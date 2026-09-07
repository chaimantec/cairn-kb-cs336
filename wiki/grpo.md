# GRPO — group relative policy optimization

GRPO is [PPO](ppo.md) with the value function deleted. Instead of learning a
network to predict the expected reward of a prompt and using it as a baseline,
GRPO samples several completions for the same prompt and scores each one
against its siblings. It is the algorithm almost all open [RLVR](rlvr.md) work
is built on, and the one CS336's assignment implements.

It comes from DeepSeek's DeepSeekMath paper and is developed in
[lecture 16](16-post-training-rlvr.md).

## The motivation is entirely practical

The lecture is blunt that GRPO exists because PPO is painful: "there's just an
enormous desire by the research community not to have to use PPO. Hopefully the
fact that DPO and GRPO have gotten adoption tells you how painful it was to get
this to work in many cases" (≈13:10).

The specific target is the value function. It is "arguably the most complicated
and annoying part of PPO... the value function is a whole neural network, it
destabilizes training, we don't want the value network" (≈13:56). It also costs
memory — asked how big the value model is, the answer is that "it's as big as
the original model. So this consumes some memory that you'd rather be using for
other stuff, like models or inference servers" (≈11:37).

But you cannot simply drop the baseline — plain REINFORCE has variance that is
"going to be really, really high."

## The idea

Replace the model-predicted baseline with the *empirical mean of siblings*:

> say you get your rollout with a score of five, you now sample ten other
> rollouts, and you say, how good was I compared to my ten other rollouts? If
> I'm doing better than my mean, then I have a high advantage. (≈14:42)

For a group of $G$ completions $o_1,\dots,o_G$ sampled from one prompt, with
reward $r_i$ for completion $i$, the advantage is the z-score within the group:

$$A_i = \frac{r_i - \operatorname{mean}(r_1,\dots,r_G)}{\operatorname{std}(r_1,\dots,r_G)}$$

This is substituted into the PPO objective, keeping the min-clipped
probability ratio and a KL penalty to the reference policy $\pi_{\text{ref}}$.

**On-policy, the clipping disappears.** This is the simplification that makes
GRPO a half-page of code: "In the online case, the clipping just kind of
disappears, because the ratio between pi-theta-old and pi-theta is one — this
clipping operator never does anything. So you just get min(A of i, A of i), so
this is just advantage minus a KL penalty" (≈16:12).

## Implementing it

The steps are: compute a reward for each of $K$ rollouts, normalize within the
group, compute the KL term, and take gradient updates on the combination
(≈16:57–17:44). The one subtlety is autodiff — "in order to do this using
autodiff you'll have to do a stop-grad somewhere."

One numerical detail from the reference implementation the lecture walks
(McGill's `nano-aha-moment`): add $10^{-4}$ to the standard deviation, "to
prevent it from blowing up when you only have a single sample, or if your
samples happen to have the exact same rewards — which does happen if you're in a
domain where you can get exactly, numerically, zero rewards, like you failed to
solve a math problem" (≈18:29).

## It is not a valid policy gradient, and this matters

The [policy-gradient theorem](advantage-estimation-and-baselines.md) lets you
subtract any state-dependent baseline and still descend the true reward. GRPO
does two things beyond that, and neither survives a first-principles derivation.

**Dividing by the standard deviation** breaks the baseline contract: "If you
really want a conceptually clear algorithm that does what's written on the tin,
that actually descends the reward, GRPO does not do that" (≈21:33).

**Normalizing by sequence length** — "almost per-token — they'll divide by the
total length of the sequence as a normalization factor" (≈22:20).

Each has a behavioural consequence:

- The length normalizer produces a bias toward **long wrong answers**, because
  dividing a negative reward by a larger length shrinks the penalty. See
  [length bias in RL](length-bias-in-rl.md). Remove it and runaway
  chain-of-thought growth "actually turns out to just cap off at a constant"
  (≈24:39) — which is why R1's growing-length plot should not be read as the
  model getting smarter.
- The standard-deviation term **upweights problems the model always gets right
  or always gets wrong**, since both have near-zero reward variance, "and that
  seems like clearly a thing we maybe don't want, because we want our models to
  learn on things that are within its solvability range" (≈25:27).

Papers written soon after GRPO — Dr. GRPO among them — remove both terms, and
the lecture treats their controlled plots as the cleaner evidence about what the
length normalizer was doing (≈39:20).

None of this makes GRPO useless. The lecture's position is that it is "not the
first-principles derivation of this idea, it's actually doing something slightly
different, with both pros and cons" (≈23:06) — and that its simplicity is what
let the open community reproduce reasoning models at all.

## Convergent evidence

[Kimi K1.5](kimi-k1-5.md) derives its update from a DPO-style analytic argument
rather than from PPO, and lands in nearly the same place: "we've reinvented the
group-mean-normalized baseline through quite different means" (≈46:16). Two
unrelated derivations arriving at the group mean is the strongest evidence in
the lecture that this is the load-bearing idea rather than an arbitrary choice.

## See also

- [PPO](ppo.md) — what it simplifies, and why that was worth doing
- [Advantage estimation and baselines](advantage-estimation-and-baselines.md)
- [Length bias in RL](length-bias-in-rl.md)
- [RLVR](rlvr.md) — the setting GRPO is used in
- [DPO](dpo.md) — the other attempt to escape PPO, for a narrower problem
- [Kimi K1.5](kimi-k1-5.md) — the convergent derivation
- [Lecture 16](16-post-training-rlvr.md)
