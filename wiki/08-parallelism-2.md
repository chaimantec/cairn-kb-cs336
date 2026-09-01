# Lecture 8 — Parallelism (Part 2)

**Tatsunori Hashimoto.** The second of two lectures called "Parallelism", and the
one that answers the questions [lecture 7](07-parallelism.md) kept deferring.
Where Percy Liang built the primitives from `torch.distributed` upward, this
lecture takes them as given and asks what you actually do with them on a real
cluster: how [ZeRO and FSDP](zero-and-fsdp.md) shard training state for free, why
[pipelines](pipeline-parallelism.md) have a bubble, what
[activation memory](activation-memory.md) really costs, and how frontier runs
combine four or more strategies at once.

The deck's own title is "Parallelism Basics". Its structure is three parts:
networking, then the parallelism primitives, then case studies.

- **Course material:** [`lecture_08.pdf`](https://github.com/stanford-cs336/lectures/blob/main/lecture_08.pdf), 73 pages, transcribed at [`raw/slides/08-parallelism-2.md`](../raw/slides/08-parallelism-2.md)
- **Transcript:** [`raw/transcripts/08-parallelism-2.md`](../raw/transcripts/08-parallelism-2.md)

## What this lecture establishes

Two bottlenecks force you off a single chip: **compute**, because the fastest
supercomputers have exaflops and one GPU does not, and **memory**, because models
do not fit ([1:38]). Everything else follows from a single distinction that runs
through the whole lecture — **fast intra-node links versus slow inter-node links**
([1:38]) — because that is what decides which cut you can afford where.

The discussion stays at the level of [collectives](collective-operations.md), not
packets: "our discussion today is going to be algorithmic" ([2:23]). One identity
in particular is flagged early as load-bearing: **an all-reduce costs the same as a
reduce-scatter plus an all-gather** ([3:08]). That single fact is why two of the
three ZeRO stages are free.

## Part 1 — the network

The [topology](network-topology.md) question splits the hardware world in two. TPUs
use a **toroidal mesh**: neighbours only, with edges wrapped, so the number of
neighbours per chip stays constant however large the machine grows ([5:26]). GPUs
use a **fat tree**: fast at the bottom, pods above, spine switches above that, with
complexity growing as you add nodes ([6:12]).

Neither is better. Mesh is cheap and scales indefinitely for predictable,
neighbour-local traffic; tree is flexible for "random, stochastic, unpredictable"
traffic ([6:12]) — which is what [MoE](mixture-of-experts.md) routing produces. And
the two are converging: Google announced TPU8i and TPU8t the morning of the
lecture, with tree-like topology and a cross-rack fabric that "looks a lot more
like a GPU" ([7:45]).

The brute-force corner is the Huawei Ascend 910: chips slower than an H200, but 384
of them in one fibre-optic rack, at four times the power of the equivalent NVIDIA
system ([9:16]). "If you're willing to pay the power cost, you can solve a lot of
communication problems by brute force" ([10:02]).

The conclusion for the rest of the course: **the unit of compute is now the data
center**, not the GPU ([10:02]).

## Part 2 — the primitives

### Data parallelism, and why it is not enough

Cut the batch across machines, compute gradients, synchronise ([12:21]).
Compute scaling is perfect; communication is $2\times$ parameters per step; memory
scaling is **zero** — every GPU holds an identical copy ([13:06]).

And that copy is expensive. Training costs roughly five copies of the weights and
**16 bytes per parameter**, most of it Adam's first and second moments ([13:52]).
So the entire memory budget is being replicated for no benefit.

### ZeRO: sharding the replicated state

[The full treatment is here](zero-and-fsdp.md). Three stages shard progressively
more: stage 1 the optimizer state, stage 2 the gradients, stage 3 (**FSDP**) the
parameters too, running slide 18's example from 120 GB down to 1.9 GB ([16:09]).

The remarkable part is the price. Stage 1 replaces DDP's one all-reduce with a
reduce-scatter plus an all-gather — *the same cost* — so "ZeRO stage 1 has the exact
same communication characteristics as naive DDP — this was free" ([18:26]). Stage 2
is free too, by reducing gradients incrementally as the backward pass sweeps.
Stage 3 does cost one extra all-gather per layer, but overlapping communication
with computation hides most of it: "if you run FSDP, you're going to see GPU
utilization very close to just the single-GPU performance" ([25:20]).

You implement FSDP yourself in the assignment ([25:20]).

### Why data parallelism runs out

Two walls ([29:07]–[29:53]):

1. **It consumes [batch size](critical-batch-size.md).** With a batch of 8 you can
   never use more than 8 accelerators, and you cannot grow the batch indefinitely
   because past the critical batch size an extra example is worth less than an
   extra step.
2. **It does not touch activations.** Slide 30 is explicit that even ZeRO stage 3
   "does not reduce activation memory".

Both walls are what [model parallelism](sharding-vs-replication.md) exists to get
past. The conceptual shift: in FSDP, *parameters* fly around the network; from here
on, **activations** do ([30:38]).

### Pipeline parallelism — the depth cut

Put different layers on different GPUs and pass activations forward, partial
gradients backward ([31:25]). Done naively the picture is "truly, truly terrible" —
one GPU active at a time, the rest idle in what is called the **bubble** ([32:56]).

Micro-batching fixes it, at a price: utilisation goes as the number of stages over
the number of micro-batches, so "we need a huge batch size in order to reduce this
bubble towards zero" ([33:42]). Batch size again, spent differently.

So why tolerate it? Because its **communication properties are the best available**
([34:27]). It moves activations of size $B \times S \times H$, and it is
point-to-point rather than all-to-all. That is why the standard placement is
counterintuitive but firm: "in practice, what we're going to do is put pipelines on
the slowest networking links" ([35:14]).

[Zero-bubble pipelining](zero-bubble-pipelining.md) is the clever refinement:
split the backward pass into propagating partials (**B**, on the critical path) and
computing weight gradients (**W**, a leaf that can wait), run the Bs eagerly and
defer the Ws into the gaps ([38:16]).

### Tensor parallelism — the width cut

Cut the matrices instead of the layers ([39:02]). Slide 40's worked example splits
$A$ and $B$ around a GeLU into two sub-matrices each. The structure to remember is
the **forward/backward duality**: in the forward pass $f$ is identity and $g$ is an
all-reduce; in the backward pass they swap ([40:34]).

In a transformer block the cuts alternate — **column-wise** at the inputs (MLP
inputs, attention projections), **row-wise** at the corresponding outputs (MLP
down-projections, attention outputs), with small layers like LayerNorm, the
nonlinearity and MoE routers left fully replicated ([41:20]–[42:05]).

The constraint is bandwidth. Tensor parallelism is "extremely communication-hungry
— every time you have a matmul, you're doing some communication", activation-sized
and frequent ([42:05]). Hence **stay inside the node, degree ≤ 8** ([42:50]) — a
GPU constraint, not a universal one, since a TPU mesh has no such boundary
([43:36]).

Against pipelines: no bubble and low complexity, but communication grows from
point-to-point $B \times S \times H$ to roughly $8 \times B \times S \times H$
all-to-all ([44:21]).

### Activation memory, and the two fixes

[The accounting](activation-memory.md). Memory is dynamic and peaks *after* the
forward pass, while the backward sweep still holds activations ([45:07]). Per
layer, storing everything:

$$sbh\left(34 + 5\frac{as}{h}\right)$$

Tensor parallelism divides the MLP's 24 and the attention term by $t$ — but **not
the remaining 10**, which is LayerNorm, dropout and the residual inputs ([48:10]).
"You're still going to suffer a $10\times SBH$ penalty" ([48:56]).

[Sequence parallelism](sequence-parallelism.md) removes exactly that penalty by
splitting those pointwise ops along the **sequence** axis instead of the hidden one
([49:43]), giving fully linear scaling. Add
[selective recomputation](activation-checkpointing.md) and you reach $sbh \cdot
34/t$, "the lower bound of what you can reasonably achieve for normal training"
([51:16]) — the number to use when working out by hand whether a model will fit.

### Expert parallelism

For [MoE](mixture-of-experts.md) models, shard whole experts rather than slices of
a matrix ([53:34]). It behaves like tensor parallelism — high bandwidth, reduces
activations — but is preferred over it whenever both apply, and Megatron's
guidelines say so directly ([54:19]). The reasons: cutting matrices too finely
starves GPU utilisation, and routing sparse token activations is easier than
routing dense tensor-parallel ones ([55:04]).

It is also the hardest to implement, which is why DeepSeek and NVIDIA both ship
dedicated libraries — DeepEP and HybridEP — for all-to-all dispatch. The DeepEP
story is the lecture's favourite piece of trivia: the DeepSeek engineers found
**undocumented [PTX](ptx.md) instructions** to accelerate networking ([57:23]).

Two composition complications, unique to expert parallelism ([58:08]–[1:00:25]):
naive implementations tie the EP domain to the DP domain, bounding how far EP can
scale; and because MoE changes only the MLPs, expert parallelism applies
**unevenly** — you want high tensor parallelism for attention and low tensor
parallelism for the experts. The modern fix decouples the two.

### Context parallelism

[Ring attention](context-parallelism.md), covered briefly: split a long sequence
across accelerators and pass blocks around a ring ([1:01:12]). Used for
long-context extension and serving.

## Part 3 — putting it together

Slide 55's table colours each strategy's drawbacks in red to make one point:
**nothing dominates** ([1:02:00]). You compose them because their limits differ.

The [prescription](3d-parallelism.md) is short ([1:05:51]): cut the model up until
it fits — tensor or expert parallel on the fast interconnect (degree 8), then
pipeline or FSDP the rest of the way — and once it fits, data-parallel with
everything left over. NVIDIA's Megatron guidance says the same in reverse
([1:07:22]).

Narayanan et al. 2021 supplies the evidence: configurations follow exactly that
pattern as scale grows, utilisation stays flat "even as you go to ludicrously large
numbers of GPUs" ([1:10:27]), tensor parallel = 8 is quantitatively the right
stopping point ([1:11:12]), and activation recomputation pays for itself by buying
batch size ([1:11:57]).

Then ten [real runs](parallelism-case-studies.md) show it in the wild: OLMo-7B on
FSDP alone, Llama 3 405B's per-stage breakdown, Gemma 2 with no pipeline at all on
a TPU mesh, and DeepSeek V3's 64-way expert parallelism.

## The through-line

The lecture's own recap ([1:18:55]–[1:19:41]): scaling past a point requires
multi-GPU, multi-node, possibly multi-data-centre parallelism; no single strategy
suffices; but the rules of thumb for combining them are simple and get you
effectively full utilisation.

Underneath, one idea recurs in four disguises — **store it sharded and materialise
it on demand**. It is ZeRO stage 2's incremental gradient reduction, stage 3's
per-layer all-gather, sequence parallelism's on-demand pointwise activations
([50:28]), and [activation checkpointing](activation-checkpointing.md)'s
recomputation. The lecture points at the resemblance itself, calling sequence
parallelism "conceptually very similar to the FSDP-style idea" ([50:28]).

Next lecture: [scaling laws](scaling-laws.md) ([1:19:41]).

## Pages from this lecture

- [ZeRO and FSDP](zero-and-fsdp.md) — the three stages, and why two are free.
- [Activation memory](activation-memory.md) — the $34sbh$ accounting and its five rows.
- [Sequence parallelism](sequence-parallelism.md) — splitting the stubborn $10sbh$.
- [Context parallelism](context-parallelism.md) — ring attention.
- [Zero-bubble pipelining](zero-bubble-pipelining.md) — separating B from W.
- [Critical batch size](critical-batch-size.md) — batch size as a budget.
- [Network topology](network-topology.md) — mesh vs tree.
- [3D parallelism](3d-parallelism.md) — the composition rules.
- [Parallelism case studies](parallelism-case-studies.md) — what ten real runs chose.

## See also

- [Lecture 7 — Parallelism](07-parallelism.md) — the primitives this builds on.
- [Lecture 5 — GPUs, TPUs](05-gpus-tpus.md) — the hardware one level down.
- [Lecture 4 — Attention alternatives and MoE](04-attention-alternatives.md) — where MoE is introduced.
