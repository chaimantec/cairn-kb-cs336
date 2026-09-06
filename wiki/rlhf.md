# RLHF — reinforcement learning from human feedback

RLHF is the second half of post-training: after
[supervised fine-tuning](supervised-fine-tuning.md) has taught the model to
follow instructions at all, RLHF shapes *which* responses it prefers to give, by
upweighting and downweighting its own outputs according to how a human — or a
model standing in for one — rates them.

This page covers the shape of the pipeline and the conceptual argument for it.
The components have their own pages: [preference data](preference-data.md),
[reward models](reward-models.md), [PPO](ppo.md), [DPO](dpo.md).

## The pipeline

The canonical picture is InstructGPT's, which CS336's lecture 15 puts on screen
three times (slides 6, 31 and 35). It has three steps, of which RLHF proper is
the last two:

1. **Collect demonstrations, train a supervised policy.** This is SFT.
2. **Collect comparisons, train a reward model.** Sample several outputs for one
   prompt, have a human rank them, fit a model that scores outputs.
3. **Optimize the policy against the reward model with RL.**

In practice (≈46:14) the sampling in step 2 is done at temperature 1 — "at the
end of SFT the model is pretty diverse, so this is a reasonable thing to do" —
and the ratings are usually pairwise rather than a full ranking.

The reason for the indirection through a reward model, rather than optimizing
human ratings directly, is throughput: humans cannot rate every rollout of a
training run. The lecture's framing is that "it might be easier to train a
verifier than to train a model that does well directly" (≈47:00).

## The conceptual shift: fitting versus maximizing

This is the part of lecture 15 that the lecturer most wants remembered, and it is
short (≈43:10).

Pre-training and SFT are **generative modelling**. You have a collection of
sequences and you fit a distribution over them. You can reason about the whole
thing with, in the lecture's phrase, your "generative-modelling brain."

RLHF is not that. It is **reward maximization**:

$$\max_{\pi_\theta} \mathbb{E}_{x\sim\mathcal{D},\, y\sim\pi_\theta(y|x)}\left[r(x,y)\right]$$

We still have a distribution, but it is now a *policy*, and nothing in the
objective asks it to resemble any reference distribution. The consequence the
lecture draws:

> Now, in the second world, I can totally collapse my distribution onto a single
> point for every input. For every prompt, my model could have a single answer —
> not a distribution — and that would be okay, as long as it got a good reward.

This is why [mode collapse](mode-collapse-and-calibration.md) is a *permitted
outcome* of RLHF rather than a bug in it, and it is why the objective is almost
never used bare. The KL term against the reference policy is what puts the
distributional pressure back in:

$$\max_{\pi_\theta} \mathbb{E}_{x\sim\mathcal{D},\, y\sim\pi_\theta(y|x)}\left[r_\phi(x,y)\right] - \beta\, \mathbb{D}_{\text{KL}}\left[\pi_\theta(y\mid x)\,\|\,\pi_{\text{ref}}(y\mid x)\right]$$

The lecture notes this "appears almost exactly in the InstructGPT paper — if you
open the paper up and go to equation two, you'll find exactly the kind of thing I
wrote down" (≈1:05:34), and that the same structure is in Stiennon et al.

## Why optimize at all?

If SFT works, why not simply collect more and better demonstrations? Lecture 15
gives two answers (≈43:55).

### The generation–verification gap

People are better at judging than at producing. The lecture's evidence is a study
of freelance writers asked to summarize news documents, in which some annotators
preferred **Instruct Davinci**'s summaries to their own — and, when interviewed,
did not retract:

> Well, I looked at it and I thought, actually, this is pretty good stuff.

Their own summaries had been independently checked for quality; these were
competent writers. "So people aren't really these optimal systems, and so when
they judge things, it's actually different. And because of this gap, sometimes
you might want to rate outputs rather than just generate demonstrations."

That gap is the reason a *rating* signal can carry information a *demonstration*
signal cannot, even from the same person.

### Verification is easier than generation in some domains

The second argument generalizes the first away from human idiosyncrasy: for
whole classes of problem, checking an answer is structurally cheaper than
producing one. "Math... is a prime example of this — verifying a proof is
probably much easier than generating the proof" (≈45:27).

Lecture 15 deliberately stops here and hands this to lecture 16, because once the
verifier is a *program* rather than a person you are no longer doing RLHF at all
— you are doing RLVR, and the failure modes change.

## Where RLHF is weak

Three things, all covered at the end of lecture 15:

- **[Overoptimization](reward-overoptimization.md)** — the reward model is a
  proxy, and pushing hard against a proxy eventually degrades the thing it
  proxies for. This is the lecture's nominated "big problem."
- **[Mode collapse and miscalibration](mode-collapse-and-calibration.md)** — the
  direct consequence of the objective shift above.
- **The data is the hard part.** "RLHF data collection is also very hard. I think
  part of the thing about post-training is that it's a very complicated, messy
  process, because a lot of it is getting good data, and getting good data is
  always very difficult" (≈1:18:41).

## A note on how much of this is public

Unusually for this course, the primary sources here are old. The lecture
recommends the **InstructGPT appendix** and **Stiennon et al., "Learning to
Summarize from Human Feedback"** — the latter "incredibly detailed," with real
annotation guidelines — precisely because they predate the competitive pressure
that closed the field (≈3:10). Nothing comparable exists for a current frontier
model. Treat every specific number on these pages as historical.

## See also

- [Lecture 15](15-mid-post-training.md) — the source lecture
- [Supervised fine-tuning](supervised-fine-tuning.md) — the phase before this one
- [Reward models](reward-models.md) · [PPO](ppo.md) · [DPO](dpo.md)
- [Preference data](preference-data.md) · [Human annotation](human-annotation.md) · [Model-based annotation](model-based-annotation.md)
- [Style and length bias](style-and-length-bias.md) — what preference optimization drifts toward
- [Course material for lecture 15](../raw/slides/15-mid-post-training.md)
