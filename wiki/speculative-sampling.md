# Speculative sampling

Also called speculative decoding. It is the one technique in
[lecture 10](10-inference.md) that costs nothing: it makes generation faster and
the tokens it emits are drawn from **exactly** the distribution the expensive model
would have produced. "There's a very elegant way of doing this in a lossless way"
([1:11:52]).

## The asymmetry it exploits

[Prefill is compute-bound and generation is memory-bound](prefill-and-generation.md),
and prefill "also gives you probabilities" — a single parallel pass over a
sequence returns the model's distribution at every position. So:

> Checking is faster than generation. If I give you a sequence, it's fast to tell
> me how good it is, much faster than it is to generate one at a time. ([1:11:52])

That is the whole idea. Producing $k$ tokens costs $k$ sequential memory-bound
steps; *verifying* $k$ proposed tokens costs one compute-bound step.

## The algorithm

A cheap **draft model** $p$ proposes; the expensive **target model** $q$ verifies.
Both sides of the trade are balanced deliberately: the draft model is memory-bound
and sequential, but it is small; the target model is large, but it is being asked
to process a batch of tokens in parallel, "so that's not too bad either" ([1:12:39]).

From Algorithm 2 of the speculative-sampling paper, reproduced in
[`raw/slides/10-inference.md`](../raw/slides/10-inference.md):

1. **Draft.** Sample $K$ tokens autoregressively from $p$:
   $\tilde{x}_t \sim p(x \mid x_1, \ldots, x_n, \tilde{x}_1, \ldots, \tilde{x}_{t-1})$.
2. **Score in parallel.** Compute $K+1$ sets of logits from $q$ — one for the
   prompt and one for each prefix of the draft — in a single forward pass.
3. **Accept or reject, left to right.** For each drafted token, draw
   $r \sim U[0,1]$ and accept if
   $$r < \min\!\left(1, \frac{q(x \mid x_{1:n+t-1})}{p(x \mid x_{1:n+t-1})}\right)$$
   "the larger $q$ is, the more likely we are to accept it" ([1:14:09]).
   On the first rejection, sample the replacement token from the clipped residual
   $(q - p)_+$ and stop accepting the rest of this round.
4. **Bonus token.** If all $K$ drafts are accepted, sample one extra token from $q$
   for free — the parallel pass already computed that distribution.

Step 4 is why a fully-accepted round advances $K+1$ positions for the price of one
target-model pass.

## Why it is exact

This is modified rejection sampling with proposal $p$ and target $q$. The
modification, stated in the source: "always generate at least one candidate", since
plain rejection sampling can loop producing nothing.

The lecture gives the proof by example, on a two-token vocabulary $\{A, B\}$ where
the draft oversamples $A$, so $p(A) > q(A)$ and therefore $p(B) < q(B)$. The
residual $\max(q - p, 0)$ is then $[0, 1]$ — all of its mass on $B$. Summing the
ways each token can be emitted:

$$P[A] = p(A) \cdot \frac{q(A)}{p(A)} + p(B) \cdot 1 \cdot 0 = q(A)$$

$$P[B] = p(B) \cdot 1 + p(A)\left(1 - \frac{q(A)}{p(A)}\right) \cdot 1 = q(B)$$

Read as cases: $A$ is emitted only by proposing it and accepting it, and the
residual contributes nothing because it has no mass on $A$. $B$ is emitted either
by proposing it and accepting outright — acceptance is certain, since the draft
undersamples $B$ — or by proposing $A$, rejecting it, and falling back to a
residual that is entirely $B$. Both land on the target model's own probability.

So the output distribution **is** $q$, not an approximation of it. Nothing about
the draft model's quality affects correctness; it affects only speed, through the
acceptance rate.

## How much it buys

From the paper's Table 1, reproduced in the slide file: Chinchilla at batch size 1
with $K = 4$ gets **1.92×** on XSum with nucleus sampling, **2.01×** with greedy
sampling, and **2.46×** on HumanEval — with benchmark scores unchanged or slightly
better (XSum ROUGE-2 0.112 → 0.114; HumanEval 45.1% → 47.0%). Mean token time
falls from 14.1 ms to 5.73–7.52 ms.

**There is a sweet spot in $K$**, and both ends of it are visible in the paper's
own measurements: "if you have too few draft tokens, you're not really leveraging the batching on the target-model side. And if you have too many, then you're going to reject more often. So, there's a sweet spot — in this case, around three or four" ([1:15:39]).
The acceptance rate falls monotonically with $K$ — on XSum, from 1.0 at $K=0$ to
about 0.45 at $K=7$ — while the per-iteration loop time rises roughly linearly, so
mean sampling time is U-shaped.

Typical size ratios in practice: a 70B target with an 8B draft, or an 8B target
with a 1B draft.

## Making the draft model better

Two extensions are named ([1:16:25]):

- **Medusa** — the draft model predicts several future tokens *in parallel* from
  the same hidden state, using multiple heads, instead of running autoregressively.
- **EAGLE** — the draft step consumes the target model's own high-level features
  alongside the token embeddings, so the draft is conditioned on what the target
  already computed.

And the unification that closes the section, which ties this page back to the rest
of the lecture: run all the [KV-cache reductions](kv-cache.md),
[quantization](quantization.md) and
[pruning](pruning-and-distillation.md) you like on your model. "if you end up with a model you're happy with, just serve that. If you're not happy with it, then at least it can be a draft model, and you can use your main model to fix things up"
([1:16:25]). A lossy compression that was *almost* good enough is not wasted work —
it becomes the draft, and speculative sampling restores exactness.

## Related pages

- [Inference](inference.md) — the workload, and where this fits among the techniques.
- [Prefill and generation](prefill-and-generation.md) — the asymmetry it exploits.
- [Pruning and distillation](pruning-and-distillation.md) — how you get a good draft model.
- [KV cache](kv-cache.md) — the lossy alternatives this one avoids.
- [Lecture 10 — Inference](10-inference.md)
