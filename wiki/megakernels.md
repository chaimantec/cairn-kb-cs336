# Megakernels

Fusing an entire model layer — in principle an entire model — into **one GPU
kernel**, so that the boundaries between operations stop costing time. A
collaboration between Stanford and Together, presented as the first of two
research threads in [Lecture 18](18-serving-megakernels-recurrence.md)
(≈34:00–≈41:42, with the trade-offs at ≈1:03:26 and multi-GPU work at ≈1:10:22).

The problem it attacks is on
[kernel launch overhead and tail effects](kernel-launch-overhead.md): decode
leaves the GPU idle at kernel boundaries, and "no matter how well you try to write
the kernel, you're always going to have downtime on your GPU" (≈36:18).

## The idea

> Instead of treating each operation in the model as its own operation and writing
> a kernel for it, let's write a single kernel to cover multiple operations at
> once. This is similar to the fusion that you see in flash attention, except done
> more aggressively, across a larger number of things. (≈37:06)

The relationship to [FlashAttention](flash-attention.md) is the right way in: that
kernel fuses the attention computation so intermediates never reach HBM. A
megakernel applies the same reasoning at the scale of a whole layer, and the
consequence is a change of mental model:

> In particular, what it does is turn the GPU from a single device, a single
> operation, into — you start thinking of the GPU as a massive distributed system,
> and saying, okay, I have all this work that I need to get done, some of it has
> dependencies on other stuff… how can I schedule it, how can I distribute the
> work to maximize my GPU utilization? (≈37:54)

That reframing is what the technique really is. Once the whole layer is inside one
kernel, the [SMs](gpu-execution-model.md) become workers in a dependency-scheduling
problem rather than a grid executing one operation in lockstep.

## What the fusion buys: overlap across operation boundaries

With no kernel boundaries to synchronize at, work from *different* operations can
run at the same time — "you see things overlapped in really weird ways… we're now
starting to overlap, say, a weight load from the next layer into the attention, or
starting to run parts of a reduction before the attention operation is over"
(≈38:39).

Two concrete instances, both from decode:

- **Load the KV cache during QKV.** In a modern attention layer, "you have the QKV
  projections, and you're going to add some [RoPE](rope.md) scaling to it," and the
  insight is "that you can start loading your KV cache into attention before
  you're finished with QKV — particularly during decode." Once QKV completes and
  the new query tokens exist, the rest of attention proceeds (≈38:39, ≈39:25).
- **Load the O-projection weights during attention.** "You have your O projection
  start loading the weights before your attention operation is over" (≈39:25).

Both are the same move: a memory-bound step and a compute-bound step that a kernel
boundary would have serialized are allowed to run together. In a phase that is
[bandwidth-bound](arithmetic-intensity.md), hiding loads behind compute is
precisely the win available.

## Results, as stated

- **30–70% speedups** applying it to the attention inference kernel alone (≈37:54).
- Applied to a **whole layer of a Llama 1B model** (≈37:54).
- **72% of achievable memory bandwidth on an H100** — "near-speed-of-light
  decoding inference… if you ignore all the complexities of what we're doing here
  and just ask how fast the GPU can physically go to do this operation, we're
  pretty close" (≈40:56). The lecturer adds that "these numbers are actually a lot
  better now".

The last figure is stated as a fraction of the hardware limit rather than a
multiple of a baseline, which is the more informative form: it says how much room
remains. A speedup number cannot.

## The implementation

> We put this together in a relatively complex CUDA framework, with basically an
> instruction-based abstraction where we can implement each subkernel in its own
> file, and then have a big virtualized shared-memory system to orchestrate the
> running of these operations. (≈40:11)

Two design elements worth noting, because they are what make a megakernel
writable at all: an **instruction abstraction** so each fused sub-operation stays
a separate source file, and a **virtualized shared-memory system** to coordinate
them. Without those the code would be one unmaintainable kernel.

It is built on **ThunderKittens**, a kernel-writing library: "you can think of it
as almost like [Triton](triton.md), except more low-level, with a lot more
fine-grained control over things" (≈40:11). [Lecture 6](06-kernels-triton.md)
teaches the Triton end of that spectrum.

## The cost, which is people

The Q&A is blunt about why this is not standard practice (≈1:03:26):

> The trade-offs are people's blood, sweat, and tears — megakernels turn out to be
> very, very labor-intensive to write. To give you some context, I think a fully
> talented kernel engineer, over the course of a year, will probably be able to
> write megakernels for one hardware, for two or three models, for batch sizes one
> to 16. You go to batch size 17, you're like, nope, start over, got to go again.

Read that as a specificity cost: the artifact is tied to one hardware generation,
a couple of model architectures, **and a batch-size range**. The current response
is tooling — "at Together, we're trying to put together some compilers that can
automate some of that process, but it's a very challenging thing to do" (≈1:04:11)
— and the lecture notes the idea is old, having "gone in peaks and troughs over
the last few decades when it comes to GPU programming" (≈1:04:11).

## Multi-GPU, and where this is heading

Can a megakernel survive communication between GPUs? Partly (≈1:10:22):

> Turns out you can also fuse the NCCL calls into the megakernel, if you set it up
> correctly. I think we haven't found a really great killer use case for that yet,
> where sometimes you're just bound by the latency of the NCCL call itself.

He cites a DeepSeek megakernel for the mixture-of-experts inference layer that
fused some communication. The forecast — the last technical statement of the
course — is partial adoption rather than total:

> You'll have a little megakernel for part of the computation, but you won't
> necessarily have a megakernel for the whole model, unless, of course, you pay
> the blood, sweat, and tears price, and then you really get the whole thing
> going. (≈1:11:09)

There is also a speculative convergence with the lecture's other thread: if a
[looped transformer](looped-transformers.md)'s recurrent block were small enough,
"you could actually write a little megakernel to just do that recurrence in a very
fast megakernel loop." It has not worked yet — "so far we haven't been able to
make those blocks small enough" (≈1:02:38).

## See also

- [Kernel launch overhead and tail effects](kernel-launch-overhead.md) — the
  problem
- [Kernels and Triton (Lecture 6)](06-kernels-triton.md) ·
  [FlashAttention](flash-attention.md) · [Triton](triton.md)
- [Decode-specialized hardware](decode-specialized-hardware.md) — the other answer
  to the same bottleneck
- [Lecture 18](18-serving-megakernels-recurrence.md)
