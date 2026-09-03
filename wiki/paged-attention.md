# PagedAttention

Virtual memory, applied to the [KV cache](kv-cache.md). Introduced in the paper
that also introduced **vLLM**, and the reason vLLM became the default open-source
serving engine. The lecture's summary of the idea: "these are systems people, so they know their operating systems. They said, "Okay, well, we solved this problem once before, so let's just use the same idea here"" ([1:20:15]).

## The problem: fragmentation

The status quo it replaced allocates a **contiguous** slice of the cache per
request, sized for the maximum length the request might reach. Two kinds of waste
follow, and the lecture names both by their operating-system names ([1:19:29]):

**Internal fragmentation** is the reserved-but-unused tail. You do not know in
advance when generation will stop, so you allocate for the maximum — say 1,024
tokens, or 2,048 — and if the response ends after twenty, the rest is dead space
you cannot lend to anyone. The paper's figure, reproduced in
[`raw/slides/10-inference.md`](../raw/slides/10-inference.md), is brutal about the
scale: a request holding a 7-token prompt and generating a handful of tokens sits
in front of **2,038 slots never used**, and a second request with a 3-token prompt
in front of **507**.

**External fragmentation** is the unusable gap *between* two requests' reserved
blocks — space that is free but too small, or too awkwardly placed, to hold
anything.

"this is what happens, or used to happen, to your hard drive, and you have to
defrag your hard drive, back in the day" ([1:19:29]).

## The solution: blocks and a block table

Split each sequence's KV cache into fixed-size **blocks** — four tokens per block
in the paper's illustration — and let them live anywhere in physical memory
([1:21:00]). A per-sequence block table maps logical block $i$ to whatever physical
block currently holds it.

"Doesn't matter where the blocks go… as long as you have the indices and keep track
of where everything is, that's fine." The figures show a sequence whose logical
blocks 0, 1 and 2 sit in physical blocks 7, 1 and 3 — deliberately out of order, to
make the point that physical contiguity is no longer required.

What this buys is precisely what paging buys an operating system:

- **External fragmentation disappears entirely.** All blocks are the same size, so
  any free block fits any request.
- **Internal fragmentation shrinks to at most one block per sequence** — the
  partly-filled last one — instead of a whole reserved tail.
- **Allocation becomes incremental.** A sequence takes another block when it needs
  one, so nothing is reserved on speculation.

## Sharing, and copy-on-write

The larger win, and the one that matters most for real traffic, is that a block
table lets two sequences point at the *same* physical block. Two cases come up
constantly ([1:21:47]–[1:22:33]):

**Shared system prompts.** Every request to a deployment usually begins with the
same instructions. Their cache can be computed once and pointed at by every
request: "if a lot of people are using the same system prompt, you don't have
to compute the KV cache for every request." Since
[prefill is compute-bound](prefill-and-generation.md), this turns real compute into
a table lookup.

**Multiple samples from one prompt.** Program synthesis, best-of-$n$, and
reinforcement-learning rollouts all sample many completions from one prompt. The
prompt's blocks are shared and only the divergent continuations are private.

When two sharers diverge, **copy-on-write** handles it at block granularity, with a
reference count per physical block. The paper's figure shows two samples from
"Four score and seven years ago our", one continuing "fathers" and the other
"mothers": the prompt block is shared and untouched, and the block where they
differ is copied, leaving one physical block per sample and dropping the original's
reference count from 2 to 1 ([1:22:33]). If two samples happen to draw the *same*
token, no copy happens at all — "you just continue with that."

## The rest of vLLM

The lecture lists three further optimizations without dwelling on them ([1:23:18]):
a fused kernel that does the block gather and the attention in one launch, current
kernels ([FlashAttention](flash-attention.md) and FlashDecoding), and CUDA graphs —
all three aimed at kernel-launch overhead, which matters because generation issues
an enormous number of very small kernels.

## Why it belongs in this lecture

Every other technique in [lecture 10](10-inference.md) shrinks the cache. This one
does not shrink it at all — it makes the memory you already have usable, which
raises the batch size you can actually run, which is the term that
[throughput](latency-and-throughput.md) depends on. A serving stack that wastes
half its HBM on reserved-but-empty slots is running at half the batch size its
hardware permits.

"The general idea is that you're using these operating-systems metaphors to manage
your inference" ([1:23:18]) — the other borrowed idea being speculative execution,
which shows up as [speculative sampling](speculative-sampling.md).

## Related pages

- [KV cache](kv-cache.md) — what is being paged.
- [Continuous batching](continuous-batching.md) — the scheduling half; vLLM does both.
- [Prefill and generation](prefill-and-generation.md) — why prefix sharing is worth so much.
- [Latency and throughput](latency-and-throughput.md) — why a bigger feasible batch matters.
- [Inference](inference.md) — the serving landscape vLLM sits in.
- [Lecture 10 — Inference](10-inference.md)
