# RL infrastructure

Reinforcement learning on language models is a systems problem as much as an
algorithmic one, because it runs a training system and an inference system in
the same loop. [Lecture 16](16-post-training-rlvr.md) devotes a section to it,
connecting back to the course's [inference](inference.md) and
[parallelism](07-parallelism.md) units:

> Training is hard, inference is hard, and RL puts the two together. So in some
> ways it's no wonder it's really horrible and difficult. (≈51:38)

Every serious open technical report now carries a section on this, which is
itself the evidence that it is hard.

## The straggler problem

The failure that is easy to miss until you hit it. Rollouts are generated in
batches, and generation length varies enormously — some problems produce a short
chain of thought, some produce an enormous one. The lecture's illustration:

> Imagine you've got your rollouts, but you've got one really hard math problem
> — let's say one of them is the Riemann hypothesis — and your model's really
> chugging along on the Riemann hypothesis, it's got this gigantic CoT. Now
> what's happening in the meantime? If you're doing naive inference, everyone
> else is waiting on this one rollout to complete, in order to move on to the
> next phase. (≈51:38)

So "long rollouts can really hurt you," and the mitigations are all awkward: "do
you need to truncate them, do you somehow set them onto a different machine, who
knows — these are all decisions you can make" (≈52:24). Note the interaction
with [length bias](length-bias-in-rl.md): an objective that encourages long
wrong answers is also an objective that makes your straggler problem worse.

## Two systems, one loop

RL alternates between rolling out and training, and neither engine wants to be
idle. The options are all costly: "either some of your machines are pure rollout
machines and some of them are training machines, or you're switching frameworks
all the time. Both of these are very costly" (≈52:24).

Whatever the split, weights have to move from the training side to the inference
side every iteration, "so you need some sort of story for how to move those
around, and you need to coordinate them closely. In some cases maybe they even
share the same machines, because as inference is running the training one might
be idle" (≈53:57).

![Slide 45 — RL Infra](../raw/images/16-post-training-rlvr/slide-45.jpg)
*Slide 45 — the training side and the inference side of an RL system, with the weight transfer between them.*

## The trap: reusing rollouts

This is the one the lecture warns about most directly, because the temptation
follows immediately from the previous problem. On-policy RL is well behaved —
"GRPO in its simple on-policy form behaves very nicely, you'll experience this
in your assignments" — and it wastes hardware. So:

> you'll get kind of greedy, you'll say, ah, but my systems utilization is so
> low — I could do so much better if only I could reuse my rollouts, then I
> could overlap my inference and computation and do all sorts of clever things.
> So you'll attempt to reuse these rollouts, and that will lead to off-policy
> problems, which then lead to destabilizing your training. (≈53:11)

Recall from [GRPO](grpo.md) that running on-policy is exactly what makes the
clipping term vanish and the algorithm simple. Reusing rollouts brings the
importance ratio, the clipping, and the instability back. The utilization win
and the algorithmic simplicity are in direct tension, and that tension is the
core of RL infrastructure design.

## See also

- [GRPO](grpo.md) — why on-policy is the simple case
- [Inference](inference.md) — the serving half of the loop
- [Length bias in RL](length-bias-in-rl.md) — why rollouts get long
- [Kimi K1.5](kimi-k1-5.md) — the report this section is taught from
- [Lecture 16](16-post-training-rlvr.md)
