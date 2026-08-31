# Benchmarking

Benchmarking measures the wall-clock time of an operation end to end. It does not
tell you where that time went — that is [profiling](profiling.md) — and Lecture 6
is blunt that this is still the number you care about, "because ultimately that's
the thing you care about — how long things are running for" ([22:22]).

It is useful for exactly two things, which the lecture source states as a pair:

- **comparing implementations** — which of these is faster?
- **understanding scaling** — how does the time move with dimension? "Because it
  distills things into one number, you can see how things are scaling, let's say
  with dimension" ([22:22]).

## The three gotchas

CS336 writes its own harness rather than using `torch.utils.benchmark`, and says
why: "because this class is language models from scratch, I'm going to do it from
scratch... I'm doing it just to highlight a few gotchas" ([23:07]). The naive
version — start the clock, run it, stop the clock — is wrong in three ways.

**1. Warm up first.** "Some things are lazily compiled, and you want to make sure
that time doesn't factor in, because most of the time you care about how fast
something is because you're going to run it over and over again, so the initial
conditions don't really matter" ([23:53]). Steady state is what you are measuring.

**2. Synchronize, or you are timing the wrong thing.** Work on the GPU is
asynchronous with respect to the CPU issuing it, so a CPU clock stops when the
*launch* returns, not when the *work* finishes. `torch.cuda.synchronize()` is "a
synchronization barrier" that waits for the CUDA threads ([24:39]) — the lecture
source marks this one "(important!)".

**3. Time on the device, and repeat.** "The proper way to time things is to use
these CUDA events — a start event and an end event, which you call record on,
actually do the computation, and then hit the end event's record" ([24:39]). CUDA
events time the GPU rather than capturing CPU overhead. And repeat, "because there
is some variance."

The harness that results (lecture source lines 179–203) is:

```python
def benchmark(run, num_warmups=1, num_trials=3):
    for _ in range(num_warmups):
        run()
    torch.cuda.synchronize()

    times = []
    for trial in range(num_trials):
        start_event = torch.cuda.Event(enable_timing=True)
        end_event = torch.cuda.Event(enable_timing=True)
        start_event.record()
        run()
        end_event.record()
        torch.cuda.synchronize()
        times.append(start_event.elapsed_time(end_event))
    return mean(times)
```

The lecture takes the mean of three trials and flags that as a simplification
rather than a recommendation: "if you are being very particular, look at the whole
distribution, the P95, or whatever — but we'll just do the average here" ([25:24]).

## What a scaling sweep shows

The lecture benchmarks matmul at dimensions 256, 512, 1024, 2048, 4096 and 8192.
Cubic scaling is expected and shows up — but only past a threshold:

> "Notice that there is this floor where, up until you get to almost
> 2000-dimensional matrices, things are basically constant. And this is because...
> these GPUs are built for fairly large matrix multiplications, and if you have
> like a 2-by-2 matrix, it's going to be very inefficient." ([25:24]–[26:10])

That floor is the whole roofline argument made visible with a stopwatch. Below it
the kernel is not compute-bound at all — it is bounded by memory traffic and launch
overhead, so making the matrix bigger costs nothing extra. See [arithmetic
intensity](arithmetic-intensity.md), and [wave quantization](wave-quantization.md)
for a second reason small or awkwardly-sized problems waste the machine.

> **No timings are quoted in this KB.** The numbers this benchmark produces are
> wall-clock measurements of whichever GPU the lecture ran on. The
> [lecture source](../raw/slides/06-kernels-triton.md) marks them
> "machine-dependent, not reproduced" and gives none, and neither does this page.
> Lecture 6's own comparison of naive, built-in and compiled GeLU is stated the way
> the lecture states it — qualitatively.

## Measuring a collective

Lecture 7 reuses this harness across GPUs, with one addition and one new number.
The addition is that **two kinds of asynchrony now have to be closed out, not
one**: "there's two forms of asynchrony here, the CUDA kernels and the different
processes" ([47:14]). So the timed region is bracketed by a
`torch.cuda.synchronize()` *and* a [`dist.barrier()`](torch-distributed.md), in
that order — barriering first lets each rank run past before its own kernels have
finished, so "the barrier doesn't really do anything" ([54:10]).

A second consequence of running $W$ processes: every rank reports its own time.
"If I look at the output here, for rank 0, 2, 1, 3, I have a different time,
potentially, because they're all different processes. Each of them is going to
report a certain measurement, and if you want to report one number, you can take
the average" ([48:00]).

### Effective bandwidth

The new number is the communication analogue of [MFU](flops-and-mfu.md) — Percy
introduces it as "analogous to when we were computing MFU" ([48:47]). A raw
duration is uninterpretable on its own; you want to know how close to the wire
speed you got. So count the bytes that *must* cross the link, and divide by the
total rank-time spent.

For an all-reduce of a payload of $S$ bytes over world size $W$ ([49:33]):

$$\text{bandwidth} = \frac{S \cdot 2 \cdot (W-1)}{W \cdot t}$$

The $(W-1)$ is the number of combining steps — "you need to iterate this
world-size-minus-one steps, because there's world-size-minus-one addition
operations." The $2$ is "because you need to both send and reduce." The $W \cdot t$
is "the total amount that all the ranks have waited."

Two properties make this the right number to quote ([51:05]):

- **Independent of world size.** As $W$ grows, $(W-1)/W \to 1$ and the expression
  tends to $2S/t$ — "so, if you grow the number of GPUs you have, the bandwidth
  doesn't change."
- **Independent of topology** — ring or tree, "which is something that
  [NCCL](torch-distributed.md#nccl) figures out."

Reduce-scatter uses the same formula **without the factor of 2** ([51:50]), and
the two should land in the same place: "all-reduce naturally is moving twice the
amount of data… But it takes twice the amount of time. But the two cancel out, so
you get the same bandwidth" ([52:36]).

The lecture's own figure is "about 400 GB per second" for both ([50:20], [51:50]),
with the caveat "sometimes, there's some stochasticity." The course's
[published four-GPU run](../raw/slides/07-parallelism.md#benchmarking-all-reduce)
bears that out — 366–426 GB/s for all-reduce and 450–490 GB/s for reduce-scatter
on 400 MiB payloads. Those are measurements of one machine, not a general fact.

## Where it sits in the workflow

The recipe is "benchmark and profile your code, you make changes, and you benchmark
and profile your code again" ([21:36]), and the ordering against kernel-writing is
deliberate: "you should always just measure what's going on in your code and figure
out what the bottlenecks are before you start writing kernels" ([22:22]). Lecture 2
made the complementary argument — that you can predict much of this with [resource
accounting](resource-accounting.md) before running anything at all.

Assignment 2 requires both this and profiling, which Percy notes leaves you no
choice ([30:02]).

## Related

- [Profiling](profiling.md) — the "where did it go" half.
- [Lecture 7 — Parallelism](07-parallelism.md) and
  [torch.distributed](torch-distributed.md) — benchmarking across GPUs.
- [Lecture 6 — Kernels and Triton](06-kernels-triton.md).
- [Resource accounting](resource-accounting.md), [FLOPs and
  MFU](flops-and-mfu.md) — predicting cost instead of measuring it.
- [Arithmetic intensity](arithmetic-intensity.md) — why small matrices sit on a
  floor.
- [Transcript](../raw/transcripts/06-kernels-triton.md),
  [lecture source](../raw/slides/06-kernels-triton.md).
