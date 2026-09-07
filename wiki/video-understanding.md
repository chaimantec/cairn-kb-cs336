# Video understanding

Video is the modality that breaks every budget in
[Lecture 17](17-multimodality.md). It is handled as "essentially a sequence of
images corresponding to a sampling of the frames" (≈36:00) — but that sampling,
and what it costs in context length, is where the design work is.

## Video is a token-count problem

A single image at LLaVA-OneVision's full resolution costs up to 7,290 tokens. A
video is many frames. Something has to give, and what gives is per-frame
resolution.

| Model | Sampling | Per-frame cost | Cap |
| --- | --- | --- | --- |
| [LLaVA-OneVision](llava-onevision.md) | up to 32 frames | 196 tokens | $32 \times 196 = 6272$ |
| [Qwen2-VL](qwen-vl-series.md) | 2 frames/second | ~66 tokens per 224×224 region | 16,384 tokens |

LLaVA-OneVision's rule is explicitly comparative — "for video, we're going to use
even lower resolution, or fewer tokens, to represent each frame, because the idea
is — well, videos can be long" — and the goal is that all three input types land
at roughly the same sequence length (≈40:42).
→ [Image resolution and token budgets](image-resolution-and-tokens.md)

Qwen2-VL's 2 frames/second with a 16,384-token cap works out to a few minutes of
video, which is the real limit on what these models can watch.

## Why this makes long context the bottleneck

The lecture flags it as a through-line: "you start running into context-length
problems for video, and later we'll see how a big part of being able to handle
multimodal is dealing with long context" (≈40:42).

[Qwen3-VL](qwen-vl-series.md) is the payoff. Its context reaches **256K**, "which
is really important if you're trying to do long video," and it buys that length
across two of its four pre-training stages — 8K, then 32K, then 262,144 — rather
than all at once.

## Position and time

Two mechanisms give the model a notion of *when*:

- **[MRoPE](multimodal-rope.md)** makes position a $(t, h, w)$ triple, so time is an
  axis of the positional encoding rather than an accident of sequence order.
- **Explicit timestamp tokens**, added in Qwen3-VL, put time into the sequence as
  content. Before, "the timestamp was kind of implicit in the positional
  encodings"; now "a token like '0 seconds' is now a token that represents the
  time," which makes it directly referable — "what happened after two seconds"
  (≈55:31).

## Video distorts training in two further ways

**Gradient share.** Long video examples dominate a per-token loss, which is what
Qwen3-VL's square-root normalization corrects.
→ [Modality balancing](modality-balancing.md)

**Data loading.** The one systems cost the course had not needed to consider:
"video data, loading it can be a bottleneck. Generally, we have not really focused
on data loading when talking about language models, because it's very cheap"
(≈1:01:02).

## What is not covered

**Video generation.** A student asked directly, and the answer is categorical for
every model in the lecture: "these models don't generate video or images — all the
multimodal stuff is on the input side. You're always generating text" (≈59:28).
[Chameleon](chameleon.md) generates images, not video.

→ [Multimodal models](multimodal-models.md) · [Lecture 17](17-multimodality.md)
