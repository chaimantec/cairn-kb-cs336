# Lecture 7 — Parallelism

Lectures 5 and 6 made a single GPU go fast. This lecture crosses the chip
boundary: how do you use *many* GPUs, and what does it cost you to move data
between them? Percy Liang's framing is that nothing conceptually new is
happening — the same principle that governed
[shared memory and HBM](gpu-architecture.md) governs NVLink and InfiniBand, only
the distances and the penalties are larger. "The game is to orchestrate the
computation to try to avoid data transfer bottlenecks" ([1:38]).

The lecture has two halves. The first builds the vocabulary and the machinery:
[collective operations](collective-operations.md), the
[interconnect hardware](gpu-interconnect.md) that carries them, and
[`torch.distributed`](torch-distributed.md), the library you actually call.
The second cuts a neural network three ways —
[data parallelism](data-parallelism.md) along the batch,
[tensor parallelism](tensor-parallelism.md) along the width, and
[pipeline parallelism](pipeline-parallelism.md) along the depth.

It is an [executable lecture](executable-lectures.md): the material is the
program [`lecture_07.py`](../raw/slides/07-parallelism.md), so there are no slide
numbers. Citations below point at function names and source line ranges.

> **Two lectures are called "Parallelism."** This is Lecture 7, Percy's. Lecture 8
> is Tatsunori Hashimoto's deck on the same subject and is **not** in this
> knowledge base. Where this page says "next time" it means Lecture 8, which
> covers FSDP and ZeRO ([1:02:44], [1:18:54]).

## Why bother with more than one GPU

Two reasons, and the lecture is careful that they are different ([3:14]).

The first is that the model does not fit. "Your parameters, or activations, or
gradients and optimizer state, don't fit on the HBM memory of a single GPU." A
B200 has 192 GB; a one-trillion-parameter model does not go into it. The
[memory accounting from Lecture 2](memory-accounting-for-training.md) is what
tells you this in advance.

The second is that it fits but you want to go faster — "even if your model could
fit on a single GPU, you might want to leverage more GPUs by splitting everything
up to train faster."

These pull in opposite directions, and the tension is the whole subject: "you
could fit everything on GPUs, but you have fewer cores. But if you spread out,
then you're going to have to pay the communication bandwidth. So, that's some
calculation you're going to have to do, to figure out how to parallelize"
([4:01]).

## The hierarchy, extended

Lecture 5 gave a memory hierarchy inside one chip. Lecture 7 extends the same
picture outward ([2:26]), and the four rungs come with numbers the source states
as constants:

| Level | Carried by | Bandwidth |
| --- | --- | --- |
| Single node, single GPU | L1 cache / shared memory | fastest |
| Single node, single GPU | HBM | 8 TB/s (B200) |
| Single node, multi-GPU | NVLink / NVSwitch | 1.8 TB/s (NVLink 5) |
| Multi-node, multi-GPU | InfiniBand | ~0.05 TB/s |
| Across pods | Ethernet | ~200 MB/s |

The joke that lands the point is that HBM, which Lecture 5 spent an hour
complaining about, is now the fast tier: "you had HBM, which we lamented was so
slow, but in this lecture HBM is going to be considered fast" ([1:38]–[2:26]).
NVLink at 1.8 TB/s against HBM at 8 TB/s is "about four x" slower ([24:01]) —
which, going *between devices*, is remarkably good.

The consequence for technique is stated as a parallel to the previous lecture:
"last week we talked about various tricks for improving memory accesses — fusion
and tiling… and this week we're going to talk about how you can reduce the amount
of communication across GPUs by replicating and sharding appropriately" ([2:26]).
See [operator fusion](operator-fusion.md) and [tiling](tiling.md) for the first
half of that sentence, and
[sharding vs. replication](sharding-vs-replication.md) for the second.

## Part 1 — the building blocks

### Collective operations

The programming model is not point-to-point messaging but
[collective operations](collective-operations.md): "collective just means that
you're specifying a general communication pattern, or a template, across multiple
devices, rather than managing point-to-point how this GPU is going to communicate
with another GPU" ([6:18]). They are old — "primitives from distributed
programming that go back to the '80s… It wasn't invented for LM training"
([5:31]).

