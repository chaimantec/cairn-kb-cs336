# RMSNorm, and dropping bias terms

Two changes that almost every modern language model makes to the original
transformer, for the same reason — and the reason is not accuracy. It is that both
LayerNorm's mean-subtraction and the network's bias terms are cheap in
floating-point operations but expensive in *memory movement*, and memory movement
is what actually costs time on a GPU.

This is the clearest place in CS336 where the systems half of the course reaches
back into the architecture half. It only makes sense if you have met
[arithmetic intensity](arithmetic-intensity.md) from Lecture 2.

## The two normalizers

**LayerNorm** normalizes across the model dimension $d_{model}$, subtracting the
mean and dividing by the standard deviation, then applying a learned scale
$\gamma$ and a learned shift $\beta$:

$$y = \frac{x - \mathrm{E}[x]}{\sqrt{\mathrm{Var}[x] + \epsilon}} * \gamma + \beta$$

**RMSNorm** drops both the mean subtraction and the bias, dividing only by the
root-mean-square of the activation and rescaling:

$$y = \frac{x}{\sqrt{\lVert x \rVert_2^2 + \varepsilon}} * \gamma$$

Slide 14 gives both formulas with their adopters. LayerNorm: GPT-1/2/3, OPT,
GPT-J, BLOOM. RMSNorm: the LLaMA family, PaLM, Chinchilla, T5. The database on
slides 7 and 29 shows the transition plainly — the `Norm` column is mixed grey
LayerNorm and blue RMSNorm until about 2022, and almost solid blue afterwards.

Note that **LayerNorm is strictly more expressive than RMSNorm**: it can represent
everything RMSNorm can, plus a mean shift. Hashimoto is explicit at [13:52] that
"there's really no representational reason why you have to use RMSNorm." The
argument for it is entirely elsewhere.

## Why drop the mean: FLOPs are not runtime

The case for RMSNorm has two steps, and the second is the interesting one.

**Step one: normalization is a negligible fraction of the arithmetic.** Slide 15
reproduces a table from Ivanov et al. 2023 breaking a transformer's work down by
operator class:

| Operator class | % FLOPs |
| --- | --- |
| Tensor contraction | 99.80 |
| Statistical normalization | 0.17 |
| Element-wise | 0.03 |

Matrix multiplies are, as the slide says, "the *vast* majority of FLOPs (and
memory)." Normalization is 0.17% of the arithmetic. On this evidence, optimizing
it would be pointless.

**Step two: it is not a negligible fraction of the runtime.** Slide 16 adds one
column to the same table, and the column changes the conclusion:

| Operator class | % FLOPs | % Runtime |
| --- | --- | --- |
| Tensor contraction | 99.80 | 61.0 |
| Statistical normalization | 0.17 | 25.5 |
| Element-wise | 0.03 | 13.5 |

An operation that is 0.17% of the FLOPs is **25.5% of the wall-clock time**. As
Hashimoto puts it at [15:24]: "it's not really about the flops — the flops are the
floating point operations we do, that's multiplying matrices, but that's not
runtime. Runtime is a much more complicated object."

The reason is [arithmetic intensity](arithmetic-intensity.md). Slide 16's dataflow
diagram tags each operation with both its FLOP count and its FLOP-to-memory ratio:
multi-head attention is **43G** FLOPs at an intensity of **153**, while LayerNorm
is **29M** FLOPs at an intensity of **3.5**. A normalization reads and writes a
whole activation tensor to do almost no arithmetic on it, so it runs at memory
bandwidth, not at compute throughput. This is the same reason ReLU is memory-bound
in Lecture 2 — see [FLOPs and MFU](flops-and-mfu.md).

A student asks about exactly this at [16:10], and the answer is the crisp version
([16:56]): for tensor contraction "the majority of the workload is multiplying,"
whereas for statistical normalization "the majority of the workload is memory
movement, and memory movement is quite slow."

**One caveat the lecture states and the KB should repeat.** Hashimoto immediately
qualifies the 25.5% figure at [16:56]: "I think the percent runtime in this case is
quite extreme; this is in tiny models with matrices that don't really generally
make sense in modern workloads." The number demonstrates the mechanism; it is not
a claim about a frontier-scale training run.

## Does it cost accuracy? Apparently not

Slide 17 reproduces Narang et al. 2020, a controlled comparison on a 223M-parameter
model where RMSNorm is the only row with bolded values:

| Model | Params | Step/s | Early loss | Final loss | SGLUE | XSum | WebQ | WMT EnDe |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Vanilla Transformer | 223M | 3.50 | 2.182 ± 0.005 | 1.838 | 71.66 | 17.78 | 23.02 | 26.62 |
| RMS Norm | 223M | 3.68 | 2.167 ± 0.008 | **1.821** | **75.45** | **17.94** | **24.07** | **27.14** |

RMSNorm is both faster (3.68 steps/s against 3.50) **and** better on every quality
column. Hashimoto's reading at [17:41]: "you actually get better performance, which
I don't think is something that you're guaranteed, but it's a nice bonus
regardless. So you got a free systems win by just moving to RMSNorm."

The table also contains four Rezero and Fixup variants, all of which are worse than
the baseline on final loss — a reminder that most proposed normalization changes do
not work, and this one is the survivor.

## Dropping bias terms

The same logic, applied more broadly. The original transformer's feedforward layer
carries biases:

$$\mathrm{FFN}(x) = \max(0, xW_1 + b_1)W_2 + b_2$$

Most modern implementations drop them entirely:

$$\mathrm{FFN}(x) = \sigma(xW_1)W_2$$

Slide 18 gives two reasons — memory, "similar to RMSnorm", and optimization
stability. Hashimoto's version at [17:41]–[18:26] is that bias terms are "generally
not that useful," that adding them is "another example of something that's not very
arithmetically intense, but fairly memory-intensive," and that while they can also
cause stability problems, "the primary reason these are dropped is just to simplify
things from the systems perspective."

This is also why the assignment-1 model has no bias terms in its linear layers or
its norms (slide 4).

## What to take from this

The honest summary is slide 19's, and it is worth reading as a statement about how
architectural knowledge in this field is actually held:

- Everyone does non-residual norm, usually pre-norm.
- Most people use RMSNorm — it works as well as LayerNorm in practice, has fewer
  parameters to move, and so saves wall-clock time.
- Bias terms get dropped because the compute-to-parameter tradeoff is poor.

Hashimoto adds the caveat at [19:12]: "you can't really reason about this
beforehand — we don't know beforehand that dropping the bias terms is okay, but
from a lot of experimentation and now collectively acquired knowledge, we roughly
know that dropping the bias terms ... is okay for typical language modeling
workloads." The justification is empirical and collective, not derived.

## Related

- [Pre-norm and post-norm](pre-norm-and-post-norm.md) — where the norm sits, as
  opposed to what it computes.
- [Arithmetic intensity](arithmetic-intensity.md) — the Lecture 2 concept this
  whole argument rests on.
- [FLOPs and MFU](flops-and-mfu.md) — why FLOP counts do not predict runtime.
- [Lecture 3 — architectures](03-architectures.md).
