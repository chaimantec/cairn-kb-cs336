# Context parallelism (ring attention)

Splitting a long sequence itself across accelerators, so that no single device
holds the whole context. Covered briefly in lecture 8, deliberately: "I'm not going
to talk about it much more, mainly because I think it overlaps conceptually with a
lot of what we've already talked about" ([1:01:12]).

## The idea

From slide 54 ([1:00:25]–[1:01:12]):

> Context parallel, or ring attention, is an idea of basically splitting
> activations, in a very long sequence, across different accelerators. And you can
> do things like pass to the device that's needed, in this ring-like way, following
> the mesh topology of TPUs.

Attention needs every query to see every key, so a split sequence has to be
circulated: each device holds one block of the sequence and passes it around the
ring, computing partial attention against whatever block it currently holds. Ring
attention "was the original paper that did this, [and] showed that this worked
really well on TPUs" — unsurprisingly, since a ring is a natural traversal of the
[toroidal mesh](network-topology.md) TPUs are wired as.

## Where it is used

Two places, both named on slide 54: **long-context extension stages** and **model
serving** ([1:01:12]).

That shows up clearly in the [case studies](parallelism-case-studies.md). Llama 3
405B runs context parallel at 1 through its main pre-training phase and cranks it
to 16 only for the long-context extension stage, dropping data parallel from 128 to
8 to pay for it ([1:15:03], slide 66). Nemotron 3 Super's long-context section
shows CP 64 ([1:17:23], slide 70). The Megatron guidance is the same: "if you're
doing long sequences, use context parallel — ring attention" ([1:07:22]).

## Not the same as sequence parallelism

The names are confusingly close and the lecture says so — context parallelism is
the one that *deserves* the name "sequence parallel", but
[sequence parallelism](sequence-parallelism.md) already has it ([48:56]). The
distinction:

- **Sequence parallelism** splits the activations of pointwise ops (LayerNorm,
  dropout, residuals) along the sequence axis, as an add-on to
  [tensor parallelism](tensor-parallelism.md), to remove the $10sbh$ term.
- **Context parallelism** splits the sequence across devices as a first-class
  parallelism dimension, so you can train on contexts no single device could hold.

They appear as separate columns in slide 72's overview table.

## See also

- [Sequence parallelism](sequence-parallelism.md) — the name collision.
- [Network topology](network-topology.md) — why a ring suits a mesh.
- [Flash attention](flash-attention.md) — the same block-wise attention accumulation, within one device.
- [3D parallelism](3d-parallelism.md) — where CP sits in the combination.
- [Lecture 8](08-parallelism-2.md) · [slide 54](../raw/slides/08-parallelism-2.md#slide-54--other-parallelism-strategies) · [transcript](../raw/transcripts/08-parallelism-2.md)
