# Lecture 4 — Attention Alternatives and Mixtures of Experts

Where [lecture 3](03-architectures.md) surveyed how the standard transformer gets
tweaked — which norm, which activation, how many heads — this lecture covers the two
places where modern models depart from it *structurally*. The first half replaces
quadratic attention with something linear or sparse. The second half replaces the
dense feedforward block with a sparsely routed mixture of experts.

Hashimoto frames them as one shape: "the first part is modifying the attention block,
and the second part is going to be modifying the MLP part" ([0:51]). Both are
motivated less by expressiveness than by cost — getting longer contexts without
quadratic attention, and more parameters without proportionally more FLOPs.

The lecture is also a marker of how fast this material moves. Hashimoto says this is
"the first year I'm talking about linear time attention, because it's clear now that
these things really work at scale and in production" ([5:26]) — after "a few false
starts" over the preceding years.

## Why attention cost became the problem

Slide 2 sets the stage with two plots: context windows across vendors over time on a
log scale, all racing upward, and beside it the ratio of compute spent in the
feed-forward block versus attention as sequence length grows. The feed-forward cost
"is fairly large at the start but grows linearly," while attention "is an all-to-all
connection between all the different positions — quadratic — so it quickly outpaces
feed-forward as the sequence length grows" ([2:22]).

That inverts an old assumption. For big models at short sequence lengths, the
feed-forward block dominated. At long ones, attention does.

Slide 3 gives what Hashimoto calls the basic toolkit, both parts of which the course
covers elsewhere. The first is **hybrid local/global attention** — "if you're only
doing global attention once every eight layers, and all the other layers are these
very local attentions, you've very much controlled the cost" ([3:09]); this is the
sliding-window and sparse-attention material from
[attention variants](attention-variants.md). The second is **systems engineering**,
and it prompts one of the lecture's recurring themes:

> Constant factors really, really matter. Part of the theme of this course is that you
> need to pay attention to the details, and it's very easy for people trained in a more
> classical, theory-oriented computer science tradition to think, "Oh, it's the big O
> that matters — linear or quadratic." ([3:09])

FlashAttention is the example — "a very clever way of rearranging the attention
operation into a much more systems-friendly form, to minimize memory transfer
overhead" ([3:55]), covered properly in the systems lectures. Slide 3's benchmark
chart shows base PyTorch attention at 36–46 TFLOP/s across sequence lengths and
running out of memory at 16k, against FlashAttention-2 at 132, 153, 162, 171, 175 and
176 TFLOP/s — better than a factor of three, from rearrangement alone.

But constant factors run out. "If we're going to five, 10 million tokens, these tricks
might not be enough" ([4:41]). That is what motivates the rest of the half.

## Linear attention

The whole of linear attention rests on one observation, which Hashimoto calls "very
silly, but surprisingly important" (slide 4): **matrix multiplication is associative**.

Standard attention is

$$\mathrm{Attn}(Q, K, V) = \rho(QK^\top)V$$

with $Q \in \mathbb{R}^{n \times d_k}$, $K \in \mathbb{R}^{n \times d_k}$,
$V \in \mathbb{R}^{n \times d_v}$, where $n$ is the sequence length and $\rho$ is the
softmax. The $QK^\top$ product is the quadratic term: it costs $n^2 d_k$.

Now drop $\rho$ — pretend, for a moment, that attention has no softmax. Then the
parentheses can move:

$$(QK^\top)V = Q(K^\top V)$$

and the cost changes from $n^2 d_k + n^2 d_v$ to $2 n d_v d_k$. The quadratic term is
gone. As Hashimoto puts it, "these $n$s are very big, they're millions — that's the
context length," whereas $d_k$ and $d_v$ "are usually on the order of thousands, tens
of thousands. No one has a million coordinates in their hidden dimension" ([6:58]).

**The softmax is what you pay for this.** Dropping $\rho$ is a real approximation, not
a rewriting, and the lecture is careful about which step is lossy — see the exchange
at [20:49] below.

### The recurrent form, and duality

