# LLaVA-OneVision

**LLaVA-OneVision** (2024) keeps [LLaVA](llava.md)'s template and replaces every
component, then spends its real effort on resolution, token budgets, data and a
transfer result. [Paper](https://arxiv.org/pdf/2408.03326). Covered in
[Lecture 17](17-multimodality.md), where it is the most figure-heavy section —
nine of the lecture's 32 figures.

## Component swaps

| | LLaVA | LLaVA-OneVision |
| --- | --- | --- |
| Vision encoder | [CLIP](clip.md) | **[SigLIP](siglip.md)** — grid features from before *and* after the last Transformer layer |
| Text decoder | Vicuna | **Qwen-2 72B** |
| Projector | linear $W$ | **2-layer MLP** |

"It's like you have a system and you're just upgrading the parts, but it's the
same rough system" (≈36:45).

## AnyRes: adaptive resolution

The motivation is OCR, which "needs to preserve very fine-grained information —
otherwise a 'J' looks like an 'I,' and that's not good." And CLIP's 336×336
center crop destroys exactly that: "if you have a document and you crop to
336-by-336, you can't read it" (≈37:32).

**AnyRes** — introduced in LLaVA 1.5 — does not change the encoder. It runs it
several times:

1. Break the image into an $a \times b$ grid of pieces, each sized to the encoder's
   native resolution.
2. Encode each piece separately and concatenate the results.
3. Separately, downsample the *whole* image to one global view and encode that too.
4. If the token count is too high, reduce it by bilinear interpolation.

The justification is a nice piece of reasoning: "the Transformer is already
adaptive — sentences can be any length, and the Transformer already handles that
pretty well. So it turns out images can be any resolution — that's kind of
piggybacking on the same dynamic ability" (≈38:20).

→ [Image resolution and token budgets](image-resolution-and-tokens.md)

## The three input types are one token budget

Single images, multiple images and video are all "technically reducible to
images," but the paper "put their thumb on the scale a little bit, because they
want to make sure all the modalities are roughly comparable, because videos can be
very long — they don't want their dataset to be dominated by a bunch of repetitive
frames" (≈39:55). The stated goal is to make all three produce **roughly the same
sequence length**.

| Input | Rule | Tokens |
| --- | --- | --- |
| Single image | highest resolution, up to 9 crops plus the global view | $729 + N \times 729$, max $(1{+}9) \times 729 = 7290$ |
| Multiple images | base resolution each | $N \times 729$, max $12 \times 729 = 8748$ |
| Video | lowest resolution per frame, up to 32 frames | $N \times 196$, max $32 \times 196 = 6272$ |

"If I have a single image, I get to look at it more carefully; if I have multiple
images, I'm just going to look at it from afar" (≈39:55). The three maxima land
within about 40% of each other, which is the budget doing its job.

This is also where long context first bites: "you start running into
context-length problems for video, and later we'll see how a big part of being
able to handle multimodal is dealing with long context" (≈40:42).
→ [Video understanding](video-understanding.md)

## Data: "quality over quantity," read sceptically

The paper's stated philosophy is quality over quantity. The lecture offers a
sharper reading: "another way to interpret this is that it's very targeted data —
if you look at many of these, it's very task-based… So this is definitely
post-training territory, where you're trying to get your model to do these tasks,
and therefore you create a bunch of these tasks" (≈41:28).

And on provenance: "this work is also unabashedly distilling GPT-4 models, so that
they can get the best performance, which is, I guess, not ideal, but this is what
you do if you don't have an annotation budget."

Two mixtures are itemized in full in the
[course material](../raw/slides/17-multimodality.md#llava-onevision): a **3.2M**
single-image stage (General 36.1%, Doc/Chart/Screen 20.6%, Math/Reasoning 20.1%,
Language 14.3%, General OCR 8.9%) and a **1.6M** final OneVision stage
(Multi-Image 43.0%, Single-Image 31.2%, Video 25.9%).

> **Before quoting the 1.6M figure**, note that the printed per-dataset counts sum
> to about 1.32M. See the
> [figure audit](../raw/slides/17-multimodality.md#figure-audit).

## Three training stages

Named in the paper as Language-Image Alignment, High-Quality Knowledge Learning,
and Visual Instruction Tuning; the philosophy is "easier to harder."

The lecturer is honest about how principled this is: "I'm not sure there's any
particular principled reason for this, except that the second stage is trying to
put in high-quality data, but focusing more on knowledge, and the final stage is
focusing on examples that look like your downstream tasks" (≈43:00).

Stage 1 trains only the projector — for a 72.7B backbone, **72.0M** trainable
parameters. Every later stage trains the full model.

## Its most interesting result

Capabilities trained in one input format appear in another that was never trained
for. → [Cross-modal transfer](cross-modal-transfer.md)

→ [Vision-language models](vision-language-models.md) ·
[Lecture 17](17-multimodality.md)
