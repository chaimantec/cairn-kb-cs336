# MoE routing

How a [mixture of experts](mixture-of-experts.md) decides which experts see which token.
Lecture 4 covers the space of possible routers, why the field converged on one of them,
and the two refinements DeepSeek added on top.

## Who chooses whom

Slide 27 lays out three possible directions of choice:

- **Token choice** — each token picks its favourite $k$ experts.
- **Expert choice** — each expert picks its favourite tokens.
- **Global assignment** — a joint optimization assigning tokens to experts across the
  whole batch.

"Basically, almost all MoEs do token-choice top-k: the token is the one that chooses the
experts, and there are k different experts that get selected into the pool" ([48:39]).

OLMoE's comparison (slide 28) supports this: token choice gets lower validation loss and
higher downstream scores than expert choice. Expert choice is not broken — "it also
trains fine" — but "token choice has generally been much easier to get working, and it's
been the standard for all the models we see today" ([49:24]). Hashimoto believes one of
the Llama 4 models used expert choice, adds "I don't think that's necessarily a strong
vote of confidence," and then corrects himself that it was an unreleased one.

## The four routing algorithms

Slides 29–30 survey what has been tried.

**Top-$k$ by inner product** — by far the most common. Each expert owns a vector; the
router takes the inner product with the token's hidden state and keeps the top $k$.
"This is by far the most common router — it's used in the classic Switch Transformer, and
GShard; those two are the early Google papers. It's used in Grok, Mixtral, Qwen, DBRX,
DeepSeek, and others" ([50:11]). The value of $k$ varies by paper.

**Hashing** — no learning at all: hash the token and send it to the corresponding FFN.
Hashimoto finds this genuinely odd and says so: "One thing that's always been a little
mysterious to me is that many, many papers have shown that you don't actually need any
sort of learned routing" ([50:11]). It sometimes gives gains, not as much as top-$k$, and
survives as a baseline rather than in deployment ([50:56]).

**Reinforcement learning** — treat the router as a policy and learn it with RL. This is
the theoretically natural framing, and Hashimoto says so: "If you're a classic
machine-learning-theory-oriented person, I think this should be the natural way to think
about the problem… I'm selecting one out of k, or k out of N, and I don't observe all of
them — this is a bandit problem" ([50:56]). It was used in the earliest work, like Bengio
2013. It is not used now, because "the approaches needed to do RL introduce a lot of
overhead, in terms of both the RL algorithm and the stochasticity" ([51:42]). Clark 2020
(slide 37) shows REINFORCE-based routing being beaten by baselines in its own paper
([1:00:08]).

**Linear assignment** — compute all pairwise token–expert scores and solve exactly for
the optimal global assignment. Hashimoto likes it on principle — "an idea that I think is
really cool, and that I love, as a person who likes things that make sense" — but it
"hasn't been seen at scale at all… because it's extremely expensive" ([52:27]).

The conclusion he draws is that heuristics won on cost, not on elegance: there is "a good
set of heuristics — these recipes you can apply to this very simple, classic top-k routing
scheme — that basically just get it working. So there's no reason to do something much
more complicated" ([51:42]).

## Top-$k$ routing in detail

Slide 31 gives the DeepSeekMoE router, which is the canonical form:

$$\mathbf{h}^l_t = \sum_{i=1}^{N} \left( g_{i,t}\, \mathrm{FFN}_i\!\left(\mathbf{u}^l_t\right) \right) + \mathbf{u}^l_t$$

$$g_{i,t} = \begin{cases} s_{i,t}, & s_{i,t} \in \mathrm{Topk}\!\left(\{s_{j,t} \mid 1 \leqslant j \leqslant N\}, K\right) \\ 0, & \text{otherwise} \end{cases}$$

$$s_{i,t} = \mathrm{Softmax}_i\!\left(\mathbf{u}^{l\,\top}_t \mathbf{e}^l_i\right)$$

where $\mathbf{u}^l_t$ is the layer-$l$ hidden state for token $t$, $\mathbf{e}^l_i$ is
expert $i$'s centroid vector, $s_{i,t}$ is the router score, $g_{i,t}$ the resulting gate,
$N$ the number of experts and $K$ how many are kept.

Read the three lines as: score every expert by inner product and softmax, zero out all
but the top $K$, then take the gated sum of those experts' outputs plus a residual.