The second observation (slide 5) is that the reordered product "looks just like an
RNN" ([7:44]). Sweeping left to right and accumulating:

$$S_t = S_{t-1} + k_t v_t^\top \qquad \text{and} \qquad y_t = q_t^\top S_t$$

where $S_t$ is a fixed-size state matrix carried forward, $k_t$ and $v_t$ are the key
and value at position $t$, and $q_t$ is the query.

This gives what the deck calls **duality**, and it is the property that makes the
whole family practical: the dense form is parallel and good for training, the
recurrent form has a fixed-size state and is good for inference, and they compute the
same thing. "You can have it in this dense form, which is great for training, since
it's parallel, or you can have it in this serial form like an RNN, which is great for
inference. So you get the best of both worlds" ([9:15]).

Slide 5 notes in passing that weighting $S_{t-1}$ by a scalar $\gamma$ gives RetNet —
which is the doorway to the next section.

Full treatment: [linear attention](linear-attention.md).

## Gated linear recurrences: Mamba-2 and Gated DeltaNet

Everything after linear attention is elaboration on the state update, and the lecture
gives an explicit rule for what elaborations are allowed: **as long as the gates depend
only on the current input and not on the state**, duality survives ([14:41]). That is
the design constraint the whole family respects.

**Mamba-2** (slide 7) adds one input-dependent gate $\gamma_t$ controlling how much
state to carry forward:

$$S_t = \gamma_t S_{t-1} + k_t v_t^\top, \qquad y_t = q_t^\top S_t + v_t^\top D, \qquad \gamma_t = f(x_t)$$

Hashimoto motivates it from LSTMs: "we know from the olden days of LSTMs that it's
important to know when to pass information forward and when to not pass information
forward — to forget things and send them to zero" ([12:21]). The $v_t^\top D$ term is
a separate residual-style pass-through that he flags as not core to the idea, added
"for completeness" ([15:28]).

**Gated DeltaNet** (slide 9) adds a second gate $\beta_t$ and an erase term:

$$S_t = \gamma_t\left(I - \beta_t k_t k_t^\top\right) S_{t-1} + \beta_t k_t v_t^\top, \qquad y_t = q_t^\top S_t$$

$\beta_t$ is a "no input operation" gate — at $\beta_t = 0$, nothing from the current
step enters the state ([16:14]). The $\left(I - \beta_t k_t k_t^\top\right)$ factor
acts as a projector that erases whatever was previously written in the direction of
the current key: "not only do I want to put in new information, I also want to erase
any previous keys that have gone into it" ([16:59]). He notes it is "not exactly"
a projector, since the keys are not unit-normalized.

The same update has been rediscovered repeatedly — in fast weight programming and in
test-time training, "through very different design principles" ([17:45]).

Full treatment: [state space models](state-space-models.md).

### Nobody runs these pure

Every deployed model in this section is a **hybrid** with periodic full attention:

| Model | Ratio | Deck |
| --- | --- | --- |
| MiniMax M1 | 7:1 linear to full softmax | slide 6 |
| Nemotron 3 | ~3:1 Mamba-2 to attention | slide 8 |
| Qwen 3.5 / Qwen Next | 3:1 Gated DeltaNet to attention | slide 10 |

"No one has thus far really proven out fully linear time attention mechanisms at
scale; everything I'm going to talk about in the next couple of slides is a hybrid"
([10:49]).

Slide 11 carries the one controlled study Hashimoto knows of, from ByteDance Seed and
UC Santa Cruz, sweeping the hybrid ratio. The shape of the result: at low ratios of
RNN layers "there's basically no hit," then past a point performance degrades, and "at
the end, as you go to full RNN, you have very noticeable performance degradation in
all these architectures" ([20:04]). He is candid that the evidence is thin — "there
hasn't been that many great controlled studies of how hybrid architectures perform"
([18:31]) — and that single-key retrieval is a task these architectures explicitly
optimize for, so QA performance is the more honest column to read.

### Which step is lossy

A student asks whether the parallel and recurrent forms should be equivalent. The
answer separates the two moves cleanly:

