# Operator fusion

Operator fusion is the second of Lecture 5's six tricks, and the one Hashimoto
calls "much more of a conceptual programming thing... a fairly simple idea, but
you'll be surprised how much it doesn't already happen" ([47:40]).

## The factory analogy

Slide 30's picture, which the lecture leans on: think of a GPU as a factory. There
is a memory warehouse, a compute factory, and a conveyor belt running between them.

Scale up the compute and the conveyor belt becomes the bottleneck. Worse, the
problem compounds with the number of operations: "imagine I've got lots of
different operations to do, and I'm shipping raw materials to my factory and back,
over and over. That's a lot of memory bottleneck you're paying for, and lots of
duplex, bidirectional memory bandwidth" ([47:40]).

The fix follows immediately: "wouldn't it be better if I just had one giant factory
that took all the raw materials and shipped back my finished products? This way I
only have to pay for memory bandwidth twice" ([48:26]).

## The worked example

Slide 32 computes $\sin^2 x + \cos^2 x$. The PyTorch computation graph is five
pointwise operations: sine, cosine, two squarings, one addition.

Done naively, "each of these will be its own factory, so to speak — you'll read and
write from global memory every time one of these is called, and incur quite a bit
of memory cost each time" ([48:26]). Five operations, five round trips to global
memory, for arithmetic that is trivial by comparison.

Fused, the whole chain becomes one kernel: read once from global memory, do
everything inside the SM, write the result back once ([49:14]). Slide 33 states
that all five pointwise operations can be fused.

This is the same reasoning as [arithmetic intensity](arithmetic-intensity.md)
applied to a chain rather than a single operation. Each pointwise op in isolation
has terrible intensity — a couple of FLOPs per several bytes moved. Chaining them
without fusion does not improve that; fusing them multiplies the arithmetic done
per byte moved by the length of the chain.

## Compilers do the easy cases for you

Hashimoto is clear that for straightforward chains you do not write this yourself.
"Anything that's easy like this — this is a very easy fusion — is like a graph that
I can just squish down into a single unit; a compiler can automatically do this for
you" ([49:14]).

Two named compilers, both of which "will fuse these kinds of basic things into a
single CUDA call":

- **Torch Compile** — PyTorch's compiler.
- **JAX Compile** — JAX's compiler, which he describes as "more deeply integrated
  into JAX."

Slide 33's figures show the ATen/Inductor graph representation this operates on.

His summary for the simple case: "you can just think of compilation as solving your
problems" ([50:00]).

## Where it stops being automatic

The caveat is stated in the same breath: "the advanced version sometimes requires
manual intervention" ([50:00]).

[FlashAttention](flash-attention.md) is the lecture's example of the advanced
version. Fusing the exponential into the attention kernel is one of the three
ingredients slide 54 lists, and no compiler derived FlashAttention on its own — it
required the online-softmax reformulation first. That is the general shape of it:
compilers fuse chains that are already fusible, but restructuring a computation so
that it *becomes* fusible is a human's job.

## Lecture 6: the same argument, measured

Lecture 6 runs the experiment that Lecture 5 describes. Three implementations of
GeLU — naive PyTorch, the built-in, and `torch.compile`d — compute identical values
at very different speeds, and the [profiler](profiling.md) shows why: the naive
version's computation graph becomes several kernels, "and the reason this is slow is
that when you launch a kernel, the kernel has to read from HBM, pull it all the way
over to your SM, do the computation, and write it back. And then the next kernel
picks it up from HBM" ([33:08]).

Two of the three are fused into a single kernel — "you read from HBM once, you write
to HBM once, per element" ([34:39]) — and the difference between them is instructive:
the built-in is a CUDA kernel a human wrote and shipped in the standard library,
while the compiled one was generated. See [torch.compile](torch-compile.md), which is
also where the lecture reveals that what the compiler emitted is a
[Triton](triton.md) kernel.

Lecture 6 also shows fusion arriving *inside* a kernel you were already writing: its
[tiled matmul](tiling.md) applies ReLU to the accumulator before the single write to
HBM, making the activation function nearly free ([1:21:06]). And [fused
softmax](fused-softmax.md) is the case where the accounting is done exactly — five
PyTorch operations costing $5MN + M$ reads against the $MN$ a fused kernel needs.

## Related

- [FlashAttention](flash-attention.md) — fusion as one of its three ingredients.
- [Arithmetic intensity](arithmetic-intensity.md) — the framework that says why
  fusion pays.
- [GPU architecture](gpu-architecture.md) — the memory hierarchy the round trips
  traverse.
- [Lecture 5 — GPUs and TPUs](05-gpus-tpus.md).
- [Transcript](../raw/transcripts/05-gpus-tpus.md), [slide deck](../raw/slides/05-gpus-tpus.md).
