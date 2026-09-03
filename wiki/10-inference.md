# Lecture 10 — Inference

**Percy Liang.** Every lecture before this one was about *training*: building the
model (1–4), making one chip and then many chips go fast (5–8), and deciding what
size model to train (9). This is the first lecture about what happens afterwards —
"you've trained a model, and you're given a prompt, and you want to produce the
response, usually as accurately and as quickly as you can" ([0:05]). It is spliced
between the two scaling-laws lectures for scheduling reasons, so the course order
runs scaling laws, inference, scaling laws again.

- **Course material:** [`lecture_10.py`](https://github.com/stanford-cs336/lectures/blob/main/lecture_10.py), 611 lines, transcribed at [`raw/slides/10-inference.md`](../raw/slides/10-inference.md)
- **Rendered:** <https://cs336.stanford.edu/lectures/?trace=lecture_10>
- **Transcript:** [`raw/transcripts/10-inference.md`](../raw/transcripts/10-inference.md)
- It is an [executable lecture](executable-lectures.md), so there are no slide
  numbers — citations below are to function names and to transcript timestamps.

## What this lecture establishes

One derivation and its consequences. The derivation is short: apply the
[arithmetic intensity](arithmetic-intensity.md) accounting from lecture 2 to a
Transformer's two phases, and you find that **[prefill is compute-bound and
generation is memory-bound](prefill-and-generation.md)** ([33:17]) — and that the
memory-bound half cannot be fixed by batching, because each sequence carries its
own [KV cache](kv-cache.md) ([31:46]).

Everything after that is a way of buying back memory traffic, and the lecture
groups the methods by what they cost you:

1. **Lossy** — change the model so there is less to read. Architectures with a
   smaller KV cache ([GQA](attention-variants.md),
   [MLA](multi-head-latent-attention.md),
   [CLA](cross-layer-attention.md),
   [local attention](attention-variants.md)),
   [quantization](quantization.md), and
   [pruning with distillation](pruning-and-distillation.md). Each is presented with
   an accuracy check, because each can lose accuracy.
2. **Lossless** — [speculative sampling](speculative-sampling.md), which guesses
   with a cheap model, verifies with the expensive one, and provably returns an
   exact sample from the expensive one ([1:14:54]).
3. **Systems** — [continuous batching](continuous-batching.md) and
   [PagedAttention](paged-attention.md), which do not touch the model at all and
   instead handle the fact that real traffic is ragged.

The sentence Percy asks the class to keep is the memory-bound one: "now whenever you hear people say "inference is memory-bound," you know why" ([34:57]).

## Why inference deserves a lecture

The economic argument is a ratio. Training is a one-time cost; inference is
"a repeated cost. You incur it every single day" ([1:37]). OpenAI is estimated
to process about **8.6 trillion tokens per day**, and for reference DeepSeek v4 was
trained on **32 trillion tokens** — so in under four days of serving, the token
volume matches an entire frontier training run ([1:37]).

The second argument is that the ceiling moved. In the chatbot era there was a
natural limit to useful speed, because "humans can only read so fast" ([3:10]). In
the agentic era there is none: a query goes in, the agent thinks, reasons, calls
tools and introspects, and "most of the tokens that the agent produces are actually not for reading" ([3:10]). So **tokens generated = compute spent**, with no
saturation point.

The lecture names the serving landscape ([3:57]): closed-API providers; open-weight
providers (Together, Fireworks, Baseten, DeepInfra, Groq, Cerebras); and four
open-source packages — **vLLM** (Berkeley, pioneered PagedAttention, "popular and
good default"), **SGLang** (Berkeley, pioneered RadixAttention, good for agentic
workloads), **TensorRT-LLM** (NVIDIA, "really fast, but it's more narrow") and
**llama.cpp** (C++, CPU inference, runs locally).

### The three metrics

They are not interchangeable, and the lecture says which application each belongs
to ([4:43]–[6:15]):

