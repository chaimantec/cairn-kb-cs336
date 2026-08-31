# Profiling

Where [benchmarking](benchmarking.md) gives you one number, profiling tells you
where the time inside it went. Lecture 6 adds a second reason to do it that has
nothing to do with speed:

> "Even if you don't care about the time — [profiling] helps you figure out what's
> actually happening under the hood, because especially with these high-level
> languages, you write some code, it runs, you get some result, and sometimes it
> can just be good to understand what's actually going on." ([26:10])

CS336 uses PyTorch's built-in profiler in the lecture and **nsight** in the
assignment, "which gives you more details" ([26:57]).

## What it reveals: PyTorch is not what you wrote

Profiling `a + b` on two tensors is the deliberately boring case. "If you're not
normally doing PyTorch, you probably don't think about it — it's like, okay, well,
these two tensors just get added. So what's actually going on underneath the hood?"
([27:43]). The answer is a CUDA kernel with a long generated name, doing the add.

Profiling `a @ b` is where it gets interesting, because the lecture profiles it at
**two different sizes** and gets **two different kernels**:

> "Similarly, there is this long name that describes this particular matmul
> kernel... Notice that if you change the dimensions — now I'm doing a 128-by-128 matmul —
> you get a different one. If you look closely, this is 64, 64, 16; this is 32, 32,
> 16. So underneath the hood, in PyTorch, it looks like you're just doing add, but
> underneath the hood there could be all sorts of things happening." ([28:30])

The two observations the source draws are that you can see which CUDA kernels are
actually being called, and that **different kernels are invoked depending on the
tensor dimensions**.

## Reading a kernel name

The names are not noise. The lecture decodes one ([29:17]):

```
cutlass3x_sm100_simt_sgemm_f32_f32_f32_f32_f32_64x64x16_1x1x1_3_nnn_align1_bi...
```

| Fragment | Meaning |
| --- | --- |
| `cutlass` | NVIDIA's CUDA library for linear algebra |
| `sm100` | the Blackwell architecture (B200) — "a kernel that's specifically designed for Blackwell" |
| `f32` | float32 |
| `64x64x16` | the **tile shape** |

That last fragment is the connection to the rest of the lecture: the tile shape in
a library kernel's name is the same tile the [tiled matmul](tiling.md) kernel
picks with `BLOCK_M`, `BLOCK_N`, `BLOCK_K`, and it is what changed when the matrix
got smaller.

## Profiling explains the GeLU race

The three-way GeLU comparison ([32:23]–[35:27]) is the lecture's worked example of
profiling as diagnosis, and the profile is what turns a speed difference into an
explanation.

- **Naive PyTorch**: many kernels — "binary functor, unary, add, tanh is a kernel
  here" — one per primitive in the computation graph, each of which "has to read
  from HBM, pull it all the way over to your SM, do the computation, and write it
  back" ([33:08]).
- **Built-in**: a single hand-written CUDA kernel for GeLU. "There's nothing
  magical about it" ([33:53]).
- **Compiled**: also a single kernel — and a **Triton** one, which is how the
  lecture arrives at its subject ([34:39]). See [torch.compile](torch-compile.md)
  and [operator fusion](operator-fusion.md).

## The harness

```python
with torch.profiler.profile(activities=[ProfilerActivity.CUDA], ...) as prof:
    run()
    torch.cuda.synchronize()
table = prof.key_averages().table(sort_by="cuda_time_total", row_limit=10)
```

Same warm-up-then-synchronize discipline as the benchmark harness (lecture source
lines 237–259), sorted by total CUDA time and truncated to ten rows.

> **The profiler tables are not reproduced in this KB.** They are the literal
> output of a run on the lecturer's GPU. The
> [lecture source](../raw/slides/06-kernels-triton.md) marks them
> machine-dependent, and the kernel name above is the one the source itself prints
> as an example rather than a reading off any particular table.

## Related

- [Benchmarking](benchmarking.md) — the end-to-end half of the same recipe.
- [torch.compile](torch-compile.md), [operator fusion](operator-fusion.md) — what
  the GeLU profiles diagnose.
- [Tiling](tiling.md) — what `64x64x16` in a kernel name means.
- [Bank conflicts](bank-conflicts.md), [warp occupancy](warp-occupancy.md) — other
  things a profiler will show you ([16:15]).
- [Lecture 6 — Kernels and Triton](06-kernels-triton.md).
- [Transcript](../raw/transcripts/06-kernels-triton.md),
  [lecture source](../raw/slides/06-kernels-triton.md).
