# Sequence parallelism

The fix for the one term [tensor parallelism](tensor-parallelism.md) cannot
divide. It is almost never used on its own — "sequence parallel is often just
something you do with tensor parallel, in order to reduce activations. It's more
like an add-on that you put in to reduce activation; it's not its own standalone
thing, in many cases" ([1:08:08]).

## The name is misleading

Said outright ([48:56]):

> This is an extremely misleading name, because when I say "sequence parallel,"
> there's actually a thing called context parallel that's more naturally suited to
> this name.

[Context parallelism](context-parallelism.md) splits a long sequence across devices
so that each holds part of the *sequence itself*. Sequence parallelism splits the
**activations of the pointwise operations** along the sequence axis. Different
technique, similar-sounding name; keep them apart.

## What it splits

After tensor parallelism, [activation memory](activation-memory.md) per layer is

$$sbh\left(10 + \frac{24}{t} + 5\frac{as}{ht}\right)$$

and the stubborn $10sbh$ is LayerNorm, dropout, and the residual inputs to
attention and the MLP. Slide 48's observation is that these are all **pointwise
operations over the sequence** ([49:43]):

> These terms are very lightweight — layer norms, stuff that doesn't have much
> computation. And what I'm going to do is split them up, over the sequence axis,
> rather than over the hidden axis.

A LayerNorm cannot be split along the hidden dimension, because it mixes across
it. But it is independent across sequence positions, so it splits along the
sequence axis perfectly. That is the whole trick.

The result is the third row of slide 49's table:

$$sbh\left(\frac{34}{t} + 5\frac{as}{ht}\right)$$

Everything now carries a $1/t$ — activation memory is finally **fully linear** in
the tensor-parallel degree ([51:16]).

## The cost: collectives at every boundary

Splitting along a different axis than the surrounding matmuls means converting
between the two layouts, "This then involves all-gathers and reduce-scatters before
every operation where we need these" ([49:43]).

The lecture points out the resemblance ([49:43]–[50:28]):

> If you think about it, this is very reminiscent of FSDP — we have this thing we
> need, we need activations at some point … but we don't need them now. So we're
> going to store them split up across the sequence axis, and materialize them on
> demand. So, I'd say this is conceptually very similar to the FSDP-style idea.

Same shape as [ZeRO stage 3](zero-and-fsdp.md): keep it sharded, gather on demand,
free afterwards.

## The forward/backward duality

As with tensor parallelism's $f$ and $g$, the two collectives swap between passes
([50:28]):

> In the forward pass, $g$ is an all-gather, $\bar{g}$ is a reduce-scatter; in the
> backward pass, this is reversed — the all-gather and reduce-scatter swap between
> $g$ and $\bar{g}$.

This is the same all-gather/reduce-scatter duality that shows up in
[collective operations](collective-operations.md) and in tensor parallelism's
backward pass. It is worth recognising as a recurring pattern rather than three
separate facts.

## In practice

Sequence parallelism travels with tensor parallelism, and the pair is often written
as a single unit: slide 68 records Gemma 2 as using "MP (=TP+SP)", and slide 72's
overview table has a single combined **TP/SP** column rather than two. DeepSeek V1
used "ZeRO stage 1, with tensor, sequence, and pipeline parallel" ([1:13:31]).

## See also

- [Activation memory](activation-memory.md) — the accounting this exists to fix.
- [Tensor parallelism](tensor-parallelism.md) — its inseparable partner.
- [Context parallelism](context-parallelism.md) — the differently-named neighbour.
- [ZeRO and FSDP](zero-and-fsdp.md) — the same shard-and-gather-on-demand pattern.
- [Lecture 8](08-parallelism-2.md) · [slides 48–49](../raw/slides/08-parallelism-2.md#slide-48--making-memory-truly-linear--sequence-parallel) · [transcript](../raw/transcripts/08-parallelism-2.md)
