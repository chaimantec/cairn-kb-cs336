# Thinking-mode fusion

Putting a reasoning model and a non-reasoning model into **one** set of weights,
switched by a tag in the prompt. [Qwen 3](qwen3.md)'s contribution, covered in
[lecture 16](16-post-training-rlvr.md) — and one the field has since partly
walked back, which is what makes it worth a page.

## What it is

Thinking and non-thinking data are mixed with tags during training, so that
"both the instant-response model and the long-CoT model basically live in the
same model, and this wasn't true in many cases — there was often a thinking mode
and a non-thinking model, even at OpenAI" (≈58:36).

The switch is in the prompt, not the serving stack. Asked about this directly, the
lecturer is precise about where the control lives:

> the interesting thing about thinking mode really is that it's actually one
> model, and there's just a little prompt tag that switches them between long
> and short CoT — long CoT and no-CoT almost mode. Whereas if you just had an
> API flag or something, that's not very difficult. The fact that they were
> putting both of them in together is kind of the interesting bit there.
> (≈1:10:11)

That distinction is the whole point: two models behind an API flag is routing;
one model with two behaviours is a training result.

## What it costs

Slide 54 measures each pipeline stage separately for thinking and non-thinking
modes. Fusion is not free — general tasks improve while maths and coding take a
small hit "because we fuse together non-thinking components, but the degradation
isn't so bad" (≈1:00:09).

![Slide 54 — Composition of the different stages](../raw/images/16-post-training-rlvr/slide-54.jpg)
*Slide 54 — benchmarks across the four stages, with thinking and non-thinking columns given separately and colour-coded deltas against the previous stage.*

## And the field reversed on it

The most useful thing on this page, because it is the kind of detail that dates
a technical report:

> even though these numbers are small in an absolute sense, in later releases —
> I think in some of the Qwen 3.5s — they've gone back on fusing both thinking
> and non-thinking into a single model. I think — they used to call these hybrid
> models — because they found this kind of drop kind of unacceptable, they
> wanted to squeeze out all the juice possible on thinking modes, and so now I
> think they've separated these models from each other. (≈1:00:55)

The lecturer hedges the specifics ("I think"), so treat the direction as the
finding rather than the release details. The reasoning is clear enough: when the
thinking mode is the product, a few points of maths and coding traded for
convenience is a bad trade.

## Related, but not the same thing

[Test-time scaling](test-time-scaling.md) via early exit is a *different* knob
that Qwen 3 also has — it truncates the chain of thought within thinking mode,
rather than switching modes. Fusion is about which behaviours coexist in the
weights; early exit is about how much budget a request gets.

## See also

- [Qwen 3](qwen3.md) — the pipeline this sits in
- [Test-time scaling](test-time-scaling.md) — the other budget control
- [Long chain-of-thought](long-chain-of-thought.md), [reasoning models](reasoning-models.md)
- [Lecture 16](16-post-training-rlvr.md)
