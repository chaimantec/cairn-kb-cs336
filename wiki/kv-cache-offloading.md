# KV cache offloading

Spilling the [KV cache](kv-cache.md) out of GPU memory and down the memory
hierarchy — to CPU DRAM, then to SSD — so that more sessions stay cached and more
requests hit. Covered in [Lecture 18](18-serving-megakernels-recurrence.md)
(≈25:30–≈29:21), where it is framed as the rediscovery of operating-system paging
inside an inference engine.

## Why the cache wants to grow without limit

Serving economics push in one direction only:

> A major piece of running large production systems is that you want to have as
> large a KV cache as possible — it's best if you can cache requests from many
> different users, or from the same user across many different sessions, and be
> able to run as many sessions as possible. (≈25:30)

Every cached prefix is prefill you never pay for again, and because
[traffic is session-structured](inference-workloads.md) the hit rate is high
enough for this to dominate. But GPU memory is also where the model's weights and
the active batch's activations live, so the cache is the first thing to run out —
and when it does the failure is admission control, not slowdown: a request "might
start queuing for that reason" (≈16:59).

## The hierarchy

The lecture walks it in order (≈25:30, ≈27:03):

1. **GPU memory** — fastest, smallest, contended with weights and activations.
2. **CPU DRAM** — "you quickly run out of GPU memory, so next you can start
   storing into CPU DRAM."
3. **Disk / SSD** — "next, you might put more KV cache onto the disk itself, and
   then you start caring about SSDs, and SSD space."
4. **"Some other global store"** (≈27:03).

Each level trades capacity for latency, and a student's interjection names the
constraint that bounds the whole scheme: **"Time to hit an SSD is very long"**
(≈27:48).

## The CPU stops being an afterthought

The best systems detail in the section. Once the cache lives in host memory, the
host's ability to read it back becomes a serving bottleneck:

> If you're paying attention to Jensen's keynotes, he's recently started getting
> very obsessed with CPU performance. One of the reasons is that a past generation
> of CPUs was actually really slow, and as a result started bottlenecking a bunch
> of very important workloads. (≈25:30, ≈26:16)

The consequence, in the lecture's phrasing:

> If your $500,000,000 machine is being bottlenecked by the thousand-dollar CPU
> that you purchase to put on top of it, that's not a great place to be. (≈26:16)

"One of the reasons that can happen is that you might be storing your KV cache on
CPU memory, and so you really care about the speed of being able to read that KV
cache back" (≈26:16). The lecture connects the same pressure to reports of
"OpenAI buying up all the SSDs, all the DRAM in the world… Part of the reason is
for stuff like this, where you want to store as much stuff in your KV cache as
you can" (≈27:03).

> **What this KB vouches for.** The market claims here are asides in a talk with
> [no slides or handout](18-serving-megakernels-recurrence.md). The technical
> point — that offloading makes host memory and storage bandwidth part of the
> serving critical path — stands on its own and is what this page is about.

## It is operating-systems paging

Asked whether particular workloads get offloaded, the lecture answers by naming
the analogy outright (≈27:48):

> I'm sure none of you guys have taken an operating systems class, but you should
> take your operating systems classes — we actually get to some pretty classic
> scheduling things. This diagram, except for the GPU things on the right, looks
> exactly like an operating systems diagram that you might have seen in the '70s
> or the '80s, because we used to have this problem where if you opened up too
> many applications on your computer, you would run out of CPU memory, and then
> you'd have to put those applications onto disk. It's exactly the same workload.

So the policy questions are the classic ones, with classic answers.

**Eviction.** LRU, and the lecture is comfortable with it: "evictions for least
recently used… is actually a pretty decent heuristic. There's probably some OS
paper somewhere that says LRU is within 2x of what's optimal" (≈28:36).

**Prefetching.** The ideal is clairvoyance — "if you could predict the future,
you would know, 'Oh, I'm about to have a request come in of a particular kind, let
me go prefetch that memory in'" (≈28:36) — and the nice observation is that in
this setting you partly can, because the **application gives you the signal**:

> When you go into your chat app and bring up some old conversation from a month
> ago, that's a very strong signal that you're going to start asking a question
> about it, and you might then want to go load that up onto a GPU. (≈28:36)

This is the advantage an inference engine has over a 1970s pager: it can see a UI
event before the request arrives.

**The objective.** Not hit rate for its own sake — throughput under a latency
constraint: "really, it's a question of how much traffic you want to put onto
your GPU footprint — I've never talked to anybody who wants to put less traffic
onto their GPUs. So, subject to those SLAs… you want to serve as much traffic as
you can" (≈29:21). See [latency and throughput](latency-and-throughput.md).

## An aside worth keeping

The slide illustrating eviction was AI-generated, and the generator invented its
content — the lecturer corrects it live: "on the left side, Nano Banana
hallucinated 'evictions for least recently used'" (≈28:36). The remark is the
reason this KB records that
[the lecture's slides were AI-generated](18-serving-megakernels-recurrence.md) and
would not have been a citable source even if they had been published.

## See also

- [Lecture 18](18-serving-megakernels-recurrence.md) — the source lecture
- [Cache-aware routing](cache-aware-routing.md) — raising hit rate by routing
  instead of by capacity
- [KV cache](kv-cache.md) · [Paged attention](paged-attention.md) ·
  [The inference request lifecycle](inference-request-lifecycle.md)