> The first step is where we drop the rho and become linear; that's the very first step
> of any of these. So this part is going to be lossy, and then after that — this linear
> form to this recurrent form — that equivalence is exact. ([20:49])

Dropping the softmax is the approximation. The linear-to-recurrent rewriting is not.

Asked what the real downside of state space models is, he answers **expressive power**,
bounded by state size: "if your state is the size of your context, then you're good to
go, but then you're paying these very large costs… if you want a really tiny state,
it's really hard to compress all the information in a big context" ([33:56]). The
historical hardware advantage of attention over LSTMs — parallel training — has been
neutralized by duality, but the information bottleneck has not ([33:07]).

## Sparse attention: DSA

A different answer to the same problem, and one that is explicitly **not** linear time.
DeepSeek Sparse Attention (slides 12–13) puts a lightweight **indexer** in front of
attention: it scores every preceding token, keeps the top-$k$, and runs full attention
on that subset only.

$$I_{t,s} = \sum_{j=1}^{H^I} w^I_{t,j} \cdot \mathrm{ReLU}\!\left(\mathbf{q}^I_{t,j} \cdot \mathbf{k}^I_s\right)$$

$$\mathbf{u}_t = \mathrm{Attn}\!\left(\mathbf{h}_t, \{\mathbf{c}_s \mid I_{t,s} \in \text{Top-}k(I_{t,:})\}\right)$$

Hashimoto is explicit that the complexity is unchanged. Asked directly, he answers
"It's quadratic… because, in order to know what to select, it does have to look at
everything" ([26:58]), and there is "no clever state-transition trick happening here —
it's really brute-force inner products" ([27:44]). What changes is the constant: the
indexer runs at low precision and low dimension, and the expensive full attention runs
on a bounded top-$k$ subset. "It's quadratic on a shorter context length, because it's
top-k, we can control k. So now it's — this is expensive, but small" ([27:44]).

Two things make it practical. It can be **bolted on after pretraining** — you train a
normal transformer, then drop the indexer in during the long-context extension stage
that models undergo anyway ([24:41], [28:29]). And it works: DeepSeek V3.2 matches
frontier models of its moment, and GLM-5's ablations show "if you do full DSA training,
you don't lose very much performance relative to full attention, even on long-context
retrieval tasks that are fairly difficult for RNN-style architectures" ([26:12]).

The top-$k$ selection is deliberate foreshadowing: "it turns out that this idea of
top-k selection is going to be core to the next part of this lecture" ([26:58]).

Full treatment: [sparse attention](sparse-attention.md).

## Mixture of experts

The second half. Conceptually the idea is small — "one way of thinking about mixture of
experts is that they're just a more efficient MLP" ([33:56]) — but the mechanics are
where the lecture spends its time.

Take the FFN, replace it with $N$ FFNs, and route each token to a few of them. "I have
four FFNs' worth of parameters, but on any forward or backward pass I'm only going to
pay one FFN's worth of cost" ([36:14]). The mental model to keep: **increase parameters
without increasing FLOPs**.

Why they took over (slides 16–23): holding compute fixed and increasing sparse
parameters reliably improves loss; MoEs train roughly twice as fast as dense
equivalents in the OLMoE ablations ([38:35]); released MoEs beat dense models at equal
*active* parameters, which is what governs inference cost. And they add an axis of
parallelism — experts are natural chunks to place on separate devices ([40:08]).

Slide 24 asks why, then, they caught on so slowly — Google was pushing them in 2022,
the field moved around 2024. The answer is complexity: hard to parallelize efficiently,
hard to fit, and "MoEs can really blow up on you" ([46:19]). Applying MoE to the
attention block rather than the FFN has been tried and mostly abandoned ([47:06]).

Full treatment: [mixture of experts](mixture-of-experts.md).

### Routing

Almost universally **token-choice top-$k$**: each token picks its $k$ experts, by inner
product against a per-expert vector. The DeepSeekMoE router (slide 31):

$$\mathbf{h}^l_t = \sum_{i=1}^{N} g_{i,t}\,\mathrm{FFN}_i\!\left(\mathbf{u}^l_t\right) + \mathbf{u}^l_t$$

