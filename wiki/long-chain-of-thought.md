# Long chain-of-thought

The extended intermediate reasoning a [reasoning model](reasoning-models.md)
produces before its answer. In CS336 it appears as three separate things, and
keeping them apart is the point of this page: a **capability** to be acquired, a
**cost** to be managed, and an **artifact** that can be mistaken for progress.

## As a capability

Long CoT is what [RLVR](rlvr.md) buys. [DeepSeek-R1](deepseek-r1.md)'s recipe
gives the model a format reward that forces it to enclose reasoning in thinking
tags, and the format reward exists because it lets them "strip out the chain of
thought later" (≈28:32).

It is also acquirable by imitation, not only by RL: "for a very good base model,
just SFT on long CoT can unlock a lot of o1-style capabilities, and that's a
great starting point for RL" (≈33:56). See
[reasoning distillation](reasoning-distillation.md).

Where it comes from in the pipeline is slightly unsettled. Asked whether long
reasoning is part of [midtraining](midtraining.md), the lecturer says long-CoT
SFT appears in both R1 and Kimi K1.5, then adds: "Long CoT is not traditionally
part of mid-training per se, but long-CoT data is often used in long-context
extension" — a phase "right before RLHF," built from "books, code and synthetic
data, because those are the things that are long enough to do extension on"
(≈1:14:04). He also flags the gap in his own coverage: "I didn't talk about
long-context extension at all, and I regret that."

## As a cost

Every reasoning token is paid for at inference. [Kimi K1.5](kimi-k1-5.md) makes
this the design driver — "we don't really want long CoTs — CoTs cost inference"
— and states the economics in terms of a subscription:

> If you're OpenAI, and your users have the $200 Pro plan, and your models are
> thinking for an hour at a time, that's not a very good place to be in. Whereas
> if your models are thinking for five minutes at a time, that is a great place
> to be in. (≈47:02)

Hence a length reward that compresses chains of thought, with the asymmetry that
wrong answers must not be compressed to nothing or the model can never recover
in a weak domain (≈48:33).

It is also a systems cost: one long rollout stalls a whole batch. See
[RL infrastructure](rl-infrastructure.md).

## As an artifact

The trap. R1's report shows CoT length rising steadily through training, and it
was widely read as the model learning to reason more deeply. Much of it is the
objective: [GRPO](grpo.md)'s length normalizer rewards long *wrong* answers, and
removing it makes length "cap off at a constant, rather than continually and
forever growing" (≈24:39).

The disaggregated evidence is Dr. GRPO's plot, which separates average, correct
and incorrect output lengths and shows the growth "is really being driven by the
incorrect ones" (≈39:20). See [length bias in RL](length-bias-in-rl.md).

**So growing CoT length is not by itself evidence of improving reasoning.**
That is the sentence to carry away.

## As a dial

[Qwen 3](qwen3.md) turns the budget into a control: a special string terminates
thinking immediately, and accuracy "degrades gracefully" as the budget shrinks,
with thinking mode beating instant-response mode on maths and code "even with
very small thinking budgets" (≈59:22). See
[test-time scaling](test-time-scaling.md).

## See also

- [Reasoning models](reasoning-models.md), [RLVR](rlvr.md), [GRPO](grpo.md)
- [Length bias in RL](length-bias-in-rl.md) — the artifact, in detail
- [Test-time scaling](test-time-scaling.md), [thinking-mode fusion](thinking-mode-fusion.md)
- [RL infrastructure](rl-infrastructure.md) — why long rollouts hurt
- [Lecture 16](16-post-training-rlvr.md)
