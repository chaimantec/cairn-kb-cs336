# Midtraining (two-phase training)

Midtraining is the practice of mixing high-quality and instruction-style data
into the **decay phase** of pre-training, rather than applying it afterwards as a
separate fine-tuning stage. CS336's lecture 15 presents it as the one genuinely
important methodological development in the SFT half of post-training — and
notes that "everyone, as far as I know, is doing it" (≈36:13).

## What changed

> It used to be that, when these things first appeared, pre-training and
> post-training were just two separate ideas — you would pre-train on web data and
> then post-train afterward. But people realized: why separate two things when you
> can mix them together?

So instruction data, chat data and other high-quality sources are now blended in
"at the tail end of training, when the decay phase of training usually happens."
Two benefits the lecture names: it "allows you to scale up instruction tuning,"
and it "allows you to essentially emphasize higher-quality data."

The clearest published example is miniCPM's, which the lecture uses because "they
show very nicely the change in the mix" (≈37:45).

![Slide 30 — 'Midtraining' / 'Two-phase training'](../raw/images/15-mid-post-training/slide-30.jpg)
*Slide 30 — miniCPM's stable-stage mixture (left, general web data) against its decay-stage mixture (right, with Stack Exchange QA, UltraChat and other SFT-style sources, and a reduced share of general pre-training data).*

The practice shows up in "most of the model-release reports — that there's a
mid-training, or second-phase pre-training, with a different data mix" (≈36:59),
and it is publicized particularly in recent Chinese-developed models: slide 30
names miniCPM and jetMoE.

## Why the decay phase specifically

A student asks whether the decay phase gets *lower*-quality data, and the answer
inverts the assumption (≈38:32):

> Usually the intuition is you want the reverse — that the decay is the most
> important part of your training. It's the part closest to deployment during
> pre-training. Also, it's the part with the lowest learning rate. And so, for
> both of those reasons, maybe you want to put the highest-quality stuff into the
> decay.

Two independent arguments, then: recency, and small steps. See
[WSD schedules](wsd-schedules.md) for the learning-rate structure that makes a
distinct decay phase available in the first place.

## "Base model" has stopped meaning what it meant

The lecturer flags this as a personal pet peeve, and it is a genuinely useful
warning when reading model cards (≈36:59):

> Now, when someone tells you something is a base model, that's kind of a lie,
> because a base model is usually about predicting the next word on internet data
> or something. Well, really, base models today are pre-trained on things like
> UltraChat and who knows what else. Those are chat datasets that are
> synthetically designed to make you good at chat. So it's very hard to say this
> is a base model in the traditional sense of the word.

The practical implication: a "base model" release may already have substantial
chat capability baked in, and comparisons that treat base models as
instruction-naive are measuring something other than what they claim.

Asked whether the prompt is masked in this data — as it typically is in SFT — the
answer is no: "this is pure pre-training, so you're also predicting the prompt as
well. But actually that's not a huge difference, because some SFT recipes do also
predict the prompt" (≈38:32).

## Midtraining is where mixture ablations actually happen

This is the part with the most practical leverage, and it reframes
[data mixture selection](data-mixture-selection.md) (≈40:05):

> The nice thing about this two-phase training, this mid-training stuff, is that
> mid-training is much shorter than full pre-training. So if you're doing data-mix
> optimization for mid-training, you can run something like ten of these for each
> one of these on the left. So this lets you not quite brute-force the problem,
> but it does let you try a lot of ablations.

And the ablations feed *backwards*:

> The usual way I've heard a lot of this done is: in the decay phase, you do a lot
> of data ablations, which are cheap to run, and then you get estimates of data
> quality, and you reflect that back even to the first stage — into the
> pre-training stage.

So the decay phase functions as the cheap experimental harness for decisions about
the expensive stable phase. This is a concrete mechanism behind lecture 9's and
lecture 14's shared observation that mixture selection is done by small-scale
proxy rather than by theory.

**Why not just make all of pre-training high-quality?** Token supply. "You run out
of tokens if you try to make Wikipedia your whole pre-training set" (≈40:51). See
[data repetition](data-repetition.md) for what happens when you try.

## The honest position on mixtures

Asked where the mixtures come from, the lecturer does not oversell the literature
(≈39:19):

> Data mixtures, both for pre-training and post-training, are very trial-and-error.
> We do have algorithms — many people have written papers about things in this
> space — but I think they're fairly unreliable. There's a lot of trial and error
> and intuition that happens for this space.

And on turning ablation results into a mix: "That's very case-by-case... There are
systematic ways — you can fit models and do these things — but they're often
brittle. What I've heard from people is that it's actually much more
trial-and-error than you think" (≈40:51).

The concrete illustration is a legal document. When Meta was sued over its use of
books, the court filings included "a bunch of documents showing researchers doing
ablations and trying to estimate how useful each of these book subsets is. And
that's exactly the kind of thing that happens" (≈41:38). See
[copyright and fair use](copyright-and-fair-use.md) for that case's other role in
this course.

## See also

- [Data mixture selection](data-mixture-selection.md) — the fuller treatment, from lectures 9 and 14
- [WSD schedules](wsd-schedules.md) — the decay phase this exploits
- [Supervised fine-tuning](supervised-fine-tuning.md) — what midtraining partly absorbs
- [Instruction-tuning datasets](instruction-tuning-datasets.md) — what gets mixed in
- [Data repetition](data-repetition.md) — the token-supply constraint
- [Pretraining datasets](pretraining-datasets.md)
- [Lecture 15](15-mid-post-training.md)