| Metric | Units | What it measures | Who cares |
| --- | --- | --- | --- |
| **TTFT** (time-to-first-token) | seconds | how long the user waits before *any* generation | interactive |
| **Latency** | seconds/token | how fast tokens appear for *one* query | interactive |
| **Throughput** | tokens/second | how fast tokens appear for *many* queries | batch processing |

Latency and throughput look like reciprocals, and for a single request they are.
[They come apart as soon as there is a batch](latency-and-throughput.md), and that
tension is the subject of a whole section.

## The derivation

The full accounting is on [arithmetic intensity](arithmetic-intensity.md) and
[prefill and generation](prefill-and-generation.md); the shape of it is this.

**The warm-up** ([13:53]–[17:48]). For one matrix multiply $X\,(B \times D)$ times
$W\,(D \times F)$ in bf16, the FLOPs are $2BDF$ and the bytes are
$2BD + 2DF + 2BF$, so

$$\text{intensity} = \frac{BDF}{BD + BF + DF} \;\xrightarrow[\;D, F \gg B\;]{}\; B$$

**The arithmetic intensity of a matrix multiply is the batch size.** An H100's own
ratio is $989 \times 10^{12} / 3.35 \times 10^{12} = 295$, so you are compute-bound
only when $B > 295$ ([17:48]). At $B = 1$ — a matrix-vector product — the intensity
is 1, "basically kind of the workload that you'll see in inference. You don't
get these full matrices — you get these very thin matrices, or tensors" ([18:35]).

**Naive generation is cubic** ([20:53]). Feeding the whole history back through the
Transformer for each new token costs $O(T^2)$ per token and so $O(T^3)$ for $T$
tokens. But in a causal model the earlier activations do not change when you append
a token — "If it were bidirectional, then if you attach a token, everything changes. But
if it's causal, then the activations here don't change based on any tokens you
append" ([22:26]) — so
you store the [KV cache](kv-cache.md) and reuse it.

**The two phases then have opposite characters** ([25:33]–[33:17]):

| | MLP intensity | Attention intensity |
| --- | --- | --- |
| **Prefill** ($T = S$) | $BS$ — compute-bound, "easy to make large" | $S/2$ — "not as good, but workable" |
| **Generation** ($T = 1$) | $B$ — workable *if* you have concurrent requests | $S/(S+1) < 1$ — **the bottleneck** |

And the reason attention cannot be rescued by batching is structural: "in MLP
layers, every sequence hits the same MLP weights… in attention layers, every
sequence has its own KV cache vectors" ([31:46]). $B$ cancels out of the attention
ratio entirely, so "increasing B doesn't help. It's like for every sequence you're
basically doing a matmul" ([32:32]). "If you're sticking with a transformer, you can't really improve this" ([34:03]).

## What the model costs, concretely

The lecture turns the derivation into a small symbolic performance model and
instantiates it for **Llama 2 13B on an H100** ([35:43]–[43:25]). Full numbers,
recomputed, are on [latency and throughput](latency-and-throughput.md); the table
is the argument:

| Batch size | Memory | Latency | Throughput |
| --- | --- | --- | --- |
| $B = 1$ | 26.87 GB | 8.02 ms/token | 124.7 tok/s |
| $B = 64$ | 79.72 GB | 23.80 ms/token | 2,689.5 tok/s |
| $B = 256$ | 240.78 GB | 71.87 ms/token | 3,561.8 tok/s — **exceeds the H100's 80 GB** |

Two things to read off it. Latency is linear in $B$ because the KV cache is $O(B)$
and must be read every step; throughput rises but asymptotes, because once the
parameter read is amortized there is nothing left to amortize. And the ceiling is
memory, not compute: at $B=256$ the configuration simply does not fit ([42:38]).

Percy's analogy for the tension is a bus: "It's basically like waiting for a bus — the latency is pretty high, you wait, and then you go. Whereas the throughput of a bus is pretty good, because you can move everyone at once" ([43:25]).

The practical corollary, since TTFT is essentially prefill time: **small batches
during prefill, large batches during generation** ([44:57]).

## Reducing the KV cache

