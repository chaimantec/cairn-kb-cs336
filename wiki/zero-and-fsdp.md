# ZeRO and FSDP

[Data parallelism](data-parallelism.md) in its naive form replicates the entire
training state on every rank. ZeRO is the observation that almost none of that
replication is necessary, and that removing most of it costs **nothing at all** in
communication. It comes in three stages, sharding progressively more of the state,
and stage 3 is what PyTorch ships as FSDP.

This is the technique [lecture 7](07-parallelism.md) repeatedly named and deferred;
lecture 8 is where it is explained.

## The problem: replication is most of your memory

Start with the accounting. Training a model does not cost one copy of the weights,
it costs about five, and the rule of thumb is **16 bytes per parameter** ([13:52]):

- the **parameters** themselves;
- the **gradients**, an accumulator the same size as the parameters;
- possibly a **higher-precision accumulator** to add into;
- and, for Adam, the **first and second moments** of the gradient, tracked over
  time, often kept in high precision for stability.

Those last two are the expensive ones. Together with the accumulator they are
called the **optimizer state**, and "if you look at the accounting, this is most
of the memory cost of doing an SGD update" ([14:37]). Slide 18 colours the three
groups consistently through the whole ZeRO sequence: **blue** parameters, **orange**
gradients, **green** optimizer state — and the green block is visibly the largest,
with parameters and gradients equal in size ([15:23]).

Under naive DDP every rank holds all of it. Memory consumption is therefore
*linear in the number of accelerators* while buying you no memory headroom at all
([15:23]). Slide 18's own worked example runs the total from **120 GB down to
1.9 GB** across the three stages, for $\Psi = 7.5$B parameters, $K = 12$ bytes of
optimizer state per parameter and $N_d = 64$ ranks ([16:09]).

The general per-rank memory expression on that slide is

$$(2 + 2 + K)\Psi$$

for the baseline, with successive stages moving terms under a $1/N_d$ divisor.

## Stage 1 — shard the optimizer state

Each rank becomes **responsible for updating one slice of the parameters**, and
keeps optimizer state only for that slice ([16:55]). Everyone still holds full
parameters and full gradients.

The step, from slide 20:

1. Every rank computes a **full gradient** on its own shard of the batch.
2. **Reduce-scatter** the gradients, so each rank receives the summed gradient for
   *only its own slice* — "they don't need all the gradients — they only need the
gradients associated with their update" ([17:41]).
3. Each rank applies the optimizer update to its slice, using the state it alone
   tracks.
4. **All-gather** the updated parameters back to everyone.

Now count the communication. Naive DDP does one all-reduce, costing $2 \times$
parameters. Stage 1 does a reduce-scatter plus an all-gather — and by
[the identity](collective-operations.md) that an all-reduce *is* a reduce-scatter
followed by an all-gather, those cost the same thing ([18:26]):

> So ZeRO stage 1 has the exact same communication characteristics as naive DDP —
> this was free. Free memory savings, wonderful.

That identity, reviewed at the top of the lecture as "very important for one part
of the talk" ([3:08]), is the entire reason stage 1 is free. It is why the lecture
spends its first five minutes on it.

## Stage 2 — also shard the gradients

The obstacle is that stage 1's step 1 materialises a full gradient vector, which
you cannot do if gradients are sharded. The fix is a scheduling trick rather than a
new algorithm ([19:12]):

> Instead of materializing the gradient vector all at once, I'm going to walk
> backwards through the compute graph, and after computing a layer's gradients,
> I'm going to immediately reduce that and send it to the right worker.

And once a gradient is no longer needed by the backward pass, free it ([19:59]).
Doing the reduction incrementally rather than all at once "is the same thing — it
doesn't really make a difference," so stage 2 is also free. Slide 21's comparison
table records the communication cost as $2 \times \#\text{params}$ in **both**
columns, naive DDP and ZeRO stage 1 alike.

## Stage 3 (FSDP) — shard the parameters too

Push the same idea onto the parameters: **send and receive parameters on demand
while stepping through the compute graph**, so that "each GPU only sees a slice of
the parameters, gradients, and optimizer states at any one time" ([20:45]).

The per-layer cycle, from slide 25 ([21:31]):

1. **All-gather** this layer's weights.
2. Forward through the layer.
3. **Free** the weights — you no longer need them.
4. For the backward: you kept the activations, so **all-gather** the layer's
   weights again on demand.
5. Backward through the layer.
6. **Reduce-scatter** the gradients out as soon as you have them, and free the
   weights again.