"The only detail you're probably learning here is: how do I learn my gates? My gates are
just learned by taking the inner product between a weight for each expert and the input I
have — very, very lightweight. There's nothing complicated about the way I select my
gates" ([54:01]).

**One variation worth noting**: where the softmax goes. As written above the softmax
precedes the top-$k$, so the surviving gates do not sum to one. Slide 31's margin notes
that Mixtral, DBRX and DeepSeek v3 instead softmax *after* the top-$k$.

This router is structurally the same primitive as [DSA's](sparse-attention.md) indexer,
and Hashimoto points that out: "if you were paying attention in the DSA slide, this looks
a lot like DSA… this will appear in many different places, so this is a good pattern to be
able to recognize" ([52:27]).

## Fine-grained and shared experts

DeepSeekMoE's two contributions (slide 32), now near-universal.

**Fine-grained segmentation.** Keep the parameter budget, cut it into more and smaller
experts, and raise $k$ correspondingly. Slide 32's panel (b) shows $N$ experts becoming
$2N$ half-sized ones with $K$ going from 2 to 4.

**Shared experts.** Designate some experts as always-on. They bypass the router entirely
and process every token. The motivation is that under pure routing, experts waste capacity
duplicating common processing:

> Sometimes you might say, "Well, there's some common processing that I want to apply to
> all tokens" — maybe you don't want experts to be different every time, you want some
> experts to always be applied, and some other experts to be applied conditionally.
> ([54:47])

> You were just kind of reusing a lot of these weights to do common modeling. So you can
> offload this into the one shared expert, letting the others specialize even more.
> ([55:33])

**The evidence is not unanimous, and the page should say so.** DeepSeek's ablations
(slide 33) show gains from both interventions, with shared experts helping notably on
TriviaQA and Natural Questions. OLMoE (slide 34) — which Hashimoto calls "the nice,
Western, carefully controlled MoE study" — agrees that fine-grained helps but concludes
"that shared experts don't help very much" ([56:18]). Both are careful studies that
disagree on this point.

## What the models actually use

Slide 35's table, which is the single most useful reference in the lecture. "Routed" is
the total number of routed experts, "Active" how many fire per token, "Shared" how many
are always on, and "Fine-grained ratio" the size of one expert relative to a full FFN.

| Model | Routed | Active | Shared | Fine-grained ratio |
| --- | --- | --- | --- | --- |
| GShard | 2048 | 2 | 0 | |
| Switch Transformer | 64 | 1 | 0 | |
| ST-MoE | 64 | 2 | 0 | |
| Mixtral | 8 | 2 | 0 | |
| DBRX | 16 | 4 | 0 | |
| Grok | 8 | 2 | 0 | |
| DeepSeek v1 | 64 | 6 | 2 | 1/4 |
| Qwen 1.5 | 60 | 4 | 4 | 1/8 |
| DeepSeek v3 | 256 | 8 | 1 | 1/14 |
| OLMoE | 64 | 8 | 0 | 1/8 |
| MiniMax | 32 | 2 | 0 | ~1/4 |
| Llama 4 (Maverick) | 128 | 1 | 1 | 1/2 |

Blank cells are blank in the original — the six older models have no fine-grained ratio
because they were not built that way. The split down the middle of the table is the whole
story: the six earlier Western models have no shared experts and no fine-graining; the six
later ones all report a fine-grained ratio, and most use shared experts.

Hashimoto's account of that transition: the first three are Google's, then "Mixtral, DBRX,
and Grok were some of the early MoE attempts in the West. DeepSeek V1 comes up with both
the fine-grained and shared-expert design, and then that catches on and everyone else
follows from DeepSeek." He compares it to "the Llama design becoming essentially the
standard for dense transformers" ([57:05]).

## One consequence for parallelism

Asked how shared experts interact with expert parallelism, Hashimoto notes they undo its
benefit: "in the shared-expert case, you wouldn't get any parallelization savings; every
activation has to route through those. You can copy the shared expert to trade memory for
reduced comms cost" ([57:51]).

## Related pages

- [Mixture of experts](mixture-of-experts.md) — the parent topic.
- [Load balancing losses](load-balancing-losses.md) — what stops routing from collapsing.
- [Sparse attention](sparse-attention.md) — the same top-$k$ primitive selecting tokens
  rather than experts.
- [Lecture 4](04-attention-alternatives.md).
