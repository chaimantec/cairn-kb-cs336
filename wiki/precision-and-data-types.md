# Precision and floating-point data types

Every number in a language model — every parameter, gradient, activation and
optimizer state — is stored in some floating-point format, and the choice of
format is the single cheapest lever on memory in the whole system. Halving the
bits per value halves the memory, and memory is what decides how large a model you
can fit on your GPUs at all.

The catch is that the formats are not interchangeable. They differ along two axes
that matter differently for deep learning, and getting the distinction wrong is a
common cause of a training run that diverges for no visible reason.

## The two axes: dynamic range and resolution

A floating-point number is stored as a sign bit, an exponent, and a mantissa
(also called the significand). Those two fields do different jobs:

- The **exponent** width sets the **dynamic range** — how large and how small a
  magnitude the format can represent at all.
- The **mantissa** width sets the **resolution** — how finely it can distinguish
  two nearby values.

Almost everything about precision in [Lecture 2](02-pytorch-resource-accounting.md)
follows from one observation: in deep learning, **dynamic range matters much more
than resolution**. Running out of range produces zeros and infinities, which
destroy a training run. Running out of resolution produces slightly noisy
gradients, which stochastic gradient descent was already tolerating.

## The formats

### fp32 — the default

fp32 (float32, single precision) is PyTorch's default dtype and the baseline
everything else is measured against. Four bytes per value.

```python
x = torch.zeros(4, 8)
assert x.dtype == torch.float32   # Default type
assert x.numel() == 4 * 8
assert x.element_size() == 4      # Float is 4 bytes
assert get_memory_usage(x) == 4 * 8 * 4   # 128 bytes
```

Memory is just the product of those two facts — **number of values × bytes per
value** — and that is the whole of memory accounting for a single tensor:

$$\text{bytes} = \text{numel} \times \text{element\_size}$$

The lecture's example of why this is not an academic concern: one matrix in the
feedforward layer of GPT-3 is $12288 \times 4$ by $12288$, which in fp32 is

```python
assert get_memory_usage(torch.empty(12288 * 4, 12288)) == 2304 * 1024 * 1024  # 2.3 GB
```

That is 2.25 GiB for **one weight matrix** in **one layer**. GPT-3 has 96 of them.

Percy's framing of where fp32 sits: in scientific computing, fp32 is the baseline
and you reach for fp64 (double precision) when you need it. In deep learning **you
can be a lot sloppier** — the pressure runs in the opposite direction, toward
fewer bits.

### fp16 — half the memory, not enough range

fp16 (float16, half precision) is two bytes per value.

```python
x = torch.zeros(4, 8, dtype=torch.float16)
assert x.element_size() == 2
```

The problem is dynamic range, especially at the small end:

```python
x = torch.tensor([1e-8], dtype=torch.float16)
assert x == 0   # Underflow!
```

$10^{-8}$ is not a pathological number in a neural network — it is an ordinary
gradient late in training. In fp16 it becomes exactly zero, and a parameter whose
gradient is zero stops learning. Do this across many values and you get the
instability fp16 training is known for.

### bf16 — the one that is actually used

Google Brain developed **brain floating point (bf16)** in 2018 specifically to fix
this. The trick is to keep fp32's exponent width and spend the savings out of the
mantissa instead:

- bf16 uses the **same memory as fp16** (2 bytes), and
- has the **same dynamic range as fp32**.

```python
x = torch.tensor([1e-8], dtype=torch.bfloat16)
assert x != 0   # No underflow!
```

The catch is that **the resolution is worse** — and, per the axis argument above,
that matters less for deep learning. This is why bf16 rather than fp16 is the
default working precision for modern training, and why the memory arithmetic in
[memory accounting for training](memory-accounting-for-training.md) uses 2 bytes
per parameter.

### fp8 — standardized in 2022

fp8 was standardized in 2022, motivated by machine learning workloads. H100s
support two variants, and the difference between them is exactly the range /
resolution trade again, now within a single byte:

| Variant | Bits | Range |
| --- | --- | --- |
| **E4M3** | 4 exponent, 3 mantissa | $[-448, 448]$ |
| **E5M2** | 5 exponent, 2 mantissa | $[-57344, 57344]$ |

