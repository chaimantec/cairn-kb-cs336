# Latency and throughput

Two ways of meaning "fast", which point in opposite directions as soon as there is
a batch. [Lecture 10](10-inference.md) builds a small symbolic performance model
to show exactly how they trade off, and the model is worth carrying around: it is
four lines of arithmetic and it predicts the shape of every serving decision on
this page.

## The three metrics

| Metric | Units | What it measures | Governed by |
| --- | --- | --- | --- |
| **TTFT** (time-to-first-token) | seconds | wait before *any* generation appears | prefill time |
| **Latency** | seconds/token | how fast tokens appear for *one* query | memory read per step |
| **Throughput** | tokens/second | how fast tokens appear across *many* queries | memory read per step, divided among $B$ |

TTFT matters because "that latency — where you're just waiting and doing nothing — the longer that is, the worse the user experience", while once tokens start
streaming, "it doesn't necessarily have to be that fast, because if you're going to read it
anyway, you can't read that fast" ([5:30]). Throughput is the metric for batch
work: "You just have, say, a petabyte of data, and you want to process it with a
language model. You just want that job to get done" ([6:15]).

## The model

Since generation is [memory-bound](prefill-and-generation.md), time is memory
traffic. That is what makes the model this small ([37:15]–[38:47]):

```python
num_params    = 2*V*D + D*F*3*L + (2*D*N*H + 2*D*K*H)*L
parameter_size = 2 * num_params                    # bf16
kv_per_seq     = S * (K*H) * L * 2 * 2             # key+value, bf16
memory         = B * kv_per_seq + parameter_size
latency        = memory / memory_bandwidth         # seconds per token
throughput     = B / latency                       # tokens per second
```

Latency is the time to read everything the step touches: all the parameters plus
the whole [KV cache](kv-cache.md). Throughput is its inverse, "but we're
generating $B$ tokens in parallel", so it carries the extra factor of $B$.

The assumption stated in the source is doing real work here: "can overlap compute
and communication perfectly and ignore overhead". These are **theoretical
maxima**, not benchmarks. Percy notes the consolation and the cost of being
memory-bound in one breath: "in some ways it's nice because it's simpler, but in
other ways it's frustrating that your accelerators are sitting there not doing
anything" ([35:43]).

## Llama 2 13B on an H100

The configuration: $S=1024$, $D=5120$, $F=13824$, $N=K=40$ (no GQA), $H=128$,
$L=40$, $V=32000$, bandwidth $3.35 \times 10^{12}$ bytes/s. Substituting
everything but $B$ (all values recomputed from the lecture's own expressions):

$$\text{memory} = 838{,}860{,}800\,B + 26{,}030{,}899{,}200 \text{ bytes}$$
$$\text{latency} = 0.000250406\,B + 0.00777042 \text{ seconds/token}$$
$$\text{throughput} = \frac{127792.36\,B}{32B + 993} \text{ tokens/second}$$

Parameter count comes out at 13,015,449,600 — "good, that's a sanity check, since it's advertised as a
13-billion-parameter model" ([39:33]).

| Batch size | Memory | Latency | Throughput |
| --- | --- | --- | --- |
| $B = 1$ | 26.87 GB | 8.02 ms/token | 124.7 tok/s |
| $B = 64$ | 79.72 GB | 23.80 ms/token | 2,689.5 tok/s |
| $B = 256$ | 240.78 GB | 71.87 ms/token | 3,561.8 tok/s |

Three things to read off it.

**Latency is linear in $B$**, with the slope set by the KV cache and the intercept
by the parameters ([41:06]). Every extra concurrent request adds 0.84 GB that must
be re-read on every step.

**Throughput rises but asymptotes.** From $B=1$ to $B=64$ it improves 21.6×; from
$B=64$ to $B=256$ only a further 1.32×. Batching amortizes the *parameter* read,
and once that is amortized there is nothing left to amortize — "throughput can't
possibly go to infinity" ([40:18]).

**Memory is the wall.** At $B = 256$ the model needs 240.78 GB against an H100's
80 GB, so the configuration is not merely slow, it does not run ([42:38]). You
never reach the asymptote, because you hit memory first.

## The tradeoff, and how to escape it

> 1. Smaller batch sizes yield better latency but worse throughput
> 2. Larger batch sizes yield better throughput but worse latency

Percy's analogy: a bus has poor latency, because you wait for it and then everyone
travels together, but excellent throughput, because it moves everyone at once
([43:25]). The reason latency degrades is that a batched query "if you're one individual query, you have to wait for everyone else to finish."

Two escapes are worth separating.

**Replication is free throughput.** "if you launch $M$ copies of the model, the latency is the same, but the throughput increases by $M$" ([44:10]). It costs $M$
times the hardware and does nothing for latency, but it is trivially available.
Sharding one model across devices is the harder version, and the lecture defers it
to the [scaling book](https://jax-ml.github.io/scaling-book/inference/).

**Reducing memory escapes the tradeoff entirely.** The tension is a property of
the *batch* dimension, not of the two metrics: shrink the KV cache and both
improve at once. "it's not that latency and throughput are always at odds — if you reduce the
amount of memory, it improves both. It's mainly the batch dimension that is the
point of tension" ([49:35]). This is why the whole lossy
section of the lecture is about the cache — see [KV cache](kv-cache.md) for the
numbers, where GQA takes the same model from 23.80 ms to 9.97 ms *and* from 2,689
to 6,417 tok/s.

**And TTFT wants the opposite of throughput.** Since it is essentially prefill
time, "if you want faster TTFT you should use smaller batch sizes, and you want
larger batch sizes to improve throughput" ([44:57]) — which is why serving systems
schedule prefill and generation as separate phases.

## A note on the source's own comments

In the GQA walk-through the lecture's written comments say "worse latency, but
better throughput" for configurations whose computed latency is *better* than the
row above them. The comments are consistent only against the $B = 1$ baseline
(8.02 ms, 124.7 tok/s), not against the immediately preceding row. The computed
numbers are the reliable part; this is recorded in
[`raw/slides/10-inference.md`](../raw/slides/10-inference.md).

## Related pages

- [Inference](inference.md) — the workload and its metrics.
- [Prefill and generation](prefill-and-generation.md) — why latency is memory traffic at all.
- [KV cache](kv-cache.md) — the term that makes latency grow with batch size.
- [Arithmetic intensity](arithmetic-intensity.md) — the underlying accounting.
- [Continuous batching](continuous-batching.md) — how $B$ is actually managed in live traffic.
- [Lecture 10 — Inference](10-inference.md)
