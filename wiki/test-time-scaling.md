# Test-time scaling

Spending more compute at inference — a longer chain of thought — to get a better
answer. [Lecture 16](16-post-training-rlvr.md) covers the measured version of
this in [Qwen 3](qwen3.md), where the thinking budget is an explicit dial rather
than an emergent behaviour.

## The mechanism

Qwen 3 can be made to stop reasoning on demand: "they also have — I thought this
was quite interesting — this way of early-exiting thinking, where if they append
a special string they'll immediately stop the CoT and the model is forced to give
an answer to whatever prompt is given" (≈58:36). The lecture notes the same
affordance is exposed to end users in interfaces like ChatGPT.

Because the exit is a string rather than a retrained model, you can sweep the
budget and measure one model at many operating points.

## The finding

Two things, and the first is the surprising one:

> as you vary the thinking budget, doing this kind of early-termination trick,
> you find that the performance of the model degrades gracefully — even though,
> at lower thinking budgets, these models are getting truncated mid-thought,
> they're able to give surprisingly reasonable responses even at that point.
> (≈59:22)

A chain of thought cut off in the middle still yields a usable answer. There is
no cliff.

Second, thinking mode dominates instant-response mode across the range: "even
with very small thinking budgets, the thinking-mode models are much better on
all these mathematical or coding tasks, compared to the instant-response mode,
which is much more like the classic instruction-following-plus-RLHF models."

![Slide 53 — Test time scaling](../raw/images/16-post-training-rlvr/slide-53.jpg)
*Slide 53 — accuracy against thinking budget under the early-termination trick.*

## Why it matters

**Operationally**, it means one model serves a spectrum of cost-quality points.
That is what makes the [long chain-of-thought](long-chain-of-thought.md) cost
problem manageable: you do not have to choose the budget at training time.

**Interpretively**, it complicates the story that longer reasoning is where the
capability lives. If most of the benefit survives severe truncation, then a
large part of what the thinking mode buys is present early in the chain. Set
this beside [length bias in RL](length-bias-in-rl.md), where a good deal of
observed length growth turns out to be an artifact of the objective, and the
picture is consistent: chain-of-thought *length* and reasoning *quality* are
much less tightly coupled than the early narrative suggested.

## The related cost

Test-time scaling is a scaling law in inference rather than training, which puts
it alongside the course's [inference](inference.md) unit: reasoning tokens are
decode-bound, sequential, and paid for per request. Kimi's argument for
compressing chains of thought (≈47:02) is the other side of the same coin.

## See also

- [Long chain-of-thought](long-chain-of-thought.md), [reasoning models](reasoning-models.md)
- [Thinking-mode fusion](thinking-mode-fusion.md) — how one model holds both modes
- [Qwen 3](qwen3.md) — where this is measured
- [Inference](inference.md) — what the tokens cost
- [Lecture 16](16-post-training-rlvr.md)
