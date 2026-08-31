# torch.compile

`torch.compile` takes an ordinary PyTorch function and returns another function
that computes the same thing faster. Lecture 6 uses it as the third horse in a
three-way race, and then as the doorway into the rest of the course's kernel
material — because what it emits is a [Triton](triton.md) kernel.

## The mechanics, as the lecture presents them

"You can take any PyTorch function, call `torch.compile` on it, and it generates
another function that does the same thing" ([30:50]). In the lecture:

```python
compiled_gelu = torch.compile(naive_gelu)
check_equal_1d(naive_gelu, compiled_gelu)  # compilation shouldn't change semantics
```

The correctness check is part of the point. All three GeLU implementations return
the same value on the same input — compilation buys speed, not different answers.

## What it actually did

The naive implementation is one PyTorch expression, but PyTorch executes it as a
computation graph in which "each primitive in the computation graph is actually
realizing a kernel" ([33:08]) — and every kernel boundary is a round trip through
HBM. The [profiler](profiling.md) shows several of them: binary functor, unary,
add, tanh.

The compiler reads that same graph and produces **one** kernel:

> "You can take a naive implementation, which, remember, in PyTorch, it has a
> computation graph, and run a compiler, which, if you look at what's underneath
> the hood, is just a single kernel. And this is because it's figured out how to
> look at the computation graph and essentially write that kernel in Triton. So you
> can see that this is actually a Triton kernel." ([34:39])

That is [operator fusion](operator-fusion.md), performed for you: "all the
operations in the GeLU have been fused together into one kernel. So you read from
HBM once, you write to HBM once, per element" ([34:39]).

## The honest scoreboard

The lecture does not tidy up the result. Compiled beat naive and **lost to the
hand-written built-in** on the day, and a student asked why:

> "So, why is a Triton kernel faster? So, a Triton kernel is actually not faster in
> this case — oh, sorry, the compiled kernel is one Triton kernel, and this is
> slower than the built-in. I think last year when I did this, it was actually
> closer, but these things change, and it's very hardware-dependent. I think none
> of this is terribly optimized — this is just giving you the general idea here."
> ([36:13])

Two things worth carrying from that. Compilation is not a guarantee of the best
kernel — the built-in GeLU is a CUDA kernel someone wrote by hand and shipped in
the standard library, "there's nothing magical about it" ([33:53]). And the result
is a fact about one afternoon on one GPU, which is why this KB quotes no timings;
see [benchmarking](benchmarking.md).

## Why the lecture cares

Because the compiled kernel is written in Triton, the compiler has just
demonstrated the thing the rest of the lecture teaches you to do by hand. The
lecture moves straight on: "so that maybe is a good segue to talk about what this
Triton thing is all about" ([35:27]).

[Lecture 5](05-gpus-tpus.md) had already set the boundary of what a compiler will
do for you: it fuses chains that are already fusible, but restructuring a
computation so that it *becomes* fusible — the [FlashAttention](flash-attention.md)
case — is a human's job. Lecture 6 is what you do on the far side of that boundary.

## Related

- [Operator fusion](operator-fusion.md) — the transformation being applied.
- [Triton](triton.md) — what it emits.
- [Profiling](profiling.md) — how the lecture found out what it emitted.
- [Lecture 6 — Kernels and Triton](06-kernels-triton.md).
- [Transcript](../raw/transcripts/06-kernels-triton.md),
  [lecture source](../raw/slides/06-kernels-triton.md).
