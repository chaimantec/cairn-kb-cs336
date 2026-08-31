# Sharding, replication, and recomputation

The closing generalization of [Lecture 7](07-parallelism.md) ([1:19:40]), and the
most portable idea in the systems unit. When a value is needed and not at hand,
there are three things you can do, and every technique in Lectures 2 through 7 is
one of them:

> You can either **recompute**, or **store in memory** — when we were talking
> about things like activation checkpointing, or when we're working with GPUs. Or,
> in this case, you can think about an extension of this: that you can **store on
> a different GPU**.

| Option | Costs you | Where the course does it |
| --- | --- | --- |
| **Recompute** | FLOPs | [Activation checkpointing](activation-checkpointing.md); [FlashAttention](flash-attention.md) recomputing attention in the backward |
| **Store** | memory | The [memory hierarchy](gpu-architecture.md); [tiling](tiling.md) into shared memory |
| **Communicate** | bandwidth | Every strategy in Lecture 7 |

The third option is what multi-GPU training adds, and the reason it is worth
having is that a neighbouring GPU's HBM can be closer, in time, than recomputing
something expensive — [NVLink](gpu-interconnect.md) is only about 4× slower than
local HBM ([24:01]).

## Replication is a choice, not a default

The clearest illustration is [data parallelism](data-parallelism.md), which looks
naive until you see what it is buying ([1:19:40]–[1:20:26]):

> You look at data-parallel — you're doing redundant work, in some sense, because
> every rank is actually updating its parameters and keeping track of all the
> parameters. But the reason you're doing that is that you don't have to move the
> optimizer state across.

Every rank stores the full parameters and optimizer state, and every rank
redundantly applies the identical update. That is real waste in both memory and
FLOPs. It buys a communication pattern of exactly one
[all-reduce](collective-operations.md) per step — the gradients move, and nothing
else ever does.

FSDP and ZeRO take the opposite side: shard the parameters and optimizer state, and
pay [all-gather plus reduce-scatter](collective-operations.md#the-identity-that-matters)
to reconstitute what you need, when you need it ([1:02:44], [1:18:54]). This is
why the lecture insists that the factorization of all-reduce matters — it is the
seam along which you trade memory for bandwidth. (Lecture 8's subject; not covered
in this KB.)

## Sharding along four axes

Given that you are going to shard something, the question is along which
dimension. The lecture's summary names four ([1:18:54]):

- **Data** (the batch) → [data parallelism](data-parallelism.md)
- **Width** → [tensor parallelism](tensor-parallelism.md), and
  [expert parallelism](expert-parallelism.md) for MoEs
- **Depth** → [pipeline parallelism](pipeline-parallelism.md)
- **Length** (the sequence) → sequence parallelism ([1:15:05])

Each axis saturates on its own, which is why real systems combine them
([1:17:22]) — data parallelism until the critical batch size, tensor parallelism
within a node, pipeline parallelism across slow links.

## Why the structure is permanent

The lecture ends on the claim that this is not a transitional problem waiting for
better hardware ([1:20:26]):

> Hardware is getting faster, but, in some sense, we'll always want bigger models.
> So, this idea of having a hierarchical structure will always be there.

The hierarchy exists because capacity and speed trade against each other at every
level — registers over shared memory over HBM over NVLink over InfiniBand — and
appetite grows to consume whatever the fast tier can hold. The
[efficiency](efficiency.md) framing of the course is the same claim from the other
direction: the binding constraint moves, but there is always one.

## See also

- [Arithmetic intensity](arithmetic-intensity.md) — the same
  compute-versus-movement tradeoff, measured.
- [Memory accounting for training](memory-accounting-for-training.md) — what has
  to fit, and therefore what has to be sharded.
- [Resource accounting](resource-accounting.md) — the habit this all serves.
