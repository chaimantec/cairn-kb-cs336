# Lecture 6 — Kernels and Triton

Lecture 5 gave the GPU from the outside: what a chip is made of, and six tricks
that make one go fast. This lecture is the hands-on half of that pair. Percy Liang
takes the same hardware and asks two practical questions — *how do you find out
where your time is actually going*, and *how do you write the kernel that fixes
it* — then answers the second one four times over, in Triton, on kernels of
steadily increasing difficulty.

The order is deliberate and he says so: "the reason I'm doing benchmarking and
profiling as opposed to teaching you Triton earlier is because you should always
just measure what's going on in your code and figure out what the bottlenecks are
before you start writing kernels" ([22:22]).

This is an [executable lecture](executable-lectures.md) — the material is the
program [`lecture_06.py`](../raw/slides/06-kernels-triton.md), not a slide deck,
so citations below name functions and source lines rather than slide numbers.

> **A note on the title.** Cairn's catalog entry calls this "Lecture 6: Kernels,
> Triton, XLA". Neither the program nor the captions mention XLA or JAX at any
> point. This KB follows the course site and calls it *Kernels, Triton*.

## The shape of the lecture

1. **A review of GPUs** that is not quite a review — five hardware details that
   the programming model hides and performance does not.
2. **Benchmarking and profiling**, with the harnesses written from scratch.
3. **The GeLU race** — naive vs. built-in vs. compiled — which ends by revealing
   that `torch.compile` emitted a Triton kernel, and hands the lecture its subject.
4. **Four Triton kernels**: elementwise, a reduction that fits in a block, a
   reduction that does not, and tiled matmul.

## Part 1 — five places the hardware shows through

The programming model is three things — thread, thread block, grid — and Percy
calls it "fairly simple, I think" ([7:02]). The abstraction is honest about
correctness and silent about speed: "if you just care about correctness, that's
all you need to know. But in practice, the performance is very sensitive to the
hardware, and so you need to really deeply understand the hardware to obtain high
performance" ([7:47]).

![One GPU: eight SMs over a shared L2, with a single link to HBM](../raw/images/06-kernels-triton/gpu-hardware.png)

