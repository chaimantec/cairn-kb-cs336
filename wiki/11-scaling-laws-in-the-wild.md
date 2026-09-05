# Lecture 11 — Scaling laws in the wild

The second of CS336's two scaling-law lectures, delivered by Tatsunori Hashimoto with
[inference](10-inference.md) in between. Where
[lecture 9](09-scaling-laws.md) derives the theory — Kaplan, Chinchilla, the
[IsoFLOP method](isoflop-method.md) — this one asks whether any of it survives contact
with the people actually training frontier-scale models, and then spends its second half
on the two hyperparameters lecture 9 said you can never inherit.

The lecturer's own description is "a bit of a grab bag" ([0:05]), which undersells it. The
lecture has a clear spine, and it is the one printed on its last slide: three challenges in
scaling in practice, and three solutions.

## What this lecture establishes

**The classical canon stops around 2022, and what replaced it is thinner than you would
expect.** Since Chinchilla the number of detailed public scaling papers has *decreased*, and
most of the recent ones come out of the open-source community in China ([1:36]). Four of the
six most recent frontier releases publish little more than an IsoFLOP curve.

**There are two coherent strategies for hyperparameters at scale, and the lecture presents
them as a genuine fork** ([22:19], [30:48]). Either you make the optimum stop moving — the
[muP](maximal-update-parametrization.md) route, taken by MiniCPM — or you accept that it
moves and fit a scaling law to *where it moves to*, the route taken by DeepSeek, Qwen and
StepFun. Both are in production use.

**Fitting a scaling law honestly is expensive, and the fix is a schedule change.** Because a
cosine schedule's shape depends on the token budget declared at the start, every point on a
scaling curve needs its own run from scratch, which makes the fit cost $n^2$.
[WSD schedules](wsd-schedules.md) let every budget branch off one stable trunk instead.

**Scale-dependence is the standing confounder, and it is not resolved.** Optimizer advantages
shrink with model size, published hyperparameters are routinely mistuned, and a scaling law
whose fit looks flawless can still blow up two decades out.

## Part 1 — MiniCPM: make the optimum stop moving

MiniCPM is a 2024 small language model, and the lecturer's reason for spending time on it is
the unusual completeness of its public account. At release it was "basically state of the art
for the 1-to-2-billion-parameter bracket" ([4:39]), and even in 2026 much of it still reads
as current ([3:53]).

### muP, and the honest gap in the ladder

MiniCPM's first technique is muP, "a special class of initializations" whose whole point is
that the optimal learning rate should not move as you scale ([5:24]). The deck prints the
concrete constants — `scale_emb = 12, scale_depth = 1.4, init_std = 0.1, lr = 0.01` (slide
[8](../raw/slides/11-scaling-laws-in-the-wild.md)) — and the recipe as a sentence: "use muP
for initialization, fix the aspect ratio, scale up the overall model size" (slide
[9](../raw/slides/11-scaling-laws-in-the-wild.md)). The fixed aspect ratio is visible in
their configuration table: $d_{ff} = 2.5\,d_m$ and head dimension 64 in every row of the
sweep.

Two caveats the lecture raises rather than glosses. The scaling ladder's largest model is
about **5× smaller** than the model actually released ([6:56]), so the recipe is an
extrapolation. And per-parameter learning rates are unfamiliar enough to be a genuine
implementation hazard ([6:10]).

**Does the optimum actually stay put?** The lecture's verdict is that it does, and cleanly:
"it's very, very clean in their experiments" ([7:43]). Across five model sizes from 0.04B to
2.1B the loss-minimising learning rate stays inside $6\times10^{-3}$ to $10^{-2}$ (slide
[10](../raw/slides/11-scaling-laws-in-the-wild.md)) — consistent with the `lr = 0.01` above.
The large vertical spread between those curves is a model-size effect on achieved loss, not
a shift in where the minimum sits.

Batch size is a separate matter, and muP does not fix it: the optimal batch size "does have a
power law structure with respect to the target loss" ([9:14]), which is the
[critical batch size](critical-batch-size.md) picture from lecture 9, and MiniCPM fits it
directly (slide [12](../raw/slides/11-scaling-laws-in-the-wild.md)).

