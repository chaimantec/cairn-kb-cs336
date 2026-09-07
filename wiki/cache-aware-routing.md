# Cache-aware routing

Routing incoming requests to prefill workers **by their KV-cache hit rate**, so
that cold, expensive requests do not share GPUs with warm, cheap ones. Presented
in [Lecture 18](18-serving-megakernels-recurrence.md) (≈31:44–≈33:14) as
"cache-aware prefill/decode disaggregation," work from Together released "a few
months ago" relative to the June 2026 talk.

It is the best effort-to-payoff ratio in the lecture, and it is included as a
worked example of the lecture's thesis — that understanding the serving system
lets you change it profitably at a layer nobody was looking at:

> It's a very simple optimization — it's like two lines of code in the routing
> layer, but it can actually make a pretty big difference. (≈31:44)

**Claimed result: "you can get up to 40% faster serving with these very simple
optimizations" (≈33:14).**

## The argument, which is entirely about traffic

Start from session structure ([inference workloads](inference-workloads.md)). If
a conversation runs about ten turns before the user leaves, then only the first
turn is a cache miss:

> Let's say your average conversation lasts 10 turns, and then the user goes
> away — that suggests that 10% of your requests are going to be very fresh, very
> new requests. (≈31:44)

Those 10% are not merely different, they are the expensive ones: "when you have a
new request come in that's going to be thousands of tokens, it's going to look
very different, it's going to be a lot more expensive to compute" (≈31:44). The
other 90% arrive with most of their prefix already in the
[KV cache](kv-cache.md), so their prefill is short.

Mixing the two classes on the same GPUs is what hurts. The lecture's example
contrasts someone who "passed in a book and says, 'Hey, talk to me about this
book'" with someone mid-conversation asking a one-line question:

> You don't want that very short question-and-answer to happen at the same time,
> on the same GPUs, as the very long request. (≈32:29)

This is a [tail effect](kernel-launch-overhead.md) at the level of the cluster
rather than the kernel: a batch finishes when its slowest member finishes, so
mixing lengths makes the short requests wait behind the long ones.

## The policy

> You can put together this really simple router that says, okay, if we have a
> new request that comes in with a very low cache-hit rate, send it to one set of
> GPUs, so those can all process things together, and then send all my other warm
> requests to another set of prefill nodes. (≈33:14)

So: partition the prefill pool, and route on predicted cache-hit rate. Cold
requests are batched with other cold requests, where their lengths are at least
comparable; warm requests keep their own pool and stay fast.

Note what the policy does *not* require. No new kernel, no model change, no
scheduler rewrite — the signal it routes on (cache-hit rate) is already computed
by the lookup stage of
[the request lifecycle](inference-request-lifecycle.md), and the pools already
exist because of
[prefill/decode disaggregation](prefill-decode-disaggregation.md). The
contribution is noticing that the signal should reach the router.

## Why it generalizes

The underlying principle is **batch by cost class, not by arrival order** — the
same reasoning that motivates disaggregating prefill from decode in the first
place, applied one level finer. Prefill and decode differ in bottleneck; cold and
warm prefill differ in length. Both splits pay off for the same reason: batching
work with similar cost profiles wastes less of the batch waiting.

The lecture's own assessment of maturity is worth keeping alongside the result,
because it is a claim about the field rather than the technique:

> The way that I would characterize where we are, in terms of the research and
> these techniques, is that we're very early — this is the type of thing that, in
> 10 or 20 years, people are going to look back on and be like, "Oh, why are these
> guys talking about this? Isn't this already obvious to folks?" (≈33:14)

## Caveat on the number

The 40% figure is stated in the talk without a benchmark, baseline, model or
workload attached, and there is
[no slide deck or handout for this lecture](18-serving-megakernels-recurrence.md)
to recover them from. Treat it as the speaker's reported result for his own
system's traffic, not as a portable expectation — the gain depends directly on
the cache-hit distribution, which is a property of the workload.

## See also

- [Lecture 18](18-serving-megakernels-recurrence.md) — the source lecture
- [Inference workloads](inference-workloads.md) — where the 10% comes from
- [Prefill/decode disaggregation](prefill-decode-disaggregation.md) — the split
  this refines
- [KV cache offloading](kv-cache-offloading.md) — the other way to raise hit rate
- [KV cache](kv-cache.md) · [Continuous batching](continuous-batching.md)
