# Quantization

Reducing the precision of the numbers a model is stored in. Because
[inference is memory-bound](prefill-and-generation.md), fewer bytes per parameter
converts almost directly into lower latency and higher throughput — "less memory
means better latency and throughput" ([1:04:11]). The cost is accuracy, and the
techniques on this page are all about paying as little of it as possible.

This page is the [lecture 10](10-inference.md) view, which is about *serving*. For
what the formats are and how they behave numerically, see
[precision and data types](precision-and-data-types.md); for the training-time
picture, see [mixed precision](precision-and-data-types.md#mixed-precision-training--the-actual-recipe).

## The mechanics

Quantization is an affine map into integers and back. The lecture works one number
through it:

```python
x = 5.2342
scale = 0.1
zero_point = 4
x_quant  = round(x / scale) + zero_point    # 56
x_approx = (x_quant - zero_point) * scale   # 5.2
```

5.2342 comes back as 5.2, and that gap is the entire cost. `scale` sets the step
size — how much real-valued range each integer step covers — and `zero_point`
shifts the grid so the available integers land where the data actually is. Both
have to be chosen per tensor, or per layer, or per channel; choosing them well is
most of what the methods below do.

## The formats

| Format | Bytes | Range | Where used |
| --- | --- | --- | --- |
| fp32 | 4 | — | "needed for parameters and optimizer states during training" |
| bf16 | 2 | — | the default for inference |
| fp8 (e4m3) | 1 | $[-240, 240]$ on H100 | "can train if you dare" |
| int8 | 1 | $[-128, 127]$ | "less accurate but cheaper than fp8, but for inference only" |
| int4 | 0.5 | $[-8, 7]$ | "cheaper, even less accurate" |

The ranges are stated for a reason. int8 and fp8 spend the same eight bits, but the
float spends some of them on an exponent, so it covers a much wider dynamic range
with unevenly spaced steps. That is why fp8 is the one you can contemplate
*training* in, while int8 is an inference-only format: training needs to represent
gradients that span orders of magnitude, and an evenly-spaced integer grid does
not.

## When to quantize

**Quantization-aware training (QAT).** Quantize and dequantize during the forward
pass so the training run experiences the rounding error and the weights adapt
around it. Pro: the weights are trained to work at the target precision. Con: "it
requires expensive large-scale training" ([1:04:58]) — you have to own the training
run.

**Post-training quantization (PTQ).** Done after the fact, so much cheaper, which
is what most people do. The naive version runs sample data through the model to
determine a scale and zero point per tensor or per layer and quantizes each
independently; the lecture notes this "generally doesn't work as well" ([1:05:45]).

**GPTQ** is the refinement: quantize layer by layer, using Hessian information to
measure how much each weight's error matters, and push the error introduced so far
into the weights that have not been quantized yet. The not-yet-quantized weights
absorb the damage, so it does not accumulate.

## AWQ — activation-aware weight quantization

The idea most worth keeping, because it identifies *which* weights matter by
looking somewhere unexpected ([1:06:31]).

The observation: some activation channels are consistently large. The weights those
channels flow through therefore contribute much more to the output than their own
magnitude suggests. So allocate precision by activation importance, not weight
magnitude — "select which weights (0.1–1%) to keep in high precision based on
activations".

The paper's figure, reproduced in
[`raw/slides/10-inference.md`](../raw/slides/10-inference.md), makes the argument
in three panels with perplexity attached to each: plain round-to-nearest int3 gives
**PPL 43.2**; keeping the 1% salient weights in FP16 gives **PPL 13.0** but is
labelled "bad hardware efficiency" because mixed-precision rows break the kernel;
and *scaling* the salient weights by their activation magnitude before quantizing
everything uniformly to int3 also gives **PPL 13.0**, with no mixed precision at
all. The third panel is the method — it recovers the accuracy of the exception
without needing the exception.

Reported result: fp16 → int3 gives **4× lower memory and a 3.2× speedup**.

## Why it fits the rest of the lecture

Quantization attacks a different term from the [KV cache](kv-cache.md) methods. The
memory a generation step must read is $B \times (\text{cache}) + 2 \times
(\text{parameters})$; GQA, MLA, CLA and local attention shrink the first term,
quantization shrinks the multiplier on the second — and on the cache too, when the
cache itself is stored at lower precision. The lecture files it as "less of an architecture thing and much more of a systems perspective on how to make things smaller"
([1:04:11]).

And like every lossy method here, it comes with the standing caution: check the
accuracy, and treat the published accuracy tables with suspicion ([51:06]).

## Related pages

- [Precision and data types](precision-and-data-types.md) — what the formats are, in detail.
- [Microscaling formats](microscaling-formats.md) — block-scaled formats and the hardware view.
- [Inference](inference.md) — the workload this serves.
- [KV cache](kv-cache.md) — the other half of the memory bill.
- [Pruning and distillation](pruning-and-distillation.md) — the other lossy compression in this lecture.
- [Lecture 10 — Inference](10-inference.md)
