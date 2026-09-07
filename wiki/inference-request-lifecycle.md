# The inference request lifecycle

What actually happens between a user pressing enter and tokens coming back. The
guest lecture calls this **"the lifetime of a token"** and uses it as the spine of
its first half, on the grounds that you cannot reason about any one optimization
until you know which stage it lives in
([Lecture 18](18-serving-megakernels-recurrence.md), ≈6:14).

This page is the serving-stack view. For the arithmetic of a single forward pass —
FLOPs, KV cache size, the three metrics — see [Lecture 10](10-inference.md) and
[inference](inference.md).

## The stages

**1. Scheduling.** The request arrives and is assigned to GPUs. Already a
structural choice has been made: "you might have disaggregated prefill and decode
on different machines" (≈7:45). See
[prefill/decode disaggregation](prefill-decode-disaggregation.md).

**2. Cache lookup.** Before computing anything, the engine asks whether it has to:

> You'll run that request against a KV cache to see: have I seen this request, or
> versions of this request, before? Is there some compute that I can actually
> save? (≈8:31)

This is [KV cache](kv-cache.md) reuse across requests and users, not within one —
see the prefix-sharing discussion below.

**3. Tokenization**, then the scheduling regime proper: "have I seen these tokens
before? Can I just look up some of the activations in a cache?" (≈13:54).

**4. Prefill**, which computes the prompt's activations — "10,000 tokens in, one
token out" — and is compute-bound (≈13:54).

**5. Decode**, which generates one token at a time and is memory-bandwidth-bound,
because "you have to load up the model every time just to generate a single
token" (≈14:39, ≈15:26). The prefill/decode asymmetry is the most consequential
fact in the whole pipeline; see
[prefill and generation](prefill-and-generation.md).

**6. Execution layout.** Whichever of these runs, it may be split: "you can split
that computation across different machines, you can parallelize across different
nodes, you can parallelize it within a node across different GPUs, depending on
the size of your model" (≈8:31). See [tensor parallelism](tensor-parallelism.md)
and [expert parallelism](expert-parallelism.md).

**7. Sampling and post-processing.** The model pass ends with "a single number
that represents a token, that then gets turned into a string." Then the
non-neural work: stop-token checks, and "you might run a safety check to say, oh,
is my user trying to hack into the system in a bad way" (≈15:26).

## It is a loop, not a pipeline

The stages above describe one request, but the engine never runs one request. It

> is really running in this loop, waiting for these requests to come in, so it's
> running this scheduling, execution, token sampling loop, and repeating.
> (≈15:26)

Which means every stage above is contended. [Continuous
batching](continuous-batching.md) is what lets requests enter and leave the loop
without draining it, and the resources they contend for are two:

> These resources are first compute resources — you have to run things over
> multiple requests at once. They can be memory resources if you're filling up a
> KV cache. (≈16:59)

The second is the one that surprises people, and it is a hard failure rather than
a slow one: when the KV cache exhausts GPU memory, a new request "might start
queuing for that reason" (≈16:59). A serving system can be compute-idle and still
unable to admit work.

## Prefix sharing is what makes the cache a shared asset

The lecture's framing of the KV cache is not *avoid recomputing my own prompt*
but *avoid recomputing everyone's*:

> You probably have a lot of users who are saying "hi" to ChatGPT or "hi" to
> Claude, and theoretically you don't need to compute new activations and run
> that again for every single user. (≈17:46)

The same applies within a conversation: prefill a long document once, and on the
next turn "you don't need to compute the whole thing again" (≈18:33). The
mechanism is a tree over token prefixes — "use a very traditional data structure
like a basic tree, look at which tokens you've seen before, which tokens are new,
and then do a lookup of what those activations are going to look like" (≈18:33).

Two consequences run through the rest of the lecture. Cache hit rate becomes a
*routing* signal, which is the whole of [cache-aware
routing](cache-aware-routing.md). And cache capacity becomes worth extending down
the memory hierarchy, which is [KV cache offloading](kv-cache-offloading.md).

## Why the lifecycle framing earns its place

Each research thread in the lecture is an attack on one stage, and the lecture is
explicit that this is the point of walking the path first (≈6:14, ≈59:30):

| Stage | Intervention | Where |
| --- | --- | --- |
| Scheduling / routing | route by cache-hit rate | [cache-aware routing](cache-aware-routing.md) |
| Cache | spill to DRAM and SSD, evict by LRU | [KV cache offloading](kv-cache-offloading.md) |
| Execution (decode) | fuse the model into one kernel | [megakernels](megakernels.md) |
| The model itself | loop blocks instead of adding parameters | [looped transformers](looped-transformers.md) |

## See also

- [Lecture 18](18-serving-megakernels-recurrence.md) — the source lecture
- [Lecture 10 — Inference](10-inference.md) — the arithmetic this page assumes
- [Inference workloads](inference-workloads.md) — what arrives at stage 1
- [KV cache](kv-cache.md) · [Continuous batching](continuous-batching.md) ·
  [Paged attention](paged-attention.md)
