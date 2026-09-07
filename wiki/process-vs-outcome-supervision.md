# Process versus outcome supervision

Two ways to reward a model that reasons before answering.

- **Outcome supervision** rewards only the final answer: correct or not.
- **Process supervision** rewards the intermediate steps, using a rubric or a
  *process reward model* (PRM) that grades a chain of thought line by line.

Process supervision is intuitively better — it tells the model *where* it went
wrong rather than only *that* it did — and for a while it looked like the
frontier. [Lecture 16](16-post-training-rlvr.md) records that it largely lost.

## The reversal

DeepSeekMath, where [GRPO](grpo.md) was introduced, used process supervision and
found it helped: the lecture's reading of its results is that "process
supervision — grading not just the final answer but grading the intermediate
steps — gives you some gains" (≈19:14).

[DeepSeek-R1](deepseek-r1.md) then abandons it. The lecturer flags this as one
of the paper's most important differences, precisely because it reverses their
own earlier finding: they "abandon process supervision, which was the thing that
worked really well in DeepSeekMath, and go only with outcome supervision"
(≈27:46), and "a lot of people thought process supervision was important; turned
out it wasn't critical for a lot of things" (≈28:32).

Reading R1 after DeepSeekMath, the question is where the PRMs went, and the
report answers it:

> they tried to get process reward models to work, and they just didn't do very
> much for them. It turns out that outcome reward models are great, they're good
> enough, and you can scale the data for those a lot better. (≈37:00)

## Why outcome supervision won

**Scalability of the supervision, not accuracy of it.** The binding question for
PRMs is "where are you going to get these step-by-step rubrics? And it's very
hard to scale that up" (≈37:00). An outcome reward needs only an answer key; a
process reward needs a graded rubric for every intermediate step of every
problem. The lecturer's summary: "at this point it's quite clear that outcome
reward models are very, very good, and that's the bulk of where the action is
happening."

This is the same shape as several other results in CS336 — a method that is
better per example loses to one that is worse per example and vastly easier to
scale.

## The related negative result: tree search

The same passage records that MCTS also failed. After o1's release "a lot of
people speculated about what was going on inside OpenAI's o1 — they were like,
oh, are they doing PRMs, are they doing tree search like AlphaGo? And, you know,
they too tried a lot of MCTS, and they describe that they couldn't get it to
work very well" (≈37:47).

The lecture makes a point of praising the reporting rather than only the result:
"I just want to highlight the fact that they're very open about all these
explorations and the things they tried that didn't work, rather than saying they
didn't try it at all."

## See also

- [DeepSeek-R1](deepseek-r1.md) — where the reversal is recorded
- [Reward models](reward-models.md), [verifiable rewards](verifiable-rewards.md)
- [GRPO](grpo.md), [RLVR](rlvr.md)
- [Lecture 16](16-post-training-rlvr.md)