$$g_{i,t} = \begin{cases} s_{i,t}, & s_{i,t} \in \mathrm{Topk}(\{s_{j,t}\}, K) \\ 0, & \text{otherwise} \end{cases} \qquad s_{i,t} = \mathrm{Softmax}_i\!\left(\mathbf{u}^{l\top}_t \mathbf{e}^l_i\right)$$

Routers are deliberately trivial — "a single matrix multiply between your input and
your whatever" ([43:12]). And they are not semantic. Asked whether experts are really
experts, Hashimoto is blunt: punctuation or a non-English character set may route
consistently, "but it's not really something where you can look at it and say, 'Oh,
this is the Wall Street Journal expert'… There's no semantics" ([1:10:05]).

DeepSeekMoE's two contributions (slide 32) are **fine-grained experts** — same budget,
more and smaller experts — and **shared experts**, always on, bypassing the router, so
common processing is offloaded and the routed experts can specialize further ([55:33]).
The evidence is not unanimous: DeepSeek's ablations show gains from both, OLMoE's find
fine-graining helps but shared experts "don't help very much" ([56:18]).

Full treatment: [MoE routing](moe-routing.md).

### Training, and why it works at all

The core difficulty, spoilered early in response to a question: **during training the
model is also sparse**. "The hard thing about MoEs is that during training you only
have one or k experts active, so you don't know what happened with the rest of them…
Despite this, you must somehow learn to route. So it's kind of got this RL-bandit
flavor" ([42:27]).

Three families of solution exist. RL on the router is the theoretically natural one and
is not used — gradient variance and overhead, and REINFORCE is beaten even by baselines
in Clark 2020 ([1:00:08]). Stochastic perturbation of the router logits, from the
original Shazeer MoE and from Fedus et al. 2022, was later removed and "it's not really
clear that it's necessary at all" ([1:02:26]). What people actually use is the third:
heuristics, principally load balancing.

The failure mode without them is **expert collapse**: chosen experts get stronger
gradients, stronger weights mean more selection, "and they kind of run away, taking on
everything" ([1:03:59]). The Switch Transformer auxiliary loss (slide 40):

$$\mathrm{loss} = \alpha \cdot N \cdot \sum_{i=1}^{N} f_i \cdot P_i$$

with $f_i$ the fraction of tokens dispatched to expert $i$ and $P_i$ the router
probability mass it receives. Hashimoto's reading of it is worth keeping, because the
objective is opaque and the gradient is not: differentiate with respect to $P_i$ and
you get $f_i$ back, so "you can think of this as a penalty in gradient space, where the
more tokens you get, the more negative gradient you get" ([1:05:30]).

Removing it is catastrophic — OLMoE's ablation (slide 43) shows losses rise and, more
tellingly, "almost all the tokens go to two experts," against even utilization with the
loss on ([1:08:33]).

Hashimoto's own summary of why the whole edifice holds:

> All you really need to do is add this balancing loss to even out the experts, and then
> treat the rest of it as if you can just pump gradients through the system — and the
> model trains very nicely. I think a big part of this is that dynamic: if an expert is
> useful, you reinforce it, and that's a positive reinforcement cycle that's nicely
> balanced out by evening things out — those two dynamics cancel each other out. ([1:09:20])

Full treatment: [load balancing losses](load-balancing-losses.md).

### Systems, stability and fine-tuning

**Expert parallelism** (slides 44–46) is the third axis after data and model
parallelism, each of which saturates — data parallelism at the batch size, model
parallelism at the natural cut points ([1:12:23]). MoE computation also maps onto
block-diagonal and structured sparse matrix multiplies that hardware supports natively,
"so there's this hardware/architecture co-design happening with MoEs" ([1:13:09]). The
cost is communication, and Nemotron 3's trick is to down-project the residual stream
before the collective call so fewer bytes cross the wire ([1:13:57]).

Full treatment: [expert parallelism](expert-parallelism.md).

