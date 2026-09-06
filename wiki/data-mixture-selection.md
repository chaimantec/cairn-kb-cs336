# Data mixture selection

How much news and how much Wikipedia? Two lectures treat this from opposite ends.
[Lecture 9](09-scaling-laws.md) uses the mixture question to show what scaling laws
buy you for data engineering and where the idealised story breaks down.
[Lecture 14](14-data-filtering-dedup-mixing.md) gives the practical method — the
baselines people actually use, the regression procedure, and a concrete
scale-dependent effect that partly undercuts lecture 9's optimism. Read the two
halves together; they disagree in an instructive way.

## The theoretical hook (lecture 9)

The reason to expect mixtures to be tractable at all is the slope/intercept split
([22:19]). Reading data scaling laws as empirical generalization bounds suggests
that **dataset composition affects the offset of the curve, while the slope is set
by the model class** — not by the distribution.

The toy model on slide 23 is a linear regression with two samplable sources: draw
from only one and error is high either way, so the best intercept comes from a
*mixture*. Writing down how that intercept behaves gives "very interesting insights
about how having more data diversity is very helpful" ([23:05]).

![Slide 23 — In practice: data mixture selection via scaling is hard](../raw/images/09-scaling-laws/slide-23.jpg)

*Slide 23 — In practice: data mixture selection via scaling is hard. [Deck](https://github.com/stanford-cs336/lectures/blob/main/lecture_09.pdf)*

## The idealised method

The obvious procedure ([23:51]):

1. Train small models on small amounts of data at various mixture ratios.
2. Fit a functional form for how the mixture level affects performance.
3. Scale up a little, check the trend holds, and extrapolate the same way you would
   any data scaling law.
4. Take the minimum of the extrapolated curve as your mixture for the full run.

This is the **data mixing laws** idea: "fit a functional form at a small amount of
compute, find the minimum, and then scale that out."

## What actually happens

Hashimoto is blunt about the gap ([23:51]):

> Unfortunately, if you talk to anyone who's done a lot of this data-mixture work,
> they'll tell you reality is a lot more noisy than this ideal world would suggest.

In practice, "as far as I know, in many cases", people train a batch of small
models, pick the best mix from those, and scale it up — **no scaling law required**
([24:38]).

**DataDecide** is the large-scale empirical study of this, and it supports the
shortcut: simply picking the best data mix at small scale works well ([24:38]).

## Why the shortcut is not a cop-out

The elegant part of the argument, and worth keeping ([24:38]):

> For what it's worth, that's consistent with the argument that the intercepts
> differ but the slopes don't change — because if the slopes don't change, the best
> mixture at small scale is also the best mixture at large scale.

In other words, the theory that motivated fitting mixture scaling laws is the same
theory that says you do not need to. If composition only moves intercepts, the
ranking of mixtures is scale-invariant, and a small-scale bake-off answers the
question directly. Fitting the law adds machinery without adding an answer.

This is a good instance of the lecture's broader methodological point: knowing
*which* part of the curve an intervention moves tells you what experiment to run.

## The caveat next door

Mixture selection assumes the pool is fixed. It is not — how aggressively you should
filter depends on your compute budget, and the optimal filter loosens as compute
grows. See [data repetition](data-repetition.md), which covers slide 26's
quality-versus-compute grid.

![Slide 26 — Data selection scaling and accounting for finiteness](../raw/images/09-scaling-laws/slide-26.jpg)

*Slide 26 — Data selection scaling and accounting for finiteness. [Deck](https://github.com/stanford-cs336/lectures/blob/main/lecture_09.pdf)*


---

## The practical treatment (lecture 14)

Lecture 14 arrives at mixtures from the pipeline side rather than the scaling side.
By this point the data has been [extracted](html-to-text-extraction.md),
[filtered](quality-classifiers.md) and [deduplicated](minhash-and-lsh.md), giving
"a smaller set of high quality documents" — but per source. "Language models are
trained on multiple data sources", and the question is how to combine them
(≈49:19).

*Figure: `images/marin-token-viewer.png`.*

![Screenshot of an interactive horizontal bar chart, "Tokens by Dataset", listing ~30 datasets by token count, colour-coded by category](../raw/images/14-data-filtering-dedup-mixing/marin-token-viewer.png)

*Marin's [token-count viewer](https://huggingface.co/spaces/marin-community/token-count-viewer): about 30 candidate sources for the next model, coloured by category (web, multilingual, code, math, specialized). Bar values are visual estimates — the tool prints no data labels — but the shape is the point: two web sources dominate everything else.*

A **data mixture** is just a distribution $p$ over sources. The Pile assigned "a
particular weight to each component" (≈50:53); the question is where those weights
come from.

### The three baselines

- **Vibes** — set $p(s)$ by hand, from intuition. The source's own word, and its
  parenthetical is "(quite common)". The lecture is blunter: "this is definitely
  fairly vibes-based, and even more recent papers, you just look at it, maybe you
  use some method, and then you just tweak things" (≈50:53).
- **Uniform sampling** — $p(s) \propto 1$.
- **Proportional mixing** — $p(s) \propto \text{num\_tokens}(s)$. "Generally a
  rational thing to do, but you might worry that if you have a huge low-quality
  dataset, that's going to eat up a lot of your tokens" (≈52:26).

Intuition says upweight the higher-quality sources. Two things complicate that
(≈52:26):

1. **Diversity.** Sources are often incomparable — "you can't really say that this
   paper is higher quality than this code, because they're just incomparable
   objects" — and a model that saw only papers will not be good.
2. **Finiteness.** Each source has a fixed size, so putting too much weight on a
   small one means **epoching** over it.

### The epoching trap

The second point is the one the lecture stops on, and it is arithmetic, not
intuition (≈53:11–55:28). Take two sources:

| Source | Available tokens |
| --- | --- |
| low quality | 10T |
| high quality | 10B |

"Generally high quality sources are smaller." Now take an innocuous 50/50 mixture
and train for **1T tokens**:

$$\text{epochs} = \frac{p(s) \cdot \text{train\_tokens}}{\text{num\_tokens}(s)}$$

- low quality: $(0.5 \times 10^{12}) / 10^{13} = $ **0.05** — you never even see
  95% of it.
- high quality: $(0.5 \times 10^{12}) / 10^{10} = $ **50** — every token is
  repeated **fifty times**.

The source's comment is "50x epochs on high quality data...can lead to
overfitting!" and the lecture's warning is sharper: "this is actually really
important, and some big model training runs have kind of messed this up" (≈55:28).
The failure mode is that nothing announces itself — "you're going to end up doing 50 epochs without realizing it, unless you're paying close attention", so the lesson
is operational: **look at how many epochs you are actually doing on each source**
(≈56:14). Upweighting a scarce source does not give you more of it; it gives you
the same tokens again. See [data repetition](data-repetition.md) for what repeats
do to the loss curve.

A question from the floor is worth recording here, because it fixes what a mixture
physically *is* at training time: mixtures are realized **per sequence, not per
token**. You sample which mixture component each element of a batch comes from and
fill the batch with sequences from that component, so a batch is itself mixed —
which is also what keeps the variance down (≈57:00).

### UniMax: cap the epochs

[arXiv 2304.09151](https://arxiv.org/abs/2304.09151), from multilingual training,
"it was very clear that some languages were very low-resource" (≈58:34).

Earlier work interpolated between the two baselines with
$p(s) \propto \text{num\_tokens}(s)^{\alpha}$ for $\alpha \in [0,1]$ — uniform at
0, proportional at 1 — "to kind of flatten out the distribution". UniMax replaces
that knob with a **constraint**: sample uniformly, but impose a hard cap $C$ on the
number of epochs for any source, so that

$$p(s) \cdot \text{num\_training\_tokens} \leq C \quad \text{for all } s$$

"I'm only going to take 20 epochs over a particular data source. If you've done
that, then too bad — you don't get any more tokens, you move on to the next thing."
It is "a safety net", and there is a simple procedure for finding the mixture
subject to the constraint (≈59:20).

### Regression-based mixing

The principled answer to where the weights come from, and structurally identical to
[scaling laws](scaling-laws.md) — cheap experiments, fit a curve, optimize, scale
up (≈1:00:07–1:01:38):

1. Pick a **distribution over mixtures** (typically **Dirichlet**) and sample from
   it.
2. Train a **swarm of small proxy models**, one per mixture. Each returns a loss on
   some target — downstream evals, perplexity, whatever you choose.
3. **Fit a regression** from mixture weights to that loss. You now have "a very
   cheap model that tells you: if I were to train on this mixture, what loss would
   I get?"
4. **Optimize the fitted function** for the best mixture, and train the large model
   on it.

*Figure: `images/regmix.png`.*

![Four-step schematic of the RegMix pipeline: proxy-model table, regression model, 3D prediction surface, best-mixture table](../raw/images/14-data-filtering-dedup-mixing/regmix.png)

*RegMix ([arXiv 2407.01492](https://arxiv.org/abs/2407.01492)) as a four-step pipeline: train small-scale proxy models over sampled mixtures (each row a mixture with its target loss), fit a regression (linear or tree), simulate new mixtures and predict the target over a surface, then train at large scale on the predicted best mixture — here Hacker News 22.8% / GitHub 67.0% / PhilPapers 10.2%, predicted target 5.34.*

The design decisions are where the judgement lives (≈1:02:25):

- **Which mixtures to sample** — you need a distribution over distributions.
- **Which regression method** — linear models, log-linear, or gradient-boosted
  trees.
- **What target to fit.** This is the dangerous one. Targets are usually downstream
  evals, and "if you're not careful — for example, if you have a bunch of code evals, then, well, guess what, you're going to upweight all the code data. That's not rocket science, and if you then go and say, I want to generate some poetry, you might realize that you've overfit." Note that uniform and proportional mixing are
  immune to this precisely because they look at no evals at all.
- **How far small scale is from large scale** — a cost/accuracy trade-off. Train
  models too small and they are unrepresentative; train them large and "there's no
  point in doing any of this, you're basically doing hyperparameter tuning at the
  largest scale, which is too expensive" (≈1:03:11).

*Figure: `images/data-mixing-methods.png`.*

![Table comparing seven data-mixing methods across swarm-construction, regression-model and mixture-optimization design choices](../raw/images/14-data-filtering-dedup-mixing/data-mixing-methods.png)

*Seven methods in the same framework — RegMix, DML, AutoScale, BiMix, ADMIRE-BayesOpt, CLIMB and OLMixBase — laid out by design choice: proxy model size (1M to 350M), swarm size, swarm distribution (Dirichlet, exponential grid, entropy-weighted), regression family (LightGBM, log-linear, power law, Gaussian process), regression granularity, whether data-repetition constraints are imposed, and the optimization solver. The lecture reads it as evidence that proxy models are "generally fairly tens, or tens of millions of parameters". The table's pink shading carries no legend in the source, so it means only "highlighted".*

The source ends this part with two lines it labels **hopes**, prayer emoji included:

> Hope 1: regression model is accurate at minimizer 🙏
>
> Hope 2: optimal data mixtures transfer from small to large scale 🙏

The lecture unpacks both as "two leaps of faith" (≈1:04:44–1:05:32). The first is
subtler than it looks: a regression fitted on sampled mixtures will predict well
*in distribution*, "but when you're optimizing, you're essentially trying to go to
potentially the extremes, where you might not have as much coverage" — optimization
walks the fit to exactly the region where it is least supported. On the second, the
honest assessment is that transfer "at least at the scales that the open community works with, seems to tend to be true, or not blatantly false" — but "clearly there are
scale-dependent effects".

### The scale-dependent effect, and simulated epoching

Here is where lecture 14 contradicts the comfortable conclusion above. Lecture 9
argued the small-scale bake-off is *sound* because composition moves intercepts and
not slopes, so mixture rankings are scale-invariant. Lecture 14 exhibits a case
where the ranking flatly does not transfer (≈1:06:19–1:07:05).

Take the same two sources — 10T low-quality, 10B high-quality — and search for a
mixture using small models on small token counts. At that scale you never exhaust
the scarce source, so nothing penalizes loading up on it: the search happily returns
something like 10% low / 90% high. "It's like, oh wow, Wikipedia is so great,
let's just train on Wikipedia." Train the large model on that mixture and you epoch
enormously on the high-quality source and overfit.

Two fixes:

- **Cap the epochs**, as UniMax does.
- **Simulated epoching** ([arXiv 2501.11747](https://arxiv.org/pdf/2501.11747)) —
  "make your small scale look like your large scale", which the lecture calls a
  general theme of the course and explicitly connects to
  [muP](maximal-update-parametrization.md): parameterize so that what you learn at
  small scale transfers (≈1:07:51).

The instantiation is to **downsample every source by the ratio of the two run
sizes**. With a 10B-token small run against a 1T-token large run the ratio is
$10^{10}/10^{12} = 0.01$, so the pool becomes 100B low-quality and 100M
high-quality tokens. Now the small experiment *feels* the scarcity: "you don't get to just train on all of Wikipedia — you're going to train on some minuscule fraction of Wikipedia, and you'll realize, oh, actually, I'm going to be epoching a lot, and that's going to have really bad loss" (≈1:08:37). The optimum moves to something
balanced — the source's illustrative 70/30.

Asked whether downsampling can leave a source too small to be usable, the answer
was yes in principle: the optimizer will put very little mass on it and recover at
scale, "but there might be rounding errors, where you end up, instead of training once on this dataset, training zero times, by accident" — worked around by ensuring
you always train at least once on it (≈1:10:11).

### Mixing *within* a source

A question from the floor drew out something the program does not contain at all
(≈1:11:44). Mixing is not restricted to the sources someone hands you: given a
single corpus like Common Crawl, you can **group it by domain** — the lecture cites
AI2's WebOrganizer, and Nemotron and OLMo as users — **and also by quality**, which
gives a two-dimensional grid of (domain × quality) cells. Each cell is then a
mixture component in its own right. That is "one automatic way to determine the
domains", on top of which you add whatever extra sources you were given.

### Summary of the section

> Regression-based mixing is a nice framework to think about things. You estimate a
> functional form that maps your mixture weights to a loss at small scale,
> optimize, and then you generalize to large scale. And you just have to be very
> careful because of epoching and overfitting issues — you have to either use
> capped epoching or simulated epoching. (≈1:09:24)

And the generalizable warning underneath it: "if you're trying to optimize
anything, you have to be very careful, because you are in danger of optimizing the
wrong thing" (≈1:10:11).

## See also

- [Data scaling laws](data-scaling-laws.md) — the slope/intercept result the
  lecture-9 half rests on
- [Data repetition](data-repetition.md) — what epochs do to the loss curve, which
  is the mechanism behind the epoching trap
- [Quality classifiers](quality-classifiers.md) — the same scale-dependence, one
  stage earlier: there is no optimal filtering threshold either
- [Scaling laws](scaling-laws.md) · [Scaling law methodology](scaling-law-methodology.md)
  — regression-based mixing is the same method pointed at a different variable
- [Maximal update parametrization](maximal-update-parametrization.md) — the other
  place this course makes small scale look like large scale
- [Lecture 9](09-scaling-laws.md) · slide [23](../raw/slides/09-scaling-laws.md) · [transcript](../raw/transcripts/09-scaling-laws.md)
- [Lecture 14](14-data-filtering-dedup-mixing.md) ·
  [course material](../raw/slides/14-data-filtering-dedup-mixing.md#data-mixing) ·
  [transcript](../raw/transcripts/14-data-filtering-dedup-mixing.md)