The KV cache has four axes — sequence $S$, layer $L$, head count $K$, and dimension
$H$ — and each method in this section picks a different one to shrink. That framing
is the through-line ([46:33]–[1:03:22]); the details are on
[KV cache](kv-cache.md) and the individual pages.

- **[GQA](attention-variants.md)** cuts $K$. Already covered in
  [lecture 4](04-attention-alternatives.md), but here it is put through the
  performance model: going from $K=40$ to $K=8$ on Llama 2 13B cuts the cache by
  exactly $N/K = 5$, and the $B=256$ configuration that did not fit in 80 GB now
  fits in 65.6 GB with throughput of 13,068 tok/s ([48:50]–[50:20]). Percy's aside
  on the endpoints: multi-query attention ($K=1$) is the one "no one uses because
  it's really bad" ([47:19]).
- **[MLA](multi-head-latent-attention.md)** cuts the stored dimension, compressing
  $NH = 16384$ down to $C = 512$ in DeepSeek v2 — "quite aggressive compression"
  ([52:38]) — at the cost of a RoPE incompatibility patched with 64 extra
  dimensions.
- **[CLA](cross-layer-attention.md)** cuts $L$, sharing KVs across layers "just as
  GQA shares KVs across heads" ([55:43]).
- **[Local (sliding-window) attention](attention-variants.md)** cuts $S$, which is
  the strongest cut available because it makes the cache independent of sequence
  length ([57:14]). It costs expressivity — "there's no free lunch here, or at
  least this was an expensive lunch" ([58:00]) — so it is deployed interleaved with
  full-attention layers.