The vocabulary is two words. A **rank** is a device; for this class, "the rank is
the GPU" ([41:50]). The **world size** is how many there are ([7:04]).

Eight operations are introduced, in three tiers ([7:04], [7:50]):

- **Warm-ups** — broadcast, scatter, gather, reduce. Individually they barely
  appear in training; they exist so the workhorses make sense.
- **Workhorses** — all-gather, reduce-scatter, all-reduce. "The ones that are
  going to show up again and again for distributed training of language models."
- **The general one** — all-to-all, "important for MoEs".

The identity the rest of the lecture turns on is
**all-reduce = reduce-scatter + all-gather** ([15:33]), demonstrated on real
tensors at [45:37] — "proof via example". It matters because breaking the
monolithic operation in two is what lets FSDP and ZeRO intervene in the middle
([16:20]).

### The hardware underneath

Full treatment on [GPU interconnect](gpu-interconnect.md). The shape of it: eight
GPUs per node on NVLink into an NVSwitch, nodes grouped into pods on InfiniBand,
pods joined by Ethernet ([23:14]). Percy flags one of his own numbers as invented
— "eight is typical, but this 256 is made up."

![One node: four GPUs on an NVSwitch, leaving over InfiniBand or Ethernet](../raw/images/07-parallelism/gpu-node-overview.png)

