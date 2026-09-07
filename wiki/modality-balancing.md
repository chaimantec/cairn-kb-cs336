# Modality balancing and multimodal training stability

Training on a mixture of modalities is not the same as training on more data,
because modalities differ in **sequence length** and in **entropy**. Both
differences destabilize or distort training if left uncorrected, and
[Lecture 17](17-multimodality.md) shows three separate fixes for them.

The lecture states the principle in its summary:

> Balance images + video (lower information density) and text for training stability

## Problem 1: length, and gradient share

A video example is enormously longer than a text example. Under a plain per-token
loss, "the normal thing is that every token is treated the same, and this would
mean that the video examples are going to dominate" (≈56:17).

**Three fixes appear, at three different levels:**

| Level | Fix | Model |
| --- | --- | --- |
| Input | a [token budget](image-resolution-and-tokens.md) making all three input types produce roughly equal sequence lengths | [LLaVA-OneVision](llava-onevision.md) |
| Loss | **square-root-normalized per-token loss** — divide each example by $\sqrt{\text{length}}$ | [Qwen3-VL](qwen-vl-series.md) |
| Mixture | reweight the [data mixture](data-mixture-selection.md) directly | general |

The square root is the interesting choice: normalizing by length itself would make
every example count equally regardless of size, which over-corrects. $\sqrt{L}$
damps long examples without flattening them.

The lecturer's answer to a student on this is worth keeping, because it deflates
the problem's apparent size: you can always reweight — "we talked, last time, or
two weeks ago, about data mixtures — you can always weight things according to
whatever makes sense" — and in any case "there's also a lot of text tokens out
there… I wouldn't say that the number of multimodal tokens vastly outnumbers text
tokens" (≈1:02:36).

## Problem 2: entropy, and norm growth

This one is specific to [Chameleon](chameleon.md)'s
[discrete-token](discrete-image-tokens.md) design, where text and image tokens
share a single vocabulary and softmax.

The two distributions are not comparable. "Most words are kind of predictable,
whereas image tokens have very high entropy — I don't know what shade of blue this
exact token is going to be" (≈1:12:41). Training on the mixture "led to the norms
of the parameters growing, and to loss instability."

**The fixes** are both already in this course:

- **QK norm** — normalize queries and keys before the attention dot product,
  bounding attention logits. → [Architectures](03-architectures.md)
- **z-loss** — an auxiliary penalty on the softmax normalizer, which "controls the
  norm growth." → [Training stability](training-stability.md)

The general lesson is stated crisply: "just calling things discrete tokens doesn't
hide the fact that there's an image living there" (≈1:11:55). A shared vocabulary
does not make two distributions alike.

## Problem 3: data loading

Raised by a student asking whether multimodal training is harder from a systems
perspective (≈1:01:02). The answer is that it is "certainly not easier," and the
specific cost is one the course had not needed to think about:

> Video data, loading it can be a bottleneck. Generally, we have not really
> focused on data loading when talking about language models, because it's very
> cheap, so that needs to be taken into account.

The remedy is the standard one — "making sure that your data loading is happening
async with actual computation" — but it is a real constraint that text-only
training never surfaces.

→ [Multimodal models](multimodal-models.md) ·
[Video understanding](video-understanding.md) · [Lecture 17](17-multimodality.md)