### The $n^2$ problem, and WSD

Here is the expensive part of any Chinchilla-style analysis ([10:00]). To fit a scaling law
you need a properly-trained model at each token budget — and with a cosine schedule you
cannot get one by stopping a longer run early. "You can't restart from the end of a 4 million
sequence model — you have to restart from scratch" ([10:46]).

Slide [13](../raw/slides/11-scaling-laws-in-the-wild.md) demonstrates why with a cosine
cycle-length ablation, and states the cost in one line: this "turns the cost of fitting a
scaling law from $n$ to $n^2$".

**WSD is the fix.** Split the schedule into warmup, a long stable phase at constant learning
rate, and a short decay ([11:31]). Then, for each budget you want on your scaling curve, take
a checkpoint from the stable trunk and "run the stable learning rate forward and re-decay the
model down" ([12:17]). One trunk, many endpoints, linear cost. See
[WSD schedules](wsd-schedules.md) for the shape and the evidence.

The lecturer's aside here is worth keeping, because it is the kind of thing that only comes
from having run these: it is "very cool, though, to see just how much of an impact learning rate
decay has, if you haven't played with that before" ([13:49]).

### The Chinchilla analysis, and a number worth arguing about

MiniCPM uses Chinchilla methods 1 and 3, which the lecturer flags as "the least reliable of
the Chinchilla methods" ([14:34]) — lecture 9 having spent real time on why method 3 in
particular is fragile.

The joint fit (slide [18](../raw/slides/11-scaling-laws-in-the-wild.md)) comes out as

$$L(N,D) = \frac{7.54\times10^{-2}}{N^{0.30}} + \frac{2.92\times10^{-1}}{D^{0.30}} + 0.25$$

and yields an optimal ratio of **95.60 tokens per parameter** at $C = 10^{21}$ — roughly
$4.8\times$ Chinchilla's ~20:1, which the slide itself calls "*very* high data-model ratios".
Note also the fitted compute-exponent of $-0.00$: on this fit the optimal ratio does not
change with compute at all.

Set that against DeepSeek's answer below, which is nearly Chinchilla's. Two serious teams,
the same question, very different numbers — and the lecture does not adjudicate between them.

## Part 2 — DeepSeek: fit a scaling law to the optimum instead

DeepSeek is the other detailed study and the other philosophy. By 2024 "every open model
builder was replicating the entire" scaling pipeline ([20:01]), and DeepSeek's is the most
thorough.

**Batch and learning rate.** Sweep at small scale, keep the runs within 0.25% of the minimum,
mark the optimum with a star, and repeat across scales ([17:37]); then fit a line through the
stars to predict the optimum at the scale you care about ([16:51], [18:25]). The published
fits are

$$\eta_{\mathrm{opt}} = 0.3118\, C^{-0.1250} \qquad B_{\mathrm{opt}} = 0.2920\, C^{0.3271}$$

**The lecture does not accept the learning-rate half.** "This is the one drawback of the
learning rate approach" ([19:11]), and the slide's own comment is that the fit "looks a bit
questionable" (slide [21](../raw/slides/11-scaling-laws-in-the-wild.md)). The figure shows
why: barely 2.5 decades of evidence, only four quantised learning-rate values, very little
visible slope — and the line then extrapolated five decades out to where the 7B and 67B
production models sit.

**Model sizing** uses IsoFLOPs, and the lecturer prefers it: DeepSeek gets "nicer scaling
curves than MiniCPM, because of their choice to go with IsoFLOPs" ([20:48]). Eight budgets
from $10^{17}$ to $3\times10^{20}$ give

$$M \propto C^{0.51} \qquad D \propto C^{0.48}$$

— compute split almost evenly between model and data, close to Chinchilla and reached
independently. One notational trap: this deck reports model size as **FLOPs per token $M$**,
not as a parameter count (slide [23](../raw/slides/11-scaling-laws-in-the-wild.md)).

