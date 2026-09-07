# Decode-specialized hardware

Chips built for the token-generation phase rather than for training or prompt
processing — and, more usefully for a learner, **the argument for why such a
category exists at all**. From
[Lecture 18](18-serving-megakernels-recurrence.md) (≈20:51–≈21:38, with the
architecture consequences at ≈1:04:11–≈1:05:43).

## Why decode is a different chip's problem

Everything follows from the asymmetry in
[prefill/decode disaggregation](prefill-decode-disaggregation.md). Prefill is
compute-bound and looks like training. Decode is memory-bandwidth-bound: it must
stream the entire model's weights to produce a single token, so its arithmetic
intensity is terrible and its parallelism is thin —

> Instead of using all this big parallelism that you get with a GPU, that you can
> use during prefill or training, you've now turned this massively parallel system
> into basically a glorified memory loader. (≈34:47)

A device whose design point is peak FLOPs is the wrong device for that. The
lecture's statement of the consequence:

> One of the reasons is that the decode workload is so different from the prefill
> workload that if you're looking at decode, you can be using very different
> chips. (≈20:51)

This is the same reasoning [Lecture 5](05-gpus-tpus.md) applies to
[arithmetic intensity](arithmetic-intensity.md), taken one step further: if a
phase is bandwidth-bound and will never use the FLOPs, buy bandwidth instead.

## What the lecture reports

- **Groq** — LPU parts, cited in the context of NVIDIA acquiring the company, with
  a described plan to use "its GPUs for the prefill side, using these LPU Groq
  chips for the decode" (≈20:51).
- **Cerebras** — "another chip that's much better at decode," named alongside an
  OpenAI compute partnership (≈21:38).
- **SambaNova** — "making bets along various parts of this space" (≈21:38).
- **Huawei** parts, inferred from the quantization choices visible in Chinese
  model releases (≈1:04:56).

> **What this KB does and does not vouch for.** These are corporate and commercial
> claims made in passing, in a June 2026 talk with
> [no slides, handout or citation](18-serving-megakernels-recurrence.md) behind
> them. This page records **that the lecturer said them**, because they are the
> evidence he offers for a technical claim that does not depend on any of them
> being true. Do not cite this KB for who acquired or partnered with whom; the
> durable content here is the bandwidth argument and the design advice below.

## The design consequence: memory first, then numerics

The Q&A asks the question that makes this page actionable — if you know the
serving platform in advance, how should the architecture change? The answer is
ordered (≈1:04:56–≈1:05:43):

**1. Memory capacity is the binding constraint.**

> If you know you're going to be taking a model and serving it on a particular
> Cerebras chip, you want to go look at the Cerebras wafer, figure out how much
> memory you have, and then size your model so that it can fit there with enough
> KV cache, or whatever, to spare. (≈1:04:56)

Note that the budget is weights **plus** [KV cache](kv-cache.md) — an architecture
that shrinks the cache, such as [MLA](multi-head-latent-attention.md), buys
parameter budget back.

**2. The numeric format is chosen by the vendor, not by you.** The concrete
example is the sharpest hardware-architecture coupling in the course:

> If you have a model that you're intending to serve on NVIDIA GPUs — for example,
> NVIDIA's Nemotron model that they released — you will train that model in
> NVFP4, an FP4 format that is proprietary to NVIDIA chips. If you're not going to
> run it on NVIDIA chips, like if you're on AMD, then you're going to run this
> other format called MXFP4. They each have their pros and cons. (≈1:05:43)

A *training* decision determined by the intended *serving* silicon. See
[quantization](quantization.md) and [precision and data types](precision-and-data-types.md).

The lecture also reads the inference backwards, as a signal: quantization choices
in recently released Chinese models "suggest they might be starting to think about
the Huawei chips that are coming out" (≈1:04:56). Format choices are legible
evidence of the deployment target.

## The small-memory endgame

The most interesting forward-looking remark ties this page to
[megakernels](megakernels.md) and [looped transformers](looped-transformers.md) at
once. Decode chips have very little on-chip memory — "those have like 250 megabytes
of memory, or something like that" — so almost nothing fits. But if a model were
designed to fit:

> Maybe you can design something that will actually fit into them, and then you
> can just keep your weights in memory the whole time and run your activations
> through as quickly as you can. So there are kind of nonlinear benefits that you
> can get if you can cross some of these thresholds. (≈1:02:38)

That is the appeal of a recurrent block small enough to be resident: it makes
decode's memory-loading problem disappear rather than optimizing it. The lecture
is candid that this has not been achieved — "so far we haven't been able to make
those blocks small enough" (≈1:02:38).

## See also

- [Lecture 18](18-serving-megakernels-recurrence.md) — the source lecture
- [Prefill/decode disaggregation](prefill-decode-disaggregation.md) — the split
  this extends into silicon
- [GPUs and TPUs (Lecture 5)](05-gpus-tpus.md) ·
  [Arithmetic intensity](arithmetic-intensity.md) ·
  [GPU architecture](gpu-architecture.md)
- [Quantization](quantization.md)
