# PTX

PTX — parallel thread execution — is the assembly-level language a Triton kernel
compiles into, and Lecture 6 spends two minutes reading some ([50:55]–[54:48]).
The point is not to teach the language. It is to show that the Python you wrote is
"in some sense, a lie" ([47:51]) and to let you see what the machine was actually
handed.

## What changes at this level

**The thread block is gone.** "This is now what a thread is actually doing, not a
thread block, because that's been compiled away" ([50:55]). Everything
[Triton](triton.md) let you say about blocks has been lowered into per-thread
instructions.

**Memory traffic becomes explicit.** `ld.global` loads from HBM into registers and
a matching global store at the bottom writes back ([51:40]). Registers come in two
flavours the listing names directly: "the R's are integer registers, FR — floating
point registers."

**The identity of a thread arrives as an argument.** Asked whether the compiler
generates one of these per thread, Percy's answer is the important one for
understanding SIMT: "This is compiled once, and it's the same piece of code that
each thread runs. And the way that the thread distinguishes itself is that this
piece of code gets passed, basically, the thread ID. So here, `%ctaid.x` is the
block index, and TID.x is the thread index" ([53:13]).

(The lecture source writes that second register `%tid.x`; the captions drop the
sigil and the transcript keeps what was said.)

**Some things are still not there.** "There's still a lot of things that are not
specified in the PTX, like, for example, which SMs things are operating on, and the
warps and everything — a lot of that is hardware controlled, so you don't even see
it" ([54:48]).

## What the listing reveals that the source did not

The GeLU kernel was written one element per lane. The PTX shows the same
computation repeated in blocks, and that is [thread
coarsening](warp-occupancy.md) applied without being asked:

> "You see all these blocks, and what's going on there is... thread coarsening —
> which is that this is one thread, but rather than processing a single element,
> it's actually processing eight elements. So the compiler decided that, well,
> actually this thread is pretty lightweight, it doesn't do that much, so let's
> just try to thicken it up a little bit." ([52:28])

The lecture's own source file records the same observation as a bullet: *one
thread processes 8 elements at the same time (thread coarsening)*. This is a good
illustration of why reading the generated code is worth the two minutes — nothing
in the Triton source says "eight."

## Should you write it?

Rarely, and not first. "There are people who do write PTX, if you really think
you're better than the compiler. And I think the NVIDIA compilers are generally
pretty mature, but some other accelerators that are less developed, I think,
sometimes you just have to reach in and actually hand-hold a bit more. But
generally, you shouldn't need to do that" ([54:48]). Asked later about
alternatives to Triton: "in the extreme, you can always go to PTX and write that,
but I wouldn't advise that as a first step" ([1:24:54]).

PTX is nonetheless one of the three rungs Percy names in his closing summary of
what a programmer controls — "PyTorch, or Triton, or PTX" ([1:21:51]) — and
[Lecture 5](05-gpus-tpus.md) made the same point from the other side, that writing
at this level is tractable precisely because it is SIMT.

## Reading the file yourself

The lecture writes the generated PTX to `var/triton_gelu-ptx.txt` during the run
(`output_ptx`, lecture source lines 735–741). That artifact belongs to whichever
machine executed the program and is not reproduced in this KB; the observations
above are the lecture's own.

## Related

- [Triton](triton.md) — the layer above.
- [The GPU execution model](gpu-execution-model.md) — SIMT, and why one compiled
  body serves every thread.
- [Warp occupancy](warp-occupancy.md) — thread coarsening, and why fatter threads
  can be the right trade.
- [Lecture 6 — Kernels and Triton](06-kernels-triton.md).
- [Transcript](../raw/transcripts/06-kernels-triton.md),
  [lecture source](../raw/slides/06-kernels-triton.md).
