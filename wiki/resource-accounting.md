# Resource accounting

Resource accounting is the practice of working out, on paper and before you run
anything, how much compute and how much memory a computation will need. It is the
subject of [Lecture 2](02-pytorch-resource-accounting.md) and the skill the rest of
CS336's systems material is built on.

The lecture's own framing of why it comes second, right after tokenization: the
goal of the course is to train the best model you can given a finite set of
resources — compute and memory, sometimes data, though data is not the limiting
factor in this class — which is to say, to maximize computational
[efficiency](efficiency.md). And before you can optimize the efficiency of a
computation you have to be able to state its compute and memory characteristics.
Accounting is the prerequisite for the course's organizing question, not a
side-topic.

## What you should take away

Percy separates three kinds of knowledge for this lecture, and the middle one is
the point:

- **Mechanics**: straightforward — this is just PyTorch semantics.
- **Mindset**: resource accounting. *Remember to do it.*
- **Intuitions**: get a sense of how resources are spent. No ML magic today.

The mechanics are the least of it. What is being taught is a habit: when you meet
a model, a layer, or a proposed change, you reach for a back-of-the-envelope
estimate before you reach for a profiler. The lecture's word for it is **napkin
math** — the aim is "not to precisely calculate every single thing, but just get
the rough shape of things."

## The two questions

The lecture opens with two questions you should be able to answer by the end of
the course, and answers both in about four lines each. They are the best
demonstration of what the habit buys you.

### How long would it take to train a 70B model on 15T tokens on 1024 H100s?

```python
total_flops = 6 * 70e9 * 15e12                                    # 6.3e24
h100_flop_per_sec = 1979e12 / 2                                   # 9.895e14
mfu = 0.5
flops_per_day = h100_flop_per_sec * mfu * 1024 * 60 * 60 * 24     # 4.377e22
days = total_flops / flops_per_day                                # 143.9
```

**About 144 days.** Four ingredients, each covered by its own page:

1. $C = 6ND$ — the [total training FLOPs](training-flops.md) for $N$ parameters and
   $D$ tokens.
2. The hardware's peak rate, **halved** because the datasheet's 1979 teraFLOP/s
   assumes sparsity. See [FLOPs and MFU](flops-and-mfu.md).
3. An **MFU of 0.5** — you will not get the peak rate, and half of it is a good
   outcome. Why that is so is the subject of [arithmetic
   intensity](arithmetic-intensity.md).
4. Multiply by the number of GPUs and the seconds in a day; divide.

### What's the largest model you can train on 8 H100s using AdamW?

```python
h100_bytes = 80e9
bytes_per_parameter = 2 + 2 + (4 + 4)   # parameters (2), gradients (2), optimizer state (4 + 4) = 12
num_parameters = (h100_bytes * 8) / bytes_per_parameter   # 5.33e10
```

**About 53 billion parameters.** The whole content is the middle line: with
[mixed precision](precision-and-data-types.md) you pay 2 bytes for the parameter,
2 for its gradient, and 8 for AdamW's two fp32 moments — 12 bytes per parameter.
See [memory accounting for training](memory-accounting-for-training.md).

And the caveat the lecture attaches immediately, which is as important as the
answer: **activations are not accounted for.** They depend on batch size and
sequence length, so 53B is an *upper bound* and a real run would not fit. An
estimate you cannot state the limitations of is not an estimate.

## What there is to account for

Everything on the GPU is a tensor, and there are five kinds:

| Kind | Scales with | Accounted in |
| --- | --- | --- |
| Parameters | model size | [memory accounting](memory-accounting-for-training.md) |
| Gradients | model size | [memory accounting](memory-accounting-for-training.md) |
| Optimizer state | model size × moments kept | [memory accounting](memory-accounting-for-training.md) |
| Activations | batch × width × depth | [activation checkpointing](activation-checkpointing.md) |
| Data | batch | — |

And two resources they consume:

- **Compute**, counted in FLOPs — see [FLOPs and MFU](flops-and-mfu.md) for the
  units and [$C = 6ND$](training-flops.md) for a whole training run.
- **Memory**, counted in bytes — the product of a tensor's element count and its
  [dtype's width](precision-and-data-types.md).

The two are not independent, and the relationship between them is the lecture's
deepest idea: whether a computation is limited by arithmetic or by moving bytes is
decided by its [arithmetic intensity](arithmetic-intensity.md), and for most
operations the answer is bytes.

## The lecture in six lines

The summary the lecture closes on, which doubles as a map of this KB's Lecture 2
pages:

- Everything is operations on tensors (parameters, gradients, activations,
  optimizer states, data).
- [einops](einops.md): a better way to think about tensor operations.
- [6 (# data points) (# parameters) FLOPs](training-flops.md) per training step.
- [Arithmetic intensity / roofline analysis](arithmetic-intensity.md):
  compute-bound or memory-bound?
- **Matrix multiplications are compute-bound, elementwise operations are
  memory-bound.**
- [Gradient accumulation, activation checkpointing](activation-checkpointing.md):
  reduce memory to use bigger batch sizes.

The fifth line is the one to carry into the rest of the course. Nearly every
systems technique in Unit 2 — kernel fusion, tiling, FlashAttention — is a way of
making a memory-bound computation less memory-bound, and none of them would be
worth the trouble if that line were not true. See [course map, Unit
2](course-map.md#unit-2--systems).

## Sources

- [Lecture 2 — PyTorch, Resource Accounting](02-pytorch-resource-accounting.md)
- [`lecture_02.py` transcription](../raw/slides/02-pytorch-resource-accounting.md)
  — `main()` lines 16–68 for the framing and summary; `motivating_questions()`
  lines 71–86 for the two calculations
- [Edited transcript](../raw/transcripts/02-pytorch-resource-accounting.md)
