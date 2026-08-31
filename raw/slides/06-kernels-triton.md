---
title: Lecture 6 — Kernels, Triton (course material)
lecture: 6
source_format: executable-python
source_file: lecture_06.py
source_repo: https://github.com/stanford-cs336/lectures
source_url: https://raw.githubusercontent.com/stanford-cs336/lectures/main/lecture_06.py
rendered_url: https://cs336.stanford.edu/lectures/?trace=lecture_06
source_lines: 744
note: >
  CS336's Percy-taught lectures are "executable lectures" — Python programs whose
  execution delivers the lecture content — rather than slide PDFs. There are no
  slide numbers. Sections below correspond to function definitions in
  lecture_06.py, and each carries the source line range so a claim can be checked
  against the program. Content is transcribed from the source text, which is the
  authoritative written form of this lecture.
title_note: >
  The Cairn catalog entry for this video is titled "Lecture 6: Kernels, Triton,
  XLA". Neither lecture_06.py nor the lecture's captions mention XLA or JAX at
  any point, so this file uses the course site's own name for the lecture,
  "Kernels, Triton". No XLA material is present in this offering's lecture 6.
runtime_values: >
  Several of this lecture's numbers do not appear in the source — they are
  produced at runtime by @inspect annotations on ordinary arithmetic. Every such
  value quoted below was recomputed by evaluating the lecture's own expressions
  verbatim, and is marked "(computed)". A second class of value — wall-clock
  benchmark timings and the PyTorch profiler tables — depends on the GPU the
  program runs on and CANNOT be reproduced without that hardware. Those are
  marked "(machine-dependent, not reproduced)" and no number is given for them.
  This lecture is unusually heavy in the second class: benchmarking and profiling
  ARE the subject, so the comparisons it draws (naive vs. built-in vs. compiled
  GeLU, add vs. matmul profiles) are stated qualitatively as the source states
  them, with no timings invented.
figures: >
  The program displays images via image() calls. Those images are recorded below
  at the point they appear, by path or URL, WITHOUT a description of what they
  show — the transcription was made from source text, not from the rendered
  images. Do not cite a figure's contents from this file.
---

# Lecture 6 — Kernels, Triton (course material)

