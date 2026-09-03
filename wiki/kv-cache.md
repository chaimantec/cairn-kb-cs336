# The KV cache

The KV cache is the stored keys and values for every token a sequence has already
seen, kept in HBM so that generating the next token does not recompute them. It is
the single most important object in [inference](inference.md): it is what makes
generation tractable at all, and then it is the thing that makes generation
[memory-bound](prefill-and-generation.md), which is why most of
[lecture 10](10-inference.md) is about shrinking it.

## Why it exists

Without a cache, generation is cubic. A black box that maps a sequence to a
distribution over the next token can be applied repeatedly — sample a token,
append it to the prompt, run the whole thing again — and that is the naive
algorithm. One forward pass over $T$ tokens is $O(T^2)$, and you do one per
generated token, so generating $T$ tokens costs $O(T^3)$ ([21:40]).

The saving is available because the model is **causal**. Appending a token does
not change any earlier activation: "If it were bidirectional, then if you attach a token, everything changes. But if it's causal, then the activations here don't change based on any tokens you append" ([22:26]). So the keys and values
computed for "never going to give" are the same whichever token comes next, and
they can be stored and reused.

## Its size

The lecture states the shape as a sentence: "for every sequence (B), token (S),
layer (L), head (K), store an H-dimensional vector"
([`raw/slides/10-inference.md`](../raw/slides/10-inference.md), and [23:59]). In
bytes, with one key and one value per position and 2 bytes per bf16 number:

$$\text{KV cache per sequence} = S \cdot (K H) \cdot L \cdot 2 \cdot 2$$

$$\text{total memory} = B \cdot S \cdot K H \cdot L \cdot 4 \;+\; 2 \cdot (\text{number of parameters})$$

For **Llama 2 13B** at $S = 1024$ — $K = 40$ key/value heads, $H = 128$, $L = 40$
layers — that is **838,860,800 bytes, about 0.84 GB, for a single request**,
against 26.0 GB for the whole model in bf16 (computed from the lecture's own
expression). One modest 1024-token conversation costs 3% of the model's entire
weight footprint, and the cache is linear in the number of concurrent requests
while the weights are not. At batch 64 the cache is 53.7 GB against 26.0 GB of
weights: "it could even be larger than the number of parameters, for a large enough batch
size" ([46:33]).

## Why it is the bottleneck, and not just an expense

Two facts compound.

First, **latency is memory traffic.** Since generation is memory-bound, the time
to produce one token is essentially the time to read everything the step touches —
all the parameters and the entire cache — so
$\text{latency} = \text{memory} / \text{bandwidth}$ ([38:01]). The cache is not
merely stored, it is re-read every single step.

Second, **batching cannot amortize it.** Every sequence hits the same MLP weights,
so a larger batch spreads one weight-read over more work; every sequence has its
own cache, so a larger batch reads proportionally more of it ([31:46]). The batch
size $B$ cancels out of attention's [arithmetic intensity](arithmetic-intensity.md)
entirely. This is the structural asymmetry that
[lecture 10](10-inference.md) is organised around.

## The four axes

The cache has four dimensions you could shrink, and each named method in the
lecture picks a different one. Reading them this way is what turns a list of
acronyms into one idea:

| Axis | Method | What it does | Factor |
| --- | --- | --- | --- |
| $K$ — heads | [GQA](attention-variants.md) | $N$ query heads share $K$ key/value heads | $N/K$ (e.g. 5×) |
| $H$ — dimension | [MLA](multi-head-latent-attention.md) | store a $C$-dimensional latent, project up on demand | $16384 \to 576$ in DeepSeek v2 |
| $L$ — layers | [CLA](cross-layer-attention.md) | a layer reuses the layer below's keys and values | 2× at CLA2 |
| $S$ — sequence | [local attention](attention-variants.md) | attend only to the last $w$ tokens | $O(S) \to O(1)$ |

$B$ is missing from that table on purpose: it is the number of concurrent
requests, which is a property of your traffic, not of your model. What you can do
about $B$ belongs to [continuous batching](continuous-batching.md) and
[PagedAttention](paged-attention.md).

The local-attention row is the strongest cut available, because it changes the
cache's asymptotic behaviour rather than its constant: "the KV cache is now
independent of the sequence length" ([57:14]). It is also the one that costs the
most accuracy, which is why it is deployed interleaved with full-attention layers
rather than alone.

## What shrinking it buys, quantitatively

Because latency is memory over bandwidth, cutting the cache converts directly into
speed — and, unusually, it improves latency *and* throughput at once. Percy makes
the point explicitly: "it's not that latency and throughput are always at odds — if you reduce the amount of memory, it improves both. It's mainly the batch dimension that is the point of tension" ([49:35]).

Cutting Llama 2 13B's cache by 5× with GQA ($K: 40 \to 8$), all computed from the
lecture's model:

| | KV cache/seq | Memory at $B=64$ | Latency | Throughput |
| --- | --- | --- | --- | --- |
| MHA ($K = 40$) | 0.84 GB | 79.72 GB | 23.80 ms/token | 2,689.5 tok/s |
| GQA ($K = 8$) | 0.168 GB | 33.41 GB | 9.97 ms/token | 6,416.7 tok/s |

And the freed memory can be spent on batch size, which is the second-order win:
$B = 256$ did not fit in an 80 GB H100 with MHA (240.78 GB) and does fit with GQA
(65.63 GB), reaching 13,068 tok/s. "Sometimes you play with these parameters
jointly — you can reduce the KV cache, but that allows you to increase the batch
size" ([50:20]).

## The standing verdict

The lecture's closing judgement is about this object specifically: "at some level,
the KV cache and the way that attention is built fundamentally makes it an
inference-unfriendly kind of architecture" ([1:24:51]). Every method on this page is a
patch on a data structure whose size is inherent to attention. The alternatives
that do not have one at all —
[linear attention](linear-attention.md) and
[state space models](state-space-models.md), which keep a fixed-size recurrent
state instead — are where the lecture says the large gains still are.

## Related pages

- [Inference](inference.md) — the workload this all serves.
- [Prefill and generation](prefill-and-generation.md) — why only one of the two phases has this problem.
- [Latency and throughput](latency-and-throughput.md) — the performance model the numbers above come from.
- [Arithmetic intensity](arithmetic-intensity.md) — the accounting that proves it is the bottleneck.
- [Attention variants](attention-variants.md) — GQA, MQA and sliding windows.
- [Multi-head latent attention](multi-head-latent-attention.md) — the dimension cut.
- [Cross-layer attention](cross-layer-attention.md) — the layer cut.
- [PagedAttention](paged-attention.md) — how it is actually laid out in memory.
- [Lecture 10 — Inference](10-inference.md)
