# Warp occupancy

Occupancy is the fraction of an SM's warp slots you are actually using. Lecture 6
introduces it as the second of five places where the hardware shows through a
programming model that pretends it is not there ([10:53]), and it is the one that
comes down to a register budget.

## The constraint

Each thread may hold **at most 255 registers**, and an SM has a fixed pool of
them. "So the more registers each thread is using, the fewer threads you can have —
that's just math. And that can reduce your occupancy" ([10:53]).

## The worked example

The lecture computes one case in full ([12:26]–[13:57]). A thread block of 128
threads, each thread using 160 registers, on a B200 whose SM has 65,536 registers
and can hold at most 64 concurrent warps:

$$\text{registers per block} = 128 \times 160 = 20{,}480$$

$$\text{blocks per SM} = \left\lfloor \frac{65{,}536}{20{,}480} \right\rfloor = 3$$

$$\text{warps} = \frac{3 \times 128}{32} = 12 \qquad
\text{occupancy} = \frac{12}{64} = 0.1875$$

Percy reads the result off as **18%**: "so you're only using 18% of the total
number of warps you have, and this is because you have a lot of register use per
thread. So this is an example where memory is constraining you in terms of how
much compute you can do" ([13:57]).

> The lecture's prose sets this example up with "thread block has 64 threads"
> while the code immediately below it uses `num_threads_per_block = 128`. The
> figures above follow the code, which is what runs. Both are recorded in the
> [lecture source transcription](../raw/slides/06-kernels-triton.md#warp-occupancy).

## Low occupancy is not automatically bad

This is the part that separates occupancy from a score to maximize:

> "That's not necessarily bad, because if you have fewer threads but each thread is
> doing more work, that can actually be good. Occupancy is something you can
> measure, but it's not necessarily the larger the better, because there are some
> other trade-offs here." ([11:39])

**Thread coarsening** is the named example. Instead of one thread per element, give
each thread several — "like a constant, maybe eight. That gives you fewer threads,
which makes scheduling and these things easier, but each thread is doing more work.
So if your threads are very light, maybe you want to fatten them up a little bit"
([11:39]).

The Triton compiler does this on its own. Reading the [PTX](ptx.md) generated for
the lecture's GeLU kernel shows each thread handling **eight** elements, a decision
nothing in the Triton source asked for ([52:28]).

## Why warps are resident in the first place

The same mechanism that makes occupancy meaningful is what hides memory latency. An
SM keeps many warps resident and its scheduler switches between them "with zero
cost" — unlike a CPU context switch — precisely so that a warp stalled on HBM does
not idle the SM ([10:07]). An HBM read "can take like 100 cycles or something. You
don't want to just sit around waiting for that warp to do nothing — you switch
immediately to another warp where it can actually do some tensor core operations"
([10:53]).

A student later restates this from the kernel's point of view and Percy confirms
it: a `tl.load` is "almost like a CPU trap call, where I'm just waiting for
something to happen... so some other warp will get scheduled over me" — "yeah,
that's right... when you get to that point, it can just find another warp to run,
and then, when this is done, the warp scheduler comes back and takes over"
([55:36]–[56:22]).

So occupancy is not an end in itself: it is how much *slack* the scheduler has to
hide stalls with. Enough resident warps to cover the latency is what matters, and
past that, fatter threads may serve you better than more of them.

## Occupancy of warps vs. occupancy of blocks

Two different things share the word, and Lecture 6 covers both:

- **Warp occupancy** (this page) — warp slots per SM, limited by registers.
- **Block occupancy** — whether your thread blocks fill whole waves of SMs, covered
  under [wave quantization](wave-quantization.md) ([18:31]).

## Related

- [The GPU execution model](gpu-execution-model.md) — warps, SIMT, control divergence.
- [GPU architecture](gpu-architecture.md) — the register file and memory hierarchy.
- [Wave quantization](wave-quantization.md) — the block-level counterpart.
- [PTX](ptx.md) — where you can see coarsening actually happen.
- [Profiling](profiling.md) — occupancy is one of the things a profiler reports ([16:15]).
- [Lecture 6 — Kernels and Triton](06-kernels-triton.md),
  [Lecture 5 — GPUs and TPUs](05-gpus-tpus.md).
- [Transcript](../raw/transcripts/06-kernels-triton.md),
  [lecture source](../raw/slides/06-kernels-triton.md).