E5M2 buys two more powers of range by giving up one mantissa bit. (Reference:
[arXiv:2209.05433](https://arxiv.org/pdf/2209.05433.pdf).)

### fp4 — nvfp4, 2025

In 2025 NVIDIA developed
[nvfp4](https://developer.nvidia.com/blog/introducing-nvfp4-for-efficient-and-accurate-low-precision-inference/):
**four bits per value.** At that width you no longer have a format so much as a
codebook — the fifteen representable values are

$$-6,\ -4,\ -3,\ -2,\ -1.5,\ -1.0,\ -0.5,\ 0.0,\ 0.5,\ 1.0,\ 1.5,\ 2,\ 3,\ 4,\ 6$$

The way this stays usable is a **separate scale factor per block** of values. The
block's scale restores dynamic range across the tensor as a whole; what you give
up is that values within a block cannot vary freely from their neighbours, since
they share a scale. Nemotron 3 Super was trained in NVFP4, so this is not a
research curiosity.

The lecture's practical note is that much of the fp8/fp4 machinery lives inside
NVIDIA libraries and is **outside user control** — you get it by using the library,
not by writing it.

## Mixed precision training — the actual recipe

The two facts that force the compromise:

- Training in fp32 works, but requires lots of memory.
- Training in fp16, and even in bf16, is risky, and you can get instability.

The resolution is **mixed precision training**
([arXiv:1710.03740](https://arxiv.org/pdf/1710.03740.pdf)), and the split is worth
memorizing because it is what the byte counts in the rest of the course assume:

- **bf16** for parameters, activations, and gradients
- **fp32** for optimizer states

The asymmetry has a reason. Optimizer state accumulates running averages over
thousands of steps — sums of squares, exponential moving averages — and small
errors in an accumulator compound in a way that a single forward pass's rounding
does not. So the one place you keep full precision is the one place where values
are added up over the whole run. This is the "customary to use fp32 for stability"
remark in the optimizer section, and it is why AdaGrad costs 4 bytes per parameter
and Adam 8, rather than 2 and 4.

PyTorch ships automatic mixed precision (AMP)
([docs](https://pytorch.org/docs/stable/amp.html)), which casts into bf16 **when
it is safe** — matmuls yes, `exp` no:

```python
with torch.amp.autocast("cuda", dtype=torch.bfloat16):
    x = torch.zeros(4, 8)
```

The safe/unsafe split follows the range argument once more: a matmul of
well-scaled values stays in range, whereas an exponential is precisely the
operation that will run off the end of it.

## Why this shows up in every other calculation

Precision is not a self-contained topic. It is a multiplier on all three of the
resources the course accounts for:

- **Memory** — bytes per parameter, hence [how large a model
  fits](memory-accounting-for-training.md). The 2 + 2 + 4 + 4 = 12 bytes per
  parameter in the lecture's opening question is a precision choice, not a law.
- **Compute** — peak FLOP/s depends heavily on dtype. An H100 delivers 67.5
  teraFLOP/s in fp32 and 989.5 in bf16, a factor of nearly 15. See
  [FLOPs and MFU](flops-and-mfu.md).
- **Arithmetic intensity** — bytes moved per element halves when you halve the
  format width, so the memory-bound / compute-bound boundary itself shifts. The
  lecture notes this explicitly: arithmetic and accelerator intensity "also depend
  on the precision (bf16 versus fp32)". See
  [arithmetic intensity](arithmetic-intensity.md).

## Block-scaled formats, and the hardware view

[Lecture 5](05-gpus-tpus.md) picks this up from the hardware side and takes it a
step further than fp8. Two things it adds:

- **A "pretty non-trivial part"** of the historical growth in GPU FLOPs is not
  better silicon at all but narrower numbers — "you start at FP32, go to BF16, and
  then to INT8" ([34:37]). Some of that famous curve is counting smaller.
- **Microscaling formats** replace one scale factor per tensor with one per small
  block of elements — MXFP8 uses an E8M0 factor per 32 values, MXFP4 goes to four
  bits per element. This buys range back at the cost of making a transpose expensive
  enough that training keeps two quantized copies of every matrix. See
  [microscaling formats](microscaling-formats.md).

Hashimoto also gives the realistic figure for what fp8 training buys once
quantization overhead is paid: **20–30% on the matrix multiplies**, not a 2×
speedup ([42:18]).

## Quantization for inference

The formats above are the training story. [Lecture 10](10-inference.md) uses the
same formats for a different purpose — shrinking a *trained* model so it can be
served — and that has its own page: [quantization](quantization.md), covering the
scale/zero-point mechanics, quantization-aware training versus post-training
quantization, GPTQ, and AWQ.

The one distinction worth carrying back here is why int8 is an inference-only
format while fp8 is not. They cost the same byte, but fp8 spends part of it on an
exponent, so it covers a wide dynamic range with uneven spacing while int8 covers
$[-128, 127]$ evenly. Training needs the range, which is the same argument this
page makes for [bf16 over fp16](#bf16--the-one-that-is-actually-used); inference,
where activations are bounded and calibratable, can live with the grid.

## Sources

- [Lecture 10 — Inference](10-inference.md) — the same formats used to compress a
  trained model, via [quantization](quantization.md).
- [Lecture 5 — GPUs and TPUs](05-gpus-tpus.md) — low precision as trick 2 of six,
  and the MXFP8/MXFP4 frontier.
- [Lecture 2 — PyTorch, Resource Accounting](02-pytorch-resource-accounting.md)
- [`lecture_02.py` transcription](../raw/slides/02-pytorch-resource-accounting.md)
  — `tensors_memory()`, lines 113–181
- [Edited transcript](../raw/transcripts/02-pytorch-resource-accounting.md)
