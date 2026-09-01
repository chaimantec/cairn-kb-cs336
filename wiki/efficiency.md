# Efficiency — the organizing principle of CS336

CS336 is not organized around the Transformer, or around training, or around any
particular technique. It is organized around **efficiency**: what is the best model
you can build given a fixed budget of compute and data? Percy Liang states this at
[10:50] in [Lecture 1](01-overview-tokenization.md) and returns to it at [1:02:23]
to show that every unit of the course is the same question asked about a different
resource.

If you only retain one framing from the first lecture, this is the one.

## accuracy = efficiency × resources

The compact statement, from [9:19]:

$$\text{accuracy} = \text{efficiency} \times \text{resources}$$

where efficiency is output-per-input and resources is the input. It reads as
almost trivial, but it is doing real work — it is Percy's correction to a common
misreading of Rich Sutton's *bitter lesson*.

- **Wrong interpretation:** scale is all that matters, algorithms don't matter.
- **Right interpretation:** *algorithms that scale* are what matter.

The distinction is that the bitter lesson is not an argument against algorithmic
work; it is an argument about *which* algorithmic work pays. An improvement that
does not survive contact with scale is worthless no matter how clever. An
improvement that does is multiplied by every dollar of resources you subsequently
spend.

## Why efficiency matters more as scale grows

The counterintuitive part, and Percy's argument at [9:19]–[10:05]: efficiency is
*more* important at large scale, not less.

At small scale, inefficiency costs you time. If a run takes twice as long, you
wait twice as long and come back later. At frontier scale, the same factor is
hundreds of millions of dollars — so, as he puts it, even a **5% improvement might
be a big deal**. The tolerance for waste falls as the bill rises.

