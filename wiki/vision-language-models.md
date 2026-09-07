# Vision-language models (VLMs)

A **VLM** takes images and text in and produces text out. Every one in
[Lecture 17](17-multimodality.md) is built to the same three-part template, which
[LLaVA-OneVision](llava-onevision.md)'s summary states outright:

> Standard VLM template: vision encoder + projector + LM

```
image ──▶ vision encoder ──▶ projector ──▶ image tokens ─┐
                                                          ├──▶ language model ──▶ text
text  ─────────────────────▶ embedding ──▶ text tokens  ─┘
```

## What the template is doing

The image is turned into vectors that sit in the language model's embedding space,
concatenated with the text's vectors, and the whole sequence is run through an
ordinary Transformer. "So we're, in some sense, converting these images into
textual tokens, so we can leverage the pre-trained language model" (≈32:54).

Nothing about the language model changes. That is the point: the vision encoder
and the LM both already exist and are both expensive, so the work is in the join.
The lecture is explicit that this is post-training rather than pre-training:
"we take an existing image encoder, we take an existing LLM, and then we kind of
stitch it together, rather than training something from scratch" (≈29:01).

## The five models, as variations on one design

| Model | Vision encoder | Projector | Language model |
| --- | --- | --- | --- |
| [LLaVA](llava.md) (2023) | CLIP ViT-L/14 | **linear** matrix $W$ | Vicuna |
| [LLaVA-OneVision](llava-onevision.md) (2024) | SigLIP | **2-layer MLP** | Qwen-2 72B |
| [Qwen-VL](qwen-vl-series.md) (2023) | OpenCLIP ViT-bigC | **1-layer cross-attention**, fixed 256 tokens | Qwen |
| [Qwen2-VL](qwen-vl-series.md) (2024) | 675M ViT, dynamic res. | 2×2 patch merge | Qwen2 |
| [Qwen3-VL](qwen-vl-series.md) (2025) | SigLIP-2 | **DeepStack** — injects into many layers | Qwen3 (dense + MoE) |

Read down the projector column and you have the entire architectural story of the
lecture's middle. → [Modality projectors](modality-projectors.md)

## Staged training, and what "frozen" tracks

Every model trains in stages, and each stage decides **which components may
move**. The organizing principle is that a stage's data quality should determine
what it is allowed to change: cheap noisy data may adjust the vision side,
expensive instruction data should only touch the parts that talk.

**LLaVA — two stages.**

| Stage | Vision encoder | Projector | LM | Purpose |
| --- | --- | --- | --- | --- |
| 1 (alignment) | frozen | **train** | frozen | make image vectors "look like natural-language token embeddings" (≈33:40) |
| 2 (fine-tuning) | frozen | train | **train** | learn the tasks |

**Qwen-VL — three stages, and the pattern is not monotonic.**

| Stage | Vision encoder | Adapter | LM | Data |
| --- | --- | --- | --- | --- |
| 1 | **train** | train | frozen | large-scale, low quality |
| 2 | train | train | **train** | higher quality, task-specific, higher resolution |
| 3 | **frozen** | train | train | instruction tuning |

Note the difference: LLaVA **never trains the vision encoder**; Qwen-VL trains it
in stages 1 and 2 and freezes it again in stage 3.

## Alignment requires a pre-trained LM

A student asked whether the language model has to be pre-trained before the
alignment stage (≈1:03:22). It does — "otherwise it doesn't make sense to align
it." The alignment stage is *only* teaching the projector a coordinate change
between two representations that are already good.

It is also not adaptive: "you pick a token budget — say, 67 billion tokens — and
you just train. There's not an adaptive threshold here."

## Where the effort actually goes

The architecture is settled; the work is data. LLaVA-OneVision's own summary says
"most work goes into data curation (heavy on synthesized, task-specific data)",
and the lecture is blunter still — the mixtures are "very targeted… very
task-based," which is "definitely post-training territory," and the work is
"unabashedly distilling GPT-4 models… which is, I guess, not ideal, but this is
what you do if you don't have an annotation budget" (≈41:28).

→ [Visual instruction tuning](visual-instruction-tuning.md) ·
[Image resolution and token budgets](image-resolution-and-tokens.md) ·
[Multimodal models](multimodal-models.md) · [Lecture 17](17-multimodality.md)