*One GPU drawn as eight streaming multiprocessors, each with its own register file and L1/shared memory, sitting over a shared L2 cache, with a single link out to HBM. This is the memory hierarchy a Triton kernel is written against. Source: [`images/gpu-hardware.png`](https://github.com/stanford-cs336/lectures/blob/main/images/gpu-hardware.png).*

The five details, each with its own page:

- **[Warps and control divergence](gpu-execution-model.md)** — 32 threads that
  must execute the same instruction every cycle, so a branch runs both sides in
  sequence ([9:21]). The upside of the same mechanism: an SM keeps many warps
  resident and switches between them "with zero cost" to hide a ~100-cycle HBM
  read ([10:07]).
- **[Warp occupancy](warp-occupancy.md)** — each thread may hold at most 255
  registers, and the more it holds the fewer threads fit on an SM. The worked
  example lands at **18%** occupancy ([13:12]).
- **[Bank conflicts](bank-conflicts.md)** — shared memory is 32 banks of 4 bytes,
  one thread per bank per cycle, so 32 threads reading one column of a matrix
  serialize completely ([14:42]).
- **[Memory coalescing](memory-coalescing.md)** — the HBM counterpart: a warp's 32
  accesses are combined into 128-byte cache lines, so reading down a column
  fetches memory you never use ([17:01]). "This can feel similar to bank
  conflicts, but it's a very different constraint: that one is about shared
  memory, and this one is about HBM" ([17:46]).
- **[Block occupancy](wave-quantization.md)** — 160 thread blocks on a 148-SM chip
  run as one full wave and one wave of 12, leaving most of the machine idle
  ([18:31]).

His own summary of why this is hard is worth keeping: the profiler "tells you a
bunch of information, but you have to know exactly how many SMs there are, and
exactly the sizing of everything, and sometimes the scheduler does something you
don't really have control over. So it's a lot messier than the programming model"
([19:18]).

## Part 2 — measure first

The recipe is three lines long: benchmark and profile, make changes, benchmark and
profile again.

**[Benchmarking](benchmarking.md)** answers "how long?" and nothing else — but
because it distils to one number, it shows you scaling. The lecture's own sweep of
matmul across dimensions 256 to 8192 has a shape worth remembering: time is
roughly *constant* until about 2000 dimensions and cubic after, because "these
GPUs are built for fairly large matrix multiplications, and if you have like a
2-by-2 matrix, it's going to be very inefficient" ([25:24]–[26:10]).

**[Profiling](profiling.md)** answers "where?", and Percy makes a second argument
for it that has nothing to do with time: "even if you don't care about the time
— [it] helps you figure out what's actually happening under the hood, because
especially with these high-level languages, you write some code, it runs, you get
some result" ([26:10]). Profiling `a + b` reveals a CUDA kernel you never asked
for; profiling `a @ b` at two different sizes reveals *two different kernels*
([28:30]).

## Part 3 — the GeLU race, and where Triton comes from

Three implementations of the same function ([30:02]–[35:27]): the naive PyTorch
expression, `torch.nn.functional.gelu`, and `torch.compile(naive_gelu)`. They
agree on every input — the lecture checks — and differ wildly in speed.

The profiler explains it, and the explanation is [operator
fusion](operator-fusion.md). The naive version's computation graph turns each
primitive into its own kernel, and "when you launch a kernel, the kernel has to
read from HBM, pull it all the way over to your SM, do the computation, and write
it back. And then the next kernel picks it up from HBM" ([33:08]). The built-in is
one hand-written CUDA kernel — "people use GeLU, so someone wrote a kernel for it
and put it in the standard library — there's nothing magical about it" ([33:53]).

The third one is the lecture's hinge. [`torch.compile`](torch-compile.md) reads the
same computation graph and emits **one kernel, written in Triton** ([34:39]).
Everything that follows is learning to write by hand what the compiler was already
writing for you.

Percy is honest about the scoreboard rather than tidying it: on the day, compiled
was faster than naive but *slower* than the built-in, and "I think last year when I
did this, it was actually closer, but these things change, and it's very
hardware-dependent" ([36:13]).

## Part 4 — four kernels

[Triton](triton.md) sits between CUDA and PyTorch. In CUDA you say what each
*thread* does; in Triton you say what each *thread block* does, and the conceptual
framework is one sentence: a block "is going to load data into shared memory,
operate on it, and write it back to global memory" ([38:32]).

![The Triton softmax kernel, one program instance per row](../raw/images/06-kernels-triton/triton-softmax.png)

*The softmax kernel: one program instance per row, pid being the row index. Row 1 is loaded with tl.load, its max subtracted, exp and sum taken with tl.exp and tl.sum, and the normalized row written back with tl.store. Source: [`images/triton-softmax.png`](https://github.com/stanford-cs336/lectures/blob/main/images/triton-softmax.png).*

![A worked trace of the Triton row-sum kernel on a 4 x 10 input](../raw/images/06-kernels-triton/triton-row-sum.png)

*A worked trace of the row-sum kernel. Block 1 (pid=1) takes row 1 and walks it in three tiles of BLOCK_SIZE 4: cols 0-3 load x = 3, 1, 4, 1 into four threads' accumulators; cols 4-7 add 5, 9, 2, 6 to give acc = 8, 10, 6, 7; cols 8-11 load 5, 3 and mask the two out-of-range lanes. tl.sum then tree-reduces across threads to out[1] = 39. Source: [`images/triton-row-sum.png`](https://github.com/stanford-cs336/lectures/blob/main/images/triton-row-sum.png).*

Every kernel in the lecture has the same skeleton, which Percy states outright at
[50:09]: "you generally have your inputs, your outputs — you wake up, you figure
out which index you're going to look at, you read, you do some stuff, and then you
write to HBM."

**1. GeLU — elementwise** ([39:18]–[45:30]). An 8192-element vector, `BLOCK_SIZE`
1024, eight blocks. The block asks "who am I?" with `tl.program_id`, turns that
into a range of `offsets`, builds a `mask` so the last block does not read past the
end, `tl.load`s, computes on the whole vector as if it were PyTorch, and
`tl.store`s. The detour through [PTX](ptx.md) that follows ([50:55]–[53:13]) shows
what the hardware actually runs, including a piece of optimization nobody asked
for: the compiler gave each thread **eight** elements rather than one, because "this
thread is pretty lightweight, it doesn't do that much, so let's just try to thicken
it up a little bit" ([52:28]).

**2. Softmax — a reduction that fits in a block** ([57:54]–[1:04:05]). Each row
becomes one block, because a row is what softmax has to normalize over. The
motivation is arithmetic: the naive PyTorch version costs $5MN + M$ reads and
$3MN + 2M$ writes where a fused kernel needs $MN$ of each. See [fused
softmax](fused-softmax.md).

**3. Row sum — a reduction that does not fit in a block** ([1:04:52]–[1:11:04]).
4096 columns, a block size of 1024, so each thread walks a *sequence of tiles*
accumulating into its own register, and only at the end is the accumulator vector
collapsed to a scalar. Percy is careful about the vocabulary here, because the
picture looks like the GeLU one and means something different: "in GeLU, we also
split a row into a bunch of pieces, but those were blocks — each of those pieces
was a block that was processed independently. These are not blocks, these are
tiles. The block corresponds to this whole row, and has to process all the tiles"
([1:11:04]). He calls this "baby tiling."

**4. Matmul + ReLU — real [tiling](tiling.md)** ([1:11:51]–[1:21:51]). Three
approaches in a ladder, measured by [arithmetic
intensity](arithmetic-intensity.md): the naive one element-at-a-time kernel is
$O(1)$; loading all of A and B into shared memory would be $O(N)$ but they do not
fit; tiling is "globally look like the naive approach, but locally look like the
idealized approach" ([1:16:29]) and gets $O(\text{tile size})$. The ReLU is fused
onto the accumulator before the single write-back — a free activation function,
which is why the lecture bothered to add one.

## What Percy wants you to take away

His own summary ([1:21:51]–[1:23:24]), in his order:

- **The programming model is a stack you can choose your level in** — PyTorch,
  Triton, or PTX — "this is what the programmer can control."
- **The hardware is finite and your code has to fit in it.** "You come in with your
  big matrix and transformer, and it has to fit in with the constraints of the
  hardware. So that's why benchmarking and profiling are really important, to
  understand how the messiness of the hardware translates to performance."
- **Thread blocks are the right unit to think in.** "Things are easier to think
  about with thread blocks than individual threads, because you don't have to
  think about explicitly synchronizing threads or doing shared memory."
- **The four examples are a difficulty ladder**, and the last rung is the one the
  assignment needs: by the end "you'll have all the ingredients you need to do the
  assignment and implement [flash attention](flash-attention.md)" ([57:54]).

Asked at the end what the alternatives to Triton are, he frames it as inductive
bias rather than ranking: "every language has an inductive bias, that makes certain
things easier and certain things harder. Most of what we do — Triton was built by
people who train transformers" ([1:24:09]). He names ThunderKittens and CuTe as
other DSLs, and PTX as the extreme you can always drop to but "wouldn't advise
that as a first step" ([1:24:54]).

Next lecture: more than one GPU.

## Related

- [Triton](triton.md), [PTX](ptx.md) — the two languages this lecture writes in.
- [Benchmarking](benchmarking.md), [profiling](profiling.md) — the measurement half.
- [Warp occupancy](warp-occupancy.md), [bank conflicts](bank-conflicts.md),
  [memory coalescing](memory-coalescing.md), [wave quantization](wave-quantization.md),
  [the GPU execution model](gpu-execution-model.md) — the five hardware details.
- [Fused softmax](fused-softmax.md), [tiling](tiling.md), [operator
  fusion](operator-fusion.md), [torch.compile](torch-compile.md) — the techniques.
- [Lecture 5 — GPUs and TPUs](05-gpus-tpus.md) — the hardware picture this builds on.
- [Lecture 2 — PyTorch and resource accounting](02-pytorch-resource-accounting.md) —
  where arithmetic intensity was introduced, and referred back to here at [1:14:10].
- [Transcript](../raw/transcripts/06-kernels-triton.md),
  [lecture source](../raw/slides/06-kernels-triton.md).
