# TPUs, and what they share with GPUs

Lecture 5 spends two slides on TPUs, and the reason is not completeness. It is that
comparing them tells you which parts of GPU design are *necessary* and which are
incidental: "TPUs are kind of the alternative evolution of GPUs, and you can see
what's the same and what's not" ([16:53]).

The headline finding is convergence. "What's interesting is that TPUs are very
similar to GPUs at a high level — this is convergent evolution. If you want to build
an energy-efficient accelerator that does machine learning, you kind of end up at
the same place" ([17:40]).

## What is the same

A TPU has the same three structural pieces as a GPU ([18:27]):

- a specialized circuit for multiplying matrices,
- units for parallel vector operations,
- a control system,

and the same shape of memory hierarchy: slow **high bandwidth memory** and a faster
local memory called **SMEM** (shared memory). "So this looks very, very similar to
the architecture of a GPU."

Slide 14, from the JAX book, tabulates the correspondence, and Hashimoto's summary
is strong: "for every concept in the GPU, there's a corresponding concept in the
TPU, and the mapping is actually pretty precise" ([19:14]). Whatever he says about
GPUs in the rest of the lecture, "you can map directly to the TPU."

The correspondence goes deeper than the block diagram. A GPU tensor core and a TPU
MXU "actually use the same underlying circuitry — they're both what's called a
**systolic array**, which streams data in and out to do a matrix multiply"
([19:14]–[20:00]). Asked what size those arrays are in each case, Hashimoto does not
know and says so.

## The naming collision

This trips people constantly, so it is worth memorizing ([20:00]):

| Term | On a GPU | On a TPU |
| --- | --- | --- |
| Tensor core | the matrix multiply unit | the **processor** (equivalent of an SM) |
| MXU | — | the matrix multiply unit |

"They're named exactly the same thing, so you will have to disambiguate them based
on context. If someone says tensor core for a TPU, that means a processor. If
someone says tensor core for a GPU, they mean a matrix multiply unit."

## What is different: few and large versus many and small

The real design divergence is in how the matmul capacity is divided up ([20:46]):

| | GPU (H100) | TPU |
| --- | --- | --- |
| Processors | ~132 SMs | 2 |
| Matrix multiply units | 528 | 8 |

"The TPU is relying on a smaller number of much bigger matrix multiply units,
whereas the GPU is relying on smaller, but much more numerous, matrix multiply
units" ([20:46]).

TPUs are also "simpler — they're more optimized for the machine learning workload.
They have lighter-weight control units, and much bigger matrix multiply units"
([17:40]).

**The trade is flexibility against size.** Many small units "give you more
flexibility — you can program them to do this and that. Whereas with a TPU, you're
locked into big, big matrix multiplies" ([21:32]).

Hashimoto has a concrete war story for that constraint. In a recent paper of his, a
batch-size sweep "goes all the way down to 64 and stops at 64. Why is that? Because
the tensor core refuses to accept anything smaller than a 64-dimensional input — it
has to be big matrix multiplies" ([21:32]). The hardware's minimum granularity
showed up as a hole in an experimental result.

## What the lecture leaves out

Twice, Hashimoto names networking as the thing he is not covering and the thing that
actually distinguishes the two platforms:

> "The main difference that I won't get into today is the networking for these
> accelerators. So if you're thinking about the big difference between a TPU and a
> GPU, actually the biggest differences come in networking, not in the individual
> chips, because really the chips just live to multiply matrices" ([18:27]).

Take this as a caveat on the comparison above: at the level of one chip they are
near-twins, and the divergence that matters at cluster scale is a topic for the
parallelism lectures.

One other difference he does mention: the TPU equivalent of L2 cache "is much
faster, because they make different design tradeoffs on silicon to enable that.
That's one of their selling points" ([28:28]).

## Why this section exists at all

Because the convergence licenses everything else in the unit. "The core concepts
I'm teaching you today are going to transfer over between GPU and TPU. The basic
ideas are always going to be the same" ([21:32]) — memory hierarchy, matrix multiply
units, and the discipline of keeping data in fast memory.

His explanation for why convergence happened is worth keeping: "there are only so
many ways to cost-effectively allocate memory, and only so many ways to multiply
matrices very fast" ([22:18]).

## Related

- [GPU architecture](gpu-architecture.md) — the side of the comparison covered in
  depth.
- [Tensor cores](tensor-cores.md) — the GPU matmul unit, and the naming collision.
- [Lecture 7 — Parallelism](07-parallelism.md) — the other half of the TPU story,
  which is software. In "Jax-and-TPU land… you can simply define the model and the
  sharding strategy, and the compiler actually handles a lot of the decision" of
  which collectives to insert ([1:18:07]). CS336 uses PyTorch and calls the
  primitives by hand on purpose — the compiler route "would take a lot of the joy
  out of actually building things from scratch" ([1:18:54]). Asked to compare TPUs
  to the GPU networking picture, Percy declines: TPUs "are generally much simpler
  objects," but "maybe we can talk about it offline" ([35:37]).
- [Lecture 5 — GPUs and TPUs](05-gpus-tpus.md).
- [Transcript](../raw/transcripts/05-gpus-tpus.md), [slide deck](../raw/slides/05-gpus-tpus.md).
