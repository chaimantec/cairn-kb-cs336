# CLIP

**CLIP** (Contrastive Language-Image Pre-training, OpenAI, 2021) is the image
encoder that most of [Lecture 17](17-multimodality.md) is built on or measured
against. Its contribution is not an architecture — it uses an off-the-shelf
[Vision Transformer](vision-transformers.md) — but a **training signal**.
[Paper](https://arxiv.org/abs/2103.00020).

Five years on, the lecture's closing assessment is that it has not been displaced:
"even CLIP, even though it's five years old, or similar ideas, are still kind of
the go-to way to capture the semantics of images" (≈1:16:30).

## The question it answers

Vision models were trained on hand-annotated datasets like ImageNet, which are
small because annotation is expensive. Captions are abundant because the web
writes them for free. So: "is it possible to leverage the much larger amount of
(image, caption) pairs?"

The bet is that noisy free supervision at scale beats clean supervision at small
scale — the same bet [data filtering](data-filtering.md) makes for text, and the
lecture draws the parallel directly, to "this idea in language, where you could
just scrape the internet — you have all this text, it's very noisy, but somehow if
you train a large enough model you can make sense out of this" (≈5:33).

## The objective

Take a batch of $N$ image-text pairs (the program says 32,768). Encode each image
to a vector $I_i$ and each text to $T_j$. Then require, for every $i$, that the
aligned pair score higher than every misaligned one — in **both** directions:

$$\mathcal{L} = \tfrac{1}{2}\left[\mathrm{CE}\big(I \cdot T^\top,\, y\big) + \mathrm{CE}\big((I \cdot T^\top)^\top,\, y\big)\right], \qquad y = (1, 2, \dots, N)$$

where $I$ and $T$ are the $\ell_2$-normalized embedding matrices and the logits
are scaled by a learned temperature. "You can imagine there's basically two times
N different softmax classification problems" (≈7:07).

Two consequences run through the whole lecture:

- **It is a ranking loss, not a reconstruction loss.** Nothing asks the model to
  produce a caption. It only has to prefer the right one.
- **The batch *is* the negative set.** So batch size is a modelling decision, not
  an optimization knob — which is precisely what [SigLIP](siglip.md) undoes.

## Data, and a circularity

400M image-text pairs, from ~500K queries at ~20K pairs each. **The dataset was
never released.** OpenCLIP reproduced the work on LAION-5B, which was released —
but "they actually used CLIP for the data filtering and then trained OpenCLIP. So
there's some bootstrapping happening" (≈8:41). The open reproduction depends on
the closed model it reproduces. This is the same loop
[data filtering](data-filtering.md) records for text: a model becomes the quality
filter for its successor's data.

## The preprocessing that everything later has to undo

Images arrive at arbitrary sizes, and "one thing you learn about neural nets is
that they don't like things to be dynamic; they want things to be fixed size"
(≈9:27). CLIP therefore:

1. resizes so the **short side is 336** px (bicubic), then
2. **center-crops** to 336×336, cutting the borders off.

The lecturer flags this as expedient at the time it is introduced, and excuses it
by the target task: "the CLIP authors were thinking about ImageNet and
classification. So usually the object is in the middle, and you're just trimming
off some background, so it doesn't matter too much" (≈10:12).

**It matters enormously for anything else.** This single decision is why
[AnyRes](image-resolution-and-tokens.md) and Qwen2-VL's dynamic resolution exist.
A document cropped to 336×336 cannot be read.

## Architecture

**Vision:** ResNet-50 and ViTs were both tried; ViTs won, so "when people say
CLIP, they usually mean the ViT version" (≈12:30). The best model is
**ViT-L/14@336px** — Large, 14×14 pixel patches, trained at 336×336. At that
resolution a 14-pixel patch tiles the image 24×24, giving 576 patches, a number
that recurs as a token count throughout the lecture.

Instead of averaging the patch vectors, CLIP uses **attention pooling**: run
attention with the query set to the global average of the activations, producing
"another vector, which is maybe a little bit more informed than just a straight-up
average" (≈14:07).

**Text:** a GPT-2-style Transformer, 63M parameters, 12 layers — small on purpose.
The sequence embedding is the `[EOS]` activation at the top layer (≈16:31).

**Position embeddings** are 1D. A student asked whether two dimensions should be
handled specially; CLIP tried a 2D variant and "found it doesn't really matter
that much," which the lecturer immediately qualifies: "you also always have to
take these results with the idea that they had classification in mind" (≈15:45).
[Multimodal RoPE](multimodal-rope.md) is where that gets revisited.

## The headline result, stated precisely

**Zero-shot CLIP outperformed a ResNet-50 trained on ImageNet's 1.2M labelled
images.** The comparison is against a supervised baseline on its own home ground,
by a model that never saw an ImageNet label.

The lecturer's gloss is about labour rather than accuracy: those 1.2M annotations
were "many, many hours of Amazon Mechanical Turk worker time," while CLIP used
annotation the web had already produced (≈17:18).

Zero-shot classification works by embedding each candidate label in a template —
"A photo of a {object}." — and taking the highest dot product with the image.

## The ablation that shapes the field

CLIP also tried the obvious alternative: predict the caption from the image, as a
bag of words or with a language model. Both are **much less compute-efficient**
than ranking — contrastive CLIP reaches ~16% zero-shot accuracy at 33M images
where bag-of-words prediction needs 134M (4×) and a full Transformer LM needs 400M
(3× beyond that).

The reading matters: "actually modeling the exact token sequences of the caption
isn't so important for getting a rough representation of the image" (≈21:12).
Generating a caption forces the model to account for its exact wording; ranking it
asks only for what distinguishes it.

*(The chart's x-axis is linear, not logarithmic — see the
[figure audit](../raw/slides/17-multimodality.md#figure-audit), which corrected
this.)*

## Known limitations, from CLIP's own summary

- **Design decisions are classification-driven, so the representation is not
  fine-grained.** This is the limitation the rest of the lecture inherits and
  works around.
- **Large batches are mandatory** — "like 30,000. If you have a batch size of one,
  clearly it doesn't work; even 10 doesn't work" (≈21:59).
- **The softmax runs over the full batch**, so the loss does not decompose across
  devices the way language-model training does. → [SigLIP](siglip.md) fixes this.
- **Captions are extremely noisy** — often alt text, and systematically incomplete,
  since "if you have an image of a dog, you don't need to say 'a dog'" (≈19:38).

→ [Contrastive image-text pretraining](contrastive-image-text-pretraining.md) ·
[SigLIP](siglip.md) · [Vision Transformers](vision-transformers.md) ·
[Lecture 17](17-multimodality.md)
