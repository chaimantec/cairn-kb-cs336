# GPU architecture — SMs and the memory hierarchy

This page covers what a GPU *is*: the physical arrangement of compute and memory
that everything else in the systems unit is reasoning about. For how code is
organized to run on it — threads, blocks, warps — see
[the GPU execution model](gpu-execution-model.md).

Hashimoto's stated goal for this part of Lecture 5 is demystification: "I want to
demystify what a GPU is... I want you to be able to visualize what the hardware of
a GPU actually is" ([8:31]).

## Design philosophy: latency versus throughput

The difference from a CPU is not incremental, and slide 8 puts the two side by side.

A **CPU** is built for fast serial execution with complex branching. It therefore
spends its transistor budget on large control units and a handful of ALUs, aiming
for **low latency** — "the time between when you get an instruction and when you
complete it should be very, very short" ([7:45]).

A **GPU** is built for **throughput**. An individual task may take a long time and
may be stopped and resumed, but the aggregate rate across all tasks is enormous.
What buys that is "having tons and tons of lightweight cores. A GPU has hundreds and
hundreds of compute units packed into a chip, and all of these can execute in
parallel" ([8:31]).

This is downstream of a hardware fact the lecture opens with: Dennard scaling ended,
clock speeds stopped improving, and "this old approach of making your clocks faster,
making instructions execute faster, is not going to work" ([5:26]). Parallel scaling
is what replaced it — "instead of making things go faster serially... you're going
to scale horizontally" ([6:12]).

## The SM

The basic unit is the **streaming multiprocessor (SM)**. Hashimoto describes it as
"kind of like a core — its own independent compute unit that has sub-components it
can use to accelerate its computation, and access to certain kinds of memory it can
connect to" ([9:16]).

An SM is not itself a single compute unit; it contains streaming processors that
execute threads in parallel. But it is the *discrete* unit — the thing a block of
threads is scheduled onto, and the thing whose count determines
[wave quantization](wave-quantization.md).

GPUs have many of them, each independently programmable and each with access to
global memory ([10:02]).

**On SM counts.** The lecture quotes two figures for an A100 and both are defensible.
At [10:02] Hashimoto says 128, reading slide 10's die annotation for the GA100
("x8 GPC, x64 TPC, 128x SM"); slide 48 says 108, which is the shipped product's
count and the one the wave-quantization arithmetic depends on. Slide 10's panel also
records the die as 7nm TSMC, 8192 FP32 units, 4096 FP64 units, 48MB L2 cache,
6144-bit HBM2(e), and roughly 826–837 mm². An H100, per [20:46], has "something like
132 streaming multiprocessors."

## The memory hierarchy

"Modern hardware and LM optimization is really defined by the memory" ([10:02]) —
so the levels, and their costs, are the part to internalize.

Slide 10's rule: **the closer the memory to the SM, the faster it is.** L1 and
shared memory are *inside* the SM; L2 is on the die; global memory is the separate
memory chips beside the GPU.

Slide 10 reproduces a benchmark table of A100 latencies:

| Memory type | Latency (cycles) |
| --- | --- |
| Global memory | 290 |
| L2 cache | 200 |
| L1 cache | 33 |
| Shared memory (load/store) | 23 / 19 |

Hashimoto reads it at [10:47]: L1 and shared memory are "something like 20 to 30
cycles"; L2 is much slower; and global memory is "10 times slower than the L1
cache."

The naming is worth pinning down, because "GPU memory" in casual usage means the
slowest level: "when an H200 tells you it's got 144 gigs of memory, you're talking
about global memory. You're not talking about shared or L1 cache" ([11:33]).

**Shared memory versus L1 cache** — a student asks twice ([12:18], [27:43]), and the
answer is about control, not speed. The L1 cache "is really just a cache — it's
storing recently accessed data elements," and you do not control it. Shared memory
"is a programmable element that you actually interact with, that you can put things
in and out of." That programmability is what makes [tiling](tiling.md) possible.

**Why L2 is slower than L1** ([27:43]): mostly physical distance and interconnect,
not a different storage technology — both are SRAM, which is why both are far faster
than DRAM global memory.

