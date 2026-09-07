# Visual instruction tuning

Fine-tuning a [vision-language model](vision-language-models.md) on
(image, instruction, response) triples so it answers questions about pictures
rather than merely embedding them. [LLaVA](llava.md) introduced the recipe and
[Lecture 17](17-multimodality.md) treats it as the point where multimodal work
becomes a **data problem** rather than an architecture problem.

## LLaVA's bootstrap

The difficulty is obvious: nobody has a large dataset of conversations *about*
images. LLaVA's answer is to manufacture one, and the mechanism is worth stating
precisely because it is easy to misread.

**GPT-4 never sees the images.**

1. MS COCO already carries human annotations — Mechanical Turk captions and
   bounding boxes.
2. GPT-4 is prompted with **those textual annotations**, and asked to generate
   questions and conversations from them.
3. The generations are paired back with the original images.

A text-only model produces multimodal training data, because somebody had already
described the images in text. 158K examples, in three response types:

| Type | What it produces |
| --- | --- |
| Conversation | question-and-answer turns about the image's contents |
| Detailed description | a long paragraph describing the scene |
| Complex reasoning | a question requiring inference, plus its worked answer |

This is [synthetic data](synthetic-data.md) generation of the kind the
[data lectures](14-data-filtering-dedup-mixing.md) cover for text, with the twist
that the teacher model is blind to the modality it is producing data about.

## What the mixtures became

[LLaVA-OneVision](llava-onevision.md) scaled this to 3.2M single-image examples
and a 1.6M final mixture spanning single-image, multi-image and video. Reading the
itemized lists, the character is unmistakable: VQA datasets, chart and document
question answering, OCR corpora, geometry problems, GUI screenshots. The lecture
names it directly:

> Another way to interpret this is that it's very targeted data — if you look at
> many of these, it's very task-based… So this is definitely post-training
> territory, where you're trying to get your model to do these tasks, and
> therefore you create a bunch of these tasks. (≈41:28)

## The honest part

Two admissions in the lecture are worth keeping, because they are what a paper's
own framing usually smooths over.

**On distillation:** "this work is also unabashedly distilling GPT-4 models, so
that they can get the best performance, which is, I guess, not ideal, but this is
what you do if you don't have an annotation budget" (≈41:28).

**On whether this is just supervised learning:** "when I first looked at this, I
said, oh boy, you're basically targeting each of these tasks — it's kind of like
supervised learning. But if you have enough tasks, these models seem to do some
transfer, which is, I guess, reassuring" (≈44:36).

That second remark is the interesting one, and it is what
[cross-modal transfer](cross-modal-transfer.md) is about: the objection is right
about the method and wrong about the outcome.

## Why the LLaVA line is teachable at all

LLaVA-OneVision "is one of the few works that open-sources not just the model
weights, but also the data, so you can really replicate and study this stuff"
(≈45:25). The [Qwen models](qwen-vl-series.md) that follow are stronger and
markedly less documented — "not too many details about the data and the data mix."

→ [Supervised fine-tuning](supervised-fine-tuning.md) ·
[Instruction tuning datasets](instruction-tuning-datasets.md) ·
[Lecture 17](17-multimodality.md)
