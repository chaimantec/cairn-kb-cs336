# Load balancing losses

The heuristic that makes [mixture of experts](mixture-of-experts.md) trainable. Without
it, routing collapses onto a handful of experts and the rest of the model's parameters do
nothing.

This is the part of lecture 4 Hashimoto finds most surprising, and he says so twice — that
a non-differentiable top-$k$ selection can be trained "as if you can just pump gradients
through the system," provided you add this one loss ([1:09:20]).

## The problem: expert collapse

Start from ordinary gradient descent on an MoE with no special handling. The dynamics are
self-reinforcing:

> You route your top-k experts, and the strongest of those experts get more signal — your
> backprop says that expert was good, increase its weight, move that parameter closer to
> it. So you get this rich-get-richer effect, where the experts that get chosen get very
> strong weights. Strong weights mean they're selected more often, and they kind of run
> away, taking on everything. ([1:03:13])

Hashimoto calls the result **expert collapse** or **expert starvation**, and rates it "a
very, very real problem, and, in some sense, this is the core issue you have to solve with
heuristic training of MoEs" ([1:03:59]).

There are two costs. The obvious one is that starved experts are wasted parameters. The
less obvious one is systems: unbalanced experts mean unbalanced devices, and a device
sitting idle while another queues is throughput thrown away.

## The Switch Transformer loss

Slide 40, from Fedus et al. 2022. Given $N$ experts and a batch $\mathcal{B}$ of $T$
tokens, add to the model loss:

$$\mathrm{loss} = \alpha \cdot N \cdot \sum_{i=1}^{N} f_i \cdot P_i$$

where $f_i$ is the **fraction of tokens dispatched** to expert $i$,

$$f_i = \frac{1}{T} \sum_{x \in \mathcal{B}} \mathbb{1}\{\arg\max p(x) = i\}$$

and $P_i$ is the **fraction of router probability mass** allocated to expert $i$,

$$P_i = \frac{1}{T} \sum_{x \in \mathcal{B}} p_i(x)$$

$\alpha$ is the loss weight. The two vectors are, in effect, a hard count and a soft
count: $f$ is what actually happened, $P$ is what the router wanted.

### Read the gradient, not the objective

The objective is opaque — minimizing a dot product of two things that both measure
expert usage is not obviously balancing. Hashimoto says as much: "this is, at least to
me, not something you'd derive from first principles" ([1:04:44]). His advice is to
differentiate instead.

$f_i$ involves an $\arg\max$ and passes no gradient. So the derivative with respect to
the router probability $p_i(x)$ picks up only the $f_i$ factor — slide 40 gives it as
$\frac{\alpha N}{T^2} \sum \mathbb{1}_{\arg\max p(x) = i}$. The gradient on each expert's
probability mass is therefore **proportional to how often that expert was actually
chosen**:

> In some sense, you can think of this as a penalty in gradient space, where the more
> tokens you get, the more negative gradient you get. So this is trying to push down the
> probability mass on very popular experts, proportional to their fraction. ([1:05:30])

That is the whole mechanism: popular experts get pushed down in proportion to their
popularity, which is exactly the counterweight to the rich-get-richer dynamic.

> The objective itself, in equation four, might not be clear to start with, but once you
> reason about the action of the gradient, it becomes clear what it's trying to do.
> ([1:05:30])

## DeepSeek V1–V2: balancing by expert and by device

Slide 41. DeepSeek uses the Switch loss essentially unchanged for per-expert balancing:

