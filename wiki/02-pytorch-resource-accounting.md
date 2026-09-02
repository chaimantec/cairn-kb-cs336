# Lecture 2 — PyTorch and Resource Accounting

This lecture teaches you to answer, on paper and before running anything, two
questions about any model: **how long will it take to train, and will it fit?**
Percy Liang builds the answer bottom-up from tensors — how they are stored, how
much memory that costs, how many floating-point operations an operation on them
takes, and how long those operations actually take on a GPU. There is, as he says
at [3:59], **no ML magic today**: the mechanics are just PyTorch, and what is
actually being taught is a habit.

It is also the lecture that explains a fact you will otherwise find inexplicable
all term — that a well-written training run uses only about half the FLOP/s its
GPU advertises, and that this is a good result.

Everything here is treated at greater length on the topic pages, which this page
links as it goes. The single best entry point is [resource
accounting](resource-accounting.md).

## Why this comes second

The lecture opens by restating the course's organizing question ([0:52]–[1:39]):
train the best model you can given a finite set of resources — compute, memory,
sometimes data, "but that's not really going to be a limiting factor for us in
this class." That is [efficiency](efficiency.md), the frame from Lecture 1.

Then the step that makes this lecture necessary:

> And our goal is simply to maximize the computational efficiency of our
> training. So, before you can optimize the computational efficiency, we need to
> understand the efficiency of a given computation, and for that we need to
> understand the compute and memory characteristics. ([1:39])

You cannot maximize a ratio you cannot measure. Percy calls the day's subject
resource accounting and places it "more on the systems side of things" ([0:52]) —
so although Lecture 2 sits in the Basics block, its content belongs to Unit 2. See
[course map](course-map.md#unit-2--systems).

Before that he reports back on the Marin run previewed in Lecture 1 ([0:05]): the
IsoFLOPs curves were fit, a compute-optimal point chosen, a loss predicted, and
the run came in **within 0.05** of the forecast. It is a small aside with a large
point — the estimates this course teaches you to make are the kind that hold. See
[scaling laws](scaling-laws.md).

## The two questions

Both are stated at the top and answered in about four lines each, which is the
whole argument for the method.

**How long would it take to train a 70B parameter model on 15T tokens on 1024
H100s?** ([1:39]–[2:24])

$$C = 6ND = 6 \times 70\text{e}9 \times 15\text{e}12 = 6.3 \times 10^{24} \text{ FLOPs}$$

Divide by the hardware's rate — 1979 teraFLOP/s halved for dense, times an
[MFU](flops-and-mfu.md) of 0.5, times 1024 GPUs, times the seconds in a day — and
the answer is **143 days**. Each ingredient gets its own treatment: [$C = 6ND$
derived](training-flops.md), [the halving and MFU](flops-and-mfu.md).

**What's the largest model you can train on 8 H100s using AdamW?** ([2:24]–[3:13])

H100s have 80 GB of HBM. The bytes per parameter are $2 + 2 + 4 + 4 = 12$ — "we'll
explain where that comes from" — so about **53 billion parameters**. See [memory
accounting](memory-accounting-for-training.md).

And immediately, the caveat that makes it an honest estimate rather than a number:
**activations are not counted**, because they depend on batch size and sequence
length. Percy's summary of the method at [3:13] is the thing to remember:

> This is all very rough back-of-the-envelope calculations, but hopefully by the
> end of this class you'll understand where these come from, and the point is not
> to precisely calculate every single thing, but just get the rough shape of
> things.

## What to take away

Percy's three-way split of knowledge, applied to this lecture ([3:59]):

- **Mechanics** — straightforward. How PyTorch works, how tensors work. "There's
  no magic here."
- **Mindset** — the real content. Resource accounting is "going to be very
  crucial, and I want everyone to get into the habit — whenever you write a line
  of code — of thinking about the performance characteristics."
- **Intuitions** — a sense of how resources are spent. "There's going to be no ML
  magic today. I'll leave that to Tatsu for the next lecture."

## Tensors, precision and memory

Everything is a tensor ([4:44]): parameters, gradients, optimizer states, data,
activations. Each has a shape and a precision, and its memory is just **number of
elements × bytes per element** ([7:01]) — a 4-by-8 fp32 matrix is 128 bytes.

