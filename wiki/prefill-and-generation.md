# Prefill and generation

Inference runs in two phases, and they have opposite bottlenecks. Getting this
distinction right is most of what [lecture 10](10-inference.md) is for: almost
every technique in the lecture, and every design decision in a real serving stack,
follows from the fact that one phase is compute-bound and the other is
memory-bound.

**Prefill** takes the prompt and encodes it, populating the
[KV cache](kv-cache.md) with the keys and values for every prompt token. Because
the whole prompt is available at once, this is "parallelizable just like in
training" ([24:46]).

**Generation** (also called decode) produces the response one token at a time.
Each step consumes the cache, emits a distribution, samples a token, and appends
that token's own key and value to the cache ([23:13]). It is sequential by
construction — "because of the autoregressive nature of inference" you cannot
parallelize across the sequence dimension ([7:47]).

## The accounting

Let $S$ be the number of tokens being conditioned on and $T$ the number being
generated or scored this step. Prefill is the case $T = S$; generation is the case
$T = 1$. The [arithmetic intensity](arithmetic-intensity.md) of each layer type,
derived in full in [`raw/slides/10-inference.md`](../raw/slides/10-inference.md):

$$\text{MLP intensity} \;=\; \frac{6BTDF}{4BTD + 4BTF + 6DF} \;\xrightarrow[\;D, F \gg BT\;]{}\; BT$$

$$\text{attention intensity} \;=\; \frac{4BSTD}{4BSD + 4BTD} \;=\; \frac{ST}{S+T}$$

Substituting the two cases gives the table the whole lecture turns on ([33:17]):

| | MLP | Attention |
| --- | --- | --- |
| **Prefill** ($T = S$) | $BS$ — large, compute-bound | $S/2$ — "not as good, but workable" |
| **Generation** ($T = 1$) | $B$ — workable if requests are concurrent | $S/(S+1) < 1$ — **the bottleneck** |

An H100's own ratio is 295 FLOPs per byte, so an intensity below 1 is roughly 300×
short of saturating the machine. "Arithmetic intensity — remember, one is bad. We
want it to be something like 295 for an H100 to saturate the compute" ([31:00]).

## Why batching rescues one and not the other

The MLP's intensity carries a $B$; attention's does not. The reason is what is
shared:

> In MLP layers, every sequence hits the same MLP weights (Wup, Wgate, Wdown don't
> depend on B). In attention layers, every sequence has its own KV cache vectors
> (Q, K, V all depend on B).

Loading the MLP weights once and using them for every sequence in the batch is
exactly how batching buys intensity ([32:32]). Attention has nothing to reuse
across sequences: "It's like, for every sequence, you're basically doing a matmul.
So, they're all independent, so doing more matmuls isn't helpful."

Percy connects this back to the notation from the start of the lecture. In the
shape algebra, $B$ appears as a **batching (blue) dimension** in the attention
contraction — present in both operands and surviving into the result — which makes
the operation a batch of dot products rather than one large matmul, and "this is basically the same as doing a dot product, which has horrible arithmetic intensity" ([33:17]).

Hence the verdict: if you keep a Transformer, generation's attention intensity is
"a fundamental bottleneck, and that's it — if you're sticking with a
transformer, you can't really improve this" ([34:03]). Everything else in the lecture reduces the *volume* of memory
traffic rather than the intensity.

## What follows in practice

**The two phases want different batch sizes.** TTFT is essentially prefill time,
so interactive latency wants prefill batches small; throughput wants generation
batches large ([44:57]). Production systems therefore schedule the phases
separately rather than treating a request as one job.

**Prefill is where prompt caching pays.** Since prefill is compute-bound and its
output is exactly the cache, reusing a prefix's cache converts compute into a
lookup — which is what [PagedAttention's](paged-attention.md) block sharing does
for system prompts and repeated samples.

**Prefill is what makes speculative sampling possible.** A prefill pass "also
gives you probabilities" ([1:11:52]) — it does not merely encode a sequence, it
scores every position of it in parallel. That is the asymmetry
[speculative sampling](speculative-sampling.md) exploits: checking $k$ tokens is
one compute-bound pass, generating them is $k$ memory-bound ones.

**Ragged batches split along the same line.** Orca's *selective batching*
([continuous batching](continuous-batching.md)) computes attention per sequence
and concatenates every sequence for the MLP layers ([1:18:43]) — the same split as
the table above, arrived at from a systems direction rather than an arithmetic
one.

## Related pages

- [Inference](inference.md) — the workload.
- [KV cache](kv-cache.md) — what prefill produces and generation consumes.
- [Arithmetic intensity](arithmetic-intensity.md) — the general tool, and the inference derivation.
- [Latency and throughput](latency-and-throughput.md) — the metrics each phase governs.
- [Continuous batching](continuous-batching.md) — the same split, from a systems angle.
- [Speculative sampling](speculative-sampling.md) — trading generation steps for prefill steps.
- [Lecture 10 — Inference](10-inference.md)
