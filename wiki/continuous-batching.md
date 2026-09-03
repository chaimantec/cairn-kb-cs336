# Continuous batching

A scheduling technique for serving, introduced by the **Orca** system (OSDI 2022).
It addresses a problem that has nothing to do with the model and everything to do
with traffic: real requests arrive at different times, share prefixes, and have
different lengths, so the neat rectangular batch that training enjoys does not
exist ([1:17:11]).

> Training: get a dense block of tokens (batch size × sequence length).
> Inference: requests arrive and finish at different times, so you have a ragged
> array.

## Iteration-level scheduling

Static batching forms a batch, runs it to completion, and then forms the next one.
That is bad twice over: a request arriving just after a batch starts waits for the
whole batch to finish generating, and a batch whose sequences finish at different
times keeps running with idle slots.

Continuous batching decodes **one step at a time and edits the batch between
steps** ([1:17:57]):

- Every step, decode exactly one token for every sequence currently in the batch.
- When a sequence emits its end token, evict it immediately.
- When a new request arrives, add it to the batch at the next step.

"That's why it's called continuous batching — your batch is dynamically being
updated, with old finished sequences being evicted and new ones coming in." No
request waits for generations it has nothing to do with, and no slot sits idle
behind a finished sequence.

This is also what makes the batch size $B$ in the
[performance model](latency-and-throughput.md) a scheduler's variable rather than a
fixed configuration. $B$ at inference is "the number of concurrent requests is essentially how many concurrent users there
are, which can be high or low, so it's a bit unpredictable, it can change over
time" ([28:43]) — and continuous batching is the mechanism that keeps it as
high as the memory budget allows.

## Selective batching

Then the objection: batching works because every element of the batch has the same
shape. If one request holds 3 tokens, another 9 and another 5, what is the tensor?

**Selective batching** answers it by splitting the model in two, and the split lands
in exactly the place the
[arithmetic intensity derivation](prefill-and-generation.md) predicted ([1:18:43]):

- **Attention is computed per sequence.** A length-3 sequence needs a $3 \times 3$
  attention computation and a length-9 sequence a $9 \times 9$ one; there is no
  useful shared tensor. This costs nothing, because each sequence has its own
  [KV cache](kv-cache.md) anyway — attention's intensity does not depend on $B$, so
  batching it was never buying anything.
- **Everything else is concatenated.** The MLP layers, which carry most of the
  FLOPs, flatten every sequence into one $[3 + 9 + 5, H]$ "mega-sequence" and
  process it as a single matrix multiply. This is where batching pays, because
  every sequence hits the same weights.

That is the same partition arrived at from two directions: the intensity algebra
says attention cannot be amortized across sequences and the MLP can, and the
systems design says attention cannot be batched across ragged lengths and the MLP
can. They agree because they are the same fact — what is shared across sequences
is the weights, not the cache.

## Related pages

- [PagedAttention](paged-attention.md) — the memory-layout half of the same problem; vLLM does both.
- [Prefill and generation](prefill-and-generation.md) — why attention and the MLP split this way.
- [Latency and throughput](latency-and-throughput.md) — what varying $B$ costs and buys.
- [KV cache](kv-cache.md) — the per-sequence state that makes the batch ragged.
- [Inference](inference.md) — the workload.
- [Lecture 10 — Inference](10-inference.md)
