# Reasoning distillation

Take a reasoning model's chains of thought, fine-tune a smaller or
non-reasoning model on them, and get much of the reasoning ability without
running RL yourself. One of [DeepSeek-R1](deepseek-r1.md)'s findings that
[lecture 16](16-post-training-rlvr.md) says has "held up, even now" (≈27:00).

## The result

R1's chains of thought, used as SFT data for Qwen2.5, "really significantly
boost the performance of these models, in some cases matching a lot of these
specialized thinking models" (≈36:15). It generalizes across model families —
"this works even for Llama models, which I think is quite fascinating."

The condition is legibility and base-model quality: "if you can get the right
kind of long CoTs that are legible to these models, it's often possible to get
them to reason for long periods of time — the base models are already
surprisingly good."

The lecturer adds that this is not only DeepSeek's finding: "many papers since
then — some, including some from my students — have shown that if you have the
right kind of distillation procedure and the right kind of base model, you can
basically get a lot of the long-CoT reasoning juice just from SFT" (≈33:56).

## Why it matters conceptually

It puts a question mark over the necessity of RL: "one of the things that's
still an open question is: do you really need RL for some of this?" (≈33:56).

The framing offered is the most useful idea on this page — **RL's role may be as
a source of supervision rather than as an optimizer**:

> maybe one way of thinking about the role of RL in language modeling is that RL
> is a great source of supervision — if you're solving frontier math problems,
> you just don't have the supervision to get detailed long CoTs, and RL allows
> you to self-generate that. But once someone has generated these long CoTs, you
> could potentially also learn from imitation, and that seems to be what a lot
> of these distillation results partially show. (≈34:42)

On that reading, RL is how the first model gets there — nobody can write down a
worked chain of thought for a frontier maths problem — and imitation is how
everyone else follows cheaply. Set against
[expert iteration](expert-iteration.md), where RL beats positives-only training
head to head, the resolution is that RL wins when you are extending the frontier
and imitation is enough when someone already has.

## It is also how the models you use get built

Distillation is a production step, not only a research finding.
[Qwen 3](qwen3.md)'s pipeline ends with it — the shipped large model is distilled
down to the smaller ones actually served (≈56:18) — and
[agentic RL](agentic-rl.md) uses it to fold four separately trained expert models
back into one.

## Reading the reports carefully

The lecture flags a habit worth having when reading tech reports. R1 says it
"construct[s] and collect[s] a small amount of long CoT data," and the lecturer
reads between the lines: "you might wonder, I wonder if that was distilled from
some other model. Not that that's a particularly negative thing for open-source
models — everyone nowadays is doing this — but I just find it funny that it's
very carefully written" (≈33:10).

## See also

- [DeepSeek-R1](deepseek-r1.md) — where the result comes from
- [Expert iteration](expert-iteration.md) — the head-to-head comparison
- [Long chain-of-thought](long-chain-of-thought.md), [reasoning models](reasoning-models.md)
- [Supervised fine-tuning](supervised-fine-tuning.md)
- [Qwen 3](qwen3.md), [agentic RL](agentic-rl.md) — distillation in production
- [Lecture 16](16-post-training-rlvr.md)
