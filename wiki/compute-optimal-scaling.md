# Compute-optimal scaling

Given a fixed compute budget, how much should go into parameters and how much into
tokens? This is the most-used application of scaling laws, the source of the
"20 tokens per parameter" rule, and — because two famous papers gave very different
answers — [lecture 9](09-scaling-laws.md)'s main case study in how much
methodological detail matters.

## The setup

The budget constraint comes from lecture 1's accounting
([training FLOPs](training-flops.md)):

$$C \approx 6ND$$

for $N$ parameters and $D$ tokens ([56:47]). Spending it badly is easy to see:
push enormous data through a tiny model and its curve flattens early — "this thing
has been flat for a very long time; this is a complete waste of compute" ([57:32]).
The same compute spent on a larger model reaches a much lower loss.

## Joint scaling laws

To optimise the split you need loss as a function of *both* variables. Kaplan and
Rosenfeld proposed near-equivalent forms almost simultaneously ([58:20], slide 43):

$$\text{Error} = n^{-\alpha} + m^{-\beta} + C
\qquad\qquad
\text{Error} = \left[m^{-\alpha} + n^{-1}\right]^{\beta}$$

Rosenfeld's is simply two scaling laws added together; Kaplan's is a little more
involved but expresses the same idea.

**Check the limits** — advice Hashimoto gives for any scaling law you are handed
([59:06]). Send data to infinity and the data term vanishes, leaving a pure
model-size law; send model size to infinity and you are left with a pure data law.
Both limits behave as they should, which is weak evidence the form is sane.

These fit well enough to extrapolate rather than merely interpolate: fit the
exponents on the small-model, small-data corner of the grid and the predictions
hold up on the large-model, large-data corner, on both ImageNet and WikiText-103
(slide 44, [59:06]). Given that, compute-optimal allocation is a constrained
optimisation — minimise the joint form subject to the FLOP budget ([59:53]).

## Kaplan's answer

$$N_{opt} \propto C^{0.73} \qquad D_{opt} \propto C^{0.27}$$

Tokens per parameter therefore *decrease* as compute grows: put new compute mainly
into parameters (slide 45).

This had consequences. "If you were around in the days of GPT-3… there was a period
where everyone was training these gigantic models, hundreds of billions of
parameters, trillion-parameter dense models, what have you. Part of that was driven
by this" ([1:00:40]).

> Hashimoto misreads the two exponents in the lecture and corrects himself
> immediately — "is that reversed? I think it's reversed" ([1:00:40]). Slide 45
> prints $N_{opt} = C^{0.73}$, which resolves it; the transcript keeps the
> self-correction as spoken.

## Chinchilla's answer

Hoffmann et al. 2022 argued those fits were badly off, and made the argument three
separate ways — which Hashimoto singles out as good practice, "a way of
robustifying yourself against modeling assumptions you may have made" ([1:02:12]).

| Approach | $a$ where $N_{opt} \propto C^{a}$ | $b$ where $D_{opt} \propto C^{b}$ |
|---|---|---|
| 1. Minimum over training curves | 0.50 | 0.50 |
| 2. IsoFLOP profiles | 0.49 | 0.51 |
| 3. Parametric modelling of the loss | 0.46 | 0.54 |
| Kaplan et al. 2020 | 0.73 | 0.27 |

*(Slide 46. The table also prints confidence intervals for each approach.)*

Roughly equal exponents mean a **fixed ratio** between tokens and parameters, and
that ratio is where the famous **20 tokens per parameter** comes from ([1:01:26]).

**Method 1 — minimum over runs** ([1:02:59]). Take every training curve and read off
its lower envelope: each envelope point is the lowest loss anyone achieved at that
FLOP count, and it belongs to a run of known model size. Scatter model size against
FLOPs for those points and fit. At Gopher's budget: **67B parameters, 1.5T tokens**
(slide 47). The weakness is practical — reliably identifying the envelope is fiddly.

**Method 2 — [IsoFLOPs](isoflop-method.md)** ([1:04:31]). At each of several fixed
budgets, sweep the parameter/data trade-off and watch terminal loss trace a
parabola; take the minima and regress them. At Gopher's budget: **63B parameters**.

**Method 3 — joint parametric fit** ([1:05:16]). Fit the hypothesised loss surface
to all runs directly. The most natural method and the most fragile: "how you do the
curve fitting is very important — you're fitting these surfaces, there are many
variables, it's kind of tricky."

## Why the two papers disagree

Not because either was careless — "they both did pretty reasonable stuff"
([1:06:50]). Two follow-up papers diagnose it, and the lecture treats this as the
real lesson.

### Porian et al. — *Resolving Discrepancies in Compute-Optimal Scaling*

