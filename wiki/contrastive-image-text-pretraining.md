# Contrastive image-text pretraining

The family of methods that learns an image encoder by **pairing images with text
and ranking the pairs**, rather than by classifying images against labels or
reconstructing them. [CLIP](clip.md) and [SigLIP](siglip.md) are the two instances
[Lecture 17](17-multimodality.md) covers, and between them they settle three
questions that shape everything downstream.

## Why text is the supervision signal

The alternative is self-supervision from **data augmentation**, which a student
raised directly (≈10:58). You take an image, "you augment it, you crop it, you
randomly perturb it, rotate it, and you want to say that that is the same image,
i.e. they should have similar embeddings" — the lecturer names SimCLR.

The answer is the clearest statement in the lecture of what the text is *for*:

> That's useful for low-level details, but the problem is that you won't
> data-augment your way from one type of dog to another dog. So by using text, it
> gives you higher-level semantic representations of the images. (≈11:43)

Augmentation teaches invariance to transformations you can apply. It cannot teach
that a dachshund and a great dane are both dogs, because no crop or rotation turns
one into the other. **Text carries the semantics because captions describe things
at the level people care about.**

The cost is inherited from the caption distribution: the encoder learns what
captions mention. Since captions do not mention what is obvious — "if you have an
image of a dog, you don't need to say 'a dog'" (≈19:38) — the representation is
shaped by what people bother to write down.

## Why ranking beats captioning

Both CLIP and SigLIP *rank* rather than *generate*. CLIP tested the alternative
directly — predict the caption from the image — and found it much less
compute-efficient at equal images processed: 4× worse for a bag-of-words
predictor, and worse again for a full Transformer language model.

The explanation is about what each objective forces the model to model. Generating
the exact caption requires accounting for its wording, word order and style, most
of which is not about the image. Ranking it against alternatives requires only
what *distinguishes* it. "Actually modeling the exact token sequences of the
caption isn't so important for getting a rough representation of the image"
(≈21:12).

The lecturer attaches the right caveat: the metric is ImageNet accuracy, so this
is evidence about classification, not about every downstream use.

## The two losses compared

|  | [CLIP](clip.md) | [SigLIP](siglip.md) |
| --- | --- | --- |
| Question | which of $N$ images matches this text? | does this image match this text? |
| Form | softmax over the batch, both directions | sigmoid per pair, with a learned bias |
| Negatives | the rest of the batch, necessarily | the rest of the batch, incidentally |
| Batch size | a **modelling** decision — change it and the loss changes | an **optimization** knob — same loss in expectation |
| Parallelism | full-batch softmax; does not decompose | ring-structured; no device holds the full matrix |
| Small batches | degrades badly below ~16K | better than CLIP there |
| Large batches | — | 1M tried, **32K sufficed** |

## What the family gets you, and what it does not

**Gets you:** a compact vector per image that captures semantics, trained on web
data nobody had to annotate, and transferable — every [VLM](vision-language-models.md)
in the lecture starts from one of these encoders rather than training vision from
scratch.

**Does not get you two things**, both of which drive the rest of the lecture:

1. **Fine-grained detail.** The design decisions "are based on image
   classification, so it's not very fine-grained." Reading small text was never an
   objective. → [Image resolution and token budgets](image-resolution-and-tokens.md)
2. **Generation.** A continuous embedding cannot be sampled from a softmax. Making
   images generable requires either a diffusion head or
   [discrete image tokens](discrete-image-tokens.md), which is
   [Chameleon](chameleon.md)'s route.

The lecture's final synthesis is that (1) and (2) are the same problem:
"comprehension and generation might demand different things (semantics versus
finer-grained details)."

→ [Multimodal models](multimodal-models.md) · [Lecture 17](17-multimodality.md)