## Why the whole chip is not fast memory

The obvious question — if shared memory is so good, build the chip out of it — gets
a three-part answer ([12:18], [1:09:49]):

1. **Cost.** SRAM is "much more expensive — hundreds of times."
2. **Physics.** It has to be physically close, and "signal propagation is hard."
3. **Energy.** SRAM "needs to be powered the whole time to hold a value, which is
   very energy hungry."

So a practical accelerator has a hierarchy you must respect, rather than one fast
pool. Hashimoto's counterexample is instructive: Groq built a design that was
essentially all SRAM, which "for certain workloads, like inference... is very
helpful" ([12:18]) — and at [1:00:39] he notes that with such a chip you would
simply hold the whole matrix in fast memory, "but very, very expensive." The
transcript's identification of this company rests partly on context outside the
deck; see the note in [the transcript header](../raw/transcripts/05-gpus-tpus.md).

## The gap that motivates everything else

Slide 18 is the chart that justifies the rest of the lecture. Three quantities are
improving at very different rates ([25:23]–[26:09]):

- **Compute throughput** — fastest.
- **Memory bandwidth** — "growing comparatively slowly."
- **Interconnect between devices** — "also going very slowly."

The consequence is historical as much as technical: early GPU programmers "probably
weren't thinking much about memory, because your compute wasn't that much faster
than your memory transfer." As the gap widens, "we're going to see memory and
communication bottlenecks more and more, and it's going to take more and more work
to fully utilize the hardware we have."

That is why five of the lecture's six tricks are memory tricks, and why
[arithmetic intensity](arithmetic-intensity.md) is the right lens for almost
everything in this unit.

Asked whether the growing gap will change chip design, Hashimoto points at inference
hardware, where it already has ([28:28]–[29:14]): **prefill/decode disaggregation**
puts the compute-heavy prefill on one chip and the memory-bandwidth-limited decode
on another, and some models go further and route attention and MLP layers to
different accelerators. "If this gap grows, any clever trick to utilize your precious
fast memory becomes very valuable."

## The three-sentence summary

Hashimoto's own, at [26:55]: GPUs are massively parallel and execute instructions
all at once; compute scales faster than memory, so memory is what matters; and
because memory matters, "everything we do has to respect the memory hierarchy — as
much as possible goes into shared memory and not global memory."

## Lecture 6's summary of the same hierarchy

Lecture 6 opens by restating this hierarchy from its own table of A100, H100 and
B200 ([0:50]–[3:10]), which is a useful second reading of the numbers on this page.
Its qualitative summary is compact:

> "This is the main hierarchy you should have in your head: large memory is slow and
> far, but big, and fast memory like registers and L1 resides on the SM — it's local
> and it's fast, but small." ([3:10])

Three details it adds. SM counts have been roughly flat across generations — "about
100, between 100 and 200" — while **HBM size is "the number that's actually going up
quite a bit"** ([0:50]–[2:23]). Bandwidth is inversely correlated with distance, with
HBM slowest — though he adds that 8 TB/s "is still not that slow in the grand scheme
of things" ([2:23]). And L1 and shared memory are physically the same memory, differing in
who controls them: "shared memory you can control; L1 you can't" ([1:35]).

It also mentions two newer structures it then sets aside: thread block clusters on
H100 and B200, which "enable some amount of distributed memory", and tensor memory on
B200 sitting between registers and shared memory, "invisible to the programmer"
([3:56]).

## Related

- [GPU execution model](gpu-execution-model.md) — threads, blocks, warps, SIMT.
- [TPUs](tpus.md) — the same problems solved slightly differently.
- [Tensor cores](tensor-cores.md) — the matmul hardware inside the SM.
- [Tiling](tiling.md) — the main technique for exploiting this hierarchy.
- [Memory coalescing](memory-coalescing.md) — how DRAM actually delivers data.
- [Arithmetic intensity](arithmetic-intensity.md) — the accounting framework.
- [Lecture 5 — GPUs and TPUs](05-gpus-tpus.md).
- [Transcript](../raw/transcripts/05-gpus-tpus.md), [slide deck](../raw/slides/05-gpus-tpus.md).