He backs this empirically with a
[2020 OpenAI result](https://arxiv.org/abs/2005.04305): a **44× improvement in
algorithmic efficiency** on ImageNet between 2012 and 2019. Hardware also improved
enormously over that period — the point is that the two multiply. Algorithmic
efficiency and hardware improvement are not competing explanations for progress;
they compound.

## Compute-constrained, for now

The framing at [10:50] is deliberately conditional. For pre-training, CS336
assumes you have far more data than compute, so **compute is the binding
constraint** and design decisions follow from squeezing hardware.

Percy is explicit that this is an assumption about the present, not a law. If
you are data-limited — or if you have "stashed away actually tons of B200s" — you
are data-bound instead and the calculus changes. The syllabus section ends with
the forward reference: *"Tomorrow, we will become data-constrained..."* ([1:03:57]).

This matters when reading the rest of the course. Several decisions that look like
timeless engineering wisdom are downstream of the compute-constrained assumption,
and would be re-derived differently under a data constraint.

## Every unit is this principle

The payoff comes at [1:02:23], where Percy shows the five units are one idea
applied to five resources. This is the most useful single map of the course:

| Unit | The resource | How efficiency shows up |
| --- | --- | --- |
| **Systems** | Compute | Directly and obviously — kernels, parallelism, inference are all about hardware utilization |
| **Tokenization** | Sequence length | Working with raw bytes is elegant but compute-inefficient with today's architectures. See [tokenization](tokenization.md) |
| **Model architecture** | Memory and FLOPs | Many changes exist to reduce one or the other — shared KV caches, sliding-window attention — and a lot are driven by inference speed |
| **Data filtering** | Gradient steps | Don't waste updates on bad or redundant data. Even if bad data doesn't actively hurt, a fixed compute budget means time on bad data is time not spent on good data |
| **Scaling laws** | Experiment budget | Do hyperparameter tuning on small models so you don't spend the large-model budget searching. See [scaling laws](scaling-laws.md) |

The resources being traded are, in Percy's list at [1:02:23]: data, and hardware —
which decomposes into compute cores, memory, and communication bandwidth.

## The three-way balance

A related framing appears at the end of the basics unit ([34:46]), and it is the
one to reach for when a design decision seems arbitrary. Every architecture and
training choice balances three things:

- **Expressivity** — can the model represent the complex dependencies in the data?
- **Stability** — do parameter and gradient norms stay in the "Goldilocks zone",
  neither blowing up nor vanishing? Percy remarks that a great deal of what
  training a language model actually consists of is stability work.
- **Efficiency** — does it run fast on the hardware, in both training and
  inference?

The interesting cases are where these conflict. His example at [35:32]: you can
make something faster by projecting into a lower-dimensional space — but does it
still work as well? Making that trade well is, in his words, the name of the game.

## From principle to practice: you have to be able to count

[Lecture 2](02-pytorch-resource-accounting.md) supplies the step this page's
framing leaves implicit. If efficiency is output per unit of resource, then
maximizing it presupposes that you can *measure* the resource — and Percy states
the dependency directly at the start of that lecture: the goal is to maximize
computational efficiency, so the prerequisite is to understand the compute and
memory characteristics of a given computation.

That is what makes [resource accounting](resource-accounting.md) the second
lecture rather than a systems-unit afterthought. The whole course is an argument
about a ratio, and the denominator has to be countable before the argument can be
made. The concrete tools are:

- [FLOPs and MFU](flops-and-mfu.md) — counting compute, and measuring what
  fraction of the hardware you are actually using
- [Training FLOPs, $C = 6ND$](training-flops.md) — the whole cost of a training run
  in one formula
- [Memory accounting](memory-accounting-for-training.md) — what fits, in bytes per
  parameter
- [Arithmetic intensity](arithmetic-intensity.md) — which of the two resources is
  the binding one for a given operation

The last of these sharpens the framing in a way worth carrying back to this page.
"Efficiency" reads as though there were one resource to be efficient with, but a
GPU has two speed limits — arithmetic and memory bandwidth — and being efficient
means knowing which one you are against. Most operations are memory-bound, so most
efficiency work is about **moving fewer bytes**, not doing less arithmetic. An
MFU of 0.5 counts as good precisely because the other half of the machine was
never reachable for that workload.

## The honest caveat

Efficiency is a *mindset*, and mindset is one of the two kinds of knowledge Percy
claims the course can actually transfer to frontier scale — along with mechanics,
and unlike intuitions ([7:01]). See
[Lecture 1](01-overview-tokenization.md#what-does-transfer) for that distinction,
which is worth keeping in view: the course can teach you to think about efficiency
and to profile and benchmark rigorously. It cannot give you a frontier lab's
intuitions about which data and modelling decisions actually pay off, because
those do not reliably survive the jump across scales.

## Sources

- [Lecture 1](01-overview-tokenization.md) — the bitter lesson at [9:19], the
  framing at [10:50], the per-unit recap at [1:02:23]
- [Lecture 2](02-pytorch-resource-accounting.md) — efficiency restated as the
  motivation for resource accounting
- [Lecture 4](04-attention-alternatives.md) — two structural bets on efficiency:
  sub-quadratic attention, and [mixture of experts](mixture-of-experts.md), which buys
  parameters without buying FLOPs. Its treatment of
  [sparse attention](sparse-attention.md) is also the course's sharpest statement that
  constant factors, not asymptotics, are often what matters.
- [Lecture 5](05-gpus-tpus.md) — efficiency as a hardware fact rather than a design
  principle. It is the lecture that says *why* the resources are hard to use well:
  compute throughput is outgrowing memory bandwidth, so most optimization is
  memory optimization, and a workload that ignores the hierarchy leaves an order of
  magnitude on the table. Its closing advice is the anti-cargo-cult version of this
  page's argument — understand the hardware reason for a rule instead of following
  the rule.
- [Lecture 6](06-kernels-triton.md) — efficiency as something you *measure before
  you optimize*. Its recipe — benchmark, change, benchmark again — is the discipline
  that keeps the rest of this page honest, and its GeLU race is the course's
  cleanest demonstration that three implementations of identical mathematics can
  differ by an order of magnitude for reasons visible only in a
  [profiler](profiling.md). It also puts a floor under the abstraction: the
  programming model is silent about speed, so performance work means knowing the
  hardware underneath it.
- [Lecture 3](03-architectures.md) — efficiency as the thing that actually decides
  architecture. RMSNorm, dropped bias terms, the ~100 aspect ratio and GQA are all
  chosen on systems grounds rather than expressiveness ones.
- [Lecture 7](07-parallelism.md) — efficiency as a communication problem. The same
  minimize-data-movement principle, one level out: "the game is to orchestrate the
  computation to try to avoid data transfer bottlenecks" ([1:38]). Its closing
  generalization — [recompute, store, or communicate](sharding-vs-replication.md) —
  is the most compact statement of the course's whole systems argument, and its
  claim that the hierarchy is permanent ("hardware is getting faster, but we'll
  always want bigger models") is this page's thesis restated in hardware terms.
- [`lecture_01.py` transcription](../raw/slides/01-overview-tokenization.md)
- [`lecture_02.py` transcription](../raw/slides/02-pytorch-resource-accounting.md)
- Edited transcripts: [Lecture 1](../raw/transcripts/01-overview-tokenization.md),
  [Lecture 2](../raw/transcripts/02-pytorch-resource-accounting.md)

## Efficiency at cluster scale

Lecture 8 is the point where the efficiency question stops being about one chip.
Its framing: "the new unit of compute is not the GPU, it's the entire data center"
([10:02]), and what you want from that unit is control over memory, control over
compute, and losslessness — "we want to use all of the resources we have"
([10:48]).

The efficiency currency at this scale is **utilisation**: keeping every accelerator
busy given that some links are fast and some are slow. Every technique in the
lecture is a trade of one resource against another —
[batch size](critical-batch-size.md) for pipeline bubble, memory for batch size,
communication for [activation memory](activation-memory.md), complexity for
utilisation. The [summary table](3d-parallelism.md) exists to show that "there is no
one strictly dominant parallelization strategy" ([1:02:00]).

Two results are worth carrying: [ZeRO stages 1 and 2](zero-and-fsdp.md) are
genuinely **free** — large memory savings at identical communication cost, from a
collective identity ([18:26]) — and, combined properly, utilisation stays flat
"even as you go to ludicrously large numbers of GPUs" ([1:10:27]). That flatness is
why data-centre-scale training is economically sensible at all.
