# Learning rate scaling and muP

One of the two hyperparameters [lecture 9](09-scaling-laws.md) says you cannot
inherit from someone else's recipe. Most architecture choices you copy — "you're
probably not radical enough to switch to an LSTM" ([40:42]) — but batch size and
learning rate you have to work out for your own run ([41:27]).

This page covers the learning-rate half; the batch-size half is
[critical batch size](critical-batch-size.md). Lecture 9 previews both and defers
the detail: "I'll go into much more detail in the advanced scaling lecture"
([48:23]).

## Why the optimum moves

The mental picture is width scaling on a plain MLP, holding depth fixed ([48:23]):

> If I'm doing width scaling, the bigger my model, the smaller my learning rate
> should be, and the reason my learning rate should be smaller is because I have
> bigger, more parameters, and I'm changing more things at once — maybe I should
> move less.

The corresponding rule of thumb is to scale the learning rate by
$1/\text{width}$ ([49:08]) — decrease it regularly as width grows.

Slide 40's left panel shows the problem it solves. Under **standard practice**,
each model width has its own loss-versus-$\log_2(\text{LR})$ curve, each descends
to its own minimum and then blows up almost vertically past it, and — the point —
**the minima are staggered**: the widest model's optimum sits furthest left, around
$\log_2(\text{LR}) \approx -15$, with narrower models' optima progressively further
right, out to about $-10$. The slide labels this "optimum shifts".

That is fatal for the small-scale-tuning programme. If the best learning rate at
your experiment scale is not the best learning rate at your target scale, tuning
small tells you nothing directly.

## The two philosophies

Both are in use, and the lecture presents them as genuinely competing ([49:54]):

1. **Predict where the minimum goes.** Estimate the minima at several scales, fit
   how they move, and extrapolate. "The way the minimum changes is pretty
   predictable, so this isn't a crazy idea." This is the scaling-law approach
   applied to the learning rate itself.
2. **Make the minimum stop moving.** Reparameterise the network — change
   initialisation scales, and change the optimiser's step sizes for different parts
   of the network — so that the loss-versus-learning-rate curve has its minimum in
   the same place at every width. Then tune once, small, and transfer. This is
   **muP** and its relatives.

Slide 40's right panel is muP working: all seven widths descend along nearly the
same path and share a broad minimum around $\log_2(\text{LR}) \approx -10$ to
$-11$. The annotation reads "optimum stable".

The slide pairs the chart with the muP scaling table from Yao et al. 2024, which
gives the concrete rules for a model $M'$ that is $r$ times the width of $M$:
matrix-like parameter tensors get their AdamW learning rate and initialisation
variance divided by $r$, while "others" — embeddings included — keep theirs; the
output multiplier is divided by $r$.

## Which one to use

Deliberately unresolved ([49:54]):

> Some people have reported great success with these approaches, others have
> reported less success… Both have been applied successfully at scale — there are
> large-scale training runs that have used both approaches.

The only tilt offered is anecdotal: "it does seem like more people are favoring the
scaling law approach, but both are certainly viable" ([50:41]).

## A warning about the underlying measurements

Worth carrying over from the discussion of variance ([54:30]). Perplexity scaling
points are clean enough that people fit them from single runs. **Learning-rate
scaling laws are not** — "if you're doing something like, say, learning rate or
critical batch size scaling laws, you will see some truly horrendous stuff."
Variance reduction there is less common than it should be.

So the minima you are fitting in philosophy 1 are noisier objects than the loss
curves elsewhere in this lecture, which is an argument for philosophy 2 that the
lecture does not quite make explicitly.

## See also

- [Critical batch size](critical-batch-size.md) — the other hyperparameter you must re-tune, and the one this interacts with.
- [Transformer hyperparameters](transformer-hyperparameters.md) — the ones that *are* roughly scale-invariant.
- [Scaling law methodology](scaling-law-methodology.md) — hyperparameter transfer as a design goal.
- [Lecture 9](09-scaling-laws.md) · slide [40](../raw/slides/09-scaling-laws.md) · [transcript](../raw/transcripts/09-scaling-laws.md)
