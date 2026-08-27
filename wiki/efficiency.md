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
- [`lecture_01.py` transcription](../raw/slides/01-overview-tokenization.md)
- [Edited transcript](../raw/transcripts/01-overview-tokenization.md)
