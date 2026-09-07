# Chameleon

**Chameleon** (Meta, 2024) is the turn in [Lecture 17](17-multimodality.md): the
one model that abandons continuous image embeddings and represents everything as
[discrete tokens](discrete-image-tokens.md), so that a single autoregressive
Transformer can both read and write images.
[Paper](https://arxiv.org/pdf/2405.09818).

## The motivation

Every [VLM](vision-language-models.md) before it encodes images into vectors and
injects them into a language model, which means it can only emit text. There are
ways around that — "you can just take a VLM, you can also attach a diffusion head,
and now you can generate images" — but Chameleon asks a different question:
"what if we mapped everything into discrete tokens?" (≈1:08:02).

The appeal is architectural simplicity, and the lecturer owns the aesthetic
judgment: "in some ways, aesthetically — well, maybe this reflects that I'm a
language person — this is kind of appealing, because now you can analyze and
generate images in the same way, since everything is a discrete token."

No projector, no adapter, no separate generation head. "The training is now
actually very straightforward — this is just normal language model training,
right? There's no adapter, there's no — you're not turning the vision encoder —
there's just a language model now" (≈1:11:09).

## How it works

- **Tokenizer:** [VQ-VAE](discrete-image-tokens.md) maps a 512×512 image to 1,024
  tokens from a codebook of 8,192, then a fresh [BPE](byte-pair-encoding.md)
  tokenizer is trained over the combined text-and-image data.
- **Model:** one Transformer over one interleaved sequence, with `Start Image` and
  `End Image` markers delimiting image spans.
- **Generation:** sample tokens; route the text spans to output and the image
  spans to the de-tokenizer, which renders pixels.

The result is genuinely interleaved output. The paper's example prompt — "I'm
bored, can you show me some birds?" — produces "text, and then there'd be images,
and then some more text and some images" (≈1:08:48). The lecture takes this as the
clearest realization of the omni-model idea available: "text and images truly live
in the same space, and this is accomplished by making everything look like text."

## Training data

| Stage | Share | Data |
| --- | --- | --- |
| 1 | 80% | large-scale unsupervised: 2.9T text tokens, 1.5T text/image, 400B interleaved |
| 2 | 20% | 50% of stage-1 data, 50% high quality |

Read the stage-1 mixture as a ratio: even in a model built to be natively
multimodal, **text is the large majority of the data**.

## The instability, and what it teaches

This is the concrete technical finding, and it generalizes past Chameleon.

Putting two token distributions through one softmax destabilizes training, because
"text and images, despite occupying the same space, just behave very differently —
so just calling things discrete tokens doesn't hide the fact that there's an image
living there" (≈1:11:55).

The mechanism is an **entropy mismatch**. Next-word prediction is relatively
predictable — "most words are kind of predictable, whereas image tokens have very
high entropy — I don't know what shade of blue this exact token is going to be"
(≈1:12:41). The consequence was parameter-norm growth and loss instability.

The fixes are ones the course has already met, arriving here for a new reason:
**QK norm** from [architectures](03-architectures.md) and **z-loss** from the
[training stability](training-stability.md) discussion.
→ [Modality balancing](modality-balancing.md)

## The verdict

The lecture's summary is a genuine trade rather than a win, and it declines even
to show the results:

> - Elegant (just autoregressive modeling of discrete tokens)
> - Not as performant (discretization loses information - think OCR)
> - Training with multiple modalities is tricky

"I'm not even going to show the results, but I wanted to highlight this work,
because there's a certain elegance here… But the downside is that it turned out
this model was not really as performant" (≈1:13:26).

**"Think OCR" is the sharp end.** Discretizing to 1,024 codes throws away exactly
the fine detail that reading small text requires — the same capability the
continuous branch spent four design iterations protecting. The two branches of the
field are optimizing against each other on one axis, which is the lecture's
closing point: comprehension and generation want different things.

→ [Discrete image tokens](discrete-image-tokens.md) ·
[Multimodal models](multimodal-models.md) · [Lecture 17](17-multimodality.md)