- **[DeepSeek v4's stack](sparse-attention.md)** — Compressed Sparse Attention
  (compress every $m$ tokens into one), DeepSeek Sparse Attention (select the top
  $k$ using cheap auxiliary queries and keys, "a lightning fast way to figure out
  what tokens you need to keep"), and Heavily Compressed Attention — supporting 1M
  context ([1:02:36]).

### A caution about the accuracy evidence

Worth carrying beyond this lecture. The GQA paper's own evals say accuracy holds
up; the DeepSeek paper's Table 8 says GQA is meaningfully worse than MHA. Percy
draws the moral explicitly: "with these accuracy evals, I think you always have to
take it with a grain of salt… take everything that's not just math with a grain of
salt here" ([51:06]). The arithmetic in this lecture is checkable; the accuracy
claims are somebody's benchmark table.

## Quantization and pruning

[Quantization](quantization.md) ([1:04:11]–[1:07:17]) reduces the precision of the
numbers: bf16 down through fp8, int8 and int4. You can do it *during* training
(quantization-aware training, which simulates the error in the forward pass so the
weights adapt, but "requires expensive large-scale training") or *after*
(post-training quantization, much cheaper — with GPTQ using Hessian information to
push each layer's quantization error into the weights not yet quantized). **AWQ**
is the idea worth keeping: some activation channels are large, the weights they hit
matter more, so keep 0.1–1% of weights in high precision and quantize the rest to
int3 — 4× less memory and a 3.2× speedup.

[Pruning with distillation](pruning-and-distillation.md) ([1:07:17]–[1:09:36]) is
cruder: "you just take a large model, rip out pieces of it, and fix it up." NVIDIA's recipe scores importance on a 1,024-sample calibration set, removes
the unimportant layers, heads and hidden dimensions, then distills the original
into the survivor — taking a 15B model down to 8B without hurting accuracy much,
for far less compute than training 8B from scratch ([1:08:50]).

Both belong to what the source calls the **distillation recipe**: define a faster
architecture, initialize it from the original model's weights even though the
architecture differs — "you kind of just make this Frankenstein thing" — then
repair it with distillation ([1:09:36]).

## Speculative sampling

The one method that costs nothing. It rests on the observation that prefill "also
gives you probabilities": scoring $k$ proposed tokens is one parallel pass, while
producing them is $k$ sequential ones, so **checking is faster than generation**
([1:11:52]).

A cheap draft model $p$ proposes a few tokens (three or four is the sweet spot —
too few wastes the target model's batching, too many get rejected, [1:15:39]); the
target model $q$ scores them all in parallel and accepts each with probability
$\min(1, q/p)$, falling back to a residual distribution on rejection. The result is
**an exact sample from the target model** ([1:14:54]) — the full argument is on
[speculative sampling](speculative-sampling.md).

The unification at the end is the nicest moment in the lecture: run all the
KV-cache shenanigans on your model; "if you end up with a model you're happy with,
just serve that. If you're not happy with it, then at least it can be a draft
model, and you can use your main model to fix things up" ([1:16:25]).

## Dynamic workloads

Nothing here changes the model. The problem is that live traffic is nothing like a
training batch: requests arrive at different times, share prefixes, and have
different lengths ([1:17:11]).

[Continuous batching](continuous-batching.md), from the Orca paper, decodes one
step at a time and edits the batch between steps — finished sequences are evicted,
arriving requests join — so a new request never waits for the current generations
to finish ([1:17:57]). Its companion, **selective batching**, splits the model in
exactly the place the intensity derivation predicted: attention is computed per
sequence, because each has its own cache and its own length, while the MLP layers
concatenate every sequence into one mega-sequence, because they share weights
([1:18:43]).

[PagedAttention](paged-attention.md), from the vLLM paper, applies virtual memory
to the KV cache ([1:19:29]–[1:23:18]). Allocating a contiguous buffer per request
up to a maximum length wastes what the request does not use (internal
fragmentation) and strands the gaps between requests (external fragmentation) —
"this is what happens, or used to happen, to your hard drive." So the cache is
split into fixed-size blocks with an index, which makes fragmentation disappear and
sharing easy: one copy of a shared system prompt for every request that uses it,
and one copy of a prompt whose completions are sampled many times, with
copy-on-write at the block level when two samples diverge.

## Recap

The lecture's own summary ([1:23:18]–[1:24:51]): inference is important, it is very
different from training even though it is the same model, it is memory-bound and
dynamic, and the techniques — quantization, new architectures, pruning and
distillation, speculative sampling — are all "driven by the same principle: reduce your KV cache, but don't hurt accuracy too much." Two ideas came straight
from operating systems: paging, and speculative execution.

The closing thought is a judgement about where the gains are: "at some level, the KV cache, and the way that attention is built, fundamentally
makes it an inference-unfriendly kind of architecture. So, if you can come up with a
new architecture that's designed for inference in the way that the Transformer was
not, this could maybe unlock a lot" ([1:24:51]) — which points at
[linear attention](linear-attention.md),
[state space models](state-space-models.md) and diffusion.

## Topics from this lecture

- [Inference](inference.md) — the workload, the metrics, and the serving landscape.
- [KV cache](kv-cache.md) — the object everything in this lecture is trying to shrink.
- [Prefill and generation](prefill-and-generation.md) — the two phases and their opposite bottlenecks.
- [Latency and throughput](latency-and-throughput.md) — the tradeoff, and the Llama 2 13B performance model.
- [Quantization](quantization.md) — formats, QAT, PTQ, GPTQ and AWQ.
- [Pruning and distillation](pruning-and-distillation.md) — rip it out, then heal it.
- [Speculative sampling](speculative-sampling.md) — free speed, with a proof.
- [Continuous batching](continuous-batching.md) — iteration-level scheduling and selective batching.
- [PagedAttention](paged-attention.md) — virtual memory for the KV cache.
- [Cross-layer attention](cross-layer-attention.md) — sharing KVs down the layer stack.

## See also

- [Lecture 4 — Attention alternatives](04-attention-alternatives.md) — where GQA, MLA and sparse attention are introduced as *architecture*; this lecture re-derives why they exist.
- [Lecture 2 — PyTorch and resource accounting](02-pytorch-resource-accounting.md) — the arithmetic-intensity machinery reused here.
- [Lecture 5 — GPUs and TPUs](05-gpus-tpus.md) — where the 295 comes from.
- [Lecture 9 — Scaling laws](09-scaling-laws.md) — the lecture before, which this one interrupts.
- [Attention variants](attention-variants.md) — MQA, GQA and sliding windows in one place.
