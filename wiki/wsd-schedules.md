# WSD learning-rate schedules (warmup–stable–decay)

**A WSD schedule replaces the cosine learning-rate curve with three phases — a short
warmup, a long stretch at a constant "stable" learning rate, and a short decay at the
end.** Its point is not that it trains better than cosine. It is that it makes fitting a
scaling law dramatically cheaper, by letting you branch many runs off a single stable
trunk instead of training each one from scratch.

[Lecture 11](11-scaling-laws-in-the-wild.md) presents it as the shared trick behind two
otherwise very different recipes: MiniCPM adopts it outright, and DeepSeek uses a
multi-step variant of the same idea.

## The problem it solves: fitting a scaling law costs $n^2$

The cosine schedule has a property that is easy to miss and expensive once you notice it.
Its shape depends on the *total* token budget you declare at the start, so a run configured
for 40N tokens is a different run from one configured for 80N — not a prefix of it.

That means, as slide [13](../raw/slides/11-scaling-laws-in-the-wild.md) puts it, "to fit a
scaling law, we need to train from scratch, not just early stop", and therefore "this turns
the cost of fitting a scaling law from $n$ to $n^2$."

The slide's own six-panel ablation is the evidence, and it is worth reading carefully
because it establishes something stronger than mere inconvenience. Six cosine cycle
lengths (1.0×, 1.1×, 1.25×, 1.5×, 2.0× and 5.0× the run length) are trained to two
different total lengths. The ordering is the same in every loss panel and in both rows:
**the shorter the cosine cycle, the lower the loss at the end of the run** — 1.0× and 1.1×
essentially tied for best, 1.25× just behind, and 5.0× clearly worst, by about 0.06 in
training loss and 0.07 in C4 loss.

The consequence is the one that costs money. The loss a run reaches at step $n$ when its
cosine cycle was *set* to $n$ is **not** recoverable by reading an intermediate point off a
longer-cycle run. Every point on your scaling curve needs its own complete training run.

One incidental detail from the same figure, since it is the kind of thing that confuses a
reader of the curves: these cosine schedules decay to a floor of about **10% of maximum
learning rate**, not to zero.

## The WSD shape

Instead of cosine, split the schedule into warmup, stable and decay phases (slide
[14](../raw/slides/11-scaling-laws-in-the-wild.md)). The MiniCPM figure shows all three
schedules sharing one identical warmup — a near-vertical ramp to the peak learning rate over
roughly the first 130 iterations — after which the WSD runs simply hold that peak, flat,
until it is time to decay, then fall linearly to the same ~10% floor.

**The payoff is stated on the slide: "for Chinchilla-style analysis, can restart the run at
the end of the stable phase."** Because every WSD run with the same peak shares the same
stable trunk, you can take a checkpoint from anywhere along it, decay from there, and get a
properly-trained model for that token budget. The MiniCPM paper's own wording, reproduced on
the slide, makes the cost claim explicit — the WSD scheduler "has the advantage of arriving
at the optimal loss of Cosine LRS after decaying from stable stage's checkpoints of any
step", so scaling properties can be measured "without re-training the models from scratch to
different amount of tokens", at **linear cost $O(mC)$**.

That is how MiniCPM assembles a scaling grid at all: six model sizes from 0.04B to 2B, each
decayed from checkpoints at $10N$ through $60N$ tokens during the stable phase.

**How long should the decay be? About 10% of the run.** Slide
[15](../raw/slides/11-scaling-laws-in-the-wild.md) prints "Decay ~ 10%", and the schedules
in the figure measure at roughly 11% and 10% of their runs.

## Does WSD actually match cosine? Mostly, with a real caveat

Slide [15](../raw/slides/11-scaling-laws-in-the-wild.md) is titled "WSD learning rates work
well in miniCPM", and the figure supports it — but **only for the longer decays**, and the
title is doing more work than the data.

During the stable phase the WSD runs are genuinely *worse* than cosine: their band sits
0.05–0.11 above the cosine curve. The decay phase is what redeems them, dropping them
sharply at the end. Reading the endpoints against the Cosine(80N) baseline at the same token
count:

| Run | Final loss | vs. cosine at that budget |
| --- | --- | --- |
| WSD(40N, 2N) | 3.856 | worse (3.843) |
| WSD(40N, 4N) | 3.820 | better |
| WSD(60N, 2N) | 3.765 | worse (3.706) |
| WSD(60N, 6N) | 3.711 | better/tied |
| WSD(80N, 2N) | 3.704 | worse (3.648) |
| WSD(80N, 8N) | **3.617** | better — the lowest point in the figure |

So the three **2N-decay** runs each finish worse than cosine, and the 4N/6N/8N runs — the
~10% decays — each finish better or tied. The slide does print "Decay ~ 10%", so this is a
qualifier rather than a contradiction, but a reader who takes only the title away gets the
wrong conclusion.

## DeepSeek's multi-step variant

DeepSeek reaches the same place by a slightly different route (slide
[22](../raw/slides/11-scaling-laws-in-the-wild.md)): "fast warmup + two decay steps of 10%
each". Its published description is precise — the learning rate peaks after 2000 warmup
steps, drops to **31.6%** of maximum after 80% of the training tokens, and to **10%** of
maximum after 90%. (31.6% is $1/\sqrt{10}$, so the two steps are equal in log space.)

The comparison against cosine has the same shape as MiniCPM's. For the first ~30B tokens the
two are indistinguishable; from ~35B to 80B the multi-step curve sits **above** cosine by
0.05–0.08; the step at 80B brings it back down onto the cosine curve, and from ~85B both
finish together at ≈2.33–2.34. The slide's summary — "generally seems to match performance
of cosine learning rates" — is fair.

The second panel is the more interesting one for anyone choosing a split. Three schedules
decaying at 60%, 70% and 80% of the token budget all converge to essentially the same final
loss, ≈2.33. Each is lower than the others only during the window after its own decay begins
and before theirs does. **Where you put the decay barely matters to the endpoint** — which
is exactly the property that makes branching off the stable trunk legitimate.

## See also

- [Scaling law methodology](scaling-law-methodology.md) — the cost of fitting a scaling law,
  which is what this schedule exists to reduce.
- [Published scaling recipes](published-scaling-recipes.md) — MiniCPM and DeepSeek in full,
  and where WSD sits in each.
- [Compute-optimal scaling](compute-optimal-scaling.md) — the Chinchilla analyses these
  schedules are feeding.
- [IsoFLOP method](isoflop-method.md) — the analysis DeepSeek runs on top of its schedule.
- [Lecture 11](11-scaling-laws-in-the-wild.md) · slides [13–15](../raw/slides/11-scaling-laws-in-the-wild.md), [22](../raw/slides/11-scaling-laws-in-the-wild.md)
