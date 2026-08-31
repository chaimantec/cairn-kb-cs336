# Collective operations

A **collective operation** is a communication pattern specified across a whole
group of devices at once, rather than as individual messages between pairs. They
are the programming model for everything in
[Lecture 7](07-parallelism.md), and they long predate deep learning: "these
primitives from distributed programming that go back to the '80s. So, the idea of
parallel programming is very old. It wasn't invented for LM training, and it's
still the case that these primitives are the ones that we use today" ([5:31]).

The argument for them over point-to-point messaging is that you say *what* you
want, not *how*: "collective just means that you're specifying a general
communication pattern, or a template, across multiple devices, rather than
managing point-to-point how this GPU is going to communicate with another GPU.
And this is going to be much easier, and the system can do a lot more work for
you" ([6:18]). The system in question is [NCCL](torch-distributed.md#nccl), which
picks the actual routes.

## Two words of vocabulary

- A **rank** is one device. "A rank corresponds to a particular device — in our
  case a GPU, could be a TPU" ([6:18]). For CS336's purposes, "the rank is the
  GPU" ([41:50]).
- The **world size** is how many devices there are ([7:04]).

Ranks are numbered $0$ to $W-1$ where $W$ is the world size; the lecture's running
example uses $W = 4$.

The naming scheme is regular once you see it ([19:25]–[20:12]):

- **Reduce** applies an associative, commutative operation — "could be sum, could
  be max, could be min."
- **Scatter is the inverse of gather.** "Scatter distributes, gather centralizes."
- **All** means the destination is every device, not just rank 0.

So *all-reduce* is a reduce whose result lands everywhere, and *all-gather* is a
gather whose result lands everywhere.

## The eight operations

The lecture sorts them into three tiers ([7:04], [7:50]). The examples below are
the source program's own, reproduced in
[the transcribed material](../raw/slides/07-parallelism.md#collective-operations--setup).

### The four warm-ups

These "are really just warm-ups, I would say" — they show what a collective
operation *is*, but "they're not really going to be the ones that are driving most
of training" ([7:04]–[7:50]). (The captions garble the clause between those two
fragments; see the `[Ed:]` note at [7:04] in the transcript.)

**Broadcast** — copy rank 0's tensor to every rank.

```
in:   rank0 = [0,1,2,3]
out:  rank0 = rank1 = rank2 = rank3 = [0,1,2,3]
```

Its real use is initialization: "you initialize — a load initial checkpoint — and
then you want to broadcast it to all the ranks. So, something that's done once"
([8:36]).

**Scatter** — cut rank 0's tensor into $W$ pieces and send piece $i$ to rank $i$.

```
in:   rank0 = [0,1,2,3]
out:  rank0 = [0]   rank1 = [1]   rank2 = [2]   rank3 = [3]
```

"Scatter just takes a big tensor at one place and spreads it out onto multiple
places. And you can see how this might be helpful, because you want all the GPUs
you're scattering to, to do some local computation on the different parts"
([9:23]).

**Gather** — the inverse: collect one piece from each rank onto rank 0 ([10:08]).

**Reduce** — same input shape as gather, but combine the pieces with an operation
instead of concatenating.

```
in:   rank0 = [0]  rank1 = [1]  rank2 = [2]  rank3 = [3]
out:  rank0 = [6]                      # 0+1+2+3
```

A nice way to hold the pair together, from [10:55]: "you can think about gather as
a reduction where the operation is concatenation, if you will."

### The three workhorses

"All-gather, reduce-scatter, and all-reduce are the ones that are going to show up
again and again for distributed training of language models" ([7:50]).

**All-gather** — gather, but the result lands on every rank ([12:26]).

```
in:   rank0 = [0]  rank1 = [1]  rank2 = [2]  rank3 = [3]
out:  every rank = [0,1,2,3]
```

Its use in training: "each rank holds part of the parameters, and then what you
need to do is all-gather the parameters to get the full parameters for the full
forward pass" ([13:12]). That is exactly what
[tensor parallelism](tensor-parallelism.md) does with activations, and what
FSDP does with parameters.

**Reduce-scatter** — reduce *per position*, and leave position $i$'s result on
rank $i$. This is the one that repays drawing out:

```
in:   rank0 = [0,1,2,3]      out:  rank0 = [6]     # 0+1+2+3, position 0
      rank1 = [1,2,3,4]            rank1 = [10]    # 1+2+3+4, position 1
      rank2 = [2,3,4,5]            rank2 = [14]    # position 2
      rank3 = [3,4,5,6]            rank3 = [18]    # position 3
```

