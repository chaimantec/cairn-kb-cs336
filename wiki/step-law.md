# Step Law and hyperparameter scaling

**Step Law is StepFun's attempt to answer, once and empirically, the question every recipe
in [lecture 11](11-scaling-laws-in-the-wild.md) answers differently: how should the learning
rate and batch size change as you scale?** Its method is brute force — grid-search the
(learning rate, batch size) plane at many model and data sizes, then fit the surface — and
its interest for a reader is less the winning formula than the three observations the sweep
produces along the way.

The paper is *Predictable Scale: Part I, Step Law — Optimal Hyperparameter Scaling*
(slide [32](../raw/slides/11-scaling-laws-in-the-wild.md)). The lecture places it as the
same approach DeepSeek and Qwen take, done more thoroughly.

## The problem: everyone publishes a different functional form

Slide [33](../raw/slides/11-scaling-laws-in-the-wild.md) collects the published laws in one
table, and the disagreement is not about coefficients — it is about **what the optimum is
even a function of**:

| Name | Learning rate | Batch size | Relative error |
| --- | --- | --- | --- |
| OpenAI Law | $3.239\times10^{-3} + (-1.395\times10^{-4})\log(N)$ | $2\times10^{18}\,\mathcal{L}^{-4.76190}$ | 9.51‰ |
| Microsoft Law | $1.3192\times10^{-5}\, N^{-0.23} D^{-0.32}$ | — | 9.25‰ |
| DeepSeek Law | $0.3188\, C^{-0.1250}$ | $0.2920\, C^{0.3271}$ | 9.26‰ |
| Porian Law | $3.7\, N^{-0.36}$ | $0.7576\, N^{0.703}$ | 3.71‰ |
| MiniCPM Law | — | $2\times10^{18} / L^{6.24}$ | — |
| MeiTuan Law | $\lambda \mathcal{L}^{-\alpha}$ | — | — |
| **Step Law** | *(the paper's own)* | | **0.94‰** |

The slide's own framing of the disagreement: some treat batch size as a function of **loss**
(the [critical batch size](critical-batch-size.md) view, OpenAI), some as a **polynomial
function of compute** (DeepSeek) — "or something else?" Note that the arguments differ
across rows: $N$ alone, $N$ and $D$, compute $C$, or loss $\mathcal{L}$.

Two columns in the source table record whether each law accounts for the **data recipe** and
for **model sparsity**; only Step Law claims both. Its relative error of 0.94‰ is about four
times better than the next-best entry and ten times better than the OpenAI, Microsoft and
DeepSeek laws — though a fit's own paper reporting its own fit as best is worth reading with
the usual caution.

## The method: map the surface rather than assume its shape

"Much like the DeepSeek paper — train models to try to map out the hparam space" (slide
[34](../raw/slides/11-scaling-laws-in-the-wild.md)). The sweep covers 18 model
configurations built from five distinct architectures, each reused at several dataset sizes.

The resulting contour map of training loss over (learning rate, batch size) is the clearest
single picture in the lecture of what "picking hyperparameters" actually means. The contours
are labelled by how far above the global minimum they sit — +0.125%, +0.250%, +0.500%,
+1.000%, +2.000% — and they are **elongated along a diagonal**: higher learning rate goes
with larger batch size, which is the same coupling that makes
[critical batch size](critical-batch-size.md) a moving target.

Two things to read off it. Step Law's predicted optimum lands **inside the innermost
+0.125% contour**, essentially on the empirical global minimum. And the OpenAI and Microsoft
laws appear as **vertical lines** — they predict a learning rate with no batch-size
dependence at all — sitting about a factor of four below the optimal learning rate.

## Observation 1 — the loss surface is convex in both

"For pre-training losses, minimizers for LR/batch can be cleanly identified" (slide
[35](../raw/slides/11-scaling-laws-in-the-wild.md)). Eight one-dimensional slices through
the smoothed surface — four holding the learning rate fixed and sweeping batch size, four
the reverse — are each single-minimum and U-shaped.

This is the observation everything else rests on. If the surface were multi-modal or flat
over a wide plateau, "the optimal learning rate" would not be a well-defined thing to fit a
scaling law to.

## Observation 2 — batch size depends on data, learning rate on both

The more actionable finding (slide [36](../raw/slides/11-scaling-laws-in-the-wild.md)):
**"batch is primarily dependent on dataset size."** In the published figure, the optimal
batch size for seven different model sizes collapses onto a *single* curve in $D$, while the
optimal learning rate needs both $N$ and $D$ — its seven fit lines stay stacked in strict
order of model size, largest lowest.

The lecture immediately qualifies the learning-rate half. Step Law also finds a **higher**
optimal learning rate as $D$ grows at fixed model size, and the slide flags this as "likely
more fragile if swapping to WSD", citing the InternLM scaling-law paper. Since
[WSD schedules](wsd-schedules.md) are what both headline recipes in this lecture use to make
their sweeps affordable, that caveat has teeth.

## Observation 3 (?) — robustness, with the question mark

The heading's own parenthetical question mark is the lecturer's, and it is well placed
(slide [37](../raw/slides/11-scaling-laws-in-the-wild.md)). Two generalization tests:

- **To MoEs** — loss landscapes at sparsity ratios $N_a/N$ from 0.09 to 0.58. Step Law's
  prediction lands on or beside the empirical optimum in each.
- **To other datasets** — landscapes for a bilingual corpus, a code-integrated mixture and a
  code-dominant one.

The second test is the one that shows the limit. Step Law's prediction is *fixed* at the
same point in all three panels, because the model and token budget do not change and only
the mixture does — but the **observed** optimum moves with the mixture, dropping from
$1.96\times10^{-3}$ to $9.8\times10^{-4}$ for the code-dominant data. A law with a data-recipe
term that nevertheless predicts the same optimum across three recipes is being tested
against exactly the variation it claims to handle.

## See also

- [Critical batch size](critical-batch-size.md) — the "batch as a function of loss" view this table contrasts against.
- [Learning rate scaling and muP](learning-rate-scaling-and-mup.md) — the other philosophy: make the optimum not move, rather than predicting where it moves to.
- [Published scaling recipes](published-scaling-recipes.md) — DeepSeek's and MiniCPM's own fits, which this study is measuring itself against.
- [Optimizer scaling](optimizer-scaling.md) — the same scale-dependence problem, for the optimizer itself.
- [Lecture 11](11-scaling-laws-in-the-wild.md) · slides [32–37](../raw/slides/11-scaling-laws-in-the-wild.md)
