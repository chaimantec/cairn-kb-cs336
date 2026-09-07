# Prefill/decode disaggregation

Running the prompt-processing phase and the token-generation phase on **different
sets of machines**, because they stress different parts of the hardware. Described
in [Lecture 18](18-serving-megakernels-recurrence.md) (≈19:18–≈21:38) as the
standard practice it has become: "a very basic optimization that pretty much all
of us have started adopting" (≈20:05).

For the two phases themselves see
[prefill and generation](prefill-and-generation.md); for the arithmetic that makes
them different, [Lecture 10](10-inference.md) and
[arithmetic intensity](arithmetic-intensity.md).

## The asymmetry that forces the split

Prefill and decode differ on three axes at once, and it is the *combination* that
makes co-locating them wasteful (≈13:54–≈15:26, ≈20:05):

| | Prefill | Decode |
| --- | --- | --- |
| Work per step | whole prompt at once — "10,000 tokens in, one token out" | one token |
| Bottleneck | compute-bound; "very flop-heavy, you can really get the most out of your GPUs" | memory-bandwidth-bound; "you have to load up the model every time just to generate a single token" |
| How often it runs | once per prompt | once per generated token |
| Resembles | training without the backward pass | nothing else in the course |

The lecture's summary: "prefill will typically take a lot longer than a single
decode step, but you're going to be running a lot more decode steps, because you
run prefill once for a prompt, and you run decode once for every token that you
generate" (≈20:05).

Decode's problem is stated most sharply in the megakernel section: because the
whole model must be loaded to produce one token, "instead of using all this big
parallelism that you get with a GPU… you've now turned this massively parallel
system into basically a glorified memory loader" (≈34:47).

## What the split buys

> You'll run prefill on one set of workers, decode on another set of workers, so
> that you can specialize those two computations to different pieces of the stack.
> (≈20:51)

Three things follow from specialization:

1. **Neither phase interferes with the other's latency.** A long prefill occupying
   a GPU delays every decode step batched with it — the concrete complaint that
   motivates [cache-aware routing](cache-aware-routing.md): "you don't want that
   very short question-and-answer to happen at the same time, on the same GPUs, as
   the very long request" (≈32:29).
2. **Each pool can be sized independently**, against a workload whose prefill and
   decode volumes are set by
   [the traffic shape](inference-workloads.md) rather than by the model.
3. **Each pool can use different hardware** — the part of this idea with the
   furthest-reaching consequences, below.

## The same split, in silicon

Because decode is bandwidth-bound rather than compute-bound, it is a poor fit for
a chip designed to maximize FLOPs. The lecture argues that the industry has
noticed:

> One of the reasons is that the decode workload is so different from the prefill
> workload that if you're looking at decode, you can be using very different
> chips. (≈20:51)

and describes NVIDIA planning to use "its GPUs for the prefill side, using these
LPU Groq chips for the decode" (≈20:51). See
[decode-specialized hardware](decode-specialized-hardware.md), which also records
what this KB does and does not vouch for in those corporate claims.

## Where it sits among the other splits

Disaggregation is orthogonal to the parallelism strategies the course already
covers. A trillion-parameter model still will not fit on one GPU, so within each
pool it is split by [tensor parallelism](tensor-parallelism.md) or, for MoE
models, by distributing [experts](expert-parallelism.md) (≈19:18). Disaggregation
partitions *by phase*; those partition *by tensor*. The lecture treats the
choice among them as the thing that sets your bottlenecks: "the choices that you
make at this point will determine what the bottlenecks are — how many GPUs do you
need to run your model, how many sessions can you serve at the same time"
(≈19:18).

One cost the lecture does not dwell on but which the structure implies: the KV
cache computed during prefill has to reach the decode workers. That transfer is
the price of the split, and it is why the routing layer — which decides *which*
prefill node, and therefore which cache — turns out to be worth optimizing at all.

## See also

- [Lecture 18](18-serving-megakernels-recurrence.md) — the source lecture
- [Cache-aware routing](cache-aware-routing.md) — the refinement built on top
- [Decode-specialized hardware](decode-specialized-hardware.md)
- [Prefill and generation](prefill-and-generation.md) ·
  [Continuous batching](continuous-batching.md) · [KV cache](kv-cache.md)
