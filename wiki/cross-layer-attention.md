# Cross-layer attention (CLA)

Share keys and values **across layers**, so that a layer with no key/value
projections of its own reuses the ones computed by the layer below. Introduced by
Brandon et al. (2024) and covered in [lecture 10](10-inference.md) as one of the
four ways to shrink the [KV cache](kv-cache.md) ([55:43]).

The one-line statement is the lecture's own, and it is the clearest way to remember
what CLA is: **"just like GQA shares KVs across heads, now I'm sharing KVs across
layers."**

## The construction

In a standard Transformer, every layer computes its own $K$ and $V$ from its input
and caches them, so the cache is $L$ layers deep. Under CLA, only a subset of
layers compute key/value projections; the others take the previous computing
layer's keys and values and attend to them with their **own** queries.

The paper's diagram, described in
[`raw/slides/10-inference.md`](../raw/slides/10-inference.md), makes the surgery
visible: in the traditional panel each of two stacked layers has its own red
"K, V Proj." box feeding its own attention; in the CLA panel the upper layer's
K,V Proj. box is simply gone, and a red arrow runs from the lower layer's box
directly into the upper layer's attention, alongside that layer's own Q Proj.

The sharing factor is named the way GQA's is: **CLA2** shares across two layers and
halves the cache; sharing across more layers cuts further.

Note what is *not* shared. Each layer keeps its own queries and its own output
projection, so the layers still compute different attention patterns over the same
keys and values. This is the same trade GQA makes one axis over: query diversity is
cheap, because queries are not cached; key/value diversity is what costs memory.

## The evidence

The paper's Pareto plot, transcribed in the slide file, is the argument. It plots
validation perplexity against KV cache bytes per token (log scale) for 1B models,
with two groups of points: baselines without CLA (MQA and GQA at various head
dimensions, from H32-MQA at ~2.5 KB/token and perplexity ~14.37 up to H128-MHA at
~160 KB/token and ~13.15), and CLA2 variants at matched cache budgets.

At comparable cache-per-token, the CLA2 points sit at lower perplexity — for
example H64-MQA-CLA2 reaches ~13.89 at ~2.6 KB/token where the non-CLA H46-MQA is
at ~13.97 for ~3.7 KB/token, and H256-MQA-CLA2 reaches ~13.50 at ~10 KB/token where
H128-MQA is ~13.54 at the same budget. The lecture's summary: it "empirically
improves the Pareto frontier of accuracy and KV cache size" ([55:43]).

The framing of that plot is worth noting, because Percy connects it to a student's
earlier question about simply shrinking the model dimension: within any given
method, you can always sweep the cache size by changing $K$ and the head dimension,
which traces out a frontier. The claim for CLA is not that one configuration is
good, but that the whole frontier moves ([56:29]).

## Where it sits

CLA cuts the **layer** axis of the cache, which makes it composable with the
methods that cut the other three — and the paper's own best configurations are CLA
stacked on top of MQA, not CLA alone:

| Axis | Method |
| --- | --- |
| heads $K$ | [GQA / MQA](attention-variants.md) |
| dimension $H$ | [MLA](multi-head-latent-attention.md) |
| **layers $L$** | **CLA** |
| sequence $S$ | [local attention](attention-variants.md) |

It is the least-adopted of the four in production models, and the lecture spends
the least time on it — but it is the cleanest illustration of the organising idea,
because it takes an axis nobody thinks of as a place to save and saves there.

## Related pages

- [KV cache](kv-cache.md) — the four axes, and why this one is available.
- [Attention variants](attention-variants.md) — GQA, the head-axis analogue.
- [Multi-head latent attention](multi-head-latent-attention.md) — the dimension-axis cut.
- [Inference](inference.md) — why any of this is worth doing.
- [Lecture 10 — Inference](10-inference.md)
