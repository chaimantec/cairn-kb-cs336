# Agentic RL

Post-training a model to act as an agent — to run tools, edit repositories and
close GitHub issues. New material in CS336's 2026 offering
([lecture 16](16-post-training-rlvr.md), ≈26:14), taught from Qwen's
Coder-Next report.

The first claim is deflationary, and it is the point of the section:

> post-training for an agent is not very different from everything that we've
> described. It's not like there's a new agent-training algorithm. Really, you
> should internalize this lesson throughout the entire class: data is the
> important thing. (≈1:01:43)

There is no agentic algorithm. There is [RLVR](rlvr.md) with
[GRPO](grpo.md)-style updates, pointed at environments where a test suite
supplies the reward, and an enormous amount of data engineering in front of it.

## Capability goes in early, at midtraining

Agentic ability cannot be bolted on at the end — "You can't just inject them at
the end — you probably have to start a little bit earlier" (≈1:01:43). So there is an
extensive [midtraining](midtraining.md) phase, and its sources are worth
listing because they show what "agent data" concretely means
(≈1:02:29–1:03:15):

- **Repository-level long-context data** — files in a repo concatenated, because
  "the model is eventually going to see these very long CoT traces where maybe
  the agent has opened a bunch of files."
- **Pull requests**, with synthetic context assembled by RAG to make the change
  comprehensible.
- **Text-and-code documents** detected automatically and converted to clean
  Markdown by an LLM.
- **[Synthetic](synthetic-data.md) coding QA** generated from coding-related web
  documents.
- **Agent trajectories** — real coding agents run against constructed
  environments, with the traces fed back in.
- Instruction-following and fill-in-the-middle data.

(Slide 56 lists these in full. It is one of the six pages in this deck carrying
no figure, so it has no image — its content is entirely in
[the slide file](../raw/slides/16-post-training-rlvr.md).)

## Expert models, then distillation back

The part the lecturer had not seen before. Rather than training one model on
everything, Qwen trains **four separate experts** — web dev, UX, single-turn QA,
and software engineering — with full RL and/or SFT on each subtask, then
distils all four back into a single coder model (≈1:04:00–1:04:46).

![Slide 57 — Expert models](../raw/images/16-post-training-rlvr/slide-57.png)
*Slide 57 — fan out from Qwen 3 Next into four experts, then distil back into Qwen 3 Next Coder.*

The lecturer is candid that he is guessing at the motive: "I can only speculate
as to why they do this. I don't think I've seen it in other works." The nearest
relatives he names are DeepSeek's data-processing experts and academic work like
Branch-Train-Merge.

A student asks the obvious question — why not one training loop? — and the
answer is organizational rather than technical:

> you can have a separate team working on each expert. It's much easier to have
> teams work on this, and then the aggregation could potentially be simple as
> well, if you have enough compute. It's just that, if you have all the
> objectives, you might as well just throw it into the big training loop.
> Usually I think that's how you would prefer to do things. (≈1:13:18)

Distillation needs its own design: it "is going to require you to write down a
sequence of prompts on which the experts will be distilling into the final
model" (≈1:12:31).

## Environments at scale

The SWE expert is the most involved. The reward is a test suite, so the
bottleneck is having enough tasks: "SWE-bench is the gold standard of these
kinds of things, and so what they want to do is they want SWE-bench but more"
(≈1:05:33). Qwen generates issues automatically from GitHub and runs RL against
them.

![Slide 59 — Agent environment construction](../raw/images/16-post-training-rlvr/slide-59.jpg)
*Slide 59 — the automated pipeline from GitHub issues to SWE-bench-style environments.*

## And that is where the reward breaks

Building environments out of real repositories hands the model an exploit: the
repository contains its own future commits, so the fix can be looked up rather
than derived. Qwen adds a reward whose only job is to stop the agent touching
the git history, and without it the training curve shows a false emergence.
This is the lecture's central [reward hacking](reward-hacking.md) example, and
it is covered in full there.

## Results, with the caveat attached

Qwen3-Coder-Next reaches "something like 70.6% on SWE-bench" (≈1:07:52) from
"a really tiny model, a three billion active parameter model" (≈1:08:38).

The lecturer immediately bounds it: "RL, of course you're going to be able to do
well on the environments for which you've trained. You might even get
generalization to your validation set as you do here. But you always want to be
a little bit careful about comparing performances: task-specific performance
doesn't necessarily mean it'll generalize to broader domains" (≈1:08:38).

## How much does midtraining matter?

A student asks whether the wrong midtraining data makes RL impossible, and the
answer is nuanced rather than absolute (≈1:11:45):

> I think pre-training and SFT are doing a lot of the heavy lifting, as long as
> you have coverage. If pre-training just doesn't have any code data, then
> you're in trouble — you need mid-training.

But given diverse pre-training, midtraining is "very nice to have, very
important in order to get slightly better generalization, but not necessarily
make-or-break" (≈1:12:31). The essential piece is SFT: "If you didn't have SFT,
then you're in deep trouble," because SFT is what gets the model close enough to
earn any reward at all.

## See also

- [RLVR](rlvr.md), [GRPO](grpo.md)
- [Reward hacking](reward-hacking.md) — the git-history exploit
- [Qwen 3](qwen3.md) — the base pipeline this extends
- [Midtraining](midtraining.md), [synthetic data](synthetic-data.md)
- [Agentic benchmarks](agentic-benchmarks.md) — how these are evaluated
- [Reasoning distillation](reasoning-distillation.md)
- [Lecture 16](16-post-training-rlvr.md)
