# Qwen 3

The third case study in [lecture 16](16-post-training-rlvr.md), and the one to
keep as a mental model of how a modern open model is assembled. Where
[DeepSeek-R1](deepseek-r1.md) is the clean experiment and
[Kimi K1.5](kimi-k1-5.md) is the alternative derivation, Qwen 3 is the
consolidation: "Qwen uses basically the, at this point, tried-and-tested
playbook for RLVR. I think they take a lot of the best parts of Kimi and
DeepSeek and get it right" (≈57:04).

## The pipeline

Base model → SFT → reasoning RL → thinking-mode fusion → RLHF → ship, with
distillation afterwards to produce the smaller models actually served (≈56:18).

![Slide 50 — Overall picture](../raw/images/16-post-training-rlvr/slide-50.jpg)
*Slide 50 — the whole pipeline on one page.*

The lecturer recommends this specifically as the picture worth carrying: "you
can basically have this be your mental picture of how frontier-ish language
models are built, putting all the components together, and I think you'd have a
pretty reasonable picture of what's happening" (≈56:18).

## Data selection, and how little RL data is needed

Qwen 3's filtering consolidates what the other two reports found (≈57:04):

- Difficulty filtering, because it saves compute.
- Removing problems the model gets right **without** a chain of thought — "we
  know that's not a thinking problem."
- Removing items too similar to validation data, to decontaminate.
- A manual pass on reference chains of thought.

Then the number that surprises people: **RL runs on roughly 4,000 examples.**
The lecture's reading is that the pipeline carries the weight, not the RL corpus:
"once again, if you have the rest of the pipeline right, you can get
surprisingly far" (≈57:50). Set this beside
[Kimi's](kimi-k1-5.md) difficulty curriculum and the point is the same — RL data
is about *which* problems, not how many.

## Thinking-mode fusion

The Qwen-3-specific contribution, and the lecturer flags that some of it was
later abandoned while "some of the ideas are actually quite interesting"
(≈57:50). Thinking and non-thinking behaviour are mixed with tags so that both
"basically live in the same model, and this wasn't true in many cases — there
was often a thinking mode and a non-thinking model, even at OpenAI" (≈58:36).
See [thinking-mode fusion](thinking-mode-fusion.md).

Alongside it is an **early-exit mechanism**: appending a special string stops the
chain of thought immediately and forces an answer. The lecture notes the same
affordance is exposed in consumer interfaces like ChatGPT (≈58:36).

## Test-time scaling, measured

The early-exit trick turns the thinking budget into a dial, which produces the
lecture's [test-time scaling](test-time-scaling.md) result: performance
"degrades gracefully — even though, at lower thinking budgets, these models are
getting truncated mid-thought, they're able to give surprisingly reasonable
responses even at that point," and thinking mode beats instant-response mode on
maths and coding "even with very small thinking budgets" (≈59:22).

![Slide 53 — Test time scaling](../raw/images/16-post-training-rlvr/slide-53.jpg)
*Slide 53 — accuracy against thinking budget under early termination.*

## What each stage buys, and what fusion costs

Slide 54 breaks results down by stage, with colour-coded deltas against the
previous stage. Reasoning RL followed by general RL improves general tasks
"across the board" — Arena-Hard, CounterFactQA and similar — while maths and
coding take a small hit "because we fuse together non-thinking components, but
the degradation isn't so bad" (≈1:00:09).

![Slide 54 — Composition of the different stages](../raw/images/16-post-training-rlvr/slide-54.jpg)
*Slide 54 — every benchmark across the four stages, thinking and non-thinking columns separately.*

**The field has since reversed on fusion.** In later releases "they've gone back
on fusing both thinking and non-thinking into a single model... because they
found this kind of drop kind of unacceptable, they wanted to squeeze out all the
juice possible on thinking modes, and so now I think they've separated these
models from each other" (≈1:00:55). Worth knowing before treating fusion as
settled practice.

## Qwen Coder-Next

The agentic successor is covered separately in [agentic RL](agentic-rl.md). The
lecturer recommends the report specifically: "if you're interested in how you
actually build an agent, this is probably a good report to go read" (≈1:00:55).

**A naming note.** The lecturer calls this model "Qwen 3.5 Next Coder" at
≈55:31 and immediately adds "the names are getting complicated"; at ≈1:00:55
he corrects the word order to "Coder-Next." Slide 55's title card prints
**Qwen3-Coder-Next**. The transcript keeps what he said and flags the
difference rather than silently renumbering it.

## See also

- [Agentic RL](agentic-rl.md) — Qwen Coder-Next in detail
- [Thinking-mode fusion](thinking-mode-fusion.md), [test-time scaling](test-time-scaling.md)
- [DeepSeek-R1](deepseek-r1.md), [Kimi K1.5](kimi-k1-5.md)
- [RLVR](rlvr.md), [GRPO](grpo.md)
- [Lecture 16](16-post-training-rlvr.md)
