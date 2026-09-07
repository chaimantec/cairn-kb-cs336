# Cross-modal transfer

[LLaVA-OneVision](llava-onevision.md)'s most interesting empirical result, and the
one [Lecture 17](17-multimodality.md) dwells on: a capability trained in **one**
input format shows up in **another** that was never trained for.

## Why it is a surprise

The training mixtures are frankly task-shaped — VQA, chart questions, OCR corpora,
GUI screenshots. That invites an obvious objection, and the lecturer raises it
against himself:

> When I first looked at this, I said, oh boy, you're basically targeting each of
> these tasks — it's kind of like supervised learning. But if you have enough
> tasks, these models seem to do some transfer, which is, I guess, reassuring.
> (≈44:36)

If the model were only doing supervised learning per task, capabilities would stay
in the format they were trained in. They do not.

## The three cases

All three have the same shape: **trained on format A, works in format B.**

**1. Chart and diagram reading: single image → multi-image.** The training data
has diagrams and charts only as single images. At test time the model is given a
table in one image and a chart in another and answers questions spanning both.
"At training time, it never saw an example where you have a table and a chart and
you're asking questions about both" (≈43:46).

**2. OCR + relational reasoning → GUI agency.** OCR appears only in single-image
data; relational reasoning only in multi-image data. Combined, they yield something
neither taught: reading a sequence of app screenshots and describing the actions
that connect them — "useful for powering GUI agents."

**3. Visual prompting: image → video.** "Visual prompting" means drawing a circle
on an image to mark what the question is about. That data exists only for still
images. The model nonetheless tracks a circled subject across video frames — "the
user says, 'describe the player highlighted in the video,' and this player is
across multiple frames" (≈44:36).

## What it suggests

The three input formats are not separate silos with separate skills. A capability
learned through one lands in a representation the others can reach — which is what
you would hope for from a design that converts every modality into tokens in one
shared sequence, and is not guaranteed by it.

The practical reading is about **data strategy**: you do not need training data in
the cross product of (capability × input format). Coverage of each capability in
*some* format, plus coverage of each format in *some* capability, appears to be
enough for the combinations to appear. That is what makes the task-shaped mixtures
defensible despite looking like plain supervised learning.

The caveat is that these are demonstrations from one paper, presented as
qualitative examples rather than measured transfer rates. The
[course material](../raw/slides/17-multimodality.md) reproduces the three figures;
none carries a benchmark number.

→ [Visual instruction tuning](visual-instruction-tuning.md) ·
[LLaVA-OneVision](llava-onevision.md) · [Lecture 17](17-multimodality.md)
