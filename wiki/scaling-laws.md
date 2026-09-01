# Scaling laws

**Start here** for scaling laws, then follow the links. This page is the hub: it
keeps Percy's framing from [Lecture 1](01-overview-tokenization.md) ([44:44]–[53:11]),
which is a preview delivered as part of the syllabus tour, and points at the proper
treatment.

> **Coverage.** [Lecture 9](09-scaling-laws.md) — the basics — is now covered in
> full. Lecture 11, the advanced treatment (muP in depth, modern open-model tech
> reports, optimizers), and Assignment 3 are **not yet covered**.

## The treatment, in order

| Page | What it covers |
|---|---|
| [Lecture 9 — Scaling laws (basics)](09-scaling-laws.md) | The whole lecture: prehistory, data scaling, model engineering, Chinchilla. |
| [Data scaling laws](data-scaling-laws.md) | The univariate law, the mean-estimation derivation, and why the exponent is ≈0.1 rather than 1. |
| [Compute-optimal scaling](compute-optimal-scaling.md) | Kaplan vs Chinchilla, the three methods, why they disagreed, and why you should overtrain anyway. |
| [The IsoFLOP method](isoflop-method.md) | The sweep-at-fixed-compute tool, and where it keeps working. |
| [Scaling law methodology](scaling-law-methodology.md) | Why the regularity is engineered, and how to avoid fooling yourself. |
| [Upstream vs downstream](upstream-vs-downstream.md) | Where predictability stops. |
| [Critical batch size](critical-batch-size.md) | Batch size as both a systems budget and an optimisation quantity. |
| [Learning rate scaling and muP](learning-rate-scaling-and-mup.md) | The two philosophies for the other un-inheritable hyperparameter. |
| [Data repetition](data-repetition.md) · [Data mixture selection](data-mixture-selection.md) | The data-composition questions. |

What follows on this page is Lecture 1's framing — still the clearest short
statement of *why* the technique exists.

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
within days, with a report promised for the Wednesday class. **The outcome is not
recorded in this knowledge base** — the promised report was a live class
announcement rather than lecture content, and no covered lecture returns to it. See
the [Marin project](https://marin.readthedocs.io/) for the retrospectives.

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

## How Lecture 9 answers Lecture 1's preview

Percy's preview sets up four claims that [Lecture 9](09-scaling-laws.md) then
delivers on, and it is worth reading them against each other:

- **"You have to will them into existence."** Lecture 9 makes this concrete by
  showing three ways Kaplan's recipe was defective — parameter counting,
  learning-rate warm-up, batch size — each of which moved the fitted exponent
  ([scaling law methodology](scaling-law-methodology.md)).
- **Hyperparameter transfer, and muP.** Lecture 9 gives the picture: staggered
  minima under standard practice, a shared minimum under muP, and the two competing
  philosophies ([learning rate scaling and muP](learning-rate-scaling-and-mup.md)).
- **IsoFLOP curves and $D \approx 20N$.** Lecture 9 derives the number three
  different ways, explains why the third disagreed, and shows the Epoch AI refit
  that resolved it ([compute-optimal scaling](compute-optimal-scaling.md)).
- **The inference caveat.** Lecture 1 notes it; Lecture 9 quantifies the trajectory
  from GPT-3's 3 tokens per parameter through Chinchilla's 20 to today's deliberate
  overtraining ([1:14:32]).

The one thing Lecture 1 does not prepare you for is
[upstream vs downstream](upstream-vs-downstream.md) — the finding that the
beautiful perplexity trend does not survive the move to benchmarks.

## Related

- [Efficiency](efficiency.md) — scaling laws as the efficiency principle applied
  to the experiment budget
- [Course map](course-map.md) — where this sits in the syllabus
- [Lecture 1](01-overview-tokenization.md) · [Lecture 9](09-scaling-laws.md)

## Sources

- [Lecture 1](01-overview-tokenization.md), scaling-laws preview [44:44]–[53:11]
- [Transformer hyperparameters](transformer-hyperparameters.md) — Lecture 3 uses
  two sweeps from Kaplan et al. 2020 for a different purpose than scaling: to show
  that the feedforward ratio and the aspect ratio each have a broad flat basin.
- [`lecture_01.py` transcription](../raw/slides/01-overview-tokenization.md#unit-3--scaling-laws-assignment-3)
- [Edited transcript](../raw/transcripts/01-overview-tokenization.md)

## The systems constraint: critical batch size

Lecture 8 introduces [critical batch size](critical-batch-size.md) not as an
optimisation topic but as the reason [data parallelism](data-parallelism.md) has a
ceiling ([29:07]):

> At a certain point there's something called the critical batch size, where the
> gain you get from an additional batch element is less than if you had taken
> another SGD step on that single element.

Below it, extra batch elements are worth as much as extra steps; above it they are
not, so "an infinitely large batch size is not infinitely better than infinitely
many single steps" ([29:53]). Since data parallelism spends batch size to buy
accelerators, this is a hard limit on how far it can scale, and it is what forces
the entire model-parallelism half of that lecture.

Lecture 8 closes by pointing here: "next week, I think, we're talking about scaling
laws" ([1:19:41]).