![The IEEE 754 single-precision bit layout](../raw/images/02-pytorch-resource-accounting/fp32.png)

*IEEE 754 single-precision: 1 sign bit, 8 exponent bits (30 down to 23), 23 fraction bits (22 down to 0). Source: [`images/fp32.png`](https://github.com/stanford-cs336/lectures/blob/main/images/fp32.png).*

![The IEEE half-precision bit layout](../raw/images/02-pytorch-resource-accounting/fp16.png)

*IEEE half-precision: 1 sign bit, 5 exponent bits (14 down to 10), 10 fraction bits (9 down to 0). Source: [`images/fp16.png`](https://github.com/stanford-cs336/lectures/blob/main/images/fp16.png).*

![The bfloat16 bit layout](../raw/images/02-pytorch-resource-accounting/bf16.png)

*bfloat16: 1 sign bit, 8 exponent bits (14 down to 7), 7 fraction bits (6 down to 0). The exponent field is the same width as fp32's, which is exactly why bf16 keeps fp32's dynamic range and gives up precision instead. Source: [`images/bf16.png`](https://github.com/stanford-cs336/lectures/blob/main/images/bf16.png).*

That the arithmetic is trivial does not make it unimportant. One matrix in GPT-3's
feedforward layer is about **2.3 GB** ([7:46]), in a model that is by 2026
standards "fairly old" and far from the biggest imaginable.

Percy walks the formats by anatomy ([5:29]–[14:42]). fp32 is 1 sign bit, 8
exponent bits, and the rest mantissa; the exponent buys **dynamic range**, the
mantissa buys **resolution**. Halving to fp16 takes the exponent down to 5 bits,
and the consequence is immediate: `torch.tensor([1e-8], dtype=torch.float16)` is
exactly **zero** ([8:32]). Training in fp16 gives you "instability — you get
underflow, you get overflow, you'll get NaNs" ([9:17]).

bf16, developed in 2018, keeps fp16's 16 bits but shifts bits from the mantissa
back to the exponent, so it has **the same dynamic range as fp32** at half the
memory. The cost is resolution — "there's no free lunch here" — and the reason
that trade is worth taking is the one insight underneath the whole topic
([10:03]):

> You want the dynamic range to not overflow and underflow, and because things are
> kind of sloppy and stochastic anyway, you don't need that much resolution.

Hence the **mixed precision** recipe ([12:24]): bf16 for parameters, activations
and gradients; fp32 for optimizer states. PyTorch's AMP library automates the
split, casting to bf16 "when it's safe" — matmuls yes, exponentiation no.

Below that: fp8, standardized with two variants trading range against resolution;
and nvfp4, four bits per value with a per-block scale factor so that a block can
be shifted up and down even though its members cannot vary freely from their
neighbours ([13:57]). Nemotron 3 Super was trained in FP4. Percy's practical note
at [14:42] is that much of this "you can't even touch — it's not like you create a
tensor and call it FP4. A lot of this is done under the hood by NVIDIA's software
stack."

Full treatment: [precision and data types](precision-and-data-types.md).

Two student questions worth keeping ([14:42]–[16:15]). On block scaling: an
individual value gets more than four bits of effective dynamic range, "but you
can't have this value be way over here and the next neighboring value way down
there." On pushing to one bit: that is a **quantization** story, not a training
one — you train at bf16 and quantize down afterwards, whereas training a one-bit
language model is something "I don't think anyone has trained anything credible
there."

## einops

Percy polls the room at [17:50] — about two-thirds know einsum — and motivates it
with code he finds genuinely confusing ([18:35]): `x @ y.transpose(-2, -1)`, where
you must work out what `-2` and `-1` are. Naming the dimensions removes the
question.

The rule to remember, from [20:07]: **anything not mentioned in the output gets
summed out.** Enumerate every named index, multiply, accumulate. And the practical
payoff at [21:37] — "there's no transpose, because in some sense I've done the
transpose by the naming. I always get confused by transposes, and the fact that I
don't have to think about transposing makes me happy."

`...` covers any number of leading batch dimensions, which matters in language
modelling where you may have batch, sequence and head axes at once ([22:24]).
`reduce` generalizes sum/mean/max/min ([23:09]); asked whether it is faster, Percy
says it reduces to the same primitives — "you can think about it as just sugar, so
it should be the same." `rearrange` splits and merges axes and "will come up in
assignment 1" ([23:55]).

His verdict at [26:57]: "once you have einops, you just think in a different way,
and all the transposes and reductions become more fluid." Full treatment:
[einops](einops.md).

## Counting FLOPs

A FLOP is a basic floating-point operation — addition or multiplication ([27:43]).
GPUs do other things, but these "are the bread and butter and are going to eat up
most of your time."

Then a distinction Percy calls a pet peeve of his: **FLOPs** (operations, a
quantity of work) versus **FLOP/s** (operations per second, a hardware speed).
They are pronounced identically and the uppercase-S convention makes it worse, so
he always writes `/s`. This KB follows him.

The spec-sheet trap ([29:16]): the H100 datasheet says 1979 teraFLOP/s for bf16.
Benchmark it and you will not get that, because a footnote says the figure assumes
**sparsity**; dense is half. "So, you always have to take these numbers and divide
by two."

The one formula everything rests on ([30:49]–[31:36]) — for `x @ w` with `x` of
shape $B \times D$ and `w` of shape $D \times K$:

$$\text{FLOPs} = 2BDK$$

one multiplication and one addition per $(i, j, k)$ triple. Two student questions
follow ([33:08]): on sub-cubic matrix multiplication algorithms, Percy says the
optimizations that matter "are going to be much more about how you co-design with
the systems, rather than these more asymptotic algorithms"; on whether addition
should be cheaper than multiplication, "the way the hardware is built, they're
kind of the same."

Regrouping $B \cdot DK$ as *data points × parameters* ([33:53]) gives $2 \times$
tokens $\times$ parameters for a forward pass — "you can kind of see the shape of
[6ND] forming."

**MFU** is then the measured FLOP/s over the promised FLOP/s, ignoring
communication and overhead ([36:58]). The calibration at [37:45]: about **0.5** is
what you should be happy with for modern models, a bare matmul might reach 0.8, and
"if something's really wrong, you'll get something like 0.1, which means you should
do something about it." Full treatment: [FLOPs and MFU](flops-and-mfu.md).

Asked why MFU is only 50% ([39:17]), Percy defers: "I'll come back to that when we
talk about memory bottlenecks." That deferral is the hinge of the lecture.

## Arithmetic intensity — the answer to that question

The cartoon of hardware ([40:49]): high-bandwidth memory at the bottom, compute
cores above. To compute anything you send inputs up, compute, and send outputs
back — so the time depends on **two** speeds, the accelerator's FLOP/s and the
memory bandwidth, which for an H100 is 3.3 TB/s ([41:34]).

![Compute and memory joined by a narrow bandwidth pipe](../raw/images/02-pytorch-resource-accounting/compute-memory.png)

*Compute and memory as two blocks joined by a narrow pipe: many small arithmetic units against one wide memory block, the thin connector standing for the bandwidth between them. Source: [`images/compute-memory.png`](https://github.com/stanford-cs336/lectures/blob/main/images/compute-memory.png).*

Assuming communication and computation overlap perfectly, the time is the **max**
of the two, and the larger term names the regime: **memory-bound** when you are
"spending most of your time just waiting for bits to show up", **compute-bound**
otherwise ([44:38]).

The ratio form is the portable one. **Accelerator intensity** is the hardware's
FLOP/s over its bytes/s — for an H100, **295**, "an intuitive number to have in
your head — about 300" ([46:11]). **Arithmetic intensity** is a workload's FLOPs
over its bytes. Compare the two and you have the answer.

Five operations, worked ([42:19]–[52:19]):

| Operation | Arithmetic intensity | Verdict |
| --- | --- | --- |
| ReLU | 0.25 | memory-bound |
| Dot product | ~0.5 | memory-bound |
| Matrix–vector | ~1 | memory-bound |
| GeLU | ~5 | memory-bound |
| Matrix–matrix | ~340 ($\approx n/3$) | **compute-bound** |

Percy's gloss on the first: "if someone tells you your arithmetic intensity is
0.25, you should say, 'Oh, this is really bad'" ([47:42]).

The GeLU result is the counterintuitive one ([49:13]). GeLU does roughly 20 FLOPs
per element to ReLU's one, and is *still* memory-bound, because both move the same
bytes:

> Even though GeLU does a lot more work than ReLU, in the way things are
> structured, it's still memory-bound. Which means that if you were just computing
> ReLU and GeLU, you'd think, well, GeLU is so complicated, it must be really
> expensive — but actually it's exactly the same, because that's not where the
> bottleneck is.

Matrix–vector is the near miss ([50:46]) — Percy polls the room on it first — and
matrix–matrix is where intensity finally scales, at roughly $n/3$, "because you're
sending N-squared things, but you're computing N-cubed things" ([51:33]).

Three consequences he draws:

- **Large matrices and large batches are not a preference, they are the mechanism**
  ([52:19]). Below the accelerator intensity, "making things smaller doesn't
  actually speed things up, it's all kind of the same."
- **Transformers are big matmuls by design** ([53:05]) — "transformers are designed
  in a certain way to have high arithmetic intensity."
- **Inference is memory-bound** ([53:05]). Generating one token at a time is a
  matrix–vector product, so it sits at intensity ~1; training processes the whole
  sequence at once.

And so, finally, the answer to the deferred question ([53:51]): low MFU comes from
memory-bandwidth bottlenecks, not from bad code. The roofline plot ([55:22]) draws
this once — intensity on the x-axis, realized FLOP/s on the y-axis, one piecewise
line per accelerator, with the kink at the accelerator intensity.

Asked why accelerators are built so lopsidedly fast relative to memory bandwidth,
Percy defers to the GPU lecture and adds: "if you have an answer, you should tell
Jensen, and maybe he can design better hardware" ([54:37]).

Full treatment: [arithmetic intensity](arithmetic-intensity.md).

## Training: gradients and the 6ND rule

The running example ([56:55]–[58:27]) is a deep network of $L$ layers, each a
$D \times D$ matmul followed by an element-wise ReLU, so the parameter count is
$D^2 L$.

The FLOPs derivation ([1:00:00]–[1:05:23]) focuses on one layer, $h_2 = h_1 w_2$,
and asks what the backward pass must produce. Two things: the gradient with
respect to the **input**, to keep propagating backward, and the gradient with
respect to the **parameters**, which is what you came for. Written in einsum both
are matmuls over the same three dimensions, so each costs what the forward pass
cost, and ([1:04:37]):

> Notice that the backward pass is exactly twice as expensive as the forward pass,
> and this is because you have to compute two gradients.

Summed over the network: forward $2ND$, backward $4ND$, total

$$C = 6ND$$

"So, this is where the 6ND comes from, that you might have seen in various places —
it's just counting forward and backward" ([1:06:09]). And the limit, stated in the
same breath: it is a good approximation for Transformers **as long as the context
length isn't too large**, since long context adds terms in the square of the
sequence length that this accounting does not include.

Percy also uses the derivation to sell einsum one more time ([1:03:04]): "if you've
learned calculus, there's, like, one of them has a transpose, and I always forget
which order to put them in. And einsum, I think, makes it very clear."

Full treatment: [training FLOPs and the 6ND rule](training-flops.md).

## Optimizer state and the memory bill

AdaGrad is used rather than Adam, deliberately, "so that I'm not just giving you
what's in assignment 1" ([1:06:54]) — Assignment 1 asks you to implement Adam. The
family in one line: momentum is the first moment of the gradient, AdaGrad the
second, Adam both.

The memory table ([1:09:14]–[1:10:48]), for $D^2 L$ parameters: parameters at 2
bytes (bf16), activations at $2BDL$, gradients "basically a copy of all the
parameters", and optimizer state at **4 bytes per parameter for AdaGrad, 8 for
Adam**. The fp32 is deliberate:

> It's customary to use FP32 for the optimizer states, for stability reasons.
> Obviously, people have tried using BF16, and what ends up happening is that, now
> you're taking squares and averaging over all those steps, and it's not very
> stable. ([1:10:01])

Two observations follow. First, **"the optimizer state is actually a lot of the
memory used"** — often more than the parameters themselves. Second, a distinction
worth carrying forward ([1:10:48]): memory serves two purposes, storage and
movement, and optimizer state is *not* a compute bottleneck because it does not
have to be shipped to the accelerators on every operation. It costs you model
size, not speed.

Full treatment: [memory accounting for training](memory-accounting-for-training.md).

## Reducing memory

Two techniques, both trading compute for memory ([1:12:21]–[1:15:28]).

**Gradient accumulation.** Large batches improve stability up to a critical batch
size ("which Tatsu will talk about later"), but activation memory scales with batch
size. So compute gradients on micro-batches, accumulate rather than zeroing, and
update every `batch_size / micro_batch_size` steps. "This is actually a very simple
code change, which allows you to save on memory."

**Activation checkpointing** — also gradient checkpointing, also rematerialization.
Training stores every layer's activations; inference, computing no gradients, needs
only the current layer's. Checkpointing keeps a subset and recomputes the rest in
the backward pass. In PyTorch it is one line, `torch.utils.checkpoint` around a
layer, and applied to a linear-plus-ReLU block it drops the pre-ReLU tensor and
"you can get away with half the memory" ([1:14:41]).

Then the frequency question ([1:15:28]): storing nothing is maximally
memory-efficient but costs $L^2$ compute, "because for every one of these layers
you have to start from the beginning," and the sweet spot is checkpointing every
$\sqrt{L}$ layers.

> **One correction.** Percy says the $\sqrt{L}$ strategy gives activation memory
> *and* recomputation overhead both of $\sqrt{L}$. The lecture's own source states
> it as $O(\sqrt{L})$ activation memory and **$O(L)$** recomputation, which is the
> version this KB uses — see
> [activation checkpointing](activation-checkpointing.md). The transcript marks the
> spoken figure inline.

## A note on this lecture's numbers

Worth knowing before citing any measured value from Lecture 2. At [17:03] Percy
says:

> Actually, I have a slight issue with the slides — they were executed on my
> laptop, which means I don't have a GPU. So some of the code I'll just show but
> not execute.

and again at [34:39], "these calculations are not going to be very meaningful,
because I'm doing this on CPU."

So the lecture's timings, measured FLOP/s, MFU and peak-memory readings were not
produced on the hardware they describe. This knowledge base marks all of them
**machine-dependent, not reproduced** and gives no number for them, while the
deterministic arithmetic — every FLOP count, byte count and arithmetic intensity on
this page — was recomputed exactly and is marked "(computed)" in
[`raw/slides`](../raw/slides/02-pytorch-resource-accounting.md). See
[executable lectures](executable-lectures.md).

## The lecture in six lines

Percy's own summary ([1:16:15]):

- Everything operates on tensors — parameters, gradients, activations, optimizer
  states, data.
- einops as a way to think about tensor operations.
- $6ND$ FLOPs per training step, now demystified.
- Arithmetic intensity and roofline analysis diagnose memory-bound versus
  compute-bound.
- **Matrix multiplications are compute-bound, basically everything else is
  memory-bound.**
- Gradient accumulation and activation checkpointing reduce memory, which lets you
  use bigger batch sizes.

## Where this goes next

"Next week, Tatsu will talk about architectures" ([1:17:01]) — Lecture 3, which is
not in this knowledge base. The accounting taught here is the prerequisite for all
of Unit 2: kernels and fusion are ways of moving fewer bytes for the same FLOPs,
parallelism is the same principle across devices, and inference is organized
entirely around the matrix–vector result above. See [course
map](course-map.md#unit-2--systems).

Assignment 1 asks you to redo this accounting properly for a Transformer, where it
is "a bit more complicated, but you're going to do that in assignment 1, and do it
more carefully" ([1:11:36]).

## Sources

- [Edited transcript](../raw/transcripts/02-pytorch-resource-accounting.md) —
  what was said, with `[MM:SS]` markers; the header lists every restoration and
  every place the spoken version and the source disagree
- [`lecture_02.py` transcription](../raw/slides/02-pytorch-resource-accounting.md) —
  what was written, with a section-to-source-line table
- [`lecture_02.py`](https://github.com/stanford-cs336/lectures/blob/main/lecture_02.py)
  and its [trace viewer](https://cs336.stanford.edu/lectures/?trace=lecture_02)
