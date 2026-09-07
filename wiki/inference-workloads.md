# Inference workloads

What production serving traffic actually looks like — and why it resembles
neither the training distribution nor a benchmark. This is the one topic in
CS336 that only the guest lecture covers
([Lecture 18](18-serving-megakernels-recurrence.md), ≈9:16–≈13:08), and it is
upstream of most serving decisions: the routing policy in
[cache-aware routing](cache-aware-routing.md), the cache capacity target in
[KV cache offloading](kv-cache-offloading.md), and the architecture advice in the
lecture's Q&A all derive from workload shape.

## The framing claim

> When you're actually serving production traffic, it doesn't necessarily look
> like, certainly doesn't look like the type of tokens that you see during
> training, but it also doesn't look like if you just made up a traffic workload
> in your head. (≈9:16)

Both halves matter. Training data is a corpus; serving traffic is a stream of
*sessions* with structure. And the intuitive synthetic workload — uniform
requests of similar length arriving independently — is wrong in ways that change
which optimization pays.

## A workload is a shape, not a number

The lecture describes a workload by five properties rather than by a request rate
(≈10:02–≈13:08):

1. **The input/output token distribution.** A coding agent with the codebase in
   context has "tens of thousands of input tokens, and then, depending on how
   you've trained your model, the model might output some amount of thinking
   tokens, or it might just output something short" (≈10:02).
2. **How many new tokens arrive per turn.**
3. **Number of turns per session** — "am I a very sticky user who keeps going back
   and forth with my Claude agent, or am I a user who's just going to ask one
   question and then leave and come back the next day?" (≈12:22)
4. **The gap between turns** — the cadence, below.
5. **The latency target**, which differs by application: "I want to get the first
   tokens back in less than a second," versus "I know I'm going to be generating
   500 tokens, and I want that whole response to come back within a certain amount
   of time so that my user can read it fast enough" (≈13:08). These are the two
   metrics [Lecture 10](10-inference.md) calls time-to-first-token and
   time-per-output-token; see [latency and throughput](latency-and-throughput.md).

Three named workloads, deliberately contrasted (≈10:02, ≈10:48):

| Workload | Shape |
| --- | --- |
| Coding agent (e.g. whole codebase in context) | very long input, short-to-medium output, many turns |
| Narrative summarization ("pasting entire books into a chat window") | very long input, back-and-forth discussion |
| One-shot chat ("explain to me first-order calculus") | short input, short output, often a single turn |

"A coding workload, for example, will look very different from a summarization
workload" (≈10:48) — and the point is not that they differ in size but that they
load different parts of the system. Long-input workloads are prefill-heavy;
long-output workloads are decode-heavy. See
[prefill/decode disaggregation](prefill-decode-disaggregation.md).

## Agentic traffic changes the cadence

The observation that dates this lecture most precisely, and the one with the most
downstream consequence. Turn-based agentic workflows do not just add volume; they
change the *arrival pattern*:

> If you're in a fast, interactive chat-based loop, or you're talking on your
> phone to ChatGPT in voice mode, you might have relatively quick responses. If,
> on the other hand, you've put together an agentic workflow where you say, "Hey,
> go do this for me, I'm going to leave you alone and just iterate on your own,"
> you'll have a different cadence. (≈11:36)

Agents also generate traffic on their own account, invoking tools and feeding
results back into the model (≈11:36) — so one user action can produce a burst of
requests rather than one. And the gaps are not bounded by human patience:

> If at some point your agent gets stuck and it's, "Hey, hey, help, I need to ask
> for advice," and you don't notice it, there might be another gap between turns.
> (≈12:22)

The speaker's own example of a long-gap session is a ChatGPT conversation about
his workout plan that he returns to "about once every other week, so that is a
very different traffic pattern than some of the other ones" (≈13:08).

## Why the gaps are the interesting variable

Session structure decides cache economics, which is where this page connects to
the rest of the lecture. A conversation resumed after two weeks has almost
certainly lost its [KV cache](kv-cache.md) entry to eviction; one resumed after
two seconds has not. So the distribution of *gaps* — not of request sizes —
determines the hit rate, and the hit rate determines both what to cache down the
hierarchy and where to route.

The lecture makes the routing consequence explicit with a worked estimate: if the
average conversation lasts ten turns, then "10% of your requests are going to be
very fresh, very new requests" (≈31:44). Those cold 10% carry nearly all the
prefill cost. That single number is the entire basis of
[cache-aware routing](cache-aware-routing.md).

It also gives the eviction policy something to predict against. Re-opening an old
conversation in the UI is, in his words, "a very strong signal that you're going
to start asking a question about it, and you might then want to go load that up
onto a GPU" (≈28:36) — a prefetch hint that exists only because sessions have structure.

## What it implies for architecture

Asked which use cases most change the optimal architecture, the lecture answers
in workload terms (≈1:07:17–≈1:09:35):

- **Agentic loops** want the KV cache "as hot as possible," which rewards
  architectures that shrink it — [MLA](multi-head-latent-attention.md), or FP8/FP4
  cache formats.
- **Batch processing**, "where you only see each document once and then translate
  it," does not care about the cache at all, and may not need causal attention
  either: "for the longest time Google was just using BERT models, and I think
  probably still uses BERT models on search."

So the same model quality can imply different architectures depending on the
traffic it will serve — the lecture's general thesis applied to this page's
subject.

## See also

- [Lecture 18](18-serving-megakernels-recurrence.md) — the source lecture
- [The inference request lifecycle](inference-request-lifecycle.md) — what happens
  to each request in the stream
- [Cache-aware routing](cache-aware-routing.md) — the optimization this page
  justifies
- [Latency and throughput](latency-and-throughput.md) ·
  [Continuous batching](continuous-batching.md)