**And then the payoff**, which the lecturer calls "really the final punchline of a lot of
scaling law work: you train a bunch of small, curated models" and predict the large one
([21:33]). The fitted curve does predict the 7B and 67B losses closely — though not exactly:
the 7B point sits about 0.019 bits-per-byte above the fit and the 67B about 0.016 below it
(slide [24](../raw/slides/11-scaling-laws-in-the-wild.md)). "Well-predicted" means within
about 0.02 bpb, not "on the line".

See [published scaling recipes](published-scaling-recipes.md) for the two recipes set out
side by side.

## Part 3 — The rapid tour, and what its thinness means

By 2026 "most people know how to do a Chinchilla scaling law" ([23:54]), and the recent
releases have moved on to narrower questions — mostly [MoE](mixture-of-experts.md) sparsity:

- **Qwen 2.5 and Qwen 3** — the same DeepSeek-style analysis, now standard recipe. By Qwen 3
  "they say, all right, we've already figured out what to do in Qwen 2.5, we're just doing
  exactly the same thing" ([23:05]).
- **Kimi K2** — sparsity scaling, so they "can make more rational decisions about how much
  sparsity to put into their model" ([24:39]).
- **Hunyuan** — a **96-to-1** data-to-active-parameter ratio ([25:26]), another very high
  number, and one more disagreement with Chinchilla's 20:1.
- **LLaMA 3** — IsoFLOPs, plus a sigmoid fit from log loss to downstream accuracy ([26:14]),
  which is the [upstream-versus-downstream](upstream-vs-downstream.md) question of lecture 9
  handled empirically.
- **MiniMax-01** — architecture *decision* scaling: use scaling laws to choose between
  architectures. The answer for hybrid and lightning attention was "for the most part, no"
  ([27:45]) — they do not change the compute-optimal picture much.

**Why so thin?** The lecturer's own explanation is not that the work stopped but that it
became unremarkable: the core machinery "is now fairly well understood by everybody, so
there's no necessary extra value in showing those experiments" ([28:31]).

## Part 4 — Step Law: grid-search the hyperparameter plane

StepFun's study is the systematic version of the DeepSeek approach — they "burned a ton of
compute grid-searching the space of hyperparameters" ([31:33]).

The motivating observation is that the published laws do not even agree on **what the optimum
is a function of** ([33:04]): OpenAI's batch-size law is written in terms of loss — the
$2\times10^{18}\mathcal{L}^{-4.76}$ form is "critical batch" thinking ([32:19]) — DeepSeek's
in terms of compute, others in terms of $N$ and $D$. Slide
[33](../raw/slides/11-scaling-laws-in-the-wild.md) tabulates seven of them.

Three findings come out of the sweep, and the middle one is the useful one:

1. **The loss surface is convex in both** learning rate and batch size, so an optimum is a
   well-defined thing to fit ([34:35], slide [35](../raw/slides/11-scaling-laws-in-the-wild.md)).
2. **Batch size depends primarily on dataset size; learning rate needs both $N$ and $D$**
   ([36:08], slide [36](../raw/slides/11-scaling-laws-in-the-wild.md)). Optimal batch size for
   seven model sizes collapses onto a single curve in $D$.
3. **Robustness across MoE sparsity and data mixtures** — presented with a question mark, and
   the lecturer flags the shakiest part himself: the finding that optimal learning rate rises
   with $D$ at fixed model size is "a little more potentially dicey" ([37:40]) and is likely
   fragile under a WSD schedule (slide [36](../raw/slides/11-scaling-laws-in-the-wild.md)) —
   which matters, since WSD is what makes these sweeps affordable in the first place.

See [Step Law and hyperparameter scaling](step-law.md).

### The honest answer to *can I just use these?*

A student asks essentially that — should we take published scaling laws off the shelf?
([39:59]) The answer is the most quotable thing in the lecture, and it is a negative result:

> We can't possibly know — there are always tiny differences ([41:30]).

You cannot establish that someone else's setup is close enough to yours for their fit to
transfer. What you can do is grid the things that are most sensitive and inherit the rest
([56:55]).

