# Reasoning models

Models trained to produce a long chain of thought before answering, and rewarded
on whether the answer is right. o1 was the first widely known example;
[DeepSeek-R1](deepseek-r1.md), [Kimi K1.5](kimi-k1-5.md) and
[Qwen 3](qwen3.md) are the open reproductions
[lecture 16](16-post-training-rlvr.md) reads.

They are the destination of the whole course arc: "We've got instruction tuning.
We've got RLHF. We got some good stuff — that's ChatGPT. And what remains is the
modern developments in thinking models: the model's ability to do very long CoT
and solve these hard, verifiable problems, like mathematics and coding in some
cases as well" (≈0:50).

## What defines one

Not an architecture — the models are ordinary transformers. What distinguishes
them is the post-training recipe, and by the end of lecture 16 the three open
reports have converged on essentially one:

1. A mid-trained base model.
2. SFT on long chains of thought, often distilled from a stronger model.
3. RL with [verifiable rewards](verifiable-rewards.md), usually
   [GRPO](grpo.md), on maths and code.
4. [RLHF](rlhf.md) at the end for everything not verifiable.

The lecture's framing of what o1 established (≈27:00): very long chains of
thought, clearly produced by RL, with strong performance on hard maths.

## The two claims that did not survive scrutiny

Because the field's early narrative came largely from R1's report, the lecture
spends real time on what that report's most-shared findings actually show.

**Chains of thought grow during training.** Read as the model learning to think
harder; substantially an artifact of GRPO's length normalizer, since removing it
makes length "cap off at a constant" (≈24:39). See
[length bias in RL](length-bias-in-rl.md).

**The "aha moment."** Read as emergent self-correction; but it is one that
"others have shown, actually appears even in the base model, so clearly it can't
just be a result of the RL algorithm" (≈30:51). Pre-training saw plenty of
humans writing "aha" mid-derivation.

The lecturer's overall verdict: "the phenomena that were highlighted aren't
particularly salient or exciting, but I do think R1 was a really important
milestone, in that it highlighted just how simple RLVR could be" (≈30:51).

## Is RL necessary?

Genuinely open, and the lecture says so. [Reasoning
distillation](reasoning-distillation.md) shows that SFT on good chains of
thought recovers much of the capability — "if you have the right kind of
distillation procedure and the right kind of base model, you can basically get a
lot of the long-CoT reasoning juice just from SFT" (≈33:56) — which raises
"one of the things that's still an open question is: do you really need RL for
some of this?"

The framing offered as an answer is about *supervision*, not optimization: RL
"is a great source of supervision" where nobody can demonstrate the answer, and
imitation can spread what RL discovers (≈34:42).

## Thinking as a budget

Later reasoning models expose the chain of thought as a dial rather than a fixed
behaviour — [Qwen 3](qwen3.md) fuses thinking and non-thinking modes into one
model and can terminate thinking early with a special string. Performance
"degrades gracefully" as the budget shrinks (≈59:22). See
[test-time scaling](test-time-scaling.md) and
[thinking-mode fusion](thinking-mode-fusion.md).

## See also

- [Long chain-of-thought](long-chain-of-thought.md)
- [RLVR](rlvr.md), [GRPO](grpo.md)
- [DeepSeek-R1](deepseek-r1.md), [Kimi K1.5](kimi-k1-5.md), [Qwen 3](qwen3.md)
- [Reasoning distillation](reasoning-distillation.md)
- [Reasoning benchmarks](reasoning-benchmarks.md) — how they are measured
- [Lecture 16](16-post-training-rlvr.md)