So the count is **two all-gathers and one reduce-scatter** per layer, against
DDP's single all-reduce — one extra all-gather ([22:17]). Slide 27 puts the
comparison plainly: DDP costs $2\times$ params, stages 1 and 2 cost $2\times$
params, stage 3 costs $3\times$ params.

### Why the extra communication does not show up in the wall clock

Two ideas rescue it, and the second is the one people miss ([22:17]–[23:02]):

1. **Sweep the graph, communicate, free immediately** — already used in stage 2.
2. **Overlap communication with computation.** While the forward pass for layer 0
   runs, the all-gather for layer 1 is already in flight; when layer 1's weights
   land, its forward starts and layer 2's all-gather begins ([23:49]).

Slide 26 draws this as three streams — CPU, GPU compute, GPU communication — from
the PyTorch FSDP paper (arXiv:2304.11277). Bubbles are visible, but "if your comms
are very fast and your computation is very big, you can easily see how the
computation can take longer than the communication" ([24:34]). The result:

> In practice, if you run FSDP, you're going to see GPU utilization very close to
> just the single-GPU performance — it's actually quite remarkable how good FSDP
> is ([25:20]).

Note that this is the same recompute-or-communicate-instead-of-storing pattern as
[activation checkpointing](activation-checkpointing.md), applied to parameters.

### It is not pipelining

A student asks whether gradients pass from one GPU to the next, and the correction
is worth keeping, because the two are easy to confuse ([26:06]):

> What you just described … is closer to pipelining. Pipelining would be like if
> one layer were on one GPU and another layer on another GPU. That's not the case
> here. In all these cases, every GPU goes through the entire model, from start to
> finish. But the difference is that no GPU is going to hold the entire set of
> parameters at the same time.

Every rank runs the whole model. It just never holds the whole model.

### Why per-layer communication is not multiplied by depth

Another good question: doesn't doing this every layer multiply the cost by the
number of layers? It multiplies the *number of operations*, but each one is
correspondingly smaller ([27:37]):

> Each of these communication pieces is smaller, because this is just one tiny MLP
> or something, and I'm comparing that to the cost of all-reducing a whole network.

## What it buys, and where it stops

Slide 28 gives the practical answer for a machine with 8×A100 80GB: the baseline
cannot fit even a 7B model, while stage 3 reaches roughly 50B ([27:37]). The slide
also notes that pure BF16 training with Kahan summation is viable, which lowers
$K$.

Two limits then force the rest of the lecture ([29:07]–[29:53]):

- **Data parallelism consumes batch size.** With a batch of 8 you can never use
  more than 8 accelerators, and you cannot simply grow the batch forever — see
  [critical batch size](critical-batch-size.md).
- **It does not touch activation memory.** Stages 1 and 2 do not reduce parameter
  memory at all, and stage 3 does, but slide 30 is explicit that ZeRO stage 3
  "is nice in principle, but *does not reduce activation memory*". For big models
  at long sequence lengths, activations dominate — see
  [activation memory](activation-memory.md).

Those two gaps are exactly what [model parallelism](sharding-vs-replication.md)
exists to close.

## In practice

FSDP alone is a complete strategy for a small model: OLMo-7B was trained "fully
with FSDP", and "many 7B-ish models are trained purely with FSDP" ([1:12:44]). At
frontier scale it appears as one component of a
[3D/4D combination](3d-parallelism.md) — ZeRO stage 1 is the most common choice
there, since it is free and composes cleanly with everything else. See
[case studies](parallelism-case-studies.md).

You implement FSDP yourself in the course assignment: "you can just write a
wrapper that wraps any module and turns it into its FSDP version. Conceptually,
all you're going to do is a bunch of all-gathers, compute, free, and repeat on the
backward pass" ([25:20]).

## A note on the deck

The slide deck misspells FSDP as "FDSP" on slides 26, 35 and 63. The transcript
and this wiki use the correct spelling; see the
[slide file's typo section](../raw/slides/08-parallelism-2.md).

## See also

- [Data parallelism](data-parallelism.md) — what ZeRO is an optimisation of.
- [Collective operations](collective-operations.md) — the all-reduce identity that
  makes stages 1 and 2 free.
- [Activation memory](activation-memory.md) — the memory ZeRO cannot reach.
- [Sharding vs replication](sharding-vs-replication.md) — the general frame.
- [Memory accounting for training](memory-accounting-for-training.md) — where the
  16 bytes/parameter comes from.
- [Lecture 8](08-parallelism-2.md) · [slides 18–28](../raw/slides/08-parallelism-2.md#slide-18--zero--solving-the-memory-overhead-issue-of-dp) · [transcript](../raw/transcripts/08-parallelism-2.md)
