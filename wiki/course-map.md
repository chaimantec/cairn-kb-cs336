# Course map — what CS336 covers, and where

> **Coverage note.** This page is the syllabus as Percy Liang presents it in
> [Lecture 1](01-overview-tokenization.md) ([27:04]–[1:03:57]). It is a map of the
> whole course, but **this knowledge base currently covers Lectures 1 and 2
> only**. Every unit below is a preview here rather than a treatment, except
> tokenization (Lecture 1) and the resource-accounting part of Systems
> (Lecture 2). See [`kb.json`](../kb.json) for exact coverage.

CS336 is five units, each paired with an assignment. The unifying question, stated
at [1:02:23], is [efficiency](efficiency.md): how do you build the best model given
a fixed set of resources — data, compute, memory, communication bandwidth?

| Unit | Assignment | Lectures | In this KB |
| --- | --- | --- | --- |
| [Basics](#unit-1--basics) | 1 | 1–4 | Lectures 1–2 |
| [Systems](#unit-2--systems) | 2 | 5–8, 10 | [Resource accounting only](resource-accounting.md) — from Lecture 2 |
| [Scaling laws](#unit-3--scaling-laws) | 3 | 9, 11 | [Preview only](scaling-laws.md) |
| [Data](#unit-4--data) | 4 | 12–14 | No |
| [Alignment](#unit-5--alignment) | 5 | 15–17 | No |

## Unit 1 — Basics

**Goal:** be able to train a language model from scratch. Roughly the first two
weeks ([27:51]).

Three components:

- **[Tokenization](tokenization.md)** — what atoms does the model operate on?
  Covered in full by Lecture 1; see [byte-pair encoding](byte-pair-encoding.md).
- **Model architecture** — starting from the original Transformer, then the
  refinements: activation functions (ReLU, SwiGLU); positional encodings
  (sinusoidal, RoPE); normalization (LayerNorm, RMSNorm, QK-norm, pre- versus
  post-norm); attention variants (full, sparse/local, GQA, MLA); recurrence and
  linear attention (Mamba, Gated DeltaNet); dense MLP versus mixture of experts;
  and the shape parameters — hidden dimension, depth, heads, experts.
- **Training** — loss function (including multi-token prediction), optimizer
  (AdamW, SOAP, Muon), initialization (Xavier, muP), learning-rate schedule
  (cosine, WSD), regularization, batch size, and MoE load balancing.

Percy's remark at [33:15] is worth carrying forward: this list *looks* like a pile
of hyperparameters you could sweep, but setting them in a principled way is the
difference between a run that blows up and a run that reaches state of the art.

**The balance to keep in mind** ([34:46]): expressivity, stability, efficiency.
See [efficiency](efficiency.md#the-three-way-balance).

**Assignment 1:** implement the BPE tokenizer, the Transformer, cross-entropy,
AdamW and the training loop; do resource accounting; train on TinyStories and
OpenWebText. Leaderboard: minimize OpenWebText perplexity given 45 minutes on a
B200.

## Unit 2 — Systems

**Goal:** squeeze the most out of the hardware ([35:32]).

> **A note on lecture numbering.** Systems is taught in Lectures 5–8 and 10, but
> its first topic — resource accounting — is delivered much earlier, in
> [Lecture 2](02-pytorch-resource-accounting.md), because Assignment 1 needs it.
> Percy opens that lecture by saying the day's subject is resource accounting and
> that it is "more on the systems side of things." So the Lecture 2 material below
> is **covered in this KB**, while the rest of the unit is still a preview. See
> [resource accounting](resource-accounting.md),
> [arithmetic intensity](arithmetic-intensity.md) and
> [training FLOPs](training-flops.md) for the treatment.

**Resource accounting** — where the FLOPs and the memory go. The formula previewed
at [36:18] is $C = 6ND$ for training a model of $N$ parameters on $D$ tokens, and
it is [derived in Lecture 2](training-flops.md).
Concrete hardware numbers from the lecture: a B200 does 2.25 PFLOP/s in bf16 with
8 TB/s of memory bandwidth. The central fact about hardware ([37:04]) is that
**your memory is not where your compute is** — parameters and activations must
move from HBM to the SMs and back, and that movement is usually the bottleneck.
Roofline analysis tells you which side you are bound by; Percy's aside is that in
general it is memory. Lecture 2 makes that concrete — see
[arithmetic intensity](arithmetic-intensity.md), where four of the five operations
worked through are memory-bound and only large matrix multiplication is not.

**Kernels** — a kernel is a function that runs on the GPU, and every PyTorch
primitive launches one whether you know it or not ([38:37]). Writing custom
kernels is about organizing computation to minimize data movement. The canonical
example is fusion: instead of read-HBM → compute A → write-HBM → read-HBM →
compute B → write-HBM, you read once, compute both, write once. Tiling
(FlashAttention) is the more sophisticated version. Written in
CUDA/**Triton**/CUTLASS/ThunderKittens.

**Parallelism** — the same minimize-data-movement principle at 1024 GPUs, where
movement between devices is even more expensive. Collective operations (gather,
reduce, all-reduce); sharding parameters, activations, gradients and optimizer
states; and the five ways to split computation: data, tensor, pipeline, sequence
and expert parallelism.

**Inference** — increasingly important, and needed for RL rollouts, test-time
compute, synthetic data and evaluation, not just chat ([41:39]). Two phases:
**prefill**, where all prompt tokens are processed at once and you are
compute-bound, and **decode**, one token at a time, which is **memory-bound** —
and that is why inference is hard. Lecture 2 supplies the reason in one line:
decoding is a matrix–vector product, whose
[arithmetic intensity is about 1](arithmetic-intensity.md#dot-product-and-matrixvector--12-and-1)
against an H100's requirement of ~295, because every weight is read from memory
and used exactly once. Speedups: cheaper models (pruning,
quantization, distillation); speculative decoding (a draft model proposes several
tokens, the full model scores them in parallel, and the decoding stays exact); and
systems work like fused kernels and continuous batching.

**Assignment 2:** a fused RMSNorm kernel in Triton, distributed data-parallel
training, optimizer state sharding, and benchmarking/profiling. Recommended
reading: [*How to Scale Your Model*](https://jax-ml.github.io/scaling-book/) —
Google, so TPU-focused, but the concepts carry.

## Unit 3 — Scaling laws

Covered in [scaling laws](scaling-laws.md). In one line: think in terms of a
scaling *recipe* mapping FLOP budgets to hyperparameters, fit it at small scale,
and extrapolate — and remember that predictability is at least as valuable as
optimality.

**Assignment 3:** a simulated training API (config in, loss out) backed by cached
offline runs; gather points under a budget, fit scaling laws, extrapolate, submit
predictions.

## Unit 4 — Data

**Goal:** decide what capabilities you want, then get the data that produces them
([53:57]).

**Evaluation** serves two distinct purposes that Percy warns are often conflated
([54:45]):

- **Internal** — guiding development. What matters is smoothness across scales and
  *relative* performance. Perplexity is the workhorse here, and he notes it remains
  a very good measure of intrinsic quality precisely because it is hard to
  benchmax.
- **External** — measuring absolute quality for a real use case, where
  **ecological validity** matters. Examples: GPQA, HLE, SWE-Bench, Terminal-Bench.

Run perplexity on data that is *not* on the internet, to avoid contamination. And
because LMs are general-purpose, you need a diverse evaluation suite — averaging
into one number conflates things ([56:17]).

**Curation** ([56:17]): data does not fall from the sky. Sources are crawled
webpages, books, arXiv, GitHub. Legal questions are live — is training on
copyrighted data fair use, when must you license it — and Percy raises the
practical case of GitHub code with no license at all: do you read that as
permissive or not?

**Processing** ([57:48]): transformation (HTML/PDF → text), filtering (quality and
harm classifiers), deduplication (Bloom filters or MinHash — saves compute and
limits memorization), data mixing (how to weight sources), and synthetic data /
rewriting.

**Three kinds of data** ([59:20]): pre-training (large and diverse), mid-training
(high quality, including long context — big code repositories, books), and
post-training (conversations, agentic traces with tool calling).

**Assignment 4:** Common Crawl HTML to text, quality and harm classifiers, MinHash
deduplication. Percy calls it "dirty work" and says that is the point.

## Unit 5 — Alignment

**Goal:** improve a model that is already reasonable, using **weak supervision**
([1:00:07]).

Why weak supervision? Because it is often easier to critique than to generate. You
cannot always produce the right response to a prompt, but you can often specify
what good looks like.

The basic template ([1:00:53]):

1. Generate responses from the model.
2. Score them with a human, a verifier, or an LM judge.
3. Update the model to prefer better responses.

Algorithms: **PPO** (from RL), **DPO** (simpler, for preference data), and **GRPO**
(removes the value function).

> **A naming note.** The lecture source writes these as "Direct Policy
> Optimization" and "Group Relative Preference Optimization." The papers call them
> *Direct **Preference** Optimization* ([arXiv:2305.18290](https://arxiv.org/pdf/2305.18290.pdf))
> and *Group Relative **Policy** Optimization*
> ([DeepSeekMath](https://arxiv.org/pdf/2402.03300.pdf)). The acronyms are the
> same either way; expect the expanded forms to differ from the course material.

**Challenges** ([1:01:38]): RL algorithms are unstable and hard to tune — Percy
says he personally prefers to stay in the fully supervised regime as long as
possible. At scale it becomes a systems problem: an inference server and a
training server, rollouts against environments that may execute code, and workers
that lag behind and drag you off-policy. The constant trade is on-policyness
against throughput. "It's a big wonderful mess."

**Assignment 5:** implement DPO and GRPO. (At lecture time the staff were still
deciding the final shape.)

## Sources

- [Lecture 1](01-overview-tokenization.md), syllabus tour [27:04]–[1:03:57]
- [`lecture_01.py` transcription](../raw/slides/01-overview-tokenization.md#syllabus-overview)
- [Edited transcript](../raw/transcripts/01-overview-tokenization.md)
- [`sources.md`](../sources.md) — every lecture and assignment, with links
