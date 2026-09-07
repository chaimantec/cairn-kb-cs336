# Reward models

A reward model is a scalar-output model trained on human preference comparisons,
used as a stand-in for a human rater inside an
[RLHF](rlhf.md) training loop. It exists for a throughput reason: a training run
makes millions of rollouts and you cannot put a person behind each one.

## How one is built

The construction CS336's lecture 15 shows is Stiennon et al.'s, reproduced on
slide 52 of the deck:

> To train our reward models, we start from a supervised baseline, as described
> above, then add a randomly initialized linear head that outputs a scalar value.
> We train this model to predict which summary $y \in \{y_0, y_1\}$ is better as
> judged by a human, given a post $x$.

So: take the SFT model, replace the language-modelling head with a fresh linear
head producing one number, and train that on comparisons. The loss is the
pairwise logistic (Bradley–Terry) loss — if the human preferred $y_i$,

$$\mathrm{loss}(r_\theta) = -E_{(x,y_0,y_1,i)\sim D}\big[\log\big(\sigma\big(r_\theta(x,y_i) - r_\theta(x,y_{1-i})\big)\big)\big]$$

where $r_\theta(x,y)$ is the scalar output for prompt $x$ and response $y$, and
$D$ is the dataset of human judgments.

Two properties of this loss are worth noticing, because they explain several
things downstream:

- **It only ever sees reward *differences*.** $r_\theta(x,y_i) - r_\theta(x,y_{1-i})$
  is invariant to adding any function of $x$ to every score for that prompt. The
  reward model is therefore identified only up to a per-prompt shift. Stiennon et
  al. pin the scale down by convention — "at the end of training, we normalize the
  reward model outputs such that the reference summaries from our dataset achieve
  a mean score of 0."
- **That same invariance is what makes [DPO](dpo.md) possible.** DPO's derivation
  produces an implied reward carrying an intractable $\beta\log Z(x)$ term, and
  it cancels precisely because this loss only reads differences.

## How it is used

The reward model's scalar becomes the reward for the whole response, and the
policy is optimized against it with [PPO](ppo.md) — "treating the output of the
reward model as a reward for the entire summary that we maximize with the PPO
algorithm [58], where each time step is a BPE token."

Critically, the reward that is actually optimized is not the reward model alone.
Stiennon et al. "include a term in the reward that penalizes the KL divergence
between the learned RL policy... and this original supervised model." That KL
term is not a regularization detail — it is the main defence against
[overoptimization](reward-overoptimization.md), because the reward model is a
proxy that degrades as you move away from the distribution it was fit on.

## Why go through a model at all

The lecture's compressed answer: "we go through the reward model because it might
be easier to train a verifier than to train a model that does well directly"
(≈47:00). This is the same
[generation–verification gap](rlhf.md#the-generationverification-gap) that
justifies preference collection in the first place, applied one level up — the
reward model is a learned verifier.

The limit of that argument is the whole of lecture 15's closing section. A
learned verifier is only a proxy for the thing you care about, and unlike a
*program* verifier ([lecture 16](16-post-training-rlvr.md)'s [RLVR](rlvr.md))
it can be gamed by the policy it is supervising.

That contrast is real but softer than it looks. Lecture 16 shows reward models
reappearing *inside* RLVR — [Kimi](kimi-k1-5.md) checks mathematical answer
equivalence with one — and shows program verifiers being gamed too. See
[verifiable rewards](verifiable-rewards.md).

## What the reward model inherits

Everything wrong with the preference data ends up in the reward model, which is
why lecture 15 spends far longer on annotation than on algorithms:

- **Style preferences.** Raters favour length and bullet-point structure, so
  reward models do too — see [style and length bias](style-and-length-bias.md).
  You can "RLHF on length alone and do quite well on many of these benchmarks"
  (≈1:04:48).
- **Annotator identity.** Who rated the comparisons shifts what the reward model
  scores highly, measurably — see [human annotation](human-annotation.md).
- **What the annotators could not check.** Non-experts weight formatting; experts
  catch factuality. A reward model built on the former cannot reward the latter.

## See also

- [RLHF](rlhf.md) — the pipeline this sits inside
- [PPO](ppo.md) — what optimizes against it
- [DPO](dpo.md) — what removes it
- [Preference data](preference-data.md) — what it is trained on
- [Reward overoptimization](reward-overoptimization.md) — what happens when you trust it too far
- [Verifiable rewards](verifiable-rewards.md) — the program-verifier alternative, and its limits
- [Lecture 16](16-post-training-rlvr.md) — where reward models reappear inside RLVR
- [Course material for lecture 15](../raw/slides/15-mid-post-training.md)
