# torch.distributed and NCCL

The software half of [Lecture 7](07-parallelism.md)'s first part: how the
[collective operations](collective-operations.md) become code you can run, and
what happens underneath when you call one.

## NCCL

At the bottom is the **NVIDIA Collective Communications Library**, which
"translates the collective operations, the all-reduce, reduce, broadcast — into
the actual low-level packets that are sent between GPUs" ([29:27]).

Three things it does for you ([30:15]):

1. **Detects the hardware topology** — how many nodes, which switches, NVLink or
   PCIe.
2. **Optimizes the path** between GPUs.
3. **Launches GPU kernels** to send and receive.

That third point is the one worth internalizing, because it connects this lecture
to the [kernels](06-kernels-triton.md) one: "at the end of the day, remember,
everything that runs on the GPU is a kernel. So, there are communication kernels
as well that actually do communication with other GPUs" ([30:15]). Communication
is not something that happens beside the computation; it is scheduled on the same
device, in the same way.

NCCL is also why you can ignore [topology](gpu-interconnect.md) when reasoning
about cost: whether a collective is implemented as a ring or a tree "is something
that NCCL figures out" ([51:05]), and the effective bandwidth comes out the same
either way.

The course does not go further: "we're not going to look too much more into NCCL,
but just know that it exists" ([30:15]).

## The PyTorch layer

`torch.distributed` "provides a clean interface into these collective operations,
so you don't have to explicitly think about NCCL" ([36:23]). Two backends matter:

- **nccl** — on GPUs.
- **gloo** — on CPUs. Worth knowing it exists, because "parallel processing has
  been around for a long time before GPUs, and you can do these collective
  operations on CPUs as well" ([36:23]) — and because it is what lets you step
  through this lecture on a laptop.

The library also ships higher-level algorithms such as `FullyShardedDataParallel`,
"but we're not going to use those in the course, because we're building things
from scratch" ([36:23]–[37:08]).

## Starting and stopping a process group

Each process calls `setup()` before anything else
([source, lines 540–549](../raw/slides/07-parallelism.md#process-group-setup-and-teardown)).
It sets a master address and port, then initializes the process group with `nccl`
if CUDA is available and `gloo` otherwise.

One thing to be clear about: **the master is only for coordination.** the master is not the data path — "this is more for just
general metadata and coordination — data goes through NCCL, otherwise it'll be
very slow" ([38:44]).

Ranks are numbered "either zero, or one, or two, all the way up to world size
minus one", and "there are world-size number of these functions that are … each
running on a process at the same time" ([37:56]).

## Barriers, and two kinds of asynchrony

`dist.barrier()` "waits for all the processes to get to this point" ([38:44]). You
need it because the processes are genuinely independent: "you can think of all the
processes as running asynchronously. So, I don't really control — one could
completely finish before the other" ([39:30]). It shows up in the lecture's output
as scrambled print order — "the print statements are coming in whatever order the
hardware feels like, because it's running async, but all the data is there"
([40:17]).

Barriers are not free. "The downside of putting more barriers in is that, well,
you end up waiting potentially unnecessarily" ([39:30]).

**There are two independent asynchronies in a distributed PyTorch program**, and
this is the detail that most often bites:

1. **CUDA kernels** are async with respect to the Python process. This is the same
   fact [Lecture 6's benchmarking](benchmarking.md) turned on — "if you're doing a
   CUDA operation, remember, by default, it's async. So, when you reach the next
   line in Python, that CUDA operation might not be done" ([53:24]).
2. **The processes** are async with respect to each other.

So closing out a timed region needs both `torch.cuda.synchronize()` and
`dist.barrier()` ([47:14]). **The order matters.** Asked whether you could barrier
first, Percy works through why not: "if you barrier first, then the CUDA might not
be done running, and you just immediately go to the barrier… which means that
you're not really synchronized — like, the barrier might, if all those operations
just return, the barrier doesn't really do anything" ([54:10]). Synchronize the device,
then the processes.

## Calling a collective

The three workhorses, as the lecture uses them:

**All-reduce** modifies its tensor in place ([41:04]):

```python
dist.all_reduce(tensor=data, op=dist.ReduceOp.SUM, async_op=False)
```

"It calls — in this case it would be gloo, but it could be NCCL — and, in which
case, it would spin up the CUDA kernels. It would do the communication — it takes
care of everything for you. And then it basically writes in place to the data."

**Reduce-scatter** takes separate input and output tensors, and "the input is not
touched" ([42:36]):

```python
dist.reduce_scatter_tensor(output=output, input=input, op=dist.ReduceOp.SUM, async_op=False)
```

**All-gather** likewise ([44:52]–[45:37]):

```python
dist.all_gather_into_tensor(output_tensor=output, input_tensor=input, async_op=False)
```

Chaining the last two reproduces the first, which is the lecture's "proof via
example" that
[all-reduce = reduce-scatter + all-gather](collective-operations.md#the-identity-that-matters)
([45:37]).

A caution on reading output buffers: they are allocated with `torch.empty`, so
before a collective fills them they hold whatever was in that memory. "The output
is — it just allocated, happens to have some values in it, but don't worry about
it" ([44:52]).

## async_op, and overlapping

Every call above passes `async_op=False`. Setting it to `True` returns immediately
and lets you do other work while the collective runs: "this code just would
return, and then you could do other things. So, a typical thing… is overlapping
computation and communication. So, for example, you can do this operation, and
then you can go ahead and load some other data for the next step, which is
independent of this operation. And then, when you want to make sure that you
actually are done, then you can call wait, or a barrier" ([43:21]–[44:07]).

The lecture keeps everything synchronous on purpose — async "would screw up all
these print statements. So, I'm trying to put more barriers than I normally would"
([41:50]) — but flags overlap as
[the main thing its implementations are missing](07-parallelism.md#what-the-lecture-deliberately-leaves-out),
and as assignment 2 material.

For point-to-point work, [pipeline parallelism](pipeline-parallelism.md) uses
`dist.send` and `dist.recv` instead of collectives.

## A wrinkle in reading the lecture

Because this lecture really does launch processes, it cannot be traced the way the
other [executable lectures](executable-lectures.md) are. Its `spawn()` helper
checks whether a tracer is attached, and if so runs the function directly in one
process with **every `torch.distributed` function replaced by a no-op**
([37:08]–[37:56], and
[the source](../raw/slides/07-parallelism.md#how-the-lecture-fakes-multiprocessing-when-traced)).
"This is actually a wrapper I wrote just to hack around the fact that I can't do
multiprocessing in this lecture."

So the stepped-through view shows the code with no communication happening and
rank hard-coded to 0. The real output is published separately, at
[`var/traces/lecture_07_stdout.txt`](https://github.com/stanford-cs336/lectures/blob/main/var/traces/lecture_07_stdout.txt).
