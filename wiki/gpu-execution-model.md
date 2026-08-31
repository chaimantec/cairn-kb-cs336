# The GPU execution model — threads, blocks, warps, and SIMT

This page covers the *programming* model of a GPU: what runs, in what groups, and
what that forces you to do differently from CPU code. For the hardware it runs on —
SMs, caches, global memory — see [GPU architecture](gpu-architecture.md).

Hashimoto sets it up in Lecture 5 at [13:05]: "There are three important players in
the execution model of a GPU."

## The three units

**Thread.** The lightweight unit of work, familiar from any parallel programming.
On a GPU, threads are extremely cheap to stop and start: "the scheduler can decide,
'okay, I have another warp that's better to run, so I'm going to stop you and start
this one'" ([23:05]). That cheapness is what lets the hardware hide stalls and keep
throughput high.

**Block.** A group of threads, and the reason blocks exist is a hardware guarantee:
**a block is guaranteed to run on a single SM** ([13:50]). Since an SM has its own
shared memory, a block is exactly the set of threads that can cooperate through
fast memory. This is the fact [tiling](tiling.md) is built on — Hashimoto flags it
forward at [14:36].

**Warp.** The scheduling unit: threads execute in groups of **32 consecutively
numbered threads** ([14:36]). Grouping them this way "decreases the overhead of a
scheduler deciding which threads to run." When the lecture says "different warps
executing," it means these 32-thread bundles.

A student asks the sharpening question at [15:21] — do all threads execute the same
instruction within a *block* or within a *warp*? The answer is **the warp**. And
the scheduler's decision is at warp granularity too: it "decides which warp is
going to be executing next."

## SIMT: every thread runs the same instruction

The model is SIMT — single instruction, multiple threads. All threads in a warp
execute the exact same instruction; only their *inputs* differ ([13:50]). They do
different useful work, but they do it in lockstep.

Hashimoto frames this as a deliberate trade: "if the threads could do all sorts of
different stuff, it would become very hard to program. GPUs are this trade-off
between programmability and efficiency." He returns to the upside at [23:05] —
writing even low-level PTX is tractable "because it's all SIMD — it's not like
you're programming every single thread, you're saying, 'here are my instructions,
here are my inputs, now run them on all of my inputs.'"

## Control divergence — the cost of an `if`

The first of the lecture's six performance tricks falls straight out of SIMT, and
it is the one trick that is not about memory ([32:19]).

Ask what a GPU does with a branch. On a CPU it is obvious: evaluate the condition,
take one path. On a GPU, every thread must execute the same instruction, so
**every thread executes both branches**, and threads on the wrong side of the
condition mask out their computation and sit idle ([33:05]).

Slide 23's picture, as Hashimoto describes it at [33:50]: the threads diverge, the
bottom-branch threads execute while the top-branch threads wait, then the
top-branch threads execute while the bottom-branch threads wait. "So to execute an
`if` statement, you're actually clock-walking through both branches, and some
threads are just waiting, doing nothing."

This is **control divergence**. Its cost is idle compute — "big compute gaps where
your GPU is doing nothing."

**The practical consequence** is a coding idiom that looks strange until you know
why. In GPU code for something like a ReLU, "you'll often find you're multiplying
by zero, or multiplying with masks, instead of an `if` statement that updates half,
because a multiplication is going to all happen at once, whereas an `if` statement
might have to wait two clock cycles" ([34:37]).

Note the classification the deck insists on: slide 23's own heading is "Control
divergence (**not a memory issue**)". Every other trick in the lecture is about
moving less data; this one is about not stalling lanes.

## The memory model, from the thread's point of view

Slide 12 lists what device code can reach, and Hashimoto walks it at [15:21]–[16:53]:

| Level | Scope | Notes from the lecture |
| --- | --- | --- |
| Registers | Per thread | Fastest and most local — array bounds, memory addresses |
| Local memory | Within a single SM | |
| Shared memory | Within a block | Where values reused across threads live |
| Global memory | Whole device | The DRAM far away; you pay the latency |
| Constant memory | Whole device | "I don't think I've seen used very much in practice" |
| Host memory | CPU side | For offloading beyond the GPU's capacity |

His summary of the whole model is the sentence the rest of the lecture elaborates:
"as soon as you go outside your shared memory, things are going to be slow. So
grouping blocks to decrease the amount of global memory reads is going to be the
name of the game throughout this lecture" ([16:53]).

## Related

- [GPU architecture](gpu-architecture.md) — the SMs and memory these units map onto.
- [Tiling](tiling.md) — the technique built on the block/shared-memory guarantee.
- [Memory coalescing](memory-coalescing.md) — why a warp's 32 addresses should land
  in one burst.
- [Lecture 5 — GPUs and TPUs](05-gpus-tpus.md).
- [Transcript](../raw/transcripts/05-gpus-tpus.md), [slide deck](../raw/slides/05-gpus-tpus.md).