*One node: four GPUs, each with four streaming multiprocessors (a register file and L1/shared memory apiece) over an L2 cache and its own HBM. Each GPU meets an NVSwitch over NVLink, and the switch leaves the node over InfiniBand or Ethernet. The file is a screen capture and carries a stray "To exit full screen, press Esc" browser banner across the top. Source: [`images/gpu-node-overview.png`](https://github.com/stanford-cs336/lectures/blob/main/images/gpu-node-overview.png).*

Two ideas carry most of the weight. **NVSwitch makes the node look flat**: "from a
programming perspective, you can think about GPUs as connected to any other GPU"
([24:01]). And **RDMA is what keeps the CPU out of the way** ([27:06]) — without
it, Ethernet forces a copy into a kernel socket buffer, packet construction, and
another copy to the NIC ([26:21]).

The reason the hierarchy exists at all is a scaling limit, stated bluntly: "you
can't have an NVSwitch handling 100,000 GPUs" ([25:34]).

### torch.distributed, and measuring it

[`torch.distributed`](torch-distributed.md) gives "a clean interface into these
collective operations, so you don't have to explicitly think about NCCL"
([36:23]), with an NCCL backend on GPUs and gloo on CPUs. Underneath, NCCL
detects the hardware topology, picks paths, and launches communication kernels —
"because, at the end of the day, remember, everything that runs on the GPU is a
kernel. So, there are communication kernels as well" ([30:15]).

The benchmarking section is short and is the distributed analogue of Lecture 6's
harness. Two forms of asynchrony now have to be closed out, not one: "there's two
forms of asynchrony here, the CUDA kernels and the different processes"
([47:14]), so a `torch.cuda.synchronize()` *and* a `dist.barrier()` bracket the
timed region — and in that order, since barriering first lets each rank sail past
before its own kernels have finished ([54:10]). See
[benchmarking](benchmarking.md#measuring-a-collective) for the effective-bandwidth
formula, which Percy introduces as "analogous to when we were computing MFU"
([48:47]).

Two properties of that number are worth remembering ([51:05]): the effective
bandwidth is **independent of world size** — as it grows, $(W-1)/W \to 1$ and you
are left with $2S/t$ — and **independent of topology**, ring or tree, "which is
something that NCCL figures out".

## Part 2 — three ways to cut a network

All three are demonstrated on deep MLPs rather than Transformers, and the source
justifies that: "MLPs are the ones that are the actual compute bottleneck in a
transformer, so this is actually pretty representative" ([54:56]). The shared
setup is a batch of 128 rows of dimension 1024 ([56:29]).

![Data parallelism drawn as a horizontal cut through the data](../raw/images/07-parallelism/data-parallelism.png)

*Data parallelism as a cut: four layers stacked above a Data block, with a horizontal orange line through the Data block only. The model is replicated; the batch is split. Source: [`images/data-parallelism.png`](https://github.com/stanford-cs336/lectures/blob/main/images/data-parallelism.png).*

![Tensor parallelism drawn as a vertical cut through every layer](../raw/images/07-parallelism/tensor-parallelism.png)

*Tensor parallelism as a cut: the same four-layer stack, with a vertical orange line running through every layer. The model is split by width. Source: [`images/tensor-parallelism.png`](https://github.com/stanford-cs336/lectures/blob/main/images/tensor-parallelism.png).*

![Pipeline parallelism drawn as a horizontal cut between layers](../raw/images/07-parallelism/pipeline-parallelism.png)

*Pipeline parallelism as a cut: the same four-layer stack, with a horizontal orange line between layer 1 and layer 2. The model is split by depth. Source: [`images/pipeline-parallelism.png`](https://github.com/stanford-cs336/lectures/blob/main/images/pipeline-parallelism.png).*

| | What is cut | What is replicated | What moves, and when |
| --- | --- | --- | --- |
| [Data](data-parallelism.md) | the batch | every parameter | gradients, once per step |
| [Tensor](tensor-parallelism.md) | each layer's width | the data | activations, once per **layer** |
| [Pipeline](pipeline-parallelism.md) | the depth | nothing | activations, rank to rank |

**[Data parallelism](data-parallelism.md)** is the elegant one. Each rank takes
32 of the 128 rows, runs an ordinary training step, and then all-reduces the
gradients — "this is the only difference between standard training and DDP"
([58:51]), "a one-line code change" ([59:37]). Losses differ across ranks,
gradients are averaged to be identical, and so the parameters never drift apart
([1:01:58]). Its ceiling is memory: DDP "does require holding all the model's
parameters in memory" ([1:02:44]), which is what FSDP and ZeRO exist to fix.

**[Tensor parallelism](tensor-parallelism.md)** cuts each weight matrix down its
columns — "also known as column tensor-parallel" ([1:04:17]) — so every rank
holds a $1024 \times 256$ stripe instead of the full matrix. Each rank computes
its slice of the activations, all-gathers, and concatenates, **every layer**
([1:05:50]–[1:06:36]). That is a lot of traffic, and it is why the lecture insists
tensor parallelism belongs inside a node on NVLink ([1:15:51]). The honest cost is
in the code: "now we have to muck around with the model. Data-parallel is very
elegant, because it's splitting by data — the model is treated as a module"
([1:07:21]).

**[Pipeline parallelism](pipeline-parallelism.md)** gives each rank a contiguous
block of layers and passes activations along the chain with point-to-point
`send`/`recv` ([1:11:13]) — the only place in the lecture that leaves the
collectives behind. Its characteristic problem is the **pipeline bubble**: "while
you're not processing, you're waiting around for other tensors to process"
([1:12:47]). Micro-batching shrinks it. Its compensating virtue is tolerance —
"this can tolerate much slower interconnects. So, some of the decentralized
training work uses pipeline-parallel" — because the GPUs may be, quite literally,
"halfway across the world" ([1:16:37]).

## What the lecture deliberately leaves out

Stated as a list at [1:14:18]–[1:15:51], and worth knowing so you do not mistake
the bare-bones implementations for the real thing:

- **Overlapping communication and computation.** Missing from all three
  implementations, and "especially crucial in pipeline parallelism." Even data
  parallelism can do it — rather than waiting for the whole backward pass, "as
  soon as the gradients are done, you can start sending that." This is assignment 2
  material.
- **Real models.** MLPs only; attention and the rest "just require a lot more
  bookkeeping, so it's harder to see the core algorithms."
- **The backward pass** for tensor and pipeline parallelism — homework exercises.
  For tensor parallelism the lecture does give the rule: all-gather in the forward
  becomes reduce-scatter in the backward, "where, in forward, if you're
  all-gathering, in the backward you're reduce-scattering" ([1:08:08]).
- **Other axes** — [sequence parallelism](tensor-parallelism.md#other-axes) and
  [expert parallelism](expert-parallelism.md), where "the all-to-all that I
  mentioned comes in" ([1:15:51]).
- **Letting a compiler do it.** In "Jax-and-TPU land… you can simply define the
  model and the sharding strategy, and the compiler actually handles a lot of the
  decision" ([1:18:07]). PyTorch is chosen deliberately so the mechanics stay
  visible — "it would take a lot of the joy out of actually building things from
  scratch" ([1:18:54]). See [TPUs](tpus.md).

## Choosing between them

The lecture's closing advice is that the choice is a hardware question before it
is an algorithmic one ([1:15:51]–[1:17:22]). Tensor parallelism goes inside a
node where bandwidth is high. Pipeline parallelism goes across slow links.
Data parallelism goes as far as your batch size allows — until you hit the
**critical batch size**, "where, if you start increasing the batch size too much,
it doesn't actually help you — in which case, you're just wasting your compute,
and then you're better off using tensor-parallel."

Real systems combine them: "tensor-parallel within a node, and then … data-parallel,
or FSDP — and then pipeline parallelism, if you need it".

## The pattern underneath

The closing generalization ([1:19:40]) is the most portable thing in the lecture,
and it connects back through the whole systems unit. When a value is needed and
not to hand, you have three options: **recompute** it, **store** it in memory, or
**store it on another GPU and communicate** it. The first two are
[activation checkpointing](activation-checkpointing.md) from Lecture 2 and the
[memory hierarchy](gpu-architecture.md) from Lecture 5; the third is this lecture.
[Sharding vs. replication](sharding-vs-replication.md) develops it.

Seen that way, data parallelism is deliberately wasteful — "you're doing redundant
work, in some sense, because every rank is actually updating its parameters and
keeping track of all the parameters. But the reason you're doing that is that you
don't have to move the optimizer state across" ([1:19:40]–[1:20:26]).

And the reason none of this goes away: "hardware is getting faster, but, in some
sense, we'll always want bigger models. So, this idea of having a hierarchical
structure will always be there" ([1:20:26]).

## Source material

- Lecture source, transcribed: [`raw/slides/07-parallelism.md`](../raw/slides/07-parallelism.md)
  — the program `lecture_07.py`, section by section with line ranges.
- Transcript: [`raw/transcripts/07-parallelism.md`](../raw/transcripts/07-parallelism.md)
  (copy-edited; the [verbatim captions](../raw/transcripts/original/07-parallelism.md)
  are kept beside it).
- The program itself: [`lecture_07.py`](https://github.com/stanford-cs336/lectures/blob/main/lecture_07.py),
  steppable at <https://cs336.stanford.edu/lectures/?trace=lecture_07>.
- **Its recorded output**, unusually for this course, is published:
  [`var/traces/lecture_07_stdout.txt`](https://github.com/stanford-cs336/lectures/blob/main/var/traces/lecture_07_stdout.txt),
  a real four-GPU run. Numbers quoted from it in this KB are marked "(recorded
  run)" and are measurements of that machine.

## The sequel

This lecture builds the machinery and cuts the network three ways. **Lecture 8 is
the other half**, and it is where the things this lecture repeatedly defers to get
explained: [ZeRO and FSDP](zero-and-fsdp.md), the
[pipeline bubble](zero-bubble-pipelining.md) and how to shrink it,
[activation memory](activation-memory.md) and
[sequence parallelism](sequence-parallelism.md), and how real runs
[combine four or more strategies at once](3d-parallelism.md).

Both lectures are titled "Parallelism" in the course catalog. The division of
labour: this one is Percy Liang's executable lecture on the primitives; lecture 8
is Tatsunori Hashimoto's deck on what you do with them at cluster scale.

Start at [Lecture 8 — Parallelism (Part 2)](08-parallelism-2.md).
