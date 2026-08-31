# Pipeline parallelism

Cut along the **depth**: each rank owns a contiguous block of layers, and
activations flow along the chain from one rank to the next. The third of
[Lecture 7](07-parallelism.md)'s strategies, and the one with a characteristic
pathology — the **bubble** — that most of the engineering goes into fixing.

It is also the only part of the lecture that leaves
[collective operations](collective-operations.md) behind and uses point-to-point
`send` / `recv`.

## The mechanism

Setup: 128 × 1024 data, four layers, **world size 2** (not 4, as in the other two
sections), four micro-batches ([1:09:41]).

**1. Split the layers.** `local_num_layers = num_layers / world_size` = 2. "Each
rank is going to get a subset of the layers now. Within each layer, it's going to
get all the dimensions" ([1:09:41]).

**2. Split the batch into micro-batches.** `micro_batch_size = 128 / 4` = 32.
Rank 0 chunks the real data; every downstream rank allocates empty buffers of the
right shape to receive into ([1:10:26]).

**3. Pass the chain** ([1:11:13]):

```python
for x in micro_batches:
    if rank - 1 >= 0:
        dist.recv(tensor=x, src=rank - 1)      # take activations from upstream

    for param in local_params:                 # my layers only
        x = x @ param
        x = F.gelu(x)

    if rank + 1 < world_size:
        dist.send(tensor=x, dst=rank + 1)      # hand downstream
```

"I'm actually using these receive and send, which are point-wise operations…
This basically says: … I'm going to receive this tensor from rank minus one. And
this says I'm going to send tensor X to rank plus one" ([1:11:13]–[1:12:01]).

In the course's [recorded run](../raw/slides/07-parallelism.md#pipeline-parallelism--implementation),
rank 0 sends four $32 \times 1024$ micro-batches to rank 1.

The backward pass is a homework exercise.

## The bubble

The problem is structural. A rank cannot start until the rank before it has
produced something, and it has nothing to do once it has handed its work on
([1:12:47]):

> This is a very natural way of dividing our deep network, but the problem is that
> you get these what are called **pipeline bubbles**, where, while you're not
> processing, you're waiting around for other tensors to process. And this ends up
> being quite inefficient.

**Micro-batching** is the first-line fix. Rather than pushing one batch of 128
through the whole chain, push four batches of 32: "the idea behind micro-batches is
that you break it up into smaller batches, so you can process it quickly, send it
on to the next one. So, this can reduce the number of pipeline bubbles"
([1:12:47]). Rank 1 can start on micro-batch 1 while rank 0 works on micro-batch 2.

Note that the lecture's implementation *creates* the micro-batches but still walks
them strictly in sequence — the source says so plainly: "Not handled: overlapping
communication/computation to eliminate pipeline bubbles." The structure is there;
the scheduling is not.

**Overlapping communication and computation** is the second fix, and the lecture
calls it "actually very important to pipeline parallelism" ([1:13:32]):

> This basically gives you the right structure. If you put an "i" before these,
> then it becomes async… And the idea here is that, while you're computing here,
> you can be receiving data or sending data. So, computation and communication
> should be overlapped, so that reduces the amount of time you're actually
> spending waiting.

> **Reading note.** The captions garble the phrase about prefixing with "i". The
> transcript reads it as `isend` / `irecv`, PyTorch's asynchronous point-to-point
> calls, and flags that reading as an editorial judgement — the lecture source
> never mentions them. See the `[Ed:]` note at [1:13:32] in
> [the transcript](../raw/transcripts/07-parallelism.md).

The cost of doing this is complexity: "you have to add more things to manage the
code."

## Why you would put up with it

Because it is the technique that survives a bad network. Where
[tensor parallelism](tensor-parallelism.md) needs NVLink, pipeline parallelism
does not ([1:16:37]):

> Pipeline parallelism — you'll see people using it, and, generally, this can
> tolerate much slower interconnects. So, some of the decentralized training work
> uses pipeline-parallel, because your GPUs are actually halfway across the world.
> But you wouldn't want to do tensor-parallel in that setting.

The reason is traffic volume and frequency. Tensor parallelism all-gathers full
activations after *every layer*; pipeline parallelism sends one activation tensor
at each *boundary between blocks of layers*, of which there are far fewer. With
fewer, larger, more predictable transfers, latency matters less and you have
something to overlap against.

The lecture's summary line: pipeline parallelism "can work with slow interconnects,
but need to work to reduce pipeline bubbles" ([1:18:54]).

## In combination

Nobody picks one. The typical arrangement stacks all three by how much bandwidth
each needs ([1:17:22]):

> Tensor-parallel within a node, and then data-parallel, or FSDP — and then
> pipeline parallelism, if you need it.

Tensor parallelism gets the fastest links, data parallelism spans nodes, and
pipeline parallelism is what you reach for when you have run out of the other two
— when the model is too deep to fit any other way, or the machines are too far
apart to do anything else. Combinations "also will show up in the assignment"
([1:15:51]).

## See also

- [Data parallelism](data-parallelism.md) — the batch cut, and the one that
  composes most easily with this.
- [Tensor parallelism](tensor-parallelism.md) — the width cut.
- [GPU interconnect](gpu-interconnect.md) — the bandwidth tiers that decide which
  technique goes where.
- [torch.distributed](torch-distributed.md#async_op-and-overlapping) — `async_op`,
  and what overlapping requires.