Yair Carmon is last author. They reproduce Kaplan's result under Kaplan's settings
and then change one thing at a time until they land on Chinchilla
([1:07:36]–[1:09:54]):

1. **Parameter counting.** Kaplan excluded embedding parameters, which is
   defensible — including them produced "very funky-looking scaling laws" ([37:40]).
   But they also excluded the **final softmax layer**, reasoning that it is dual in
   shape to the input embedding: vocab×hidden against hidden×vocab. Whether those
   are counted materially changes the law.
2. **Learning-rate warm-up.** Kaplan's smallest models "were so small that they were
   not actually converging by the time the learning-rate warm-up was done", so their
   learning rates were badly set and the models undertrained. Hashimoto calls this
   "much more of an oversight than a genuine difference of opinion" ([1:09:08]).
3. **Batch size.** Kaplan fixed one large batch size across all models, which is
   suboptimal for the small ones. Tune per model and "you get results that agree
   exactly with Chinchilla" ([1:09:54]).

### Pearce and Song — *Reconciling Kaplan and Chinchilla Scaling Laws*

A cleverer design: they train nothing ([1:10:41]). They simulate training curves
from Chinchilla's own fitted functional form, then ask what those curves would look
like measured under Kaplan's conventions — non-embedding parameter counts, Kaplan's
compute range.

Their diagnosis differs slightly from Porian's: it is the **low compute scale** that
makes Kaplan sensitive to small perturbations, combined with the **nonlinearity**
introduced by using non-embedding rather than total parameters ([1:11:27]).

Slide 52 carries their reconciliation chart, where the true frontier is a single
curved line that looks like Kaplan's steeper 0.78 exponent at small compute and
Chinchilla's shallower 0.51 exponent at large compute — each paper's law being
locally right in its own range.

### The generalisable lesson

**A scaling law is a lower bound on a recipe** ([1:09:54]):

> Scaling laws are kind of lower bounds, in some sense — they're saying, "if I
> continue this recipe and scale it up, this is what I'll get." But if you're
> scaling up a recipe you don't actually want to scale up — your warm-up is crazy,
> or your batch sizes are crazy — you're going to get bad scaling laws.

So the fitted line is only as good as the run underneath it. Stay as close to a
proper full run as you can.

## The method-3 mystery

Chinchilla's own third method never agreed with the other two — 0.46/0.54 against
0.50/0.50 — and its authors were "pretty unperturbed" ([1:12:13]). Hashimoto is
less relaxed: unequal exponents are not a rounding difference, they mean that
asymptotically you end up with far more tokens than parameters, which is a
qualitatively different prescription.

**Epoch AI** resolved it ([1:13:00]). Unable to obtain either the raw data or the
code, they extracted the data points from the paper's own published plots and
refitted the parametric surface. The original fit was **underfit**: refitting
achieves lower losses, and it recovers almost exactly the 20-tokens-per-parameter
rule the other two methods gave. Method 3 was fine; the fit was not.

> "It turns out the authors were more right than they knew. They'd made a mistake,
> and that was the only reason method three disagreed with methods one and two."
> ([1:13:46])

## Why you probably should not use the ratio

Compute-optimal means **training**-compute-optimal, and that is the wrong budget
for a production model ([1:14:32]). Surveys of frontier-lab compute show most of it
going to R&D and serving rather than to the training run itself. For serving you
want a small capable model — "you don't want big, bloated models that cost a lot to
serve, even if that minimizes training cost".

Hence deliberate **overtraining**, a word Hashimoto puts in scare quotes because it
is a misnomer: "really, overtraining is what we want, that's the right amount of
training."

The trajectory ([1:15:17]):

| Era | Tokens per parameter | Why |
|---|---|---|
| GPT-3 | ~3 | Undertrained; pre-Chinchilla |
| Chinchilla and after | 20 | Compute-optimal, and models were not yet served at scale |
| Serving era | well past 20 | Inference cost dominates; also the shift to [MoE](mixture-of-experts.md) |

The lasting value is methodological, not numerical: Chinchilla matters "not because
I think the 20:1 ratio is the one golden ratio… but because it tells us quite a bit
about how we should fit scaling laws".

## See also

- [Lecture 9](09-scaling-laws.md) · [The IsoFLOP method](isoflop-method.md)
- [Scaling law methodology](scaling-law-methodology.md) — the recipe-as-lower-bound idea generalised.
- [Training FLOPs](training-flops.md) — where $C = 6ND$ comes from.
- [Data scaling laws](data-scaling-laws.md) — the univariate case.
- [Scaling laws](scaling-laws.md) — the hub.
- Slides [43–53](../raw/slides/09-scaling-laws.md) · [transcript](../raw/transcripts/09-scaling-laws.md)
