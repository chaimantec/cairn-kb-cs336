# Published scaling recipes

[Lecture 11](11-scaling-laws-in-the-wild.md) is built around a question the theory in
[lecture 9](09-scaling-laws.md) cannot answer on its own: **when people actually train
frontier-scale open models, does the Chinchilla-style approach survive contact with
practice?** The lecture answers it by reading the published recipes of eight model families
off their own figures.

Two are treated in depth because they take genuinely different approaches to the same
questions — MiniCPM and DeepSeek. The rest are a rapid tour.

The lecturer's framing of the corpus is worth keeping: since the classical canon (Kaplan,
Hestness, Chinchilla, ~2022) the number of detailed scaling papers has *decreased*, and most
of the recent ones come out of the open-source community in China.

## The two headline recipes, side by side

Slide [30](../raw/slides/11-scaling-laws-in-the-wild.md) states them compactly, and the
contrast is the lecture's spine:

| | **DeepSeek** | **MiniCPM** |
| --- | --- | --- |
| Architecture hyperparameters | Assume most transformer hypers are invariant to scale | Use [muP](maximal-update-parametrization.md) to make transformer + LR invariant to scale |
| Batch size and learning rate | Fit a **scaling analysis** on batch/LR to find optimal scaling | muP is supposed to make the LR transfer, so no fit needed |
| Model sizing | **IsoFLOP** analysis ([Chinchilla method 2](isoflop-method.md)) | Chinchilla methods 1 (lower envelope) and 3 (joint fit) |
| Making the sweep affordable | Piecewise-linear ([WSD-style](wsd-schedules.md)) schedule | Piecewise-linear schedule, to sample for method 3 |

Both reach for the same trick on the last row — [WSD schedules](wsd-schedules.md) — because
both face the same $n^2$ cost problem in fitting a scaling law. Where they part company is
the second row, and that is the choice a practitioner actually has to make: **trust a
parametrization to transfer your hyperparameters, or measure how they scale and
extrapolate.**

## MiniCPM

A 2024 high-performance small language model, and — the lecturer's reason for spending time
on it — an unusually detailed public account of how its hyperparameters were chosen.

**The recipe** (slide [9](../raw/slides/11-scaling-laws-in-the-wild.md)): "Use muP for
initialization, fix the aspect ratio, scale up the overall model size." The published
configuration table bears the aspect-ratio claim out — across all seven model sizes in the
sweep, $d_{ff}$ is exactly $2.5 \times d_m$ and the head dimension $d_h$ is 64 in every row.
The concrete muP-related constants are printed on slide
[8](../raw/slides/11-scaling-laws-in-the-wild.md): `scale_emb = 12, scale_depth = 1.4,
init_std = 0.1, lr = 0.01`.

Two caveats the slides themselves raise. The gap between the largest model in the scaling
sweep and the model actually trained is about **5×**, so the recipe is an extrapolation.
And optimal batch, LR and token-to-size ratios are "directly fitted via scaling analysis"
even here.

**Does muP deliver a stable learning rate?** Slide
[10](../raw/slides/11-scaling-laws-in-the-wild.md) asks exactly that, and the answer is
broadly yes: across five model sizes from 0.04B to 2.1B, the loss-minimising learning rate
stays in the narrow band $6\times10^{-3}$ to $10^{-2}$ — consistent with the `lr = 0.01`
above. The large vertical spread between the curves is a model-size effect on the achieved
loss, not a shift in where the minimum sits. The paper's own caption claims the shift
"becomes minimal".

**The Chinchilla analysis.** MiniCPM uses methods 1 and 3 (slide
[16](../raw/slides/11-scaling-laws-in-the-wild.md)), fitting

$$L(N,D) = C_N N^{-\alpha} + C_D D^{-\beta} + L_0$$

with `scipy.curve_fit`, then deriving the optimal ratio at fixed compute $C = 6ND$. Method 1
(slide [17](../raw/slides/11-scaling-laws-in-the-wild.md)) gives "fairly clear (though maybe
not linear?)" trends and "relatively low diminishing returns due to data".

**The striking result is method 3** (slide
[18](../raw/slides/11-scaling-laws-in-the-wild.md)). The joint fit comes out as

$$\frac{7.54\times10^{-2}}{N^{0.30}} + \frac{2.92\times10^{-1}}{D^{0.30}} + 0.25$$

