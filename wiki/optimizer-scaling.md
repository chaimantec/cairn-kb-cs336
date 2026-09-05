# Optimizer scaling

**The optimizer you pick, and the hyperparameters you pick for it, are both scale-dependent
— and [lecture 11](11-scaling-laws-in-the-wild.md) argues that most published optimizer
comparisons are confounded by getting one or the other wrong.** The slide's own bottom line:
"optimization is a core part of LLMs (but also tricky, due to scale dependence!)"

This page covers the lecture's optimizer material: the comparison evidence, three problems
with how such comparisons are usually run, and Muon.

## What the comparisons show

Two figures recur through this part of the deck (slides
[38](../raw/slides/11-scaling-laws-in-the-wild.md),
[40](../raw/slides/11-scaling-laws-in-the-wild.md),
[43](../raw/slides/11-scaling-laws-in-the-wild.md)), and it is worth knowing they are the
same two figures, because the deck reuses them to make three different points.

**By wallclock, at very small scale** — a NanoGPT speedrun, validation loss against minutes,
five optimizers. All five runs are the same number of steps; what differs is time per step.
Final losses: SOAP\* 3.275 (25.7 min), Muon 3.276 (12.1 min), DistributedShampoo
UpdateFreq=10 3.285 (15.3 min), UpdateFreq=32 3.292 (13.2 min), Adam 3.34 (11.9 min). So the
matrix-based methods reach a materially better loss than Adam, and Muon reaches SOAP's loss
in less than half the time.

**By speedup over AdamW, against model size** — at 8× Chinchilla, four model sizes from 130M
to 1.2B. Every non-AdamW optimizer falls from a **1.18–1.41× speedup at 130M to 1.09–1.10×
at 1.2B**, and all three converge to nearly the same value at the top. The chart's own
annotation reads "Optimizers' speedup w.r.t. AdamW decreases with model size."

That annotation is directionally right and worth one qualification: the decrease is **not
monotone** for two of the three optimizers. SOAP dips to 1.236 at 300M and rises again to
1.289 at 520M; NAdamW dips to 1.060 and rises to 1.100 over the same interval. Only Muon
decreases at every step.

A second annotation, on the Chinchilla-ratio panel, claims "matrix-based optimizers (solid)
consistently outperform scalar-based optimizers (dashed)". The *ordering* does hold at all
four ratios — but the margin collapses from 0.015 at ratio 1 to 0.004–0.006 at ratios 2, 4
and 8, roughly a threefold shrink.

## Problem 1 — the hyperparameters are usually wrong

Slide [39](../raw/slides/11-scaling-laws-in-the-wild.md), drawing on *Fantastic Pretraining
Optimizers and Where to Find Them* (Wen, Hall, Ma and Liang — Stanford). "Different
optimizers can require different hyperparameters! (*and* likely have different optimal
hyperparameter scaling)."

The sharpest evidence is a comparison of AdamW **with itself**. At 130M parameters, AdamW at
learning rate $8\times10^{-3}$ reaches a given loss in about **2.25× fewer steps** than AdamW
at $6\times10^{-4}$. Same optimizer, same model, same data — only the learning rate differs,
by a factor of 13. Any comparison that gave one optimizer a tuned learning rate and another
an inherited one has measured the tuning, not the optimizer.

The second panel makes the same point with weight decay: Lion's optimum is at $wd \approx
0.6$, AdamW's at $wd = 0.1$ — **six times smaller** — and AdamW at its own optimum beats
anything Lion achieves anywhere in the sweep.

## Problem 2 — the advantage shrinks with scale

Slide [40](../raw/slides/11-scaling-laws-in-the-wild.md), and the practical instruction is
stated as a general rule rather than a finding about optimizers: "**always** check scaling
with respect to compute and chinchilla ratios. These are often major confounders to
performance!"