$$\mathcal{L}_{\mathrm{ExpBal}} = \alpha_1 \sum_{i=1}^{N'} f_i P_i, \qquad f_i = \frac{N'}{K'T} \sum_{t=1}^{T} \mathbb{1}\!\left(\text{token } t \text{ selects expert } i\right), \qquad P_i = \frac{1}{T} \sum_{t=1}^{T} s_{i,t}$$

with $N'$ routed experts, $K'$ selected per token, $T$ tokens, and $s_{i,t}$ the routing
score.

Then it adds a **second, device-level** version of the same objective:

$$\mathcal{L}_{\mathrm{DevBal}} = \alpha_2 \sum_{i=1}^{D} f'_i P'_i, \qquad f'_i = \frac{1}{|\mathcal{E}_i|} \sum_{j \in \mathcal{E}_i} f_j, \qquad P'_i = \sum_{j \in \mathcal{E}_i} P_j$$

where $D$ is the number of devices and $\mathcal{E}_i$ the experts placed on device $i$.
Structurally identical — "instead of experts, you're just operating on each device's
fraction" ([1:07:02]) — but aggregated per device.

The motivation is pure systems, and Hashimoto uses it to make a broader point about the
DeepSeek papers: "DeepSeek folks are very savvy with their systems design, so they don't
just balance the experts — they also want to balance by device… you want those two
machines to be balanced, both running at full utilization" ([1:06:16]).

### Why two losses instead of one

A student asks the obvious question: if experts are evenly split across devices, wouldn't
perfect per-expert balancing give per-device balancing for free? Hashimoto's answer is
that you deliberately do not enforce per-expert balance that hard:

> The per-expert loss, in principle, if perfectly enforced, would also enforce per-device
> balancing, if your experts are evenly split. But I think, at least in my interpretation,
> you don't want to crank up the per-expert loss so high that you get true, full
> uniformity, because that has deleterious effects on training dynamics. But per-device is
> important enough that you're willing to add a little extra loss to encourage per-device
> balancing over the others. ([1:10:51])

The router is supposed to learn something. Forcing uniform expert usage would destroy
that; forcing uniform *device* usage costs nothing semantically, because which device an
expert lives on is arbitrary.

## DeepSeek V3: per-expert biases

Slide 42. Instead of an auxiliary loss, add a learned bias $b_i$ per expert that shifts
its position in the top-$k$ competition — but is *not* applied to the gate value:

$$g'_{i,t} = \begin{cases} s_{i,t}, & s_{i,t} + b_i \in \mathrm{Topk}\!\left(\{s_{j,t} + b_j \mid 1 \leqslant j \leqslant N_r\},\, K_r\right) \\ 0, & \text{otherwise} \end{cases}$$

The biases are updated online — raised for underused experts, lowered for overused ones.
Note the asymmetry in the equation: the bias decides *selection*, the raw score $s_{i,t}$
supplies the *weight*. So balancing does not distort the gate magnitudes.

DeepSeek call this **auxiliary-loss-free balancing**, and slide 42 records the honest
caveat in the deck's own words: "(but the approach is not fully aux loss free..)" — a
complementary sequence-wise balance loss remains, to "prevent extreme imbalance within any
single sequence."

Hashimoto's verdict:

> DeepSeek V3 and others have started to get rid of some of these, I'd say, uglier
> auxiliary losses, but there's no solution that fully gets rid of them so far. ([1:07:02])

## What happens without it

Slide 43 is OLMoE's ablation, and it is decisive. Removing the load-balancing loss raises
validation loss on both C4 and the Pile and worsens training loss. But the second figure
is the one Hashimoto calls more telling — per-expert token share over training:

> Without load balancing, almost all the tokens go to two experts: the yellow expert and
> the pinkish expert you see here. Whereas with the load-balancing loss, even though it's
> a heuristic thing, all of the experts are being utilized across the tokens — nice, even
> utilization. ([1:08:33])

With balancing, the eight expert traces converge to roughly $1/8$ each. Without it, two
experts take nearly everything. "We've thrown away a ton of parameters — those experts are
doing absolutely nothing for most of training."

## The two alternatives that lost

Worth knowing, because both are what a theoretician would reach for first.

**RL on the router** (slide 37). Natural framing — it is a bandit problem — and used in
the earliest work. Clark 2020 finds REINFORCE-based routing beaten by its own paper's
baselines, and the gradient variance and overhead sink it ([1:00:08]).

**Stochastic perturbation** (slides 38–39). The original Shazeer MoE injects
input-scaled noise into the router logits before the top-$k$, so near-tied experts get
explored:

> If you have two experts that are closely tied, or very close to each other, then
> stochastically you'll pick one of them and not the other. And as you backprop, you'll
> backprop in a way that the experts that are helpful get high weights. ([1:01:40])

Fedus et al. 2022 use a uniform multiplicative jitter for the same purpose. Both were
abandoned: "it was later removed in later Google routing papers, and it's not really clear
that it's necessary at all — you see later ablations showing that not doing any of these
stochastic robustness tricks actually helps with both stability and the overall quality of
the final trained MoE" ([1:02:26]).

> **A note on slide 39.** The code screenshot on that slide writes the jitter as
> `router_logits += mtf.random_uniform(..., minval=1-eps, maxval=1+eps)` — a perturbation
> drawn around 1 but applied *additively* rather than multiplicatively. The slide
> transcription records this as printed rather than silently correcting it; the surrounding
> text describes it as a multiplicative jitter.

## Why the whole thing works

Hashimoto's own explanation, and the best summary of the topic:

> All you really need to do is add this balancing loss to even out the experts, and then
> treat the rest of it as if you can just pump gradients through the system — and the
> model trains very nicely. I think a big part of this is that dynamic: if an expert is
> useful, you reinforce it, and that's a positive reinforcement cycle that's nicely
> balanced out by evening things out — those two dynamics cancel each other out. ([1:09:20])

Two opposing forces, roughly cancelling. That is the entire theory.

He expects the pattern to generalize beyond MoEs — the same top-$k$-plus-auxiliary-loss
combination appears in [DSA](sparse-attention.md) and in H-Net's attempt to remove
tokenizers, and "will be an ingredient of future architecture design" ([1:10:05]).

## Related pages

- [MoE routing](moe-routing.md) — the routers these losses correct.
- [Mixture of experts](mixture-of-experts.md) — the parent topic.
- [Expert parallelism](expert-parallelism.md) — why device balance is worth its own loss.
- [Training stability](training-stability.md) — z-loss, which the router also needs.
- [Lecture 4](04-attention-alternatives.md).
