# PPO — proximal policy optimization

PPO is the reinforcement-learning algorithm that InstructGPT, Stiennon et al. and
most early [RLHF](rlhf.md) pipelines use to optimize a policy against a
[reward model](reward-models.md).

CS336's lecture 15 gives it deliberately brief treatment — "I'm only going to
briefly talk about PPO, both in the interest of time and because I'll do a more
extended treatment next lecture" (≈1:05:34) — and [lecture 16](16-post-training-rlvr.md)
delivers that treatment, which is where this page's "In practice" section comes
from. Lecture 15 presents it as the third of
three attempts, each fixing a problem with the previous one. That framing is the
useful thing to carry away, because each step has a clear motivation.

## The objective being optimized

$$\max_{\pi_\theta} \mathbb{E}_{x\sim\mathcal{D},\, y\sim\pi_\theta(y|x)}\left[r_\phi(x,y)\right] - \beta\, \mathbb{D}_{\text{KL}}\left[\pi_\theta(y\mid x)\,\|\,\pi_{\text{ref}}(y\mid x)\right]$$

Maximize the reward model's score on your own samples, minus $\beta$ times the KL
divergence from the reference (post-SFT) policy. The KL term is there because "I
want to stay close to my pre-trained model, because I don't want to go too far
and become degenerate" (≈1:06:21).

The lecture is careful that this is a very easy RL problem by RL standards:
"This is like baby reinforcement learning — people might say we're playing
bandits, that this isn't true multi-turn, interesting RL, and that's roughly
right. So our algorithms will also be quite simple." Each prompt is one episode;
the reward arrives once, at the end.

## Attempt 1 — policy gradients

Differentiate the expected reward and push the gradient inside the expectation:

$$\nabla_\theta E_{p_\theta}[R(z)] = E_{p_\theta}[R(z)\nabla_\theta \log p_\theta(z)]$$

The gradient of the log-probability of a sample, weighted by that sample's
reward. The lecture's gloss is the one to remember: "This really just looks like
SFT, but with weighted examples, effectively" (≈1:07:06).

**The problem:** variance, and cost. The expectation is over your *current*
policy, so every optimization step requires fresh samples. And sampling is the
expensive direction — as the course's systems half established, "inference is
often very complicated and hard, whereas training is very arithmetically
intensive and good" (≈1:07:52). See [inference](inference.md) for why.

## Attempt 2 — off-policy, and TRPO

The fix is to roll out once and reuse those samples for several gradient steps.
That makes the updates **off-policy**: you are computing gradients for a policy
that is no longer the one that generated the data, corrected by an
importance-weighting ratio.

That correction is only trustworthy nearby. "I want to take multiple steps, but
not go too far, because if I go too far, my estimates of my local rewards kind of
blow up." TRPO enforces this as a hard constraint — maximize the
importance-weighted advantage subject to a KL trust region:

$$\underset{\theta}{\text{maximize}} \quad \hat{\mathbb{E}}_t\left[\frac{\pi_\theta(a_t \mid s_t)}{\pi_{\theta_{\text{old}}}(a_t \mid s_t)}\hat{A}_t\right]$$

$$\text{subject to} \quad \hat{\mathbb{E}}_t\big[\mathrm{KL}[\pi_{\theta_{\text{old}}}(\cdot \mid s_t),\, \pi_\theta(\cdot \mid s_t)]\big] \le \delta$$

where $\hat{A}_t$ is the estimated advantage and $\delta$ is the trust-region
radius.

Note that this KL is a *different* KL from the one in the RLHF objective above.
The objective's KL keeps the final policy near the SFT model; this one keeps each
update near the policy that produced the current batch of rollouts. They are
easily confused and do different jobs.

## Attempt 3 — PPO

The constrained problem is awkward to solve. PPO replaces the constraint with a
clipped surrogate — "a heuristic clipping thing that discourages the RL algorithm
from going to places where I'm too far from the original policy" (≈1:08:38):

