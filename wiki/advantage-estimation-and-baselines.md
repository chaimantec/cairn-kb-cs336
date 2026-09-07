# Advantage estimation and baselines

In policy-gradient RL you push up the probability of actions that did better
than expected. "Better than expected" is the *advantage*, and how you estimate
it is most of what separates one algorithm from another.
[Lecture 16](16-post-training-rlvr.md) treats this carefully, because
[GRPO](grpo.md)'s advantage is the lecture's main technical criticism.

## The starting point

Everything descends from the REINFORCE gradient. For a policy $p_\theta$ over
completions $z$ with reward $R(z)$:

$$\nabla_\theta \mathbb{E}_{p_\theta}[R(z)] = \mathbb{E}_{p_\theta}\left[R(z)\,\nabla_\theta \log p_\theta(z)\right]$$

The lecture's gloss is the one to keep, because it makes the whole family
intuitive: "What we are always going to be doing is gradient descent on our
rewards, and we're going to do so by taking essentially weighted SFT updates,
where the weights might be positive or negative" (≈3:54).

Used raw, this estimator has very high variance — you are weighting by the raw
reward, which has an arbitrary offset.

## What you are allowed to subtract

The classical fix is REINFORCE with baseline, and the lecture goes to Sutton and
Barto for the licence:

> I want to take my gradient step in the direction of my rewards, and the thing
> you can do is take your rewards and subtract any what they call a
> state-dependent baseline. In this case, our state, if we're in the bandit
> world, is just the prompt, so you can have a prompt-dependent baseline that
> you subtract, and anything that does just this is a valid form of a policy
> gradient. (≈20:47)

Two consequences:

- **Any** state-dependent $b$ leaves the gradient direction unbiased — "you're
  still going to descend in the same direction as long as you do this."
- The *choice* of $b$ only changes variance: "depending on your choice of b,
  you'll have either lower or higher variance in doing this gradient-descent
  process" (≈21:33).

That is the contract. Subtracting is free; anything else is not.

![Slide 23 — GRPO doesn't use a "valid" baseline](../raw/images/16-post-training-rlvr/slide-23.jpg)
*Slide 23 — the Sutton and Barto baseline result beside GRPO's actual advantage, with the offending division marked in red.*

## Three ways to get a baseline

**A learned value function — [PPO](ppo.md).** A network predicts the expected
return for the state and you subtract it. Correct, and expensive: the value
model — asked how big it is, the answer is that "it's as big as the original
model" (≈11:37). It destabilizes training, and it is the single largest source
of PPO's implementation difficulty.

**Generalized advantage estimation (GAE) — PPO, in principle.** GAE interpolates
between low-variance, high-bias and high-variance, low-bias estimates using
$\gamma$ and $\lambda$, with a value function estimating reward at every token.
In practice for language models, the lecture reports that this structure is
usually thrown away: "people often just use gamma equals lambda equals one,
which is just a degenerate setting that turns this back into a bandit problem.
So you've kind of thrown away a lot of the structure that you get from PPO"
(≈10:06). Worth knowing when reading a PPO codebase — the GAE machinery may be
present and inert.

**The group mean — [GRPO](grpo.md).** Sample $G$ completions for one prompt and
use their mean as the baseline. No network, no extra memory, and the baseline is
prompt-dependent by construction, so it satisfies the contract.

## Where GRPO steps outside the contract

GRPO does not stop at subtracting the mean. It also **divides by the group
standard deviation**, and separately **normalizes by sequence length**. Neither
is a baseline, and neither falls out of the derivation: "if you try to derive
GRPO from first principles, following the policy-gradient and baseline theorems,
you'll end up with something different — you won't have a length normalizer, and
you won't have the standard-deviation normalization" (≈22:20).

So GRPO "does not" descend the reward objective as written (≈21:33). The two
extra terms have real effects — a bias toward
[long wrong answers](length-bias-in-rl.md), and an upweighting of problems that
are always solved or never solved — and both are covered on their own pages.

The lecture's verdict is balanced rather than dismissive: GRPO is "not the
first-principles derivation of this idea, it's actually doing something slightly
different, with both pros and cons" (≈23:06).

## A convergent check

[Kimi K1.5](kimi-k1-5.md) derives its update from a completely different
starting point — a [DPO](dpo.md)-style analytic solve — and arrives at "a policy
gradient with a baseline, r-bar — and that r-bar is actually the mean,
conditioned on each of these x's" (≈45:30), plus a KL term. Two derivations
landing on the group mean is good evidence that the mean is the right baseline
and the extra normalizations are the optional part.

## See also

- [GRPO](grpo.md) — where this analysis lands
- [PPO](ppo.md) — the value-function approach
- [Length bias in RL](length-bias-in-rl.md) — the length normalizer's cost
- [Kimi K1.5](kimi-k1-5.md) — the convergent derivation
- [Lecture 16](16-post-training-rlvr.md)