**Token dropping** (slide 47) is a historical curiosity with a memorable consequence.
When an expert's queue overflowed, older infrastructure silently dropped the token and
returned zeros — so "if other users are sending queries that hit the experts you're
using, you could actually get a worse result, because they'd bump you out of the expert
queue" ([1:15:30]). Dropless implementations such as MegaBlocks have since removed this.

**Stability** (slides 48–49) follows lecture 3's rule that exponentials and divisions
are danger zones — and MoE adds another softmax in the router. The fixes are float32
for the router specifically and a [z-loss](training-stability.md) on it, which OLMoE's
ablation supports with visibly spikier loss curves when removed ([1:17:48]).

**Fine-tuning** (slide 50) overfits badly: dense models keep train and validation close,
sparse models show an extremely large gap. Mitigations are to fine-tune only the
non-MoE feedforwards or only attention, or "the bitter-lesson version" — use enough
data to effectively retrain the MoE ([1:19:18]).

### Upcycling

Take a trained dense model, copy its MLP into $N$ experts, add a randomly initialized
router, and continue training; stochastic routing makes the copies diverge and
specialize ([1:20:05]). It beat continued dense training in the original papers, and
MiniCPM and Qwen both shipped upcycled models. Hashimoto includes it for completeness
while noting it has fallen out of use — "you don't really train dense models and then
convert them — you might as well just train your big hero run as an MoE to start with"
([1:21:37]).

Full treatment: [upcycling](upcycling.md).

## The DeepSeek walkthrough

The lecture closes by tracing one lineage (slides 54–59), on the grounds that the
DeepSeek papers are unusually well documented — Hashimoto says the lecture's first
iteration was going to be nothing but a DeepSeek paper walkthrough ([43:59]).

- **V1** is already "the prototypical, Platonic ideal of an MoE model": shared and
  fine-grained experts, top-$k$ routing, auxiliary-loss balancing ([1:22:22]).
- **V2** scales it and adds systems-motivated losses — device routing and communication
  balancing. His comment on this is the section's real point: "successful language-model
  training isn't just about deep learning, it's also about really respecting your
  systems" ([1:23:08]).
- **V3** keeps the structure but replaces auxiliary-loss balancing with per-expert
  biases updated online, and switches to sigmoid-plus-softmax weighting. Slide 42 notes
  the honest caveat: DeepSeek call it "auxiliary loss free balancing," but a
  complementary sequence-wise balance loss remains, so "the approach is not fully aux
  loss free."

Two bonus mechanisms get a slide each. **Multi-head latent attention** (slides 57–58)
projects to a low-dimensional latent $\mathbf{c}^{KV}_t$ and reconstructs keys and
values from it, so only the latent needs caching — with the wrinkle that RoPE breaks the
matrix pre-merging that makes it cheap, solved by keeping a few unrotated dimensions.
**Multi-token prediction** (slide 59) predicts several future tokens, which both
sharpens the training signal and "gives you a speculative decoder built in" ([1:24:41]).

Full treatments: [multi-head latent attention](multi-head-latent-attention.md),
[multi-token prediction](multi-token-prediction.md).

## What this lecture does not cover

Expert parallelism is *introduced* here but taught in the systems lectures — Hashimoto
defers it explicitly ([40:08], [1:11:37]), as he does FlashAttention ([3:09]) and
speculative decoding ([1:24:41]). None of those lectures are in this knowledge base
yet; see [`kb.json`](../kb.json) for coverage.

## Sources for this page

- [Edited transcript](../raw/transcripts/04-attention-alternatives.md) — timestamps
  above refer to it. [Verbatim captions](../raw/transcripts/original/04-attention-alternatives.md)
  are kept alongside.
- [Slide-by-slide transcription](../raw/slides/04-attention-alternatives.md) of
  [`lecture_04.pdf`](https://github.com/stanford-cs336/lectures/blob/main/lecture_04.pdf),
  60 pages. Slide numbers are PDF page numbers — the deck prints none.
- Figure descriptions on 8 chart- and table-heavy pages were audited against the PDF at
  600–2200 dpi. See [`kb.json`](../kb.json) for what that audit found.
