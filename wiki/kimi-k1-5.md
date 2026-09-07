# Kimi K1.5

Released around the same time as [DeepSeek-R1](deepseek-r1.md), and also beating
o1, but far less discussed. [Lecture 16](16-post-training-rlvr.md) includes it
deliberately, because it reaches similar results by different means — and that
disagreement is what tells you which components are essential:

> I think Kimi does some sets of things quite differently than DeepSeek, and we
> can learn quite a bit from the fact that both of these work. I think this is
> part of the theme of this class, which is: by reading all these tech reports
> and looking at broader patterns, we can roughly start to understand what are
> the valid, or easy, spaces to get these algorithms to work in. (≈40:08–40:53)

## A different derivation, the same destination

Kimi does not start from [PPO](ppo.md). It starts where everyone starts — maximize
expected reward with a KL regularizer to the reference policy:

$$\max_{\theta} \mathbb{E}_{(x,y^*)\sim \mathcal{D}} \Big[ \mathbb{E}_{(y,z)\sim \pi_\theta} \left[ r(x,y,y^*) \right] - \tau\, \mathrm{KL}\!\left(\pi_\theta(x) \,\|\, \pi_{\theta_i}(x)\right) \Big]$$

— and then follows a [DPO](dpo.md)-style route: assume the objective can be
maximized analytically, solve for the reward that corresponds to that maximizer,
and substitute back (≈43:57).

The final step is a heuristic, and the lecturer flags it as one rather than
dressing it up:

> if something's equal at the minimizer, why don't we just put a squared loss on
> that thing and minimize the squared loss? I think optimization people looking
> at this would be horrified, but I think this is a totally reasonable
> intuition, in some ways. (≈44:44)

Take the gradient and you get "a policy gradient with a baseline, r-bar — and
that r-bar is actually the mean, conditioned on each of these x's" plus a KL
term (≈45:30). In other words, [GRPO](grpo.md)'s group mean, reached from a
different direction: "we've reinvented the group-mean-normalized baseline
through quite different means" (≈46:16).

This convergence is the lecture's evidence that the group-mean baseline is the
load-bearing idea, while GRPO's standard-deviation and length normalizations are
optional extras. See
[advantage estimation and baselines](advantage-estimation-and-baselines.md).

![Slide 42 — Kimi RL](../raw/images/16-post-training-rlvr/slide-42.jpg)
*Slide 42 — the objective, and the note that the RL algorithm is "inspired by DPO-type derivation."*

## Length as a cost, not a virtue

Kimi's objective does not normalize by sequence length, so it never had
GRPO's [bias toward long wrong answers](length-bias-in-rl.md). It then goes
further and actively compresses chains of thought, because inference is paid for
at serving time (≈47:02).

The design subtlety is that you cannot just make wrong answers short, or the
model loses the room it needs to recover in its weak domains — the geometry
example at ≈48:33. So the length reward pushes incorrect answers to be *shorter
than the center of the range of rollouts* rather than as short as possible.
Slide 43 gives the reward in full, along with the note that it is only enabled
later in training because of its effect on performance.

## Data curriculum

Kimi's other contribution is treating RL data selection as a first-class problem
(≈41:38–43:11). The framing is that RL has a difficulty constraint SFT does not:
"if your problems are too hard, you get no reward, and if you get no reward, you
have no signal, and if you have no signal, you can't learn."

The methods:

- **Broad domain coverage**, excluding multiple choice on the grounds that it
  does not require long, deep thought.
- **Best-of-$k$ filtering.** If the model already solves a problem within eight
  samples, it teaches little; filtering on both sides keeps problems that are
  neither too easy nor too hard. "I think the general consensus in the research
  community is that doing this kind of medium-range difficulty filtering is very
  good, if what you want is for RL to progress at a steady pace" (≈43:11).
- **Retirement.** Once a model masters a problem it is removed from the pool,
  which saves compute (≈49:19).

Note this is the data-side answer to the same problem GRPO's standard-deviation
term addresses badly — both are about spending gradient on problems inside the
model's solvability range.

As with DeepSeek, the SFT stage is undocumented: "there's no description of SFT
— we can speculate about what that is, but we have no concrete information"
(≈43:11).

## The reward is a model after all

For code, Kimi generates test cases from ground-truth solutions. For math, it
uses a reward model to check answer equivalence — the fact the lecture builds
its punchline on. See [verifiable rewards](verifiable-rewards.md).

## RL infrastructure

Kimi's report is where the lecture hangs its
[RL infrastructure](rl-infrastructure.md) section: "Training is hard, inference
is hard, and RL puts the two together. So in some ways it's no wonder it's
really horrible and difficult" (≈51:38).

![Slide 45 — RL Infra](../raw/images/16-post-training-rlvr/slide-45.jpg)
*Slide 45 — the training side and the inference side, and the weight movement between them.*

## RL beats expert iteration

Kimi runs the ablation that answers the obvious question — why not just train on
the correct answers? — with "very large-scale ablations showing that these kinds
of RL methods work consistently better than expert iteration... So you can't
really avoid RL if you want to squeeze out all of your performance" (≈54:44).
See [expert iteration](expert-iteration.md).

![Slide 48 — Ablation: comparison to expert iteration](../raw/images/16-post-training-rlvr/slide-48.jpg)
*Slide 48 — twelve benchmarks, RL (orange) against ReST (blue). Values are hand-read from small charts and approximate; the direction is consistent.*

## See also

- [GRPO](grpo.md), [DPO](dpo.md) — the two derivations it sits between
- [Advantage estimation and baselines](advantage-estimation-and-baselines.md)
- [Length bias in RL](length-bias-in-rl.md)
- [Verifiable rewards](verifiable-rewards.md)
- [RL infrastructure](rl-infrastructure.md), [expert iteration](expert-iteration.md)
- [DeepSeek-R1](deepseek-r1.md), [Qwen 3](qwen3.md)
- [Lecture 16](16-post-training-rlvr.md)
