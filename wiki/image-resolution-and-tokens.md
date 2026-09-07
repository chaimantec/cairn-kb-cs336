# Image resolution and token budgets

How many tokens an image is worth, and at what resolution it is seen, is the
single most-revised design decision in [Lecture 17](17-multimodality.md). Four
successive answers appear, each fixing the previous one's failure.

The whole sequence is driven by one task: **OCR**. Reading text in a picture
"needs to preserve very fine-grained information — otherwise a 'J' looks like an
'I,' and that's not good" (≈36:45).

## 1. Fixed square crop — [CLIP](clip.md)

Resize so the short side is 336 px, then center-crop to 336×336. Chosen because
"neural nets… don't like things to be dynamic; they want things to be fixed size"
(≈9:27), and excused because CLIP targeted classification, where "usually the
object is in the middle, and you're just trimming off some background" (≈10:12).

**The failure:** it destroys detail and discards the frame. "If you have a
document and you crop to 336-by-336, you can't read it" (≈37:32).

## 2. Fixed token count — [Qwen-VL](qwen-vl-series.md)

A cross-attention adapter mapping to exactly **256 tokens**, whatever came in.

**The failure:** the compute an image gets is independent of how much is in it. A
dense screenshot and a snapshot cost the same. Flagged as provisional when
introduced (≈46:12).

## 3. AnyRes — [LLaVA 1.5](llava.md) / [LLaVA-OneVision](llava-onevision.md)

Do not change the encoder; run it several times.

1. Split the image into an $a \times b$ grid of pieces, each at the encoder's native
   resolution.
2. Encode each piece; concatenate.
3. Also downsample the whole image to one global view and encode that.
4. If the result is too many tokens, interpolate down.

The reasoning is the nicest argument in the lecture: "the Transformer is already
adaptive — sentences can be any length… So it turns out images can be any
resolution — that's kind of piggybacking on the same dynamic ability" (≈38:20).

**The remaining problem:** token count now grows with $a \times b$, so something has
to allocate it.

## The token budget

That allocation is what LLaVA-OneVision's three input modes are. The goal is to
"make all of the modalities produce roughly the same length" (≈39:55), because
otherwise video swamps everything.

| Input | Rule | Formula | Maximum |
| --- | --- | --- | --- |
| Single image | high resolution, up to 9 crops + global view | $729 + N \times 729$ | $10 \times 729 = 7290$ |
| Multiple images | base resolution each | $N \times 729$ | $12 \times 729 = 8748$ |
| Video | low resolution per frame, ≤32 frames | $N \times 196$ | $32 \times 196 = 6272$ |

Read as a budget, the three rules are obvious rather than arbitrary: "if I have a
single image, I get to look at it more carefully; if I have multiple images, I'm
just going to look at it from afar."

## 4. Dynamic resolution — [Qwen2-VL](qwen-vl-series.md)

The token count becomes a function of the input's actual size: "this picture here
might be mapped to 11,000 tokens. This tiny picture of an equation might only be
mapped to eight tokens" (≈49:16).

Each 224×224 region is encoded with a ViT/14 and every **2×2** block of adjacent
patches is merged into one token — a 4× reduction in sequence length before the
language model sees anything.

This is the end state of the sequence: an image costs what it is worth, and
nothing is cropped away.

## Why this is really a context-length problem

Every step above trades detail against sequence length, and the constraint
underneath is context. Video makes it acute — "you start running into
context-length problems for video, and later we'll see how a big part of being
able to handle multimodal is dealing with long context" (≈40:42) — which is why
[Qwen3-VL](qwen-vl-series.md) spends two of its four pre-training stages buying
context length, ending at 256K.

→ [Video understanding](video-understanding.md) ·
[Vision-language models](vision-language-models.md) ·
[Lecture 17](17-multimodality.md)
