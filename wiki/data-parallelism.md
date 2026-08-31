# Data parallelism

Cut along the **batch**: each rank gets a slice of the data, a complete copy of
the model, and runs an ordinary training step — with one collective inserted after
the backward pass to keep the copies in agreement. It is the first of
[Lecture 7](07-parallelism.md)'s three strategies and by some distance the
simplest.

The version here is **DDP** (distributed data parallel). Its sharded successors,
FSDP and ZeRO, are Lecture 8's subject and are not covered in this knowledge base
([1:02:44]).

## The mechanism

Setup: a batch of 128 rows of dimension 1024, a four-layer MLP, world size 4
([56:29]).

**1. Slice the batch.** Each rank takes `batch_size / world_size` rows — 32 here
([57:17]). Rank $r$ takes rows $32r$ through $32(r+1)$. The lecture notes that
slicing a full copy is an illustration, not a design: "in practice, each rank
should probably load its own data, rather have this bottleneck" ([57:17]).

**2. Every rank holds every parameter**, and its own optimizer state ([58:04]).
All ranks start from identical weights, because `get_init_params` seeds with
`manual_seed(0)`.

**3. Forward and backward as normal**, on the local 32 rows.

**4. All-reduce the gradients.** This is the entire difference:

```python
for param in params:
    dist.all_reduce(tensor=param.grad, op=dist.ReduceOp.AVG, async_op=False)
```

"This is the key step that makes data parallelism work: we're going to synchronize
the gradients across all the workers. This is the only difference between standard
training and DDP. It's actually pretty nice and elegant" ([58:51]). And: "it's a
one-line code change" ([59:37]).

**5. Step the optimizer.** Since every rank began with the same parameters and
just applied the same averaged gradient, they stay identical — no further
communication needed.

Note the reduction is `AVG`, not `SUM`. Each rank computed a mean loss over its
own 32 rows, so averaging the gradients yields the gradient of the mean loss over
all 128.

## What is and is not the same across ranks

The lecture's own summary ([1:01:58]), and the published run confirms each line:

| | Same across ranks? |
| --- | --- |
| Data slice | no |
| Loss | **no** — computed on local data |
| Gradients *before* the all-reduce | no |
| Gradients *after* the all-reduce | yes |
| Parameters | yes, always |

In the course's [recorded four-GPU run](../raw/slides/07-parallelism.md#data-parallelism--implementation)
the four losses come out 0.0115, 0.0121, 0.0120 and 0.0128 while all four
parameter summaries are byte-identical — which is the property the whole scheme
rests on. (Those loss values depend on an unseeded random batch and are not
reproducible.)

The effect, stated well at [1:00:24]: "each rank is basically performing parameter
updates as if it were — have all the data on it, but it's only actually processing
a part of the data."

## Why it is the pleasant one

Because it never touches the model. "DDP has a nice thing that is very modular —
you do the forward pass… DDP just averages the parameters here — it doesn't care
what your forward pass looks like" ([1:01:12]). Asked what it would look like for
a Transformer instead of an MLP: "it would actually be basically the same."

Contrast [tensor parallelism](tensor-parallelism.md), where "now we have to muck
around with the model" ([1:07:21]).

## The constraints

**Batch size ≥ world size**, obviously, and in practice much more: "your batch
size has to be at least world size for this to really make sense, and usually it
should probably be quite a bit larger" ([1:00:24]).

**Divisibility.** A batch that divides evenly by the world size "would be nice —
yes. I mean, if it's not, then you can pad it with zeros or something, so there's
ways, but it's just easier for everyone if it is" ([1:01:12]). The lecture's own
`int_divide` helper asserts on a remainder.

**Memory is the real ceiling.** Of all-reduce: "It's a very simple monolithic operation,
but it does require holding all the model's parameters in memory. But what if the
model parameters don't fit in memory? Then you're going to have to be more clever,
and that's the topic for next class" ([1:02:44]). See
[memory accounting](memory-accounting-for-training.md) for what has to fit.

**The critical batch size.** Data parallelism scales by growing the global batch,
and that stops helping: "you might be able to do data-parallel quite a bit, but
then you start hitting something called the critical batch size, where, if you
start increasing the batch size too much, it doesn't actually help you — in which
case, you're just wasting your compute, and then you're better off using
tensor-parallel" ([1:17:22]). The lecture names the phenomenon and moves on; it
does not derive it.

## What DDP is trading away

The closing framing ([1:19:40]–[1:20:26]) is worth sitting with, because it makes
data parallelism look like a deliberate choice rather than the naive baseline:

> From that perspective, you look at data-parallel — you're doing redundant work,
> in some sense, because every rank is actually updating its parameters and
> keeping track of all the parameters. But the reason you're doing that is that
> you don't have to move the optimizer state across.

Every rank redundantly stores the optimizer state and redundantly applies the same
update. That waste buys you a communication pattern of one all-reduce per step.
FSDP and ZeRO take the other side of the trade — shard the state, and pay
[all-gather plus reduce-scatter](collective-operations.md#the-identity-that-matters)
instead. See [sharding vs. replication](sharding-vs-replication.md).

## Overlapping, which this version does not do

The implementation waits for the whole backward pass and then all-reduces
everything. It need not: "if you're clever, then, on the backward pass, as soon as
the gradients are done, you can start sending that. And that's something you'll be
exploring in assignment two" ([1:14:18]). Gradients for the last layer are
finished long before the first layer's, so most of the communication can hide
under the remaining computation.

## See also

- [Collective operations](collective-operations.md) — all-reduce, and why it
  factors.
- [Tensor parallelism](tensor-parallelism.md) and
  [pipeline parallelism](pipeline-parallelism.md) — the other two cuts.
- [Expert parallelism](expert-parallelism.md) — a fourth axis, for MoE models.
