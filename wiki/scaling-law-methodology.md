# Scaling law methodology

The part of [lecture 9](09-scaling-laws.md) that is about *how to do this
properly* rather than about any particular law. Its thesis, stated flatly: scaling
laws are not a natural phenomenon you observe, they are a property of a recipe you
construct.

## Predictability is engineered

The central claim ([38:26]):

> Scaling laws aren't magic. Percy likes to make this point, that scaling laws — and
> this kind of predictability across scales — is engineered, it doesn't happen
> automatically. We need to pick the right x-axes to look at, we need to make sure
> the hyperparameters are set right, and only under those conditions does it become
> possible to get predictable scaling across many orders of magnitude of compute.

This is the same point [lecture 1](01-overview-tokenization.md) makes as "scaling
laws are not laws of nature. They don't just happen automatically. You kind of have
to will them into existence" ([47:47] of that lecture), arrived at from the other
direction — lecture 1 asserts it, lecture 9 demonstrates it by showing three
separate ways the Kaplan fit went wrong.

"This is kind of what a lot of scaling law researchers do — they think about the
right way to get very predictable scaling" ([38:26]).

## A scaling law is a lower bound on a recipe

The formulation to carry away ([1:09:54]):

> Scaling laws are kind of lower bounds, in some sense — they're saying, "if I
> continue this recipe and scale it up, this is what I'll get." But if you're
> scaling up a recipe you don't actually want to scale up — your warm-up is crazy,
> or your batch sizes are crazy — you're going to get bad scaling laws.

So the fitted line inherits every defect of the runs underneath it. The practical
instruction: stay as close to a proper, full-quality run as you can at every point
you fit. Kaplan's small models had not converged past learning-rate warm-up, and a
fixed large batch size was wrong for them — neither is a modelling error, both are
recipe defects, and both moved the exponent
([compute-optimal scaling](compute-optimal-scaling.md)).

## Choose the right x-axis

The most consequential-looking piece of housekeeping in the whole lecture. Kaplan
found that including embedding parameters produced "very funky-looking scaling
laws" and so counted only non-embedding parameters ([37:40]) — reasonable, and the
sort of decision nobody would flag in review.

It is one of the three things that put Kaplan and Chinchilla a factor of several
apart ([1:09:08]). Hashimoto's framing of the general problem: "not all parameters
are created equal, and because not all parameters are created equal, your scaling
laws may look good or bad depending on how you define what a parameter is"
([37:40]).

The same question resurfaces in a new guise for [MoE](mixture-of-experts.md), where
total and active parameters diverge and you must decide which one the law is about
([39:12]).

### Scale-invariant quantities

The constructive version of the same idea ([36:07]). Some quantities should be
expected to move with scale and some should not:

- **Number of layers is not scale-invariant** — bigger models genuinely want more
  layers.
- **Aspect ratio $d_{model}/n_{layer}$ roughly is** — the minima sit near 100 across
  model sizes, drifting only slightly toward smaller ratios for deeper models
  ([36:54]).

Finding a scale-invariant parameterisation is what makes "fix this and scale up"
a defensible strategy: "you can make plots like this and convince yourself you're
probably good, because your optimum isn't shifting too much as you go to larger and
larger models" ([36:54]). Where no such parameterisation exists naturally, muP
manufactures one — see
[learning rate scaling and muP](learning-rate-scaling-and-mup.md).

## Run more than one estimator

Chinchilla's use of three independent methods is held up as the right instinct:
"a way of robustifying yourself against modeling assumptions you may have made"
([1:02:12]). Two of the three agreeing closely, and the third disagreeing, is
exactly the signal such a design is supposed to produce — and the disagreement
turned out to be a fitting error rather than a real difference ([1:13:00]).

## Know which measurements are clean

Variance is not uniform across the quantities you might scale ([53:44]–[54:30]):

- **Perplexity** is clean enough that essentially all published points are single
  runs; reruns differ in the second decimal place, because the data is homogeneous
  and the eval sets are large.
- **Learning-rate and critical-batch-size** scaling laws are noisy — "you will see
  some truly horrendous stuff" — and variance reduction is "not as common as it
  should be".

The corresponding discipline is to **establish regularity where the signal is
clean, then argue about transfer separately** ([55:15]), rather than fitting a law
directly to whatever you ultimately care about. See
[upstream vs downstream](upstream-vs-downstream.md).

## Beware short compute ranges

Two related traps ([29:14]–[29:59]):

- **Check the axes.** A chart that looks linear may have a doubling x-axis, and a
  y-range narrow enough that linear and log are visually indistinguishable.
- **Over a narrow range, functional forms are unidentifiable.** "It's very, very
  difficult to tell if something is scaling polynomially or if it's scaling
  exponentially — because Taylor approximations are a thing, everything looks
  linear if you zoom in enough." So "you always want to be a little careful and
  skeptical" about a form fitted over a small slice.

This is also part of why Kaplan was fragile: operating at a much lower compute
scale made the result sensitive to small changes ([1:11:27]).

## Interventions move intercepts, not slopes

The empirical regularity that recurs throughout, and a useful prior when reading
anyone's scaling plot ([26:55], [34:35]):

> It's rare to get different slopes, even with an intervention as big as SGD versus
> Adam — if you were the one training with SGD or Adam, you'd think that's a huge
> change, and yet the scaling trends remain roughly the same. ([35:21])

Hashimoto says he is "surprised by this every time I see it, and yet it's very
true". Two consequences:

- A claimed **slope** change is a strong claim and should be scrutinised. It is also
  the only kind of change that matters asymptotically — a worse slope means your
  architecture eventually loses no matter what ([32:16]).
- If slopes really are fixed, small-scale rankings transfer, and you may not need to
  fit a law at all — see [data mixture selection](data-mixture-selection.md).

## The paradigm, stated

The attitude the method produces, for better and worse ([34:35]):

> A lot of people use scaling laws as almost a paradigmatic way of saying, "if it
> doesn't show up in the scaling law, it's not a good intervention."

And on the culture around it ([2:23]): "if you talk to some people who are really in
these big labs doing scaling work, it's almost kind of a way of life — they really
believe in the scaling laws. It's almost a belief, and you'll see why scaling laws
can sometimes be quite tricky objects."

## See also

- [Compute-optimal scaling](compute-optimal-scaling.md) — the worked example of methodology mattering.
- [The IsoFLOP method](isoflop-method.md) — the most robust of the estimators.
- [Upstream vs downstream](upstream-vs-downstream.md) — the limit of the paradigm.
- [Data scaling laws](data-scaling-laws.md) · [Lecture 9](09-scaling-laws.md)
- [Scaling laws](scaling-laws.md) — the hub.
