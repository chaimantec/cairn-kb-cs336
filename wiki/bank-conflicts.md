# Bank conflicts

A bank conflict is what happens when several threads in a warp reach into shared
memory at the same time and land on the same bank. The accesses serialize, and the
parallelism you thought you had disappears. Lecture 6 introduces it as the third of
five hardware details that the programming model hides ([13:57]).

## The hardware

Shared memory — the same physical memory as L1, sitting on the SM — is divided
into **32 banks, each 4 bytes wide** ([13:57]). The rule is one access per bank per
cycle:

> "Every clock cycle, each bank can only be accessed by one thread — at most one
> thread — assuming that's not the same location... Which means that if you have
> multiple threads trying to access the same bank, the accesses have to be
> serialized — this is called a bank conflict." ([14:42])

The exception matters: threads reading the *same exact location* are fine, since
that can be broadcast. It is different locations within one bank that queue up.

## The worst case is a column

Lay a matrix out so that each row spans all 32 banks. Now have 32 threads read the
first *column*:

> "You think, wow, this 32 is so great, I can massively parallelize this — and you
> try to all access this first column. They're just going to wait in line. This is
> a 32-way bank conflict, which is the worst possible setting you can be in."
> ([15:29])

Thirty-two threads, thirty-two cycles, one useful access each.

## Why you cannot simply avoid it

The obvious answer — read along rows instead — does not survive contact with
matrix multiplication:

> "This is unavoidable, because, for example, if you're doing a matmul, you have to
> access rows of one matrix and columns of the other matrix, and sometimes you do
> transposes. So you can't always just get away with choosing the order in which
> you go down. For elementwise operations it's fine — you can go in any order. But
> if you're doing matmuls, you can't control, for every matrix, whether it's
> row-major or column-major." ([15:29]–[16:15])

$A B$ needs rows of $A$ and columns of $B$. One of them is going to be read across
the grain.

## The fix: swizzling

The lecture names the solution without going into it: **swizzling**, which
"arranges your shared memory so that when you're going through, you can avoid bank
conflicts" ([16:15]). The lecture source gives the canonical form — permute the
layout by something like *row xor col*, so that a column of the logical matrix
lands on distinct banks.

## Not the same thing as coalescing

This is the confusion the lecture pre-empts, and the distinction is the level of
the memory hierarchy:

| | Bank conflicts | [Memory coalescing](memory-coalescing.md) |
| --- | --- | --- |
| Where | **shared memory** (on the SM) | **HBM** (global memory) |
| Unit | 32 banks × 4 bytes | 128-byte cache lines |
| Failure | accesses to one bank serialize | you fetch bytes you never use |

"This can feel similar to bank conflicts, but it's a very different constraint:
that one is about shared memory, and this one is about HBM" ([17:46]).

Both are visible in a profiler — "when you're profiling, you can look at the bank
conflicts, you can look at occupancy, and you can see what's happening" ([16:15]).

## Related

- [Memory coalescing](memory-coalescing.md) — the HBM-side analogue.
- [GPU architecture](gpu-architecture.md) — where shared memory sits, and why it is
  the same silicon as L1.
- [Tiling](tiling.md) — the technique that puts matrix tiles in shared memory in the
  first place.
- [Warp occupancy](warp-occupancy.md) — the other shared-memory-adjacent limit.
- [Lecture 6 — Kernels and Triton](06-kernels-triton.md).
- [Transcript](../raw/transcripts/06-kernels-triton.md),
  [lecture source](../raw/slides/06-kernels-triton.md).
