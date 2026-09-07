# RLVR — reinforcement learning from verifiable rewards

RLVR is reinforcement learning where the reward comes from *checking the answer*
rather than from a learned model of human preference. A math problem has a
correct answer; a program either passes its tests or does not. Where such a
check exists, you can replace the [reward model](reward-models.md) that
[RLHF](rlhf.md) depends on with something that cannot be overfitted in the same
way — and that changes how much compute the method can absorb.

CS336 develops this in [lecture 16](16-post-training-rlvr.md).

## Why it exists: RLHF has a compute ceiling

[Lecture 15](15-mid-post-training.md) ends by arguing that RLHF is
annotation-bottlenecked. You collect preferences, fit a reward model, and
optimize against it — but "you can't keep putting compute into the same reward
model. Eventually you're going to overfit your reward model. And no matter how
good a job you do at regularizing, eventually you're going to run into this
problem with overfitting" (≈1:36). That is
[reward overoptimization](reward-overoptimization.md), and no amount of KL
regularization removes it, because the problem is the proxy, not the tuning.

The contrast the lecture uses is AlphaGo:

> in AlphaGo, we are optimizing exactly what we want. You get the win-loss
> conditions of the game of Go — you don't have any sort of sloppiness to that
> definition. So you can just put in as much compute as you want, and as long as
> the objective improves, you're doing well. (≈2:22)

The lecturer offers "search problems" versus "learning problems" as a way to
name the difference, but explicitly hedges it as "not quite a precise
distinction."

So the question becomes whether any *language* domain has that property.
Mathematics — formal or natural-language — and code plausibly do, since both
admit an external check. That is the whole bet.

## What actually changes, and what does not

Almost nothing changes in the algorithm. "The algorithms aren't going to be that
different fundamentally, but where we will end up will actually be surprisingly
different" (≈2:22). In practice RLVR means:

- a [GRPO](grpo.md)-style policy-gradient update, usually run on-policy;
- a reward that is mostly a correctness check, plus small shaping terms
  (format, language consistency, length);
- heavy investment in *data difficulty*, because a problem the model always
  fails and a problem it always solves both carry no gradient signal.

What changes is the ceiling. Because the verifier does not overfit the way a
learned reward model does, you can keep scaling RL compute — which is what
produced the long-chain-of-thought reasoning models.

## The catch: "verifiable" is weaker than it sounds

This is the part of the lecture most worth carrying, because the term promises
more than it delivers. See [verifiable rewards](verifiable-rewards.md) for the
full treatment; two facts summarize it.

**Even math ends up using a reward model.** Kimi checks mathematical answers for
equivalence with a model, and the lecturer draws the irony out deliberately:

> we started out this lecture by saying we want to work on formal math, or
> something truly verifiable, where a compiler can check the correctness of your
> math, and we've gone through most of the lecture, and then, in the end, where
> have we ended up? Well, we ended up with a reward model — a reward model that
> checks the correctness of math answers. (≈50:05)

**And real verifiers are attackable.** The lecturer's own group found that
"the Lean compiler is not adversarially robust. There are strings that you can
put in it that will allow you to verify proofs that are not meant to be
verified, in certain modes" (≈1:07:52).

Hence the summary line, which is the honest statement of what RLVR guarantees:

> RLVR is only as robust as your reward, and your rewards can sometimes be not
> very robust at all. (≈1:07:06)

See [reward hacking](reward-hacking.md).

## Where it has got to

Three open recipes are read side by side in the lecture, and they have largely
converged: [DeepSeek-R1](deepseek-r1.md), [Kimi K1.5](kimi-k1-5.md) and
[Qwen 3](qwen3.md). The shared shape is base model → SFT on long chains of
thought → reasoning RL with verifiable rewards → RLHF for the non-verifiable
remainder. [Agentic RL](agentic-rl.md) extends the same machinery to software
engineering, where the verifier is a test suite and the environments are
generated from GitHub at scale.

## See also

- [GRPO](grpo.md) — the algorithm nearly all open RLVR uses
- [PPO](ppo.md) — what it simplifies
- [Verifiable rewards](verifiable-rewards.md) — what "verifiable" actually buys
- [Reward hacking](reward-hacking.md) — what happens when it does not hold
- [Reward overoptimization](reward-overoptimization.md) — the problem RLVR answers
- [RLHF](rlhf.md) — the method it extends
- [Reasoning models](reasoning-models.md), [long chain-of-thought](long-chain-of-thought.md)
- [Lecture 16](16-post-training-rlvr.md)