## Part 5 — Optimizers, and three ways comparisons go wrong

"Optimization is a core part of LLMs (but also tricky, due to scale dependence!)" (slide
[38](../raw/slides/11-scaling-laws-in-the-wild.md)). Much optimizer research hill-climbs on
the NanoGPT speedrun — reach a target loss "as quickly as possible" ([42:16]) — and the
lecture's complaint is that this is exactly the setting where scale-dependence bites: "we have
a small-scale experiment, we have something that's scale-dependent" ([43:02]).

**Problem 1 — the hyperparameters are usually wrong** (slide
[39](../raw/slides/11-scaling-laws-in-the-wild.md)). The sharpest evidence compares AdamW
*with itself*: at 130M, AdamW at $8\times10^{-3}$ reaches a given loss in **2.25× fewer
steps** than AdamW at $6\times10^{-4}$. Same optimizer, same model — only the learning rate
differs. And optimizers want different hyperparameters: Lion's optimal weight decay is ~0.6
against AdamW's 0.1, six times smaller, and picking the wrong one gives "a much worse result"
([44:35]).

**Problem 2 — the advantage shrinks with scale** (slide
[40](../raw/slides/11-scaling-laws-in-the-wild.md)). Every non-AdamW optimizer falls from a
1.18–1.41× speedup at 130M to 1.09–1.10× at 1.2B. The general instruction is stated as a rule:
**always** check scaling against compute *and* the Chinchilla ratio, because these "are often
major confounders" ([46:08], [46:56]).

**Problem 2.5 — a clean fit can still blow up** (slide
[41](../raw/slides/11-scaling-laws-in-the-wild.md)). Taken from Will Held's write-up of work
on [Marin](scaling-laws.md), Percy Liang's open-source LM project ([47:42]). Seven IsoFLOP
parabolas, seven optima sitting on the fitted line to within ±0.16% — and then the held-out
runs come in 0.8% worse one decade out, 2.5% two decades out, and at two and a half decades
the run **diverged**, about 24% worse than predicted. "It's totally blown up on you… even
good-looking scaling" can fail ([48:29]).

### Muon

**Muon is momentum SGD with the momentum buffer orthogonalized before use** ([49:15],
[50:01]). The motivation is that matrix parameters deserve different treatment from vector
parameters, and matrices have spectra you can manipulate ([50:47]): write $B_t = USV^{\top}$
and replace it with $UV^{\top}$ ([51:34]), setting every singular value to 1. It is defined
only for matrix-valued parameters ([52:20]).

**Does it work at scale?** The lecture is careful here in a way the slide's own closing line
is not. Kimi K2 is "a very good model, and the training curves look reasonable" ([53:52]) —
but that panel has no baseline, so it shows Muon *works*, not that it works *better*. On
whether Muon beats AdamW at scale: "those are interesting questions to which we don't yet have
the answers" ([54:37]). See [optimizer scaling](optimizer-scaling.md).

## Part 6 — muP, derived

The final third derives muP properly, from what the lecturer cheerfully calls "a very
physicist way of thinking about this" ([1:00:45], [1:07:39]). The accessible reference he
recommends ([59:59]) is Yang, Simon and Bernstein's *A Spectral Condition for Feature
Learning*.

Two conditions, as functions of layer width $n_l$ (slide
[46](../raw/slides/11-scaling-laws-in-the-wild.md)):

- **A1** — activations at initialization stay $\Theta(1)$.
- **A2** — after one gradient step, the change in activation is $\Theta(1)$ ([1:06:53]).

A2 is the [feature-learning](maximal-update-parametrization.md) condition: a gradient step
should change the network by a meaningful amount, neither vanishing nor exploding as width
grows ([1:01:30]). Because these are per-coordinate statements, the norm version is
$\Theta(\sqrt{n_l})$ ([1:02:15]).