"For each component of this tensor, I'm going to do a reduction, and then I'm
going to put it on a different rank" ([13:59]). Its use: "you need to sum all of
the gradients from the different shards, and then you're going to … redistribute …
this storage" ([14:47]) — the reduction you want, without every rank having to
store the whole result.

**All-reduce** — reduce, replicated everywhere.

```
in:   rank0 = [0,1,2,3] … rank3 = [3,4,5,6]
out:  every rank = [6,10,14,18]
```

"You have a bunch of tensors, you reduce — in this case, sum — and then you
replicate them on all the nodes" ([15:33]). This is the operation
[data parallelism](data-parallelism.md) is built on.

### The general one

**All-to-all** — every rank sends a distinct piece to every other rank. "You
basically specify how each rank sends a particular message to another rank"
([16:20]).

```
in:   rank0 = [0,1,2,3]        out:  rank0 = [0,4,8,12]
      rank1 = [4,5,6,7]              rank1 = [1,5,9,13]
      rank2 = [8,9,10,11]            rank2 = [2,6,10,14]
      rank3 = [12,13,14,15]          rank3 = [3,7,11,15]
```

Position $j$ of rank $i$'s input goes to rank $j$ — "the position here is going to
denote … which rank is going to be the ultimate destination" ([17:07]). Rank $j$'s
output is therefore column $j$ of the input matrix, so **when the splits are
balanced, all-to-all is a transpose**: "if you think about this as a matrix, all
you're doing is transposing that matrix" ([18:39]).

It exists for [mixture-of-experts](mixture-of-experts.md) models, where "each rank
has both a split of the data and also a subset of experts", and because
[routing](moe-routing.md) is dynamic, "you have to look at your data to figure out
which experts … you need to route those activations to. So, it ends up being an
all-to-all communication" ([17:54]–[18:39]). Unbalanced splits are allowed — you
can send any number of bytes to any rank — but the transpose picture is the ideal,
which is the systems reason
[load-balancing losses](load-balancing-losses.md) matter: "remember Tatsu's
lecture, where we had load balancing to make sure that things were as balanced as
possible. So, morally, the ideal goal is to have the all-to-all look like this"
([19:25]). See [expert parallelism](expert-parallelism.md).

## The identity that matters

$$\text{all-reduce} = \text{reduce-scatter} + \text{all-gather}$$

Reduce-scatter leaves $[6], [10], [14], [18]$ spread across the four ranks;
all-gather then puts all four on all four. "If you understand reduce-scatter and
all-gather, it's basically you do one and then you do the other" ([14:47]). The
lecture demonstrates it on real tensors and calls the result "proof via example"
([45:37]).

This is not a curiosity. All-reduce is monolithic and leaves every rank holding
the full tensor; splitting it in two gives you a point to intervene, which is what
sharded optimizers need: "later, we're going to see how to get to fancier things
like ZeRO or FSDP — we need to break the all-reduce into reduce-scatter and
all-gather, because then you can intervene and manage things a bit more"
([16:20]). The treatment is Lecture 8, which this KB does not cover.

There is a matching duality in autograd. For [tensor
parallelism](tensor-parallelism.md), "in forward, if you're all-gathering, in the
backward you're reduce-scattering" ([1:08:08]).

## Practical notes

- **The target rank is not fixed.** Asked whether rank 0 is always the
  destination for gather and reduce: "you basically specify the GPU ID, or the
  rank, and it goes there. So, it doesn't have to be determined way in advance,
  but it has to be determined basically when you execute the call" ([20:58]).
- **It is not NumPy broadcasting**, though the intuition rhymes. "Conceptually the
  same idea, where you have one thing that goes to many things… But the
  instantiation — this is for collective communication, so it's a bit different"
  ([11:41]–[12:26]).
- **These are real code**, not just diagrams — see
  [`torch.distributed`](torch-distributed.md) for the API and
  [benchmarking](benchmarking.md#measuring-a-collective) for what they cost.

## Where this goes

- [Data parallelism](data-parallelism.md) — all-reduce on gradients.
- [Tensor parallelism](tensor-parallelism.md) — all-gather on activations, every
  layer.
- [Pipeline parallelism](pipeline-parallelism.md) — the exception, built on
  point-to-point `send`/`recv` instead.
- [Expert parallelism](expert-parallelism.md) — all-to-all.
