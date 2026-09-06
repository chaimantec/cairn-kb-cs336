# Preference data

The comparison judgments that [reward models](reward-models.md) and
[DPO](dpo.md) are trained on: for a given prompt, which of two (or more) model
responses is better.

This page covers the artifact — how it is collected and what the instructions to
raters look like. Who does the rating, and what difference that makes, is
[human annotation](human-annotation.md); the shift to models doing it is
[model-based annotation](model-based-annotation.md).

## The standard shape

From [RLHF](rlhf.md)'s pipeline (≈46:14):

1. Put a prompt into the post-SFT model.
2. Sample a few outputs, "usually... by just sampling with temperature one — at
   the end of SFT the model is pretty diverse, so this is a reasonable thing to
   do."
3. A rater ranks them — "sometimes even a binary ranking."
4. Train a reward model on those rankings; optimize against it.

Note the dependence in step 2: the procedure assumes the SFT model still has
enough output diversity for sampled responses to differ meaningfully. That
assumption gets weaker as RLHF proceeds, which connects to
[mode collapse](mode-collapse-and-calibration.md).

The collection interface is unremarkable and roughly universal — two responses
side by side, pick one:

![Slide 36 — RLHF and data – standard setups](../raw/images/15-mid-post-training/slide-36.jpg)
*Slide 36 — a pairwise annotation interface of the kind most preference collection uses. "This is just a standard annotation interface I went and found, but most places have similar things."*

## What raters are actually told

The interesting content is in the *guidelines*, and there are essentially two
public examples in existence.

### InstructGPT

"Kind of the last point at which we have a glimpse into data-collection processes
from industry," and the lecture's explicit recommendation for anyone who wants to
read further (≈47:00).

The guidelines ask raters to balance three properties (≈47:45):

- **Helpful** — clarity of writing, sensitivity to internationality, not giving
  overly long answers.
- **Truthful** — don't hallucinate.
- **Harmless** — flag unsafe content; upweight responses that decline questionable
  prompts.

The lecturer's characterization is worth keeping: "this makes sense as an odd,
omnibus objective for training chatbots." Three properties that trade off against
each other, collapsed into one scalar preference, with the trade-off resolved
implicitly by whoever happens to be rating.

![Slide 37 — RLHF and data – instruct GPT guideline](../raw/images/15-mid-post-training/slide-37.jpg)
*Slide 37 — the InstructGPT annotation guidelines.*

Note also that "helpful" here includes *not* being overly long — the opposite of
what raters actually reward in practice. See
[style and length bias](style-and-length-bias.md).

### Google Bard

The only other public example, and public by accident: "their annotations got
leaked by some annotators" (≈48:33). Structurally similar — helpfulness, quality
of presentation — but rated on a **Likert scale** rather than pairwise.

The specifics are ordinary in a way that is itself informative: "don't contain
inaccurate information, or be coherent and easy to consume, so short for the user
to read." As the lecturer says, "it's not a super long set of instructions, but
you get the sense of what kinds of things are included."

![Slide 38 — Another old example – bard annotations](../raw/images/15-mid-post-training/slide-38.jpg)
*Slide 38 — the leaked Bard side-by-side rating guidelines.*

*(The introductory sentence of this screenshot is cut off mid-word in the deck
itself — see the slide file's Known gaps table.)*

The same leak has a second life later in the lecture, as evidence about working
conditions: these annotators were disputing a requirement to fact-check long chat
responses in under a minute. See
[human annotation](human-annotation.md#the-conditions-are-part-of-the-data-quality-story).

## Everything here is historical

Both examples predate the competitive scramble, and nothing comparable exists for
a current model. This is the lecture's standing caveat about the whole topic
(≈3:55):

> Now that competition has heated up, basically none of the vendors want to
> release any information about their post-training processes. The data is very
> much a trade secret.

For anyone wanting real depth, the recommendation is the appendices of
**Stiennon et al., "Learning to Summarize from Human Feedback"** — "that's
actually incredibly detailed... it contains things like detailed annotation
guidelines, how they were instructing people to give feedback to the models"
(≈3:10) — and Anthropic's 2022 HH paper.

## See also

- [RLHF](rlhf.md) — the pipeline this feeds
- [Reward models](reward-models.md) — what is fit to this data
- [Human annotation](human-annotation.md) — who produces it
- [Model-based annotation](model-based-annotation.md) — what largely replaced them
- [Style and length bias](style-and-length-bias.md) — the systematic distortion in the labels
- [Lecture 15](15-mid-post-training.md)
