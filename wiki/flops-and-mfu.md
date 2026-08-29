# FLOPs, FLOP/s, and model FLOPs utilization

Compute is the resource CS336 assumes is binding, so counting it correctly is the
prerequisite for everything else. This page covers the units, the hardware
numbers, and the one metric — MFU — that tells you whether you are using the
hardware you are paying for.

## FLOP, FLOPs, FLOP/s

A **floating-point operation (FLOP)** is a basic operation like an addition
$x + y$ or a multiplication $x \cdot y$.

Then there are two terribly confusing acronyms, and — as the lecture points out —
they are **pronounced the same**:

- **FLOPs** — floating-point operation*s*. A measure of **computation done**. A
  quantity of work. "Training GPT-3 took 3.14e23 FLOPs."
- **FLOP/s** — floating-point operations **per second**, also written FLOPS. A
  measure of **hardware speed**. A rate. "An H100 does 9.895e14 FLOP/s."

They differ by a division by time, and mixing them up turns a sensible calculation
into nonsense. This knowledge base writes the rate as `FLOP/s` throughout, which is
the convention the lecture adopts for exactly this reason.

## Intuitions for the numbers

Two anchors for the scale of a training run:

- Training **GPT-3** (2020) took **$3.14 \times 10^{23}$ FLOPs**
  ([article](https://lambdalabs.com/blog/demystifying-gpt-3))
- Training **GPT-4** (2023) is speculated to take **$2 \times 10^{25}$ FLOPs**
  ([article](https://patmcguinness.substack.com/p/gpt-4-details-revealed))

And one for what hardware supplies. An H100 has a peak of **1979 teraFLOP/s with
sparsity, 50% of that without**
([spec](https://resources.nvidia.com/en-us-tensor-core/nvidia-tensor-core-gpu-datasheet)):

$$\text{h100\_flop\_per\_sec} = \frac{1979 \times 10^{12}}{2} = 9.895 \times 10^{14} \text{ FLOP/s}$$

**Always halve the datasheet number.** The headline figure assumes structured
sparsity that ordinary dense training does not have. This is the most common
factor-of-2 error in back-of-the-envelope compute estimates, and every calculation
in the course uses the halved value.

Putting supply and demand together — 8 H100s for 2 weeks:

```python
total_flops = 8 * 2 * (60 * 60 * 24 * 7) * h100_flop_per_sec   # 9.575e21
```

$9.6 \times 10^{21}$ FLOPs, against GPT-3's $3.14 \times 10^{23}$. So two weeks on
eight H100s is about **3% of the compute** that trained GPT-3, and about
$5 \times 10^{-4}$ of speculated GPT-4. That is the gap the course is honest about
from Lecture 1 onward — see [efficiency](efficiency.md).

## Counting the FLOPs in a matmul

The one formula worth internalizing. For `y = x @ w` with `x` of shape $B \times D$
and `w` of shape $D \times K$:

$$\text{FLOPs} = 2 \cdot B \cdot D \cdot K$$

The derivation is the entire content: the result has $B \times K$ entries, each is
a dot product of length $D$, and each step of a dot product is **one
multiplication and one addition** — that is, one multiply-add per $(i, j, k)$
triple, or 2 FLOPs. Hence the factor of 2.

```python
B = 16384  # Number of points
D = 32768  # Dimension of each point
K = 8192   # Number of outputs

x = torch.ones(B, D, device=cuda_if_available())
w = torch.randn(D, K, device=cuda_if_available())
y = x @ w

actual_num_flops = 2 * B * D * K   # 8.796e12
```

Everything else in the course's compute accounting is built on this one count.
The [6ND training rule](training-flops.md) is this formula applied to a whole
network, forward and backward.

## Measuring what you actually get

The lecture times the matmul and divides:

```python
actual_time = benchmark(lambda: x @ w)
actual_flop_per_sec = actual_num_flops / actual_time
```

`benchmark()` runs the operation five times and averages, and — importantly for
anyone reproducing this — calls `torch.cuda.synchronize()` before and after.
CUDA kernel launches are asynchronous, so timing them without synchronizing
measures how fast you can queue work, not how fast the GPU does it, and yields
impossibly good numbers.

The actual timing and throughput are **machine-dependent**: they are facts about
the GPU the lecture was run on, so this knowledge base does not quote a value for
them. Run the lecture yourself and the number you get is the answer for your
hardware.

## Peak FLOP/s depends heavily on dtype

The denominator — the hardware's "promised" rate — is not one number per GPU. It
is one number per (GPU, dtype) pair, and the spread is large. From
`get_promised_flop_per_sec()` in the lecture source:

| GPU | fp32 | bf16 / fp16 (dense) |
| --- | --- | --- |
| A100 | 19.5e12 | 312e12 |
| H100 | 67.5e12 | 989.5e12 (= 1979e12 / 2) |
| B200 | 75e12 | 2.25e15 (= 4.5e15 / 2) |

Two things to read off this table. Within a row, **bf16 is worth roughly 15–30×
fp32** — which is a much larger effect than the memory saving, and the real reason
mixed precision is universal. Across rows, note that fp32 barely improves from
H100 to B200 (67.5 → 75) while bf16 more than doubles (989.5 → 2250): the hardware
vendors are optimizing for the format deep learning actually uses. See
[precision and data types](precision-and-data-types.md).

## MFU — model FLOPs utilization

$$\text{MFU} = \frac{\text{actual FLOP/s}}{\text{promised FLOP/s}}$$

ignoring communication and overhead. It is the fraction of the hardware you are
actually using, and it is the number to look at when deciding whether a training
run is worth optimizing.

**An MFU of ≥ 0.5 is quite good.** That rule of thumb is worth pausing on: half
the machine idle is a *good* outcome. It is also the assumption behind the
lecture's opening napkin calculation, which uses `mfu = 0.5` to estimate 144 days
to train a 70B model on 15T tokens over 1024 H100s — see [resource
accounting](resource-accounting.md).

Which raises the question the lecture asks and then spends the rest of the hour
answering: **why is MFU not closer to 1?** The answer is that peak FLOP/s is only
one of the machine's two speed limits, and for most operations the other one binds
first:

$$\text{MFU} = \min\left(1,\ \frac{I_{\text{arith}}}{I_{\text{accel}}}\right)$$

See [arithmetic intensity](arithmetic-intensity.md) for what those terms are and
why the answer is usually memory.

## Sources

- [Lecture 2 — PyTorch, Resource Accounting](02-pytorch-resource-accounting.md)
- [RMSNorm](rmsnorm.md) — Lecture 3's sharpest illustration that FLOPs are not
  runtime: an operator class that is 0.17% of a transformer's FLOPs and 25.5% of
  its wall-clock time.
- [`lecture_02.py` transcription](../raw/slides/02-pytorch-resource-accounting.md)
  — `tensor_operations_flops()`, lines 279–335; `get_promised_flop_per_sec()`,
  lines 795–827
- [Edited transcript](../raw/transcripts/02-pytorch-resource-accounting.md)