The derivation then runs: model the weights as a $\sigma^2$-scaled Gaussian and take the
spectral norm ([1:03:02]); carry an inductive assumption on the activation norm at layer
$l-1$ ([1:03:47]); recover the familiar $1/\sqrt{\text{fan-in}}$ initialization ([1:04:33]);
turn to the updates for SGD ([1:05:21]); expand $\Delta h_l$ into its terms and require them
all to be the same order ([1:06:06], [1:08:25]); and arrive at the learning-rate ratio
$n_l / n_{l-1}$ ([1:09:58]). For Adam, the rule becomes one over the fan-in — "layers with big
fan-ins get smaller learning rates" ([1:11:29], [1:13:01]).

The full derivation, with every equation as the deck writes it, is at
[maximal update parametrization](maximal-update-parametrization.md).

### Does it hold up?

**Where it works:** the replication finds the optimal base learning rate fixed at $2^{-6}$
across widths 128, 512 and 2048, for both baseline muP and a projection-biases variant —
"both of those transfer very nicely" ([1:13:48]). At larger scale it holds from 2M to 10B
parameters. And muP has a second, quieter benefit: with it, "their scaling law fits are
generally" better behaved ([59:13]).

**Where it breaks:** RMSNorm gains, exotic optimizers such as Lion, and strong weight decay
each move the optimum (slides [54](../raw/slides/11-scaling-laws-in-the-wild.md)–[56](../raw/slides/11-scaling-laws-in-the-wild.md)).
The RMSNorm-gain result comes with a practical shrug — those gains "can be removed in many
cases without necessarily hurting you, so maybe it's not such a bad idea to drop them"
([1:14:33]).

**And the verdict is deliberately modest.** muP "generally seems useful — insofar that SP is
quite a bit more unstable" (slide [57](../raw/slides/11-scaling-laws-in-the-wild.md)); the
demonstration that widening a model keeps the optimum in place is "actually pretty useful"
([1:15:19]). But the lecture closes by refusing the strong version of the claim: you cannot
run these procedures and "know for sure what will happen at scale… in reality it's a lot
messier and a lot more unknown than that" ([1:16:06]).

## Recap

The last slide states the lecture as three problems and three answers (slide
[58](../raw/slides/11-scaling-laws-in-the-wild.md)):

| Challenge in scaling *in practice* | The solution on offer |
| --- | --- |
| Setting model architecture hyperparameters (width, etc.) | Assume stability, or use [muP](maximal-update-parametrization.md) |
| Setting optimizer hyperparameters (LR, batch) | Search at small scale, then either keep fixed or [predict the scaling](step-law.md) |
| The compute needed for the big Chinchilla sweep | Use alternative learning schedules ([WSD-like](wsd-schedules.md)) |

## Topics from this lecture

- [Published scaling recipes](published-scaling-recipes.md) — MiniCPM, DeepSeek, and the tour.
- [Maximal update parametrization](maximal-update-parametrization.md) — muP derived, and its failure modes.
- [WSD schedules](wsd-schedules.md) — warmup–stable–decay, and the $n^2$ problem it solves.
- [Step Law and hyperparameter scaling](step-law.md) — the grid search over LR and batch.
- [Optimizer scaling](optimizer-scaling.md) — Muon, and why comparisons mislead.

## See also

- [Lecture 9 — scaling laws (basics)](09-scaling-laws.md) — the theory this lecture tests.
- [Compute-optimal scaling](compute-optimal-scaling.md) · [The IsoFLOP method](isoflop-method.md) · [Scaling law methodology](scaling-law-methodology.md)
- [Critical batch size](critical-batch-size.md) · [Learning rate scaling and muP](learning-rate-scaling-and-mup.md) — the two un-inheritable hyperparameters.
- [Mixture of experts](mixture-of-experts.md) — what most of the recent scaling work is really about.
- [Training stability](training-stability.md) — divergence as a failure mode.
- [Scaling laws](scaling-laws.md) — the hub.
- Material: [slide deck](../raw/slides/11-scaling-laws-in-the-wild.md) (`lecture_11.pdf`, 58 pages) · [transcript](../raw/transcripts/11-scaling-laws-in-the-wild.md)
