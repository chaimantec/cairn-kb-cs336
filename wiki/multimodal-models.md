# Multimodal models

A **multimodal model** accepts, and sometimes produces, data in more than one
modality — text, images, audio, video. CS336 covers them in
[Lecture 17](17-multimodality.md), and the lecture's organizing insight is that
the problem reduces to a single question about interfaces.

## Why tokens are the whole problem

The argument, as the lecture states it:

> - Transformers work really well. So we gotta use them.
> - Transformers speak tokens (discrete or continuous), where a token represents some ~semantic unit of information.
> - Therefore, we must convert everything into tokens.

The first premise is empirical rather than principled: "despite the best efforts
of people to try other things, across all these modalities they are still, at
scale, the best thing we have" (≈1:39). Given that, the architecture is fixed and
the only open question is what to feed it.

Two clarifications in the second bullet carry the rest of the lecture.

**A token need not be discrete.** Text tokens are symbols drawn from a vocabulary,
with an embedding table behind them. Image tokens, in most of the models below,
are *continuous vectors* produced directly by an encoder, with no vocabulary at
all. Both are tokens in the sense that matters: a Transformer can attend over
them. Only [Chameleon](chameleon.md) makes image tokens discrete, and it pays for
it.

**A token must be roughly a semantic unit.** This rules out the obvious approach.
"In natural language, tokens are subwords, and these are somewhat meaningful,
whereas a pixel is certainly not meaningful by itself" (≈2:26). You cannot feed
pixels to a Transformer and expect a language model's machinery to apply.

So the lecture frames image encoding as the sequel to
[tokenization](tokenization.md): "what is the equivalent of the BPE tokenizer that
will take an image and produce things that a Transformer can digest" (≈3:12).
Tokenization was already an unloved compromise for text; this is the same problem
one notch harder.

## The omni model, and the two questions

The stated destination is an **omni model**: "the ability to take any combination
of these modalities as input and output any combination of these modalities"
(≈0:51). From it fall the two questions the lecture is built on:

> 1. How do we input non-text data (e.g., understand images)?
> 2. How do we output non-text data (e.g., generate audio)?

**Almost everything known publicly answers question 1.** CLIP, SigLIP, LLaVA,
LLaVA-OneVision and the three Qwen-VL models all take images *in* and emit text.
Question 2 is structurally harder: a continuous embedding is not something you can
sample from a softmax, so generation needs either a separate diffusion head or a
discrete vocabulary.

## The two branches, and why they trade against each other

Everything in the lecture sorts into one of two answers.

| | Continuous encoders | Discrete tokens |
| --- | --- | --- |
| Examples | [CLIP](clip.md), [SigLIP](siglip.md), the whole [VLM](vision-language-models.md) family | [Chameleon](chameleon.md) via [VQ-VAE](discrete-image-tokens.md) |
| Supervision | paired text — keep what a caption would mention | reconstruction — keep what rebuilds the pixels |
| Can generate images? | not without a separate diffusion model | yes, in the same autoregressive loop as text |
| Fine detail | preserved by raising [resolution](image-resolution-and-tokens.md) | lost at the codebook — "think about OCR" |
| Architecture | encoder + [projector](modality-projectors.md) + LM | one language model, nothing else |

The lecture's synthesis is that this is not a bug in either branch but a real
tension: **"comprehension and generation might demand different things (semantics
versus finer-grained details)."** Comprehension can afford to throw detail away —
that is what makes a contrastive encoder cheap and effective. Generation needs the
detail back.

Hence the lecture's account of what frontier practice probably is, which it
explicitly labels speculation: *continuous encoders + Transformer + diffusion
models for generation* — a compromise rather than a unification (≈1:14:58).

## The one rule that applies to every design

Modalities differ in **information density**, and a model trained on a mixture has
to correct for it. "Video certainly has lower information density than text, so
you don't want video to overwhelm your text" (≈1:16:30). This shows up three
separate times in the lecture, in three different guises — as
[LLaVA-OneVision's token budget](image-resolution-and-tokens.md), as Qwen3-VL's
square-root-normalized loss, and as
[Chameleon's training instability](modality-balancing.md).

→ [Vision-language models](vision-language-models.md) ·
[Modality balancing](modality-balancing.md) ·
[Lecture 17](17-multimodality.md)
