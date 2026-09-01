# Critical batch size

The reason [data parallelism](data-parallelism.md) cannot scale forever, and the
reason batch size is treated throughout lecture 8 as a **consumable resource**
rather than a free parameter.

## Batch size is a budget

Data parallelism spends batch size to buy accelerators ([29:07]):

> Data parallel is consuming an important resource, and that resource is the batch
> size, so to speak. If you have a batch size of eight, you can never have more
> than eight accelerators.

The obvious response is to grow the batch as you add machines. That fails
([29:07]):

> You can't do that, because at a certain point there's something called the
> critical batch size, where the gain you get from an additional batch element is
> less than if you had taken another SGD step on that single element.

Below the critical batch size, adding examples is genuinely equivalent to taking
more optimisation steps. Above it, returns diminish ([29:53]): "an infinitely large
batch size is not infinitely better than infinitely many single steps."

## The trade-off it forces

Stated as a hard choice ([29:53]):

> Do we want small batch sizes and let our GPUs be more idle, or do we want big
> batch sizes and take the hit on optimization? Kind of hard to say.

Neither answer is right in general, which is precisely why the rest of the lecture
exists: [model parallelism](sharding-vs-replication.md) lets you use more
accelerators *without* spending more batch size.

## Where the budget gets spent

Once you see batch size as a budget, several otherwise-unrelated facts line up:

- **[Pipeline parallelism](pipeline-parallelism.md) spends it on micro-batches.**
  The bubble shrinks as one over the number of micro-batches, so "we need a huge
  batch size in order to reduce this bubble towards zero" ([33:42]). The lecture
  makes the connection explicitly: "this is kind of why I said batch size is a
  useful resource, because now we can spend it in a different way — pipeline more,
  to reduce the amount of idle time" ([33:42]). Slide 36's sweep shows utilisation
  collapsing at small batch sizes and large pipeline degrees.
- **[Activation recomputation](activation-checkpointing.md) buys it back.** Saving
  activation memory frees room for a larger batch, which improves utilisation —
  "memory can be turned into batch size" ([11:57], slide 62).
- **Gradient accumulation converts it.** In the standard recipe, "if your batch
  size ends up too small, you can do gradient accumulation, to get better GPU
  utilization" ([1:06:37]).
- **Slide 55's table lists it as a cost.** FSDP's recorded drawback is not only
  that it leaves activations alone but that "it consumes your global batch size in
  order to do the parallelization" ([1:02:00]), whereas tensor parallelism "doesn't
  touch global batch size" ([1:02:46]).

## The utilisation curve

Slide 56 plots this directly: efficiency against per-chip batch size, with a
threshold above which you are compute-bound ([1:05:06]):

> If your batch size is big enough, FSDP only is good. If you have a batch size of
> 2,000 per chip, how far can you go? Well, you've got FSDP doing extremely well,
> you're fully compute-bound. But the important thing is, as the batch size goes
> down, FSDP only will hit a point where you're now communication-bound.

Adding model parallelism pushes the curve out, keeping you compute-bound into
smaller per-chip batch regimes — which is the argument for
[3D parallelism](3d-parallelism.md) in one picture.

## See also

- [Data parallelism](data-parallelism.md) · [ZeRO and FSDP](zero-and-fsdp.md) — what spends the budget.
- [Pipeline parallelism](pipeline-parallelism.md) — the other big consumer.
- [Activation memory](activation-memory.md) — how memory converts into batch size.
- [Scaling laws](scaling-laws.md) — the optimisation side of the same question.
- [Lecture 8](08-parallelism-2.md) · [slide 29](../raw/slides/08-parallelism-2.md#slide-29--issues-remain-with-data-parallel--compute-scaling) · [transcript](../raw/transcripts/08-parallelism-2.md)