$$L(s,a,\theta_k,\theta) = \min\left(\frac{\pi_\theta(a|s)}{\pi_{\theta_k}(a|s)}A^{\pi_{\theta_k}}(s,a),\ \ \mathrm{clip}\left(\frac{\pi_\theta(a|s)}{\pi_{\theta_k}(a|s)},\, 1-\epsilon,\, 1+\epsilon\right)A^{\pi_{\theta_k}}(s,a)\right)$$

Taking the *minimum* of the clipped and unclipped terms is what makes this a
pessimistic bound: the objective stops rewarding you for moving the probability
ratio beyond $1\pm\epsilon$ in the helpful direction, while still penalizing you
for moving too far in the harmful one.

## In practice

- **PPO is complex, and that complexity is the reason DPO exists.** "Already the
  equations look a little bit gnarly. And so, for many years, a lot of people have
  asked the following question: can we get rid of PPO?" (≈1:08:38). See
  [DPO](dpo.md).
- **GRPO is the simpler variant CS336's assignment uses.** "Thankfully, we also
  have a simpler variant called GRPO that works pretty well — that's what you'll
  do in your assignments" (≈1:18:41). See [GRPO](grpo.md).
- **Whether PPO beats DPO is genuinely unsettled**, and lecture 15 uses the
  dispute as a lesson in how fragile these comparisons are: AI2 published a result
  favouring PPO and a Tulu 2 result favouring well-executed DPO (≈1:15:36).

## What lecture 16 adds: implementing it is the hard part

[Lecture 16](16-post-training-rlvr.md) returns to PPO at length, and its
argument is that the algorithm's difficulty is not conceptual but practical.
Read from OpenAI's Spinning Up pseudocode, "you look at this and you say, this
is not that bad, this is actually pretty easy, I could implement this in one
go" (≈6:13). The rebuttal is a blog-post title:

> Because if you see a blog post that says "The 37 Implementation Details of
> PPO," you know that this is an algorithm that is very sensitive to your
> implementation decisions. (≈6:59)

Worse than sensitivity: "there are papers saying that the baselines some people
use in PPO aren't even baselines at all, that they fundamentally change the
optimization problem." See
[advantage estimation and baselines](advantage-estimation-and-baselines.md).

Three specifics from that lecture worth carrying:

- **It is not really a bandit problem.** The KL term "actually operate[s] token
  by token. So it's not actually just a bandit problem, it's like a whole
  multi-step RL problem" (≈7:47), which is where much of the machinery comes
  from.
- **A real implementation contains a defensible-looking hack.** In the
  implementation the lecture walks through, the KL penalty is clipped at zero,
  "which, of course, if you know anything about KL divergences, totally ruins
  the point of a KL divergence — you have both positive and negative values
  being summed. If you remove this, it blows up immediately" (≈9:20).
- **GAE is often inert.** The generalized advantage estimator is designed around
  a per-token value function, but "people often just use gamma equals lambda
  equals one, which is just a degenerate setting that turns this back into a
  bandit problem. So you've kind of thrown away a lot of the structure that you
  get from PPO" (≈10:06).

The lecture is careful not to overstate: PPO works, and "many of the labs have
very turnkey solutions for getting PPOs to work at scale. So this is not
impossible, but for people like researchers who are implementing this from
scratch, PPO can just be really finicky and complicated" (≈11:37).

That is the case for [GRPO](grpo.md), which removes the value model entirely.

## See also

- [RLHF](rlhf.md) — the pipeline
- [Reward models](reward-models.md) — what PPO optimizes against
- [DPO](dpo.md) — the attempt to remove PPO entirely
- [GRPO](grpo.md) — PPO with the value function removed
- [Advantage estimation and baselines](advantage-estimation-and-baselines.md)
- [RL infrastructure](rl-infrastructure.md) — the systems cost of the rollout loop
- [Inference](inference.md) — why sampling is the expensive half
- [Lecture 15](15-mid-post-training.md), [Lecture 16](16-post-training-rlvr.md)
- [Course material for lecture 15](../raw/slides/15-mid-post-training.md)