The evidence is the speedup-vs-model-size panel above: a roughly fourfold shrink in the
advantage over one decade of model size. An optimizer benchmarked at 130M and reported as a
1.4× speedup is a 1.1× speedup at 1.2B, and the trend gives no reason to expect it to stop
there.

## Problem 2.5 — a clean-looking scaling law can still blow up

Slide [41](../raw/slides/11-scaling-laws-in-the-wild.md) is the cautionary case, and it is
the most useful page in this section because the failure is invisible until it happens.

Seven IsoFLOP parabolas, one per compute budget, each with a clean minimum. The seven optima
fall on a fitted line to within ±0.16%. Then the held-out runs:

| Extrapolation | Result |
| --- | --- |
| $10^{21}$ FLOPs — one decade past the fit | 0.8% worse than predicted |
| $10^{22}$ FLOPs — two decades past | 2.5% worse |
| $10^{23}$ FLOPs — two and a half decades past | **run diverged**, about 24% worse |

The setup that produced it is named on the slide: "Cautious AdamC + Sqrt batch-size scaling
of learning rates", and it was "fixed with some more careful parametrization / scaling /
optimizer changes". The lesson is that the quality of a fit inside its own range tells you
very little about how far past it you can go — which is the same warning
[lecture 9](09-scaling-laws.md) attaches to scaling laws generally, here with a concrete
blow-up attached.

## Muon

**Muon is momentum SGD with one extra step: the momentum buffer is orthogonalized before it
is used as the update** (slide [42](../raw/slides/11-scaling-laws-in-the-wild.md)). It is an
optimizer for *matrix-valued* parameters.

$$B_t \leftarrow \mu B_{t-1} + G_t, \qquad O_t \leftarrow \mathrm{NewtonSchulz5}(B_t),
\qquad \theta_t \leftarrow \theta_{t-1} - \eta O_t$$

The Newton–Schulz iteration approximately maps $B_t = USV^{\top} \to UV^{\top}$ — that is,
it sets every singular value to 1, keeping only the directions. The deck spells the routine
"NewtonSchultz" in its prose and "NewtonSchulz5" in the pasted algorithm; the latter is the
implementation's name.

That construction connects directly to
[muP](maximal-update-parametrization.md): both are arguments about the **spectral norm** of
the update. muP asks what $\|\Delta W_l\|_*$ should be as width grows; Muon fixes the
update's singular values by construction.

**Does it work at scale?** Slide [43](../raw/slides/11-scaling-laws-in-the-wild.md) closes
with "scaling gains are tricky to measure, but clearly muon 'works' at scale", supported by
three panels — and they support it unevenly, which is worth knowing before citing the claim:

- The NanoGPT speedrun is labelled by the deck itself as "very small!"
- The scaling study is the panel that actually measures a gain, and it shows that gain
  *shrinking*, from 1.38× at 130M to 1.10× at 1.2B.
- The Kimi K2 panel is a single training curve with **no baseline at all** — it shows that a
  large Muon-trained run is stable and converges (falling to ~1.60 by 2T tokens, flat from
  about 5T to 11T, then dropping again to end near 1.33), which is evidence that Muon works,
  but not evidence that it works *better*.

So "works at scale" is well supported in the sense of *trains a frontier-scale model without falling over*. The stronger reading — that its advantage persists at scale — is the one the
centre panel actively argues against.

## See also

- [Maximal update parametrization](maximal-update-parametrization.md) — the other spectral-norm argument, and the one Lion breaks.
- [Step Law and hyperparameter scaling](step-law.md) — the systematic sweep over learning rate and batch size.
- [Critical batch size](critical-batch-size.md) · [Learning rate scaling and muP](learning-rate-scaling-and-mup.md) — the two hyperparameters you cannot inherit.
- [Training stability](training-stability.md) — divergence as a failure mode.
- [Lecture 11](11-scaling-laws-in-the-wild.md) · slides [38–43](../raw/slides/11-scaling-laws-in-the-wild.md)
