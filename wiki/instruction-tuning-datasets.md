# Instruction-tuning datasets

The open world's SFT data, in roughly chronological order. CS336's lecture 15
walks this progression on slide 9 "top left to bottom right — that's roughly my
chronological ordering" (≈7:01), and uses it to trace how the field's beliefs
about post-training data changed.

![Slide 9 — Progression of SFT data (in the open world)](../raw/images/15-mid-post-training/slide-9.jpg)
*Slide 9 — the progression this page follows.*

The caveat that frames the whole thing: this is the *open* world. "In the
closed-source world... how you're collecting the human data is a big, important
ingredient that's not reflected in any of this discussion" (≈10:04).

## FLAN

The oldest and, the lecturer argues, "in some ways the most visionary" (≈7:46).
Used to train Google's T5. The idea: NLP has already collected hundreds of
supervised input/output datasets, so train the model on all of them.

It is also the most instructive failure in the sequence, in two distinct ways.

**The examples are unnatural.** FLAN is manufactured from existing benchmarks, so
its inputs have the shape of a benchmark rather than a request. The lecture's
example is an Enron email with the instruction *appended after the body*: "here's
an email... write a subject line for this email." As the lecturer puts it, "I
don't think you've ever probably prompted a model to do a task like this, let
alone with the instructions at the very end" (≈11:36). The Enron Emails corpus
has a body and a subject, so it can be mechanically turned into a
subject-prediction task — which is exactly why the result reads as it does.

**It inherits its sources' defects.** The summarization examples come from one of
the standard summarization corpora (the lecturer is unsure whether CNN/Daily Mail
or XSum), and the targets are conspicuously short — "In fact, if you look
really closely, the summaries are often hallucinated — there's a bunch of details
that are not in the inputs" (≈12:23). Train on that and "you also inherit a whole
bunch of deficiencies from those datasets."

**It bet on scale.** "When FLAN was originally constructed, part of the theory of
how post-training should work was that you needed scale. Like, in pre-training we
know we need a lot of scale, so you might also think that in post-training we need
a ton of scale" (≈13:09). The subsequent datasets get *smaller* and work better —
see [supervised fine-tuning](supervised-fine-tuning.md#1-you-need-far-less-data-than-you-would-guess).
The lecturer is fair about this: "it's kind of the wrong point, in some ways, on
the quality/quantity trade-off — although there was no way to know until we
explored the full space of these datasets."

![Slide 7 — What are the ingredients in SFT?](../raw/images/15-mid-post-training/slide-7.jpg)
*Slide 7 — FLAN's task mixture: T0-SF, Muffin, Natural Instructions v2 and CoT, and the resulting dataset composition.*

## Self-Instruct

"also sort of very forward-looking in its thinking, saying: well, why can't we use
the model itself to generate data? Models are getting better all the time, and in
fact they might even be better than some of our annotators" (≈8:33). The seed of
everything on [model-based annotation](model-based-annotation.md), and the
capability-focused sibling of Constitutional AI's self-training loop.

## Alpaca and Vicuna

Distillation from ChatGPT. Alpaca distilled ChatGPT traces into input/output
pairs; Vicuna, from Berkeley, used online user-shared prompts as the inputs.

The result was the field's turning point: "you get more natural-looking inputs,
longer, chattier outputs, because of course it's taken from ChatGPT. And what we
found was that these kinds of examples now reliably induced ChatGPT-like
behavior on models" (≈14:41).

With one condition the lecturer is careful to state — this worked "importantly,
only when we did it on the original Llama models. So pre-training and
post-training both need to work out." A good post-training recipe cannot rescue a
weak base model, the same point made at ≈2:24.

## Open Assistant

The maximal human effort, and the road not taken. After Alpaca, "there was this
enormous optimism that if only we could collect a sufficiently high-quality and
large instruction-tuning dataset, we could then catch up and match a lot of the
performance of these big closed-source labs" (≈15:28).

Open Assistant was the crowdsourced attempt: volunteers writing hard prompts and
high-quality responses, "in the same way that Wikipedia managed to produce
something very high quality at scale" (≈16:15). The data is chat-shaped, with
long, detailed, expert responses.

The lecturer's verdict is admiring and blunt: "a very admirable, very impressive
effort that generated a decent amount of data — I forget if it's like 10,000 or
more examples — before stalling out as a project."

Open Assistant is also the source of the lecture's
[hallucination argument](hallucination-and-knowledge-extraction.md): one of its
high-quality responses carries an academic citation, and SFT'ing on it teaches
both the citation and the habit of citing.

## WizardLM and Tulu 3

The return to synthesis, now deliberate: "we know that language models are very
good synthetic data generators, so why don't we come up with increasingly
complicated ways of generating instruction-following data using language models?"
(≈9:18).

**Tulu 3** is the lecture's recommended reference for post-training as a whole —
the pipeline behind Allen AI's OLMo models, and "maybe the only good reference
right now" for how a performant post-training pipeline actually fits together
(≈30:04). Its safety component alone is ~50k examples. It uses model-based
annotation throughout its pipeline (≈1:02:27).

![Slide 24 — Pipeline with most details](../raw/images/15-mid-post-training/slide-24.jpg)
*Slide 24 — the Tulu 3 pipeline, the most detailed public description of open post-training.*

## Nemotron and the agentic turn

The newest generation is not about chat. "We've really started to move, as a
field — in terms of the products that people want — from just a chat interface to
something that's like a full agent system. We don't want just textual responses —
we want tool calls, we want to-do lists. If you've ever used Claude Code or
Codex, you know it produces a to-do list to check off as it goes" (≈17:00).

NVIDIA's **Nemotron** open SFT data is largely agentic: responses in the
assistant field *plus* tool calls that can happen in parallel alongside the text,
"and this is SFT data, so this is explicitly supervised into the model." See
[agent trajectory data](agent-trajectory-data.md) for lecture 14's treatment of
where such data comes from.

## The three shifts

The lecture's own summary of what changed across the sequence (≈17:47):

1. **Chattiness.** The classic NLP datasets were "very input-to-programmatic-output,
   almost. But people don't really want to talk to an NLP benchmark, they want to
   talk to people."
2. **Higher-quality annotators and more detail.** Open Assistant is the exemplar —
   experts writing the responses. See [human annotation](human-annotation.md).
3. **Tool use**, and the interface question that comes with it.

## Style varies enormously across these

Slide 16 tabulates the datasets side by side, and the differences in length and
register are large. Those differences are *choices*, and they propagate into the
model's tone and into your evaluation numbers — see
[style and length bias](style-and-length-bias.md).

![Slide 16 — Style variations in data and models](../raw/images/15-mid-post-training/slide-16.jpg)
*Slide 16 — "Table 1: Instruction datasets investigated in this work" — the datasets compared directly.*

## See also

- [Supervised fine-tuning](supervised-fine-tuning.md) — what you do with these
- [Post-training data](post-training-data.md) — lecture 14's taxonomy
- [Synthetic data](synthetic-data.md) — the generation side
- [Model-based annotation](model-based-annotation.md) — where this ended up
- [Style and length bias](style-and-length-bias.md)
- [Agent trajectory data](agent-trajectory-data.md) — the agentic generation
- [Lecture 15](15-mid-post-training.md)
