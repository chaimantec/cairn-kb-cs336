# Lecture 5 — GPUs and TPUs

This lecture opens the course's **systems** unit. Lectures 3 and 4 asked what to
build; this one asks what the hardware does when you run it. Hashimoto's promise at
the start is unusually concrete: by the end you will understand a specific benchmark
plot — throughput of a square matrix multiply against matrix size — that looks, at
first sight, like noise ([0:51]).

He also makes a pitch for the unit as a whole: systems is "part of the stack that is
most reasonable, in the sense that you can reason through all the pieces and there
are logical steps you can go through to get the result" ([0:05]). Nothing here is
mysterious once you know what the machine is doing.

The argument for caring, even if you are not a systems person, is the course's own
thesis ([1:37]):

> "Thinking back to Percy's initial lecture: a big part of scaling is making sure you
> use your resources effectively, and without understanding your systems, you will
> never be efficient at using your resources."

See [efficiency](efficiency.md) for that framing, and
[resource accounting](resource-accounting.md) for the accounting it produces.

## Three parts

The lecture is organized as ([3:09]–[3:55]):

1. **What a GPU is** — the hardware model and the programming model.
2. **Six tricks** for making ML workloads fast on one.
3. **FlashAttention** as the victory lap, assembled entirely out of parts 1 and 2.

Hashimoto names his sources on slide 3 and is worth quoting on his own standing:
"I am not a systems person — surprise" ([2:22]). The credits are Horace He's blogs,
the GPU MODE community (renamed from CUDA MODE, which a student corrects him on
mid-sentence), and the JAX book by the Google team — "a really incredible resource"
whose exercises resemble the course assignment.

## Part 1 — what a GPU is

**Why GPUs at all.** Dennard scaling ended in the 2000s: transistors kept shrinking
but stopped getting faster, so "this old approach of making your clocks faster,
making instructions execute faster, is not going to work" ([5:26]). The replacement
was horizontal scaling — many slow things instead of one fast thing ([6:12]) — which
is what a GPU is.

**The hardware.** A GPU trades latency for throughput: small control units, enormous
numbers of lightweight cores, organized into **streaming multiprocessors**. Memory
sits in a hierarchy where speed tracks physical distance, from registers and shared
memory inside the SM out to global memory on separate chips. Full treatment in
[GPU architecture](gpu-architecture.md).

**The programming model.** Threads run in lockstep within 32-thread **warps** under
SIMT; **blocks** of threads are guaranteed to land on one SM and can therefore
cooperate through its shared memory. See
[the GPU execution model](gpu-execution-model.md).

