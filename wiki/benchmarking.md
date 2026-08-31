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
- [Lecture 6 — Kernels and Triton](06-kernels-triton.md).
- [Resource accounting](resource-accounting.md), [FLOPs and
  MFU](flops-and-mfu.md) — predicting cost instead of measuring it.
- [Arithmetic intensity](arithmetic-intensity.md) — why small matrices sit on a
  floor.
- [Transcript](../raw/transcripts/06-kernels-triton.md),
  [lecture source](../raw/slides/06-kernels-triton.md).
