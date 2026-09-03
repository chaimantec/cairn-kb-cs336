# Inference

Inference is the problem of turning a trained model and a prompt into a response,
"usually as accurately and as quickly as you can" ([0:05]). It is the subject of
[lecture 10](10-inference.md), the only lecture in CS336 about what happens after
training, and the course's argument is that it deserves more attention than its
one-lecture allocation suggests.

## Why it matters more than it used to

**Training is a one-time cost; inference is a repeated one** ([1:37]). The
comparison the lecture uses: OpenAI is estimated to process about **8.6 trillion
tokens per day**, while DeepSeek v4 was trained on **32 trillion tokens**. Under
four days of serving equals an entire frontier training run.

**Agents removed the ceiling.** In the chatbot era there was a natural limit to
how much speed was worth buying, because a human reads at a fixed rate. In the
agentic era, a query goes in and the model thinks, reasons, calls tools and
introspects before producing anything a person reads, so "most of the tokens that the agent produces are actually not for reading" ([3:10]). The consequence is the
line worth keeping: **tokens generated = compute spent**, with no saturation
point — "if you have an ambitious enough problem, you're going to need much more
compute and a lot of tokens."

**And it is not only a product concern.** Inference shows up inside the research
loop too: model evaluation needs generation, and reinforcement learning needs
rollouts — "you generate rollouts, you score them, and you update the weights
appropriately" ([0:50]). Anything in [post-training](course-map.md) that samples
from the model pays inference costs.

## Why it is a different problem from training

Training sees all the tokens at once. The sequence is just another tensor
dimension, so a forward pass is a small number of large matrix multiplies and the
hardware is easy to saturate. Inference has to generate autoregressively, one
token at a time, so it cannot parallelize along the sequence — "this is the
fundamental problem of why inference is a very different workload than training"
([7:47]).

The formal version of that difference is the arithmetic-intensity table in
[prefill and generation](prefill-and-generation.md), which finds that generation's
attention layers have intensity below 1 against hardware that needs 295 to
saturate. Hence the sentence the lecture wants you to leave with: "now whenever
you hear people say \"inference is memory-bound,\" you know why" ([34:57]).

## The three metrics

Not interchangeable, and each belongs to a different application ([4:43]–[6:15]):

- **TTFT** (time-to-first-token), in seconds — how long the user waits before
  anything appears. A function of prefill.
- **Latency**, in seconds/token — how fast tokens appear for one query. Interactive
  applications care.
- **Throughput**, in tokens/second — how fast tokens appear across many queries.
  Batch processing cares.

Latency and throughput look reciprocal and are not; see
[latency and throughput](latency-and-throughput.md) for the tradeoff and the
performance model.

## The techniques, grouped by what they cost

The lecture's own three-way split ([45:47] onward):

**Lossy — change the model.** [Reduce the KV cache](kv-cache.md) with
[GQA](attention-variants.md), [MLA](multi-head-latent-attention.md),
[CLA](cross-layer-attention.md) or local attention; reduce precision with
[quantization](quantization.md); reduce size with
[pruning and distillation](pruning-and-distillation.md). Every one of these can
cost accuracy, which is why the lecture pairs each with an accuracy check — and
why it warns that those checks are somebody's benchmark table: "take everything
that's not just math with a grain of salt" ([51:06]).

**Lossless — use a shortcut but verify it.**
[Speculative sampling](speculative-sampling.md) drafts with a cheap model and
checks with the expensive one, and returns a provably exact sample from the
expensive one.

**Systems — don't change the model at all.**
[Continuous batching](continuous-batching.md) and
[PagedAttention](paged-attention.md) address the shape of live traffic rather than
the shape of the model.

The lecture's summary compresses all three: they are "driven by the same principle: reduce
your KV cache, but don't hurt accuracy too much", plus two ideas
borrowed from operating systems — paging, and speculative execution ([1:23:18]).

## The serving landscape

Named in the lecture ([3:57]), and useful context for which package you would
actually reach for:

- **vLLM** — from Berkeley, pioneered [PagedAttention](paged-attention.md),
  "popular and good default".
- **SGLang** — also from Berkeley, pioneered RadixAttention, "particularly good for
  agentic workloads, but maybe not as popular yet".
- **TensorRT-LLM** — from NVIDIA, highly optimized for GPUs, "really fast, but it's
  more narrow".
- **llama.cpp** — C++ only, supports CPU inference, runs locally.

Commercially, both the closed-API providers and a layer of open-weight serving
providers (Together, Fireworks, Baseten, DeepInfra, Groq, Cerebras) exist entirely
to do this well.

## Where the lecture thinks the gains are

Not in any of the techniques above, which are patches. "At some level, the KV cache, and the way that attention is built, fundamentally
makes it an inference-unfriendly kind of architecture. So, if you can come up with a
new architecture that's designed for inference in the way that the Transformer was
not, this could maybe unlock a lot"
([1:24:51]). The candidates named are
[linear attention](linear-attention.md),
[state space models](state-space-models.md), and diffusion models as a
non-autoregressive way to generate ([1:04:11]).

## Related pages

- [Lecture 10 — Inference](10-inference.md) — the lecture page.
- [KV cache](kv-cache.md) — the object at the centre of it.
- [Prefill and generation](prefill-and-generation.md) — the two phases.
- [Latency and throughput](latency-and-throughput.md) — the metrics and the model.
- [Efficiency](efficiency.md) — the course-wide theme this is one face of.
- [Course map](course-map.md) — where this sits in CS336.