**TPUs**, covered in two slides, turn out to be near-twins at the chip level —
convergent evolution toward the same memory hierarchy and the same systolic-array
matmul circuit, differing mainly in granularity (a TPU has 2 processors and 8 matmul
units against an H100's ~132 and 528) and, above the chip, in networking. See
[TPUs](tpus.md).

**The one hardware fact that shapes architecture design**: since the V100, matmuls
run on dedicated circuits and everything else does not, leaving a gap of **more than
10×** in throughput ([25:23]). That is why "any near-future machine learning
architecture that scales with compute is going to have a matrix multiply in it"
([24:37]). See [tensor cores](tensor-cores.md).

**The gap that motivates the rest of the lecture**: compute throughput is improving
much faster than memory bandwidth or interconnect ([25:23]). Optimization is
therefore mostly memory optimization, now and increasingly.

## Part 2 — six tricks

Slide 22 lists them. The first is about idle lanes; the other five are all about
moving less data.

| # | Trick | Where it is covered |
| --- | --- | --- |
| 1 | Control divergence | [GPU execution model](gpu-execution-model.md) |
| 2 | Low precision | [Microscaling formats](microscaling-formats.md), [precision and data types](precision-and-data-types.md) |
| 3 | Operator fusion | [Operator fusion](operator-fusion.md) |
| 4 | Recomputation | [Activation checkpointing](activation-checkpointing.md) |
| 5 | Memory coalescing | [Memory coalescing](memory-coalescing.md) |
| 6 | Tiling | [Tiling](tiling.md) |

The organizing frame is the **roofline model**, which Percy introduced earlier in the
course: below a threshold of arithmetic per byte moved you are memory-bound and extra
compute buys nothing; above it you are compute-bound. "What you need to do is be on"
the right side, "which means we want to increase the operational intensity"
([31:32]–[32:19]). See [arithmetic intensity](arithmetic-intensity.md).

A few things worth pulling out of the individual pages:

- **Control divergence** is the one non-memory trick. Under SIMT a branch makes
  *every* thread execute *both* sides, masking out the inapplicable one — so GPU code
  multiplies by masks where CPU code would branch ([33:05]–[34:37]).
- **Low precision** is where Nvidia is investing most, and a "pretty non-trivial
  part" of the historical FLOPs growth is simply counting in fewer bits ([34:37]).
  Modern formats are stranger than they look: MXFP8 attaches a separate E8M0 scale
  factor to every 32 elements, which makes a transpose expensive enough that training
  keeps **two quantized copies of every matrix** ([40:47]). The realized gain is
  20–30% on the matmuls, not 2×, because quantization has overhead ([42:18]).
- **Recomputation** is presented as pure systems arithmetic. Three stacked sigmoids
  cost 8 memory accesses if you store activations and 5 if you throw them away and
  recompute in the backward pass — "5/8 of the total memory accesses" for the same
  result ([52:17]). Slides 35 and 36.
- **Coalescing** turns on a DRAM property: a read returns a whole ~128-byte burst
  section, so a warp whose 32 addresses fall inside one section gets its data free,
  and one whose addresses scatter pays for 32 bursts and wastes most of each
  ([53:04]–[53:49]).
- **Tiling** is "the big one" ([57:37]): cut the matrices into tiles, load a tile into
  shared memory once, and reuse it. Global memory reads drop from $N$ per input to
  $N/T$ — a factor of $T$ (slide 42).

## The matrix mystery, solved

The benchmark from the opening is explained in three pieces, and this is the
lecture's set piece.

1. **The rising trend** is arithmetic intensity — bigger matmuls have more work per
   byte, so they climb the roofline ([1:05:59]).
2. **The bands** are tile alignment. Colour the points by the divisibility of the
   matrix size and they separate cleanly: sizes divisible only by 1 or 2 have "very
   bad throughput," while 16 and 32 "perform equally well" ([1:06:45]). Not because
   powers of two are magic, but because a tile that fits the burst window gets
   coalesced reads ([1:07:32]).
3. **The periodic cliffs** are [wave quantization](wave-quantization.md). Between
   $N = 1792$ and $N = 1793$ the tile count goes from 98 to 120 against an A100's 108
   SMs, so a second wave begins and most of the GPU idles through it ([1:08:17]).

The same reasoning explains Karpathy's nanoGPT result, reproduced on slide 45:
padding the vocabulary from 50257 to 50304 gave a ~25% speedup. More arithmetic, less
wasted bandwidth ([1:05:14]).

## Part 3 — FlashAttention

The finale assembles the tricks. FlashAttention changes no mathematics; it is "all
systems — going from PyTorch's naive implementation of attention to a single, very
cleverly fused kernel" ([1:12:09]), and its gains come from moving less data to and
from global memory.

The obstacle is that softmax is global and so resists tiling. The resolution is the
**online softmax**: keep a running maximum and a running denominator, and rescale the
accumulator by $e^{m_{j-1}-m_j}$ whenever the maximum changes. That makes the softmax
computable tile by tile, and the $N \times N$ score matrix never has to be
materialized. The backward pass then uses recomputation rather than storing those
activations. Full derivation in [FlashAttention](flash-attention.md).

## What Hashimoto wants you to take away

His own summary, at [1:17:30]:

- **Understand the hardware, down to the low-level details.** "I don't want you to be
  the kind of people who cargo-cult, say, multiples of 32 for your matrices — I want
  you to really understand why we do these things."
- **Think about matmuls**, because they are the arithmetically dense operations the
  hardware privileges.
- **Think carefully about data movement**, because of the compute–memory gap.
- **Hardware-aware architecture work pays.** FlashAttention is the example.

## Related

- [Course map](course-map.md) — where this lecture sits in the syllabus.
- [Arithmetic intensity](arithmetic-intensity.md) and
  [resource accounting](resource-accounting.md) — the accounting this lecture assumes.
- [Attention variants](attention-variants.md) — the architectural approach to
  attention cost, as against this lecture's systems approach.
- [Lecture 4 — attention alternatives](04-attention-alternatives.md), the previous
  lecture.
- [Transcript](../raw/transcripts/05-gpus-tpus.md) —
  [verbatim captions](../raw/transcripts/original/05-gpus-tpus.md).
- [Slide deck transcription](../raw/slides/05-gpus-tpus.md) — 55 pages;
  [`lecture_05.pdf`](https://github.com/stanford-cs336/lectures/blob/main/lecture_05.pdf).
