# Data scaling laws

The simplest scaling law, and the one [lecture 9](09-scaling-laws.md) uses to
build intuition for all the others: fix the training procedure, grow the dataset,
and watch the loss fall on a straight line in log-log space.

## The setup

Three conditions ([13:04]):

1. **Fix the model training procedure.** This is a univariate relationship — only
   data varies.
2. **Keep the model larger than the dataset.** Otherwise you leave the power-law
   regime and hit the irreducible-error floor, where extra data buys nothing
   because you have already fitted the data as well as your model class allows
   ([20:46]). Asked how much larger, Hashimoto's answer is loose on purpose: "it
   can be basically any amount larger than the data, as long as it's in this power
   law regime — usually, 10 times bigger than the data will do it."
3. **Tune hyperparameters**, or the relationship will not even be monotone
   ([13:50]).

What you expect in the abstract is a sigmoid: random guessing at one end, the
task's entropy floor at the other ([13:50]). What you *measure*, in the middle, is
a line.

## The observation

Plot log dataset size against log test loss and the points are strikingly linear
([14:35], slide 15, taken from Kaplan et al. 2020). Assignment 3 has you reproduce
something like it yourself.

A line on a log-log plot carries two pieces of information ([15:22]):

- Error is decaying **polynomially**, not exponentially.
- You are **far from the asymptote**. Approaching the floor would bend the curve.

## Why polynomial? The mean-estimation argument

Deliberately the simplest possible case ([16:09]). Draw
$x_1,\dots,x_n \sim N(\mu, \sigma^2)$, estimate $\mu$ by the empirical mean
$\hat\mu$. Then

$$\mathbb{E}\,\|\hat\mu - \mu\|^2 = \frac{\sigma^2}{n}$$

Take logs: $\log(\text{error}) = \log \sigma^2 - \log n$. That is a scaling law
with slope $-1$.

The general form ([16:09]) — anything that looks like

$$\text{error} \;\approx\; \frac{1}{n^{\alpha}} + c$$

becomes a straight line once you subtract the constant $c$. So **polynomial decay
in error is exactly what produces a scaling law**, and classical parametric
estimation — means, linear regression — gives $\alpha = 1$ as a matter of course
([16:54]). If neural scaling worked like classical statistics, every one of these
plots would have slope $-1$.

## The exponent mystery

They do not. Reading off the Hestness and Kaplan fits, the exponents are roughly
$-0.1$, $-0.3$, $-0.1$ ([17:41], slide 18) — still polynomial, but "much, much
slower convergence" than estimating a mean.

The resolution offered is **non-parametric** estimation ([18:28]). Suppose you want
to estimate an arbitrary smooth function rather than a finite-dimensional
parameter. Cut the unit cube into boxes of side $n^{-1/4}$; in the 2-D case you get
$\sqrt{n}$ boxes with $\sqrt{n}$ samples each and error around $n^{-1/2}$. In $D$
dimensions the same argument gives

$$\text{error} \;\approx\; n^{-1/D}$$

so the log-log slope is $-1/D$ rather than $-1$. Read backwards, an exponent near
$-0.1$ says the network learns at about the rate of **a non-parametric estimator in
ten dimensions** ([19:16]) — a nearest-neighbour method, say. That is a genuinely
informative thing to recover from a curve fit.

### How much to believe it

Bahri et al. and other scaling-law theorists push the claim harder, arguing the
exponents really do reveal non-parametric smoothing over the data's intrinsic
dimension ([19:16]). Hashimoto is explicitly non-committal: "I don't quite know how
much I truly buy this argument. Some of the evidence might be a little sketchy — it
relies on estimators of intrinsic dimension" ([20:01]).

Worth keeping as intuition, not as a result.

## Slopes move less than you expect

The single most reused fact in the lecture. Thinking of data scaling laws as
empirical generalization bounds suggests that **dataset composition sets the
intercept while the model class sets the slope** ([22:19]).

The consequences turn up everywhere:

- Data mixtures change the offset — which is why picking the best mixture at small
  scale tends to be enough ([24:38], see [data mixture selection](data-mixture-selection.md)).
- Regularisation and ensembling under [infinite compute](data-repetition.md) improve
  performance while leaving the slopes "surprisingly similar" ([26:55]).
- Even Adam versus SGD moves the intercept and not the slope ([34:35]).

Hashimoto states it as a warning to anyone about to fit their own: "very often your
slopes don't change, often the interventions you do just change the intercept"
([26:55]). If your intervention appears to change a slope, that is the surprising
result and deserves scrutiny.

## What they are and are not good for

Good for: forecasting how fast a model learns from data.

Not much else on their own ([21:33]). The engineering questions — what mixture,
whether to repeat, what to filter — need the composition results above, not the
univariate law. And extrapolating a *loss* curve says nothing directly about
downstream behaviour; see [upstream vs downstream](upstream-vs-downstream.md).

## A caution about narrow ranges

Raised in response to a student question about a chart that looked linear on
seemingly linear axes ([29:14]). Two separate points, both worth carrying:

- The axis in question was doubling — log scale in disguise — and its y-range was
  so narrow that linear and log are visually identical anyway ([29:14]).
- More generally, over a small slice of compute you **cannot distinguish polynomial
  from exponential scaling**, "because Taylor approximations are a thing,
  everything looks linear if you zoom in enough" ([29:59]).

## See also

- [Lecture 9](09-scaling-laws.md) — where this is developed.
- [Compute-optimal scaling](compute-optimal-scaling.md) — the joint model-and-data version.
- [Scaling law methodology](scaling-law-methodology.md) — why the regularity is engineered.
- [Data repetition](data-repetition.md) · [Data mixture selection](data-mixture-selection.md)
- Slides [15](../raw/slides/09-scaling-laws.md), [16–20](../raw/slides/09-scaling-laws.md) · [transcript](../raw/transcripts/09-scaling-laws.md)