with both exponents equal to 0.30 and an irreducible loss of 0.25 — and an optimal
data-to-parameter ratio at $C = 10^{21}$ of **95.60 tokens per parameter**, roughly $4.8\times$
Chinchilla's ~20:1. The slide's own text calls these "*very* high data-model ratios". Note
also the fitted $\eta = -0.00$: on this fit the optimal ratio does not change with compute
at all.

## DeepSeek

The other detailed study, and the lecture's example of the opposite philosophy — **do not
rely on muP; measure how the hyperparameters scale and fit power laws to them.**

**Batch and learning rate** (slide [21](../raw/slides/11-scaling-laws-in-the-wild.md)). Run
small-scale sweeps, collect the models within 0.25% of the minimum loss, and fit:

$$\eta_{\mathrm{opt}} = 0.3118 \cdot C^{-0.1250} \qquad B_{\mathrm{opt}} = 0.2920 \cdot C^{0.3271}$$

The lecture does not take this at face value — the slide's own comment is that the
"learning rate fit looks a bit questionable", and the figure shows why: the evidence covers
barely 2.5 decades of compute and only four quantised learning-rate values, with very little
visible slope, and the fitted line is then extrapolated another five decades out to where
the 7B and 67B production models sit.

**Schedule** (slide [22](../raw/slides/11-scaling-laws-in-the-wild.md)): a multi-step
WSD-style curve — peak after 2000 warmup steps, down to 31.6% of maximum after 80% of tokens,
down to 10% after 90%. See [WSD schedules](wsd-schedules.md).

**Model sizing** (slide [23](../raw/slides/11-scaling-laws-in-the-wild.md)): a straightforward
[IsoFLOP](isoflop-method.md) analysis over eight compute budgets from $10^{17}$ to
$3\times10^{20}$, giving

$$M \propto C^{0.51} \qquad D \propto C^{0.48}$$

i.e. compute split almost evenly between model and data — close to Chinchilla's own
conclusion, and reached independently. One notational trap: **this deck reports model size as
FLOPs per token $M$, not as a parameter count.**

The final loss prediction (slide [24](../raw/slides/11-scaling-laws-in-the-wild.md)) is the
payoff — the fitted curve predicts the 7B and 67B models' achieved loss closely, though not
exactly: the 7B star sits about 0.019 bits-per-byte *above* the fit and the 67B about 0.016
*below* it. "Well-predicted" here means within about 0.02 bpb, not "on the line".

## The rapid tour

Slide [30](../raw/slides/11-scaling-laws-in-the-wild.md) files these as "recent (late 2024+)
but less detailed":

- **Qwen** — LR and batch scaling fits, with very few details published (slide [25](../raw/slides/11-scaling-laws-in-the-wild.md)).
- **Kimi K2** — [MoE](mixture-of-experts.md) sparsity scaling and attention-head scaling (slide [26](../raw/slides/11-scaling-laws-in-the-wild.md)).
- **Hunyuan** (slide [27](../raw/slides/11-scaling-laws-in-the-wild.md)) and **LLaMA 3** (slide [28](../raw/slides/11-scaling-laws-in-the-wild.md)) — "just IsoFLOPs (no other scaling details)".
- **MiniMax-01** — architecture choice and decision scaling; its published figure covers models from 70M to 7B (slide [29](../raw/slides/11-scaling-laws-in-the-wild.md)).

The thinness of this list is itself the finding. Four of the six most recent frontier-scale
releases publish little more than an IsoFLOP curve.

## See also

- [Compute-optimal scaling](compute-optimal-scaling.md) — the Chinchilla question these recipes are all answering.
- [IsoFLOP method](isoflop-method.md) — Chinchilla method 2, the one most of these use.
- [Maximal update parametrization](maximal-update-parametrization.md) — MiniCPM's route.
- [WSD schedules](wsd-schedules.md) — how both headline recipes make the sweep affordable.
- [Step Law and hyperparameter scaling](step-law.md) — the study that tries to unify all of these fits.
- [Scaling laws](scaling-laws.md) · [Lecture 9](09-scaling-laws.md) for the theory these test.
- [Lecture 11](11-scaling-laws-in-the-wild.md) · slides [3–30](../raw/slides/11-scaling-laws-in-the-wild.md)
