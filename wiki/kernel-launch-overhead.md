# Kernel launch overhead and tail effects

Where a GPU's time goes when it is *not* computing — and why that idle time
dominates decode. This is the problem [megakernels](megakernels.md) exist to
solve, set out in [Lecture 18](18-serving-megakernels-recurrence.md)
(≈34:47–≈37:06).

## The picture

The lecture's central figure plots time on the x-axis against the GPU's streaming
multiprocessors on the y-axis:

> On an H100 there are 132 of these; on the B200 there are, I think, 148, etc.
> The bars indicate useful work — when there's a bar here, one of the processors
> on the GPU is actually doing work, and the empty space is just waiting, waiting
> for other operations to finish so that something else can go on. (≈36:18)

The conclusion drawn from it is unusually strong, and it is what justifies the
whole megakernel line of work:

> Basically you get into this position where, no matter how well you try to write
> the kernel, you're always going to have downtime on your GPU. (≈36:18)

That "no matter how well" is the point. These are not inefficiencies inside a
kernel — the kind [Lecture 6](06-kernels-triton.md) teaches you to remove with
tiling, fusion and [memory coalescing](memory-coalescing.md). They are structural
costs of the *boundaries between* kernels, and no amount of care inside one kernel
touches them.

## The three sources of idle time

**1. Kernel launch and teardown.** Every kernel invocation has fixed setup and
shutdown cost — "the kernel launch and kernel teardown, that's these big gaps in
the red and the yellow" (≈36:18). Because a language model layer is many small
operations, this is paid many times per token.

**2. Tail effects.** A batch finishes when its slowest member finishes:

> This is just the same way that if you have a short prompt that gets processed
> with a very long prompt — this same thing goes all the way down to the basic
> attention operation. If you're processing a batch of inputs and one input is
> very short, one is very long, you're going to be waiting for the very long input
> to finish. (≈36:18, ≈37:06)

Note that the identical phenomenon appears one level up, between requests on a
cluster, where it motivates [cache-aware routing](cache-aware-routing.md). Mixing
costs in one batch wastes the batch, at every scale.

**3. Accumulated inter-kernel gaps.** "Because you're running these across
multiple kernels, you'll actually start to see these gaps between kernels add up"
(≈37:06). Individually negligible, they are paid once per operation per token.

## Why decode makes this acute

For training or prefill, each kernel does a great deal of work, so fixed
per-launch costs amortize. Decode is the opposite: one token at a time, the whole
model loaded to produce it, very little arithmetic per launch. The parallel
machine is reduced to "basically a glorified memory loader" (≈34:47), so the
overhead is a large fraction of a small total. See
[prefill/decode disaggregation](prefill-decode-disaggregation.md).

## Why we write kernels this way anyway

Not carelessness — tractability. The one-kernel-per-operation convention exists
because kernels are hard:

> Kernels tend to be pretty challenging to write, I'm sure you all had a lot of
> fun writing flash attention — but that means that typically what we do is look
> at all the different operations in a language model, and we will write a single
> kernel for that operation at a time. This makes things a lot easier to program,
> because you just have to write your norm kernel, or your map kernel, or
> attention kernel — but it ends up introducing a lot of downtime into your
> system. (≈34:47, ≈35:33)

So this page describes a genuine engineering trade-off rather than a mistake:
per-operation kernels buy modularity and reusability, and pay for it in boundary
overhead. [Megakernels](megakernels.md) take the opposite side of that trade, and
pay for it in labour — "a fully talented kernel engineer, over the course of a
year… for two or three models, for batch sizes one to 16" (≈1:03:26).

## See also

- [Megakernels](megakernels.md) — the response to this problem
- [Kernels and Triton (Lecture 6)](06-kernels-triton.md) — optimizing *within* a
  kernel
- [GPU execution model](gpu-execution-model.md) ·
  [GPU architecture](gpu-architecture.md) ·
  [Arithmetic intensity](arithmetic-intensity.md)
- [Lecture 18](18-serving-megakernels-recurrence.md)
