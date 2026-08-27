# Scaling laws

> **Coverage note.** This page records what [Lecture 1](01-overview-tokenization.md)
> says about scaling laws, which is a **preview**, delivered at [44:44]–[53:11] as
> part of the syllabus tour. CS336 teaches the material properly in Lectures 9 and
> 11 (Tatsunori Hashimoto) and Assignment 3, none of which this knowledge base
> covers yet. Treat what follows as the framing, not the treatment.

## The problem

The setting Percy poses at [45:30]: suppose you have $10^{25}$ FLOPs of compute —
tens of millions of dollars — and you have to decide what model to train.

You cannot do ordinary hyperparameter tuning, because at that budget **you only
get to train one model**. Mess it up and the money is gone. This is the problem
that distinguishes large-scale pre-training from anything you encounter
fine-tuning or working at small scale, and it is why scaling laws exist as a
discipline rather than a curiosity.

## The conceptual shift: recipes, not models

The key move, at [46:16], is to stop thinking about a single model and start
thinking about a **scaling recipe**: a mapping from a FLOP budget to a set of
hyperparameters — effectively a function that emits a config file.

$$\text{scaling recipe}: \quad C \;\longmapsto\; \text{hyperparameters}$$

Given a recipe, the procedure is:

1. Run experiments at various **smaller** scales (say up to $10^{24}$ FLOPs) and
   measure the loss each one achieves.
2. **Fit a scaling law** to those points.
3. **Extrapolate** to the target scale.

That primitive buys two things, and the second is the one people underrate:

- You can **optimize the recipe** for a large scale using only small-scale
  experiments.
- You can **predict the loss** before running the expensive job. Percy's gloss at
  [47:02] is commercial as much as scientific: you can go and raise money on the
  strength of "I ran the small-scale experiments and I think I can get a
  GPT-5-level model."

## Scaling laws are not laws of nature

The correction at [47:47], and the most important sentence in the section:
**scaling laws don't happen automatically — you have to will them into existence.**
They are a property of a carefully constructed recipe, not something the universe
provides.

What makes a recipe extrapolate is that its hyperparameters vary with scale in a
*predictable* way: as scale grows, maybe the learning rate holds constant, maybe
it drops, maybe batch size grows — and by how much is exactly what the recipe has
to pin down.

### Hyperparameter transfer

This is why parameterization matters ([48:32]). You want **hyperparameter
transfer**: the hyperparameters you use at small scale are either the same ones
you use at large scale, or predictable functions of them. Percy's argument by
contradiction — if your best learning rate is sometimes `1e-5` and sometimes
`1e-4` depending on scale, you have no way to guess it at the scale that matters.
The lecture cites [muP](https://arxiv.org/abs/2203.03466) as the technique here.

### Predictability over optimality

The resulting shift in values, stated at [48:32]: **predictability is at least as
important as optimality.** The instinct is to tune for the best possible number.
But a slightly worse configuration whose behaviour you can extrapolate is more
valuable than a slightly better one that surprises you at 100× the budget.

Percy makes the same point about the *fit* at [50:05]: when you plot optimal
parameter count against FLOP budget, if you are lucky the points lie roughly on a
line, and if you are unlucky they are all over the place — in which case you
should have **no confidence** in your extrapolation. The scatter is the diagnostic.

## Compute-optimal scaling: Kaplan and Chinchilla

The classic question ([49:18]): given a FLOPs budget, do you train a bigger model
or train on more tokens? The budget constraint is

$$C = 6ND$$

for $N$ parameters and $D$ training tokens. (Lecture 2 derives the factor of 6;
Lecture 1 only previews it at [36:18].)

The method — **ISOFLOP curves**, from
[Kaplan et al.](https://arxiv.org/pdf/2001.08361.pdf) and
[Chinchilla](https://arxiv.org/pdf/2203.15556.pdf):

1. For each of several small FLOP budgets — the lecture describes a sweep from
   about `6e18` to `3e21` — sweep model size $N$ and find the one minimizing loss.
2. Fit a curve through those optima to predict $N$ from $C$.
3. Extrapolate to large budgets.

The headline result:

$$D \approx 20N$$

So a 70B-parameter model should be trained on roughly **1.4 trillion tokens**.
Percy flags this as "quite crude" and notes the number varies with dataset and
architecture ([50:51]).

### The caveat that changed practice

Compute-optimal scaling optimizes *training* cost and ignores **inference** cost
([50:51]). This is why so many current models are smaller than Chinchilla-optimal
and trained on far more tokens than the rule suggests: if you are going to serve
a model to many users, you want a small one, and you will happily overspend on
training to get it.

## Pre-registration: the Marin example

At [50:51]–[51:38] Percy describes what his own group does — **pre-registering**
predictions. Fit scaling laws at several compute budgets, predict the loss for a
much larger run, publish the prediction, then train the model and see whether you
hit it.

He mentions a run in progress at the time of the lecture, expected to finish
within days, with a report promised for the Wednesday class. This knowledge base
does not cover Lecture 2, so **the outcome is not recorded here** — see the
[Marin project](https://marin.readthedocs.io/) for the retrospectives.

The scientific point stands independent of the result: predicting the loss of a
model you have never trained, and being held to it, is a much stronger claim than
fitting a curve after the fact.

## Assignment 3

The assignment simulates the high-stakes setting without the budget ([51:38]).
The staff train models offline and expose a cached **training API**: you submit a
config, you get a loss back. You spend a FLOPs budget gathering points, fit
scaling laws, extrapolate, and submit predicted hyperparameters and a predicted
loss. A leaderboard scores how well you landed.

Percy's framing: it is meant to replicate the stress of having, say, $100M you
must spend carefully — "of course, this is low-stakes."

## Related

- [Efficiency](efficiency.md) — scaling laws as the efficiency principle applied
  to the experiment budget
- [Course map](course-map.md) — where this sits in the syllabus
- [Lecture 1](01-overview-tokenization.md)

## Sources

- [Lecture 1](01-overview-tokenization.md), scaling-laws preview [44:44]–[53:11]
- [`lecture_01.py` transcription](../raw/slides/01-overview-tokenization.md#unit-3--scaling-laws-assignment-3)
- [Edited transcript](../raw/transcripts/01-overview-tokenization.md)