This is the written content of CS336 Lecture 6, transcribed from
[`lecture_06.py`](https://github.com/stanford-cs336/lectures/blob/main/lecture_06.py).
Run it, or step through it in the browser, at
<https://cs336.stanford.edu/lectures/?trace=lecture_06>.

Lecture 5 gave the hardware picture from the outside — what a GPU is made of and
which six tricks make one go fast. This lecture is the hands-on half of that
pair: how to *measure* where time goes (benchmarking and profiling), and how to
*write* the kernel that fixes what you measured (Triton). Its four worked kernels
climb a ladder of difficulty — elementwise, then a reduction that fits in one
block, then a reduction that does not, then tiled matrix multiplication.

The spoken lecture follows this program closely but not exactly — Percy digresses,
answers questions, and expands on points that the source states in one line. For
what was *said*, see [the transcript](../transcripts/06-kernels-triton.md). For
what was *written*, use this file.

## Sections → source lines

| Section | Function | Lines |
| --- | --- | --- |
| [Framing and roadmap](#framing-and-roadmap) | `main()` | 13–36 |
| [Review of GPUs — hardware](#review-of-gpus--hardware) | `review_of_gpus()` | 39–57 |
| [Review of GPUs — programming model](#review-of-gpus--programming-model) | `review_of_gpus()` | 58–72 |
| [Interaction between programming model and hardware](#interaction-between-programming-model-and-hardware) | `review_of_gpus()` | 74–79 |
| [Warps and control divergence](#warps-and-control-divergence) | `review_of_gpus()` | 83–92 |
| [Warp occupancy](#warp-occupancy) | `review_of_gpus()` | 94–113 |
| [Bank conflicts](#bank-conflicts-shared-memory) | `review_of_gpus()` | 115–124 |
| [Memory coalescing](#memory-coalescing-hbm) | `review_of_gpus()` | 126–130 |
| [Block occupancy and wave quantization](#block-occupancy-and-wave-quantization) | `review_of_gpus()` | 132–138 |
| [Benchmarking and profiling](#benchmarking-and-profiling) | `benchmarking_and_profiling()` | 144–153 |
| [Benchmarking](#benchmarking) | `benchmarking()` | 156–176 |
| [The benchmark harness](#the-benchmark-harness) | `benchmark()` | 179–203 |
| [Profiling](#profiling) | `profiling()` | 206–234 |
| [The profile harness](#the-profile-harness) | `profile()` | 237–259 |
| [Naive vs. built-in vs. compiled GeLU](#naive-vs-built-in-vs-compiled-gelu) | `naive_vs_builtin_vs_compiled_gelu()` | 262–302 |
| [Triton — the programming model](#triton--the-programming-model) | `triton_introduction()` | 305–314 |
| [Triton GeLU (elementwise)](#triton-gelu-elementwise) | `triton_gelu_example()`, `triton_gelu()`, `triton_gelu_kernel()` | 317–389 |
| [Triton softmax (row fits in a block)](#triton-softmax-row-fits-in-a-block) | `triton_softmax_example()`, `naive_softmax()`, `triton_softmax()`, `triton_softmax_kernel()` | 392–484 |
| [Triton row sum (row does not fit in a block)](#triton-row-sum-row-does-not-fit-in-a-block) | `triton_row_sum_example()`, `builtin_row_sum()`, `triton_row_sum()`, `row_sum_kernel()` | 487–535 |
| [Triton matmul + ReLU (tiling)](#triton-matmul--relu-tiling) | `triton_matmul_relu_example()`, `naive_matmul_relu()`, `triton_matmul_relu()`, `matmul_relu_kernel()` | 538–674 |
| [Summary](#summary) | `main()` | 28–36 |
| [Helper functions](#helper-functions) | `run_operation1`, `run_operation2`, `naive_gelu`, `builtin_gelu`, `pytorch_softmax`, `check_equal_*`, `mean`, `output_ptx` | 677–741 |

The top-level order is set by `main()` (lines 13–36):

```python
def main():
    text("Last lecture: high-level overview of GPUs and performance")
    text("This lecture: benchmarking/profiling + writing kernels")

    review_of_gpus()
    benchmarking_and_profiling()           # Where are the bottlenecks?
    naive_vs_builtin_vs_compiled_gelu()    # Apply it to the GeLU example

    # Write Triton kernels
    triton_introduction()
    triton_gelu_example()      # Elementwise operation
    triton_softmax_example()   # Reduction (row fits in a block)
    triton_row_sum_example()   # Reduction (row doesn't fit in block)
    triton_matmul_relu_example()    # Tiling: use shared memory
```

---

## Framing and roadmap

Last lecture: high-level overview of GPUs and performance.

This lecture: benchmarking/profiling + writing kernels.

The comments in `main()` label what each part is for: benchmarking and profiling
answer "where are the bottlenecks?", the GeLU section applies that to a concrete
example, and the four Triton examples are ordered as *elementwise operation*,
*reduction (row fits in a block)*, *reduction (row doesn't fit in block)*, and
*tiling: use shared memory*.

---

## Review of GPUs — hardware

*Figure: `images/gpu-hardware.png` (width 800).*

The following table is printed verbatim by the source (lines 42–55):

```
| Accelerator                        | A100      | H100      | B200      |
+------------------------------------+-----------+-----------+-----------+
| # SMs                              |       108 |       132 |       148 |
+------------------------------------+-----------+-----------+-----------+
| Register size (per SM)             |    256 KB |    256 KB |    256 KB |
| L1 cache + shared memory (per SM)  |    192 KB |    256 KB |    256 KB |
| L2 cache size                      |     40 MB |     50 MB | 96-126 MB |
| HBM size                           |     80 GB |     80 GB |    192 GB |
+------------------------------------+-----------+-----------+-----------+
| Register bandwidth                 | ~116 TB/s | ~401 TB/s | ~447 TB/s |
| L1 cache + shared memory bandwidth |  ~19 TB/s |  ~33 TB/s |  ~19 TB/s |
| L2 cache bandwidth                 | ~5-8 TB/s |  ~12 TB/s |   ~9 TB/s |
| HBM bandwidth                      |    2 TB/s | 3.35 TB/s |    8 TB/s |
```

(B200s also have tensor memory (TMEM) for tensor cores (between registers and
shared memory) that are invisible to programmer.)

---

## Review of GPUs — programming model

*Figure: `https://docs.nvidia.com/cuda/parallel-thread-execution/_images/grid-with-CTAs.png` (width 600).*

- *Thread*: executes code on a small part of the data
- *Thread block* or concurrent thread array (CTA): a group of threads
- *Grid*: collection of thread blocks

(H100s and B200s also have thread block clusters that enable distributed shared
memory.)

**Why thread blocks?**

For elementwise operations (e.g., GeLU), threads are most natural: each thread
processes one element — $f(i)$ for $i = 0, \ldots, N-1$.

However, for non-elementwise operations like softmax or matrix multiplication,
threads need to communicate. Reading/writing from HBM is slow, so use shared
memory (local to SM). A thread block is a collection of threads that access the
same shared memory. Consequently, a thread block is scheduled on one SM.

In Triton, think natively in terms of thread blocks (later).

---

## Interaction between programming model and hardware

The programming model provides an abstraction of the hardware. In principle, you
don't need to think about anything else (for correctness). In practice,
performance is very sensitive to the hardware, so you need to understand it to
obtain high performance.

Let's go over some considerations.

---

## Warps and control divergence

**Warps**:

- Within a thread block, threads are grouped into warps (32 threads per warp).
- Example: thread block has 64 threads => it has 2 warps.

```
| TTTTTTTTTTTTTTTTTTTTTTTTTTTTTTTT | TTTTTTTTTTTTTTTTTTTTTTTTTTTTTTTT |
```

- All threads within a warp must execute same instructions in lockstep on an SM.
- Control divergence: if different threads in a warp need to execute different
  instructions (if A, else B), must be done sequentially (bad)

```
| AAAAAAAAA....................... |
| .........BBBBBBBBBBBBBBBBBBBBBBB |
```

- SM runs multiple warps and switches between them (e.g., when one warp is
  blocked on HBM reads/writes) with zero cost.

---

## Warp occupancy

**(Warp) occupancy**:

- Each thread can use between 0 and 255 registers.
- The more registers threads use, the fewer threads can be scheduled on an SM
  (low occupancy).
- Low occupancy isn't necessarily bad if each thread is doing more work.
- Example: thread coarsening (each thread processes multiple elements).
- Example: thread block has 64 threads, each using 160 registers, SM has 65536
  registers

**Note a discrepancy in the source**: the bullet above says "thread block has 64
threads", but the code immediately below it sets `num_threads_per_block = 128`.
The computed values that follow are those of the code, which is what actually
runs.

```python
# What we want to run
num_threads_per_block = 128
num_registers_per_thread = 160

# What hardware offers
max_registers = 65536  # Registers allowed per SM
max_warps = 64         # Concurrent warps allowed per SM

# What we can run at once
assert num_registers_per_thread <= 255
num_registers_per_block = num_threads_per_block * num_registers_per_thread  # 20480 (computed)
num_blocks = max_registers // num_registers_per_block  # Limited by registers — 3 (computed)
num_warps = num_blocks * num_threads_per_block / 32  # 12.0 (computed)
occupancy = num_warps / max_warps  # 0.1875 (computed)
```

So a thread that asks for 160 registers gets you three blocks resident per SM,
twelve warps out of the 64 the SM can hold, and an occupancy of **0.1875** —
under a fifth of the SM's warp slots.

---

## Bank conflicts (shared memory)

**Bank conflicts** (shared memory):

- Shared memory is divided into 32 banks, each 4 bytes wide.

```
B00 B01 B02 B03 B04 B05 B06 B07 B08 B09 B10 B11 B12 B13 B14 B15 B16 B17 B18 B19 B20 B21 B22 B23 B24 B25 B26 B27 B28 B29 B30 B31
... ... ... ... ... ... ... ... ... ... ... ... ... ... ... ... ... ... ... ... ... ... ... ... ... ... ... ... ... ... ... ...
... ... ... ... ... ... ... ... ... ... ... ... ... ... ... ... ... ... ... ... ... ... ... ... ... ... ... ... ... ... ... ...
... ... ... ... ... ... ... ... ... ... ... ... ... ... ... ... ... ... ... ... ... ... ... ... ... ... ... ... ... ... ... ...
```

- Each cycle, each bank can only be accessed by one thread (if not the same exact
  location).
- If multiple threads access the same bank, accesses serialized (bank conflict).
- Worst case example: matrix where each row spans all banks; 32 threads accessing
  first column results in 32-way bank conflict!
- Unavoidable: when doing matmul `A @ B`, access rows of A and columns of B
- Solution: swizzling rearranges shared memory (e.g., row xor col) to avoid bank
  conflicts

---

## Memory coalescing (HBM)

**Memory coalescing** (HBM):

- When the 32 threads in a warp access HBM, memory accesses combined into
  transactions of 128 bytes (cache lines).

```
M00 M01 M02 M03 M04 M05 M06 M07 M08 M09 M10 M11 M12 M13 M14 M15 M16 M17 M18 M19 M20 M21 M22 M23 M24 M25 M26 M27 M28 M29 M30 M31
M32 M33 M34 M35 M36 M37 M38 M39 M40 M41 M42 M43 M44 M45 M46 M47 M48 M49 M50 M51 M52 M53 M54 M55 M56 M57 M58 M59 M60 M61 M62 M63
```

- Best case: full coalescing, all threads access the same cache line (32 threads
  x 4 bytes = 128 bytes).

---

## Block occupancy and wave quantization

*Figure: `https://developer-blogs.nvidia.com/wp-content/uploads/2019/06/pasted-image-0.png` (width 400).*

**Block occupancy**:

- Thread blocks scheduled onto SMs in waves.
- B200 has 148 SMs, if we launch 160 thread blocks, first wave has 148 blocks,
  second wave has 12 blocks.
- Wave quantization problem: last wave has fewer thread blocks, leaving some SMs
  idle (low block occupancy).
- Solution: make number of thread blocks divide # SMs.

**Summary of the review:**

- Programming model: grid (HBM) -> thread block (shared memory) -> thread
  (registers)
- Details of hardware (warps, bank conflicts, memory coalescing, occupancy)
  determine performance

---

## Benchmarking and profiling

Recipe for success:

1. Benchmark and profile your code
2. Make changes
3. Benchmark and profile your code again

The section runs `benchmarking()` ("How long does it take?") and then
`profiling()` ("Where time is being spent?"), and closes with the imperative:
**benchmark and profile your code!**

---

## Benchmarking

Benchmarking measures the wall-clock time of performing some operation. It only
gives you end-to-end time, not where time is spent (profiling).

It is still useful for:

- comparing different implementations (which is faster?), and
- understanding how performance scales (e.g., with dimension).

You can use
[`torch.utils.benchmark`](https://pytorch.org/tutorials/recipes/recipes/benchmark.html).
We will roll our own to make benchmarking more transparent.

```python
# Benchmark matrix multiplication
matmul = run_operation2(dim=1024, operation=lambda a, b: a @ b)
result = benchmark(matmul)  # @inspect result — machine-dependent, not reproduced

# See how timing scales with dimension
results = {}
for dim in [256, 512, 1024, 2048, 4096, 8192]:
     results[dim] = benchmark(run_operation2(dim=dim, operation=lambda a, b: a @ b))
     # @inspect results — machine-dependent, not reproduced
```

The dimensions swept are **256, 512, 1024, 2048, 4096, 8192**, and the source
states the shape of the answer without printing it:

> Note: time is roughly constant when dimension is small, then cubic scaling.

The individual timings are wall-clock measurements of whichever GPU the lecture
was run on and are not reproduced here.

---

## The benchmark harness

`benchmark()` (lines 179–203) is written out in full because the details are the
lesson — a naive timing loop measures the wrong thing on a GPU.

```python
def benchmark(run: Callable, num_warmups: int = 1, num_trials: int = 3) -> float:
    """Benchmark `func` by running it `num_trials`.  Return the average time."""
    # Warmup: first times might be slower due to compilation, etc.
    # Since we will run the kernel multiple times, the timing that matters is steady state.
    for _ in range(num_warmups):
        run()
    torch.cuda.synchronize()  # Wait for CUDA threads to finish (important!)

    # Time it for real now!
    times: list[float] = [] # @inspect times
    for trial in range(num_trials):  # Do it multiple times to capture variance
        # Use CUDA events for accurate GPU timing (avoid capturing CPU overhead)
        start_event = torch.cuda.Event(enable_timing=True)
        end_event = torch.cuda.Event(enable_timing=True)

        start_event.record()  # Start timing
        run()  # Actually perform computation
        end_event.record()  # End timing

        torch.cuda.synchronize()  # Wait for CUDA threads to finish

        times.append((start_event.elapsed_time(end_event)))  # @inspect times

    mean_time = mean(times)   # @inspect mean_time
    return mean_time
```

Three things the source calls out in its own comments: **warm up first**, because
the first runs might be slower due to compilation; **synchronize**, because CUDA
launches are asynchronous and the CPU would otherwise finish timing before the
GPU finishes working ("important!"); and **repeat**, to capture variance. The
timing itself uses `torch.cuda.Event` rather than a CPU clock, to avoid capturing
CPU overhead.

---

## Profiling

While benchmarking looks at end-to-end time, profiling looks at where time is
spent. Independent of time, profiling also helps you understand what's going
under the hood.

PyTorch has a built-in
[profiler](https://pytorch.org/tutorials/recipes/recipes/profiler_recipe.html).
In your assignment, you will use nsight to get more details.

The lecture profiles three calls and prints each table verbatim:

```python
## add(dim=2048)
add_profile = profile(run_operation2(dim=2048, operation=lambda a, b: a + b))

## matmul(dim=2048)
matmul_profile = profile(run_operation2(dim=2048, operation=lambda a, b: a @ b))

## matmul(dim=128)
matmul_profile = profile(run_operation2(dim=128, operation=lambda a, b: a @ b))
```

**The three profiler tables are machine-dependent and are not reproduced here** —
they are the actual output of the profiler on the GPU the lecture ran on. What
the source says about them:

Observations:

- You can see which CUDA kernels are actually being called (the long names).
- Different CUDA kernels are invoked depending on the tensor dimensions.

Note that `matmul` is profiled at **two** dimensions, 2048 and 128, precisely to
show that second point: the same PyTorch expression dispatches to a different
CUDA kernel depending on how big the tensors are.

The name of the CUDA kernel tells us something about the implementation. The
example the source gives is

```
cutlass3x_sm100_simt_sgemm_f32_f32_f32_f32_f32_64x64x16_1x1x1_3_nnn_align1_bi...
```

- `cutlass`: NVIDIA's CUDA library for linear algebra
- `sm100`: corresponds to the NVIDIA Blackwell architecture (B200)
- `f32`: float32
- `64x64x16`: tile shape (more on this later)

---

## The profile harness

```python
def profile(run: Callable, num_warmups: int = 1):
    # Warmup
    for _ in range(num_warmups):
        run()
    torch.cuda.synchronize()

    # Run the code with the profiler
    with torch.profiler.profile(activities=[ProfilerActivity.CUDA],
            experimental_config=torch._C._profiler._ExperimentalConfig(verbose=True)) as prof:
        run()
        torch.cuda.synchronize()

    # Print out table
    table = prof.key_averages().table(sort_by="cuda_time_total",
                                      max_name_column_width=100,
                                      row_limit=10)

    # Append to profiles.txt
    with open("var/profiles.txt", "a") as f:
        f.write(f"Profile at {time.ctime()}:\n")
        f.write(table)
        f.write("\n\n")
    return table
```

Same warmup-and-synchronize discipline as `benchmark()`. The table is sorted by
`cuda_time_total` and truncated to ten rows.

---

## Naive vs. built-in vs. compiled GeLU

Let's benchmark and profile the
[GeLU activation function](https://pytorch.org/docs/stable/generated/torch.nn.GELU.html).

Three implementations are compared on the same input:

```python
x = torch.tensor([1.])  # @inspect x

# 1. Implementation naively from scratch in PyTorch (non-fused)
y1 = naive_gelu(x)  # 0.8412 (computed)

# 2. Built-in PyTorch implementation (fused)
y2 = builtin_gelu(x)  # 0.8412 (computed)
check_equal_1d(naive_gelu, builtin_gelu)  # Check it works

# 3. Use PyTorch compiler on the naive implementation
compiled_gelu = torch.compile(naive_gelu)
y3 = compiled_gelu(x)  # 0.8412 (computed)
check_equal_1d(naive_gelu, compiled_gelu)  # Check it works (compilation shouldn't change semantics)
```

All three return the same value on $x = 1$ — the tanh approximation gives
$0.5 \cdot 1 \cdot \left(1 + \tanh\!\left(0.79788456 \cdot (1 + 0.044715)\right)\right) \approx 0.84119199$
(computed by evaluating `naive_gelu`'s own expression in double precision; the
program evaluates it in float32, so the last digits it displays may differ) — which is the point:
compilation and fusion change the speed, not the semantics. `check_equal_1d`
re-checks that on a random 2048-element vector with `atol=1e-6`.

```python
# Benchmarking
naive_time = benchmark(run_operation1(dim=16384, operation=naive_gelu))
builtin_time = benchmark(run_operation1(dim=16384, operation=builtin_gelu))
compiled_time = benchmark(run_operation1(dim=16384, operation=compiled_gelu))
```

All three timings are **machine-dependent and not reproduced**. The source's own
statement of the result is:

> The builtin and compiled versions are significantly faster!

To understand why, the lecture profiles each of the three
(`naive_gelu`, `builtin_gelu`, `compiled_gelu` at `dim=16384`) and prints the
three profiler tables — again machine-dependent, not reproduced. The conclusion
the source draws from them:

- Naive implementation: multiple kernels, requires many reads/writes from/to HBM
  (**no fusion**).
- Builtin and compiled versions: one kernel (**kernel fusion**), one read from
  HBM, one write to HBM.
- The compiled kernel is a Triton kernel.

That last line is the hinge of the lecture: `torch.compile` emits Triton, so the
rest of the lecture is about writing by hand what the compiler was writing for
you.

---

## Triton — the programming model

*Figure: `https://docs.nvidia.com/cuda/parallel-thread-execution/_images/grid-with-CTAs.png` (width 600).*

In CUDA (developed by NVIDIA), specify what each **thread** does.

- Pros: fine-grained control
- Cons: need to manage more things (e.g., shared memory)

In Triton (developed by OpenAI), specify what each **thread block** does.

- Generally powerful enough (especially when getting started)
- Conceptual framework: load data into shared memory, operate on it, write back
  to global memory

---

## Triton GeLU (elementwise)

Let's write the Triton kernel for GeLU.

```python
x = torch.randn(8192, device=cuda_if_available())
y = triton_gelu(x)

check_equal_1d(triton_gelu, naive_gelu)  # Check for correctness
```

**The launcher.** The host-side function checks the input, allocates the output,
works out how many blocks the grid needs, and launches:

```python
def triton_gelu(x: torch.Tensor):
    # Check input
    assert x.is_cuda
    assert x.is_contiguous()

    # Allocate output tensor
    y = torch.empty_like(x)

    # Determine grid (elements divided into blocks)
    # | T T T T T T T T | T T T T T T T T | T T T T T T T T | T T T T T T T T |
    # |    Block 0      |    Block 1      |     Block 2      |    Block 3     |
    num_elements = x.numel()  # 8192 (computed)
    BLOCK_SIZE = 1024  # Number of threads
    num_blocks = triton.cdiv(num_elements, BLOCK_SIZE)  # 8 (computed)

    # Launch the kernel
    kernel = triton_gelu_kernel[(num_blocks,)](x, y, num_elements, BLOCK_SIZE=BLOCK_SIZE)

    # Write out PTX (look at this later)
    output_ptx("triton_gelu", kernel)

    return y
```

With 8192 elements and a block size of 1024, the grid is **8 blocks**
(`triton.cdiv` is ceiling division, so a non-multiple would round up and the
mask in the kernel would handle the ragged tail).

**The kernel.**

```python
@triton.jit
def triton_gelu_kernel(x_ptr, y_ptr, num_elements, BLOCK_SIZE: tl.constexpr):
    # Input starts at `x_ptr`
    # Output starts at `y_ptr`

    # | T T T T T T T T | T T T T T T T T | T T T T T T T T | T T T T T T T T |
    # |    Block 0      |    Block 1      |     Block 2      |    Block 3     |

    pid = tl.program_id(axis=0)      # Identifies the block
    start = pid * BLOCK_SIZE         # Starting index of this block

    # Indices where this thread block should operate
    offsets = start + tl.arange(0, BLOCK_SIZE)

    # Don't read/write past the end of the tensor
    mask = offsets < num_elements

    # Read
    x = tl.load(x_ptr + offsets, mask=mask)

    # Approx gelu is 0.5 * x * (1 + tanh(sqrt(2/pi) * (x + 0.044715 * x^3)))
    # Compute (tl.tanh doesn't exist, use tanh(a) = (exp(2a) - 1) / (exp(2a) + 1)
    a = 0.79788456 * (x + 0.044715 * x * x * x)
    exp = tl.exp(2 * a)
    tanh = (exp - 1) / (exp + 1)
    y = 0.5 * x * (1 + tanh)

    # Store
    tl.store(y_ptr + offsets, y, mask=mask)
```

The shape of every Triton kernel in this lecture is visible here: get your block
id with `tl.program_id`, turn it into a vector of `offsets`, build a `mask` so the
last block does not run off the end of the tensor, `tl.load`, compute on whole
vectors rather than scalars, `tl.store`. Note the source's own aside that
`tl.tanh` doesn't exist, so the kernel uses the identity
$\tanh(a) = \dfrac{e^{2a} - 1}{e^{2a} + 1}$ — the constant `0.79788456` is
$\sqrt{2/\pi}$.

**PTX.** Triton compiles down to PTX (parallel thread execution), an assembly
language for GPUs. We can see the PTX code generated by Triton — the program
writes it to `var/triton_gelu-ptx.txt` and links it
(*link: `var/triton_gelu-ptx.txt`, a local artifact of the run; not reproduced
here*).

Observations:

- `ld.global.*` and `st.global.*` reads and writes from global memory
- `%ctaid.x` is block index, `%tid.x` is thread index
- `%f*` are floating point registers, `%r*` are integer registers
- One thread processes 8 elements at the same time (thread coarsening)

That last observation connects back to the occupancy discussion: with
`BLOCK_SIZE = 1024` and 8 elements per thread, Triton has chosen to coarsen
rather than launch 1024 threads doing one element each.

---

## Triton softmax (row fits in a block)

So far, we've looked at elementwise operations in Triton (e.g., GeLU). Now let us
look at operations that aggregate over multiple values.

We will roughly follow the
[Triton fused softmax tutorial](https://triton-lang.org/main/getting-started/tutorials/02-fused-softmax.html).

Recall the softmax operation is used in attention and generating probabilities.
Exponentiate and normalize each row of a matrix:

```
[0 0 0]      =>   [1/3 1/3 1/3]
[1 1 -inf]        [1/2 1/2 0  ]
```

**The naive implementation, with reads and writes counted.** This is the part of
the lecture that motivates fusion arithmetically:

```python
def naive_softmax(x: torch.Tensor):
    # M: number of rows, N: number of columns
    M, N = x.shape

    # Compute the max of each row (MN reads, M writes)
    x_max = x.max(dim=1)[0]

    # Subtract off the max (MN + M reads, MN writes)
    x = x - x_max[:, None]

    # Exponentiate (MN reads, MN writes)
    numerator = torch.exp(x)

    # Compute normalization constant (MN reads, M writes)
    denominator = numerator.sum(dim=1)

    # Normalize (MN reads, MN writes)
    y = numerator / denominator[:, None]

    # Total: 5MN + M reads, 3MN + 2M writes
    # In principle, should have MN reads, MN writes (speedup of 4x!)
    return y
```

Five separate PyTorch operations, each of which round-trips the whole matrix
through HBM: **$5MN + M$ reads and $3MN + 2M$ writes**, where in principle a
fused kernel needs only $MN$ reads and $MN$ writes — the source's own note puts
the headroom at a **speedup of 4x**.

The example input is

```python
x = torch.tensor([
    [5., 5, 5],
    [0, 0, 100],
], device=cuda_if_available())
y1 = naive_softmax(x)
```

which gives `[[1/3, 1/3, 1/3], [≈0, ≈0, 1]]` (computed) — the second row's first
two entries are $e^{-100}$, which is a subnormal in float32 and displays as
roughly `3.7e-44`. The row is the max-subtraction trick doing its job: without
subtracting 100 first, $e^{100}$ overflows.

**The kernel.** Each row gets its own block, and the block is sized to the whole
row:

```python
def triton_softmax(x: torch.Tensor):
    # Allocate output tensor
    y = torch.empty_like(x)

    # Determine grid
    M, N = x.shape                          # Number of rows x number of columns
    block_size = triton.next_power_of_2(N)  # Each block contains all the columns
    num_blocks = M                          # Each block is a row

    # Launch kernel
    triton_softmax_kernel[(M,)](
        x_ptr=x, y_ptr=y,
        x_row_stride=x.stride(0), y_row_stride=y.stride(0),
        num_cols=N, BLOCK_SIZE=block_size
    )

    return y
```

```python
@triton.jit
def triton_softmax_kernel(x_ptr, y_ptr, x_row_stride, y_row_stride, num_cols, BLOCK_SIZE: tl.constexpr):
    assert num_cols <= BLOCK_SIZE

    # Process each row independently
    row_idx = tl.program_id(0)
    col_offsets = tl.arange(0, BLOCK_SIZE)

    # Read from global memory
    x_start_ptr = x_ptr + row_idx * x_row_stride
    x_ptrs = x_start_ptr + col_offsets
    x_row = tl.load(x_ptrs, mask=col_offsets < num_cols, other=float("-inf"))

    # Compute
    x_row = x_row - tl.max(x_row, axis=0)
    numerator = tl.exp(x_row)
    denominator = tl.sum(numerator, axis=0)
    y_row = numerator / denominator

    # Write back to global memory
    y_start_ptr = y_ptr + row_idx * y_row_stride
    y_ptrs = y_start_ptr + col_offsets
    tl.store(y_ptrs, y_row, mask=col_offsets < num_cols)
```

*Figure: `images/triton-softmax.png` (width 600).*

Two details worth noticing. The padding value on the load is `-inf`, not zero, so
that padded lanes contribute nothing to either the max or the sum. And the whole
five-operation sequence above now happens between one `tl.load` and one
`tl.store` — that is the fusion, written out.

Correctness is checked both ways, against PyTorch's own softmax:

```python
check_equal_2d(pytorch_softmax, naive_softmax)
check_equal_2d(pytorch_softmax, triton_softmax)
```

---

## Triton row sum (row does not fit in a block)

In the softmax example, an entire row fits in a block, so the reduction happens
within a block (handled by Triton). What if the row doesn't fit in a block?

Example: 4096 columns, but block size is 1024...

Strategy:

- Break up row into tiles (4 in the example above)
- Each thread iterates over tiles and accumulates a sum
- Do final reduction (sum) over accumulators of each thread (shared memory or
  warp shuffles)

Consider the simpler example (row sum instead of softmax):

```python
x = torch.tensor([[1., 2, 3, 4], [5, 6, 7, 8]], device=cuda_if_available())
y1 = builtin_row_sum(x)  # [10., 26.] (computed)
```

*Figure: `images/triton-row-sum.png` (width 600).*

```python
def builtin_row_sum(x: torch.Tensor):
    return x.sum(dim=1)


def triton_row_sum(x: torch.Tensor, BLOCK_SIZE: int = 1024) -> torch.Tensor:
    M, N = x.shape
    y = torch.empty(M, device=x.device, dtype=x.dtype)
    row_sum_kernel[(M,)](x, y, N, BLOCK_SIZE=BLOCK_SIZE)
    return y


@triton.jit
def row_sum_kernel(x_ptr, out_ptr, N, BLOCK_SIZE: tl.constexpr):
    row = tl.program_id(0)  # Which row are we processing?

    # Accumulator for each thread
    # One row: T1 T2 T3 T4 | T1 T2 T3 T4 | T1 T2 T3 T4 (N = 12, BLOCK_SIZE = 4)
    acc = tl.zeros([BLOCK_SIZE], dtype=tl.float32)

    # Loop over tiles
    for start in range(0, N, BLOCK_SIZE):
        cols = start + tl.arange(0, BLOCK_SIZE)
        mask = cols < N
        x = tl.load(x_ptr + row * N + cols, mask=mask, other=0.0)
        acc += x

    # Final reduction from BLOCK_SIZE (all threads) to a scalar
    result = tl.sum(acc, axis=0)

    tl.store(out_ptr + row, result)
```

The two-stage structure is the whole point, and the comment in the source draws
it: thread $T_i$ visits column $i$, then column $i + \texttt{BLOCK\_SIZE}$, then
$i + 2\,\texttt{BLOCK\_SIZE}$, accumulating into its own register; only at the end
does `tl.sum` collapse the `BLOCK_SIZE`-wide accumulator vector down to one
scalar. Padding is `0.0` here rather than `-inf`, because the reduction is a sum.
Note also that the accumulator is `float32` regardless of the input dtype.

---

## Triton matmul + ReLU (tiling)

Matrix multiplication is the bread and butter of deep learning.

```python
a = torch.randn(1024, 1024, device=cuda_if_available())
b = torch.randn(1024, 1024, device=cuda_if_available())
c = naive_matmul_relu(a, b)
```

How should we build a matmul kernel? The source sets up the block structure:

```
|        k                  n
|   [ A1 A2 A3 ]       [ B1 B2 B3 ]   [ C1 C2 C3 ]
| m [ A4 A5 A6 ]  *  k [ B4 B5 B6 ] = [ C4 C5 C6 ]
|   [ A7 A8 A9 ]       [ B7 B8 B9 ]   [ C7 C8 C9 ]
```

**Naive approach:**

Fix any $(m, n)$. For each $k$:

- Read `A[m, k]` and `B[k, n]` from HBM.
- Multiply and accumulate.

Write result to `C[m, n]` in HBM.

Bottleneck: $MKN$ reads, $MN$ writes. Arithmetic intensity: $O(1)$.

Computing C4 and C5 both need A4, A5, A6. Can we read A4, A5, A6 from HBM once to
compute both? Answer: yes, using shared memory!

**Idealized approach:**

- Load all of A and B into shared memory, then compute C.
- Now we get $MK + KN$ reads and $MN$ writes.
- This yields the idealized $O(N)$ arithmetic intensity from before.
- However, A and B are usually too large to fit in shared memory.

**Tiling:**

*Figure: `images/gemm_tiled.png` (width 600).*

Key idea: divide the matrix C into output tiles (thread blocks). Fix an output
tile in C. For each pair of (row tile of A, column tile of B):

- Load the corresponding A tile and B tile from HBM into shared memory.
- Perform matrix multiplication on the tiles.
- Accumulate into the partial sum (in shared memory).

Write output tile to HBM.

Arithmetic intensity: $O(\text{tile\_size})$.

Bonus:

- Often, you want to apply an elementwise activation function.
- Example: `GeLU(A @ B)`
- Solution: kernel fusion!

**Implementation.**

Review: each matrix is linearized in memory.

```python
x = torch.tensor([[0., 1, 2, 3], [4, 5, 6, 7]])
stride_row, stride_col = x.stride()  # 4, 1 (computed)
row = 1
col = 2
index = row * stride_row + col * stride_col  # 6 (computed)
```

A row-major 2x4 matrix has strides $(4, 1)$, so element $(1, 2)$ — the value `6`
in this matrix — sits at flat index $1 \cdot 4 + 2 \cdot 1 = 6$. Every pointer
expression in the kernel below is built this way.

```python
def naive_matmul_relu(x: torch.Tensor, y: torch.Tensor):
    # Matmul followed by ReLU
    return torch.nn.functional.relu(x @ y)


def triton_matmul_relu(a: torch.Tensor, b: torch.Tensor):
    assert a.is_cuda and b.is_cuda
    assert a.is_contiguous() and b.is_contiguous()
    assert a.shape[1] == b.shape[0]

    # A is M x K, B is K x N
    M, K = a.shape
    K, N = b.shape

    # Allocate output tensor
    c = torch.empty((M, N), device=a.device)

    # Determine grid
    BLOCK_M, BLOCK_N, BLOCK_K = 64, 64, 32
    grid = (triton.cdiv(M, BLOCK_M), triton.cdiv(N, BLOCK_N))

    matmul_relu_kernel[grid](
        a, b, c,
        M, N, K,
        a.stride(0), a.stride(1),
        b.stride(0), b.stride(1),
        c.stride(0), c.stride(1),
        BLOCK_M, BLOCK_N, BLOCK_K,
    )

    return c
```

For the 1024x1024 inputs used above, `BLOCK_M = BLOCK_N = 64` gives a **16 x 16
grid of output tiles** (computed), and each block's `k` loop runs
$1024 / 32 = 32$ iterations (computed).

```python
@triton.jit
def matmul_relu_kernel(
    a_ptr, b_ptr, c_ptr,    # Compute c = a @ b
    M, N, K,                # a is M x K, b is K x N, c is M x N
    stride_am, stride_ak,   # How to navigate a
    stride_bk, stride_bn,   # How to navigate b
    stride_cm, stride_cn,   # How to navigate c
    BLOCK_M: tl.constexpr,
    BLOCK_N: tl.constexpr,
    BLOCK_K: tl.constexpr,
):
    # We are working on the (m, n)-th tile
    pid_m = tl.program_id(0)
    pid_n = tl.program_id(1)

    # Indices
    indices_m = pid_m * BLOCK_M + tl.arange(0, BLOCK_M)  # Row indices of a [BLOCK_M]
    indices_n = pid_n * BLOCK_N + tl.arange(0, BLOCK_N)  # Column indices of b [BLOCK_N]
    indices_k = tl.arange(0, BLOCK_K)                    # Row indices of a = column indices of b [BLOCK_K]

    # Initial matrix of pointers of a and b
    a_ptrs = a_ptr + indices_m[:, None] * stride_am + indices_k[None, :] * stride_ak  # [BLOCK_M, BLOCK_K]
    b_ptrs = b_ptr + indices_k[:, None] * stride_bk + indices_n[None, :] * stride_bn  # [BLOCK_K, BLOCK_N]

    acc = tl.zeros([BLOCK_M, BLOCK_N], dtype=tl.float32)

    # Move along row tiles of a, column tiles of b
    for k in range(0, K, BLOCK_K):
        a = tl.load(a_ptrs, mask=(indices_m[:, None] < M) & (indices_k[None, :] + k < K), other=0.0)
        b = tl.load(b_ptrs, mask=(indices_k[:, None] + k < K) & (indices_n[None, :] < N), other=0.0)
        acc += tl.dot(a, b)
        a_ptrs += BLOCK_K * stride_ak  # Advance to the next row tile of a
        b_ptrs += BLOCK_K * stride_bk  # Advance to the next column tile of b

    # Apply activation function (e.g., ReLU)
    acc = tl.maximum(acc, 0.0)

    # Write output tile
    c_ptrs = c_ptr + indices_m[:, None] * stride_cm + indices_n[None, :] * stride_cn
    tl.store(c_ptrs, acc, mask=(indices_m[:, None] < M) & (indices_n[None, :] < N))
```

The grid is two-dimensional here — `tl.program_id(0)` and `tl.program_id(1)` pick
the output tile's row and column — and the pointers are *matrices* of pointers,
`[BLOCK_M, BLOCK_K]` and `[BLOCK_K, BLOCK_N]`, advanced by
`BLOCK_K * stride` each time round the `k` loop rather than recomputed. The
accumulator lives in `float32` across the whole loop, `tl.dot` is what gets
mapped onto tensor cores, and the ReLU is applied to the accumulator **before**
the single `tl.store` — the fusion the "bonus" note promised, costing one extra
instruction rather than a second full pass over C in HBM.

---

## Summary

The source's closing summary (lines 28–36):

- Know the programming model (PyTorch, Triton, PTX) to give you correctness
- Understand the hardware (SMs, warps, occupancy, bank conflicts, etc.) to
  optimize performance
- Benchmark to understand scaling
- Profile to see what's being executed for how long
- Triton: think in terms of thread blocks (read to shared memory, do stuff
  (fusion), write back HBM)
- Examples: GeLU (elementwise), softmax (row-wise), row sum (baby tiling), matmul
  (tiling)

Next time: more than one GPU!

---

## Helper functions

Small utilities used above, transcribed for completeness (lines 677–741).

```python
def run_operation1(dim: int, operation: Callable) -> Callable:
    # Setup: create one random dim x dim matrices
    x = torch.randn(dim, dim, device=cuda_if_available())
    # Return a function to perform the operation
    return lambda : operation(x)


def run_operation2(dim: int, operation: Callable) -> Callable:
    # Setup: create two random dim x dim matrices
    x = torch.randn(dim, dim, device=cuda_if_available())
    y = torch.randn(dim, dim, device=cuda_if_available())
    # Return a function to perform the operation
    return lambda : operation(x, y)


def naive_gelu(x: torch.Tensor):
    # tanh approximation to the gelu activation function
    # https://docs.pytorch.org/docs/stable/generated/torch.nn.GELU.html
    return 0.5 * x * (1 + torch.tanh(0.79788456 * (x + 0.044715 * x * x * x)))


def builtin_gelu(x: torch.Tensor):
    # PyTorch's built-in GeLU with the tanh approximation
    return torch.nn.functional.gelu(x, approximate="tanh")


def pytorch_softmax(x: torch.Tensor):
    return torch.nn.functional.softmax(x, dim=-1)


def check_equal_1d(f1, f2):
    x = torch.randn(2048, device=cuda_if_available())
    y1 = f1(x)
    y2 = f2(x)
    assert torch.allclose(y1, y2, atol=1e-6)


def check_equal_2d(f1, f2):
    x = torch.randn(2048, 2048, device=cuda_if_available())
    y1 = f1(x)
    y2 = f2(x)
    assert torch.allclose(y1, y2, atol=1e-6)


def mean(xs: list[float]) -> float:
    return sum(xs) / len(xs)


def output_ptx(name: str, kernel):
    """Print out the PTX code generated by Triton for the given `kernel`."""
    ptx_path = f"var/{name}-ptx.txt"
    with open(ptx_path, "w") as f:
        ptx = kernel.asm["ptx"]
        f.write(ptx)
```

`check_equal_2d_2d` (lines 723–729) is defined in the source with the same shape
as `check_equal_2d` but taking two input matrices; it is not called anywhere in
this lecture.

Note that every allocation goes through `cuda_if_available()`, imported from the
course's `gpu_util` module, so the program still runs (slowly, on CPU) where no
GPU is present — but the Triton kernels themselves require CUDA, as
`triton_gelu`'s `assert x.is_cuda` makes explicit.
