# Discrete image tokens and VQ-VAE

The alternative to continuous image embeddings: map an image to a sequence of
symbols from a finite vocabulary, so that it becomes the same kind of object as
text. This is what makes image *generation* possible in a plain autoregressive
Transformer, and it is [Chameleon](chameleon.md)'s route in
[Lecture 17](17-multimodality.md).

## Why discreteness buys generation

A [CLIP](clip.md)-style encoder produces a continuous vector, and **you cannot
sample a continuous vector from a softmax**. That is the structural reason every
model in the first two thirds of the lecture can read images but not write them:
"because it's a language model, you can only generate text — you can't generate
images" (≈1:07:16).

If an image is instead a sequence of codebook indices, generating one is exactly
what a language model already does. No projector, no separate head, no diffusion
model — the architecture collapses back into
[lecture 1](01-overview-tokenization.md)'s.

## VQ-VAE

**Vector Quantized Variational Autoencoder** (2017) is the mechanism. The lecture
calls it "rather old" and describes it in three steps (≈1:09:36):

1. **Encode** a patch to a continuous vector.
2. **Quantize** — round it to the nearest entry in a learned **codebook** of, in
   Chameleon's case, 8,192 entries. "These are like prototypical vectors that
   correspond to patches — and then you pick the one that's closest, and that
   would be your representation."
3. **Decode** the codes back to pixels, and train by minimizing
   **reconstruction loss**.

$$\mathcal{L} = \mathcal{L}_{\text{recon}} + \mathcal{L}_{\text{vq}}$$

The second term exists because rounding is not differentiable — "there are some
other terms that you add, because this is not differentiable, which I won't have
time to get into" (≈1:10:22). It covers the codebook and commitment losses that
keep encoder outputs near their assigned codes.

In Chameleon the numbers are: a **512×512 image → 1,024 tokens**, from a codebook
of **8,192**.

## The supervision is completely different from CLIP's

This is the distinction that explains the whole trade-off.

| | [CLIP](clip.md) / [SigLIP](siglip.md) | VQ-VAE |
| --- | --- | --- |
| Trained against | paired **text** | **the image itself** |
| Must preserve | what a caption would mention | whatever rebuilds the pixels |
| Free to discard | everything else | nothing, in principle |
| Output | one continuous vector | a sequence of discrete codes |
| Generative? | no | yes |

A contrastive encoder is *allowed* to throw detail away, and that is what makes it
cheap and semantic. A reconstruction encoder is not — but it is bottlenecked by
the codebook, and 1,024 codes is a hard ceiling on what a 512×512 image can carry.

## BPE over image codes

Chameleon then trains a fresh tokenizer over the result — "they train a new
tokenizer, now, because your data looks different than if it were just normal
natural language" (≈1:11:09). This is [BPE](byte-pair-encoding.md), the algorithm
from lecture 1, run over image codes rather than bytes. The unification is literal:
one vocabulary, one merge table, one model.

## What it costs

**Discretization loses information**, and the lecture names exactly where: "think
about OCR, again: if you discretize, very small print, you're not going to be able
to read it anymore" (≈1:13:26). Which is precisely the capability that
[AnyRes and dynamic resolution](image-resolution-and-tokens.md) were invented to
protect on the continuous side.

## Why the branch receded

Not because it was refuted, but because a better generator arrived. VQ-VAE-style
discretization was popular for image generation for a structural reason — "you
have a Transformer — how do you generate from a Transformer? Well, you have to
generate discrete things, so you basically put all your data into this discrete
form." Then "diffusion models came out and became popular and viable for
generation. So this flavor of method is somewhat less popular than it used to be"
(≈1:14:12).

Once diffusion could generate images well, the reason to force images into a
discrete vocabulary largely went away.

→ [Chameleon](chameleon.md) · [Multimodal models](multimodal-models.md) ·
[Lecture 17](17-multimodality.md)
