# Vision Transformers (ViT)

A **Vision Transformer** applies the ordinary Transformer encoder to an image by
cutting it into fixed-size patches and treating each patch as a token.
[Paper](https://arxiv.org/pdf/2010.11929). It is the vision backbone for every
model in [Lecture 17](17-multimodality.md) — [CLIP](clip.md) tried ResNets
alongside ViTs and found ViTs best, so "when people say CLIP, they usually mean
the ViT version" (≈12:30).

## The construction

1. **Patch.** Split the image into a grid of non-overlapping patches. The original
   ViT paper used 16×16 pixels; CLIP uses 14×14 (≈12:30).
2. **Flatten and project.** Each patch is a small tensor of pixels — 14×14×3 for
   RGB — flattened and linearly projected to the model dimension. "You have these
   little patches, which are basically vectors. So each patch is, in some sense, a
   token for this Vision Transformer" (≈13:18).
3. **Add position embeddings**, "just like you would if you were training a
   language model."
4. **Run a standard Transformer encoder.** Nothing about the blocks is
   vision-specific — the same
   [pre-norm blocks, attention and MLPs](03-architectures.md) as a text model.

The patch count follows from resolution and patch size, and is the number that
matters downstream. At CLIP's 336×336 with 14×14 patches the image tiles 24×24,
so **one image becomes 576 patch tokens**.

## Pooling to a single vector

A Transformer encoder emits a *sequence*; a contrastive objective needs one
vector. Two options appear in the lecture:

- **The `[class]` token.** The original ViT prepends an extra learnable embedding
  and reads its output. This is the standard construction shown in the deck's ViT
  figure.
- **Attention pooling**, which is what CLIP does. Take the global average of the
  activations, then run one more round of attention using that average as the
  **query** against the keys and values at each position. The result is "another
  vector, which is maybe a little bit more informed than just a straight-up
  average" (≈14:07).

## Reading the model names

The naming convention recurs constantly, so it is worth decoding once.

**ViT-L/14@336px** = **L**arge, **14×14** pixel patches, trained at **336×336**
resolution. Other sizes seen in the lecture: **ViT-bigC** (as the deck prints it —
the cited OpenCLIP paper's own name is ViT-bigG) for [Qwen-VL](qwen-vl-series.md),
and a 675M-parameter ViT for Qwen2-VL.

## Position embeddings, and a question deferred

A student asked in the CLIP section whether position embeddings should treat an
image's two dimensions differently from text's one (≈15:45). CLIP tried a 2D
variant and "found it doesn't really matter that much" — with the lecturer's
immediate qualification that "you also always have to take these results with the
idea that they had classification in mind."

That qualification turns out to be right. Once resolution becomes dynamic and
video adds a time axis, the question returns and gets a real answer in
[multimodal RoPE](multimodal-rope.md).

## Why vision encoders stay small

A student asked why the vision tower has so many fewer parameters than the
language model — ViTs are "generally less than a billion parameters" against a
72B LLM (≈1:04:09). The answer is about the job:

> The vision encoder is in some sense doing a very local operation. It's looking
> at a patch — a patch is very small, and it's just trying to understand the
> patch. There's not much knowledge there — so, per patch, we're not reasoning. So
> most of the capabilities of the model are still in the language model. (≈1:04:54)

This is the structural reason [VLMs](vision-language-models.md) are built by
attaching a small encoder to a large pre-trained LM rather than the other way
round.

→ [CLIP](clip.md) · [Vision-language models](vision-language-models.md) ·
[Lecture 17](17-multimodality.md)
