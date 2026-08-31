# Fused softmax

Softmax is Lecture 6's second Triton kernel and its first *reduction* — the first
operation where threads have to cooperate rather than each mind their own element.
It is also the clearest arithmetic case for fusion in the course, because the
lecture counts the memory traffic on both sides.

## Why the naive version is slow

The naive implementation is five ordinary PyTorch operations on an $M \times N$
matrix: take each row's max, subtract it, exponentiate, sum, divide. Written in
PyTorch, each of those is a separate kernel, and "unless you call `torch.compile`,
these are going to be different operations, and each operation is going to read and
write, read and write, from HBM" ([59:26]).

The lecture source counts them line by line and totals:

$$5MN + M \ \text{reads}, \qquad 3MN + 2M \ \text{writes}$$

against what the computation actually requires — $MN$ reads and $MN$ writes. The
source's own comment calls that a **speedup of 4x** left on the table, and Percy's
version is "in principle you should really only have much fewer" ([59:26]).

Note that the max-subtraction is not one of the wasted steps: it is there "for
numerical stability" ([58:39]). Without it, a row containing 100 would exponentiate
to $e^{100}$ and overflow. The lecture's example input is exactly that case — a row
of `[0, 0, 100]`, which comes out as $[\approx 0, \approx 0, 1]$.

## The kernel: one row, one block

The design decision is a single line, and the reason is the reduction:

> "What we're going to do is say each row is a block. And why do I make each row a
> block? Well, because, remember, each row I have to normalize and sum. So it's not
> elementwise — softmax is not elementwise, but it is sort of row-wise. So the
> blocks don't interact — they don't need shared memory across blocks." ([1:00:11])

Independent rows mean no cross-block communication, which is the thing
[Triton](triton.md) gives you no mechanism for anyway.

The host side sizes the block to the whole row — `triton.next_power_of_2(N)`, which
Percy annotates "go to the next power of two, for good luck" ([1:00:57]) — and
launches one block per row. It also passes the **strides**, "which tell me how far
to move down" ([1:01:44]).

Inside, after the pointer arithmetic, the code is the naive version again:

```python
x_row = tl.load(x_ptrs, mask=col_offsets < num_cols, other=float("-inf"))
x_row = x_row - tl.max(x_row, axis=0)
numerator = tl.exp(x_row)
denominator = tl.sum(numerator, axis=0)
y_row = numerator / denominator
tl.store(y_ptrs, y_row, mask=col_offsets < num_cols)
```

"This part is essentially the same as the naive softmax — I'm just going to compute
the max, subtract it off, exponentiate, sum, and divide, and then write it back"
([1:03:18]). The difference is that all five now happen between **one** load and
**one** store.

## The `-inf` detail

The padding value on the masked load is not zero:

> "Here I do this thing where, if it's masked out, then I put minus infinity,
> because I know that's going to be the equivalent of a zero for the softmax
> operation." ([1:03:18])

$e^{-\infty} = 0$, so a padded lane contributes nothing to either the max or the
sum. Compare the [row-sum kernel](06-kernels-triton.md), where the reduction is a
plain sum and the padding value is `0.0` instead. The rule is to pad with the
identity of whichever reduction follows.

## The limit of this design, and what comes next

This kernel assumes the row fits in one block — it literally asserts
`num_cols <= BLOCK_SIZE`. A student asks the obvious question and gets deferred to
the next example: "what if my number of columns and number of rows are bigger than
the block size?" — "yeah, so we'll get to that" ([1:04:05]).

The answer is tiling within a block, which the lecture demonstrates on the simpler
row-sum before applying it to matmul: each thread walks a series of tiles
accumulating its own partial, and the accumulator vector is reduced once at the end
([1:05:39]). Percy calls it "baby tiling", and is careful that the word *tile* is
not the word *block* — "these are not blocks, these are tiles. The block corresponds
to this whole row, and has to process all the tiles" ([1:11:04]).

Asked whether a column-wise softmax would work, the answer is that nothing about
the kernel is committed to rows: "we're tracking these pointers, and the pointers
can be anything... all you would have to do is change the stride" ([1:04:52]).

## Where this goes

Fused softmax is the load-bearing piece of [FlashAttention](flash-attention.md),
which the assignment asks you to implement, and which needs one thing this kernel
does not do: softmax over a row *too long to see at once*, computed in a single
pass. That is the online-softmax reformulation from [Lecture
5](05-gpus-tpus.md) — this kernel plus the tiling idea from the next example.

The lecture follows the [Triton fused-softmax
tutorial](https://triton-lang.org/main/getting-started/tutorials/02-fused-softmax.html)
"roughly" ([57:54]).

## Related

- [Triton](triton.md) — the kernel skeleton and masking rules.
- [Operator fusion](operator-fusion.md) — the general form of the argument.
- [Tiling](tiling.md) — what to do when a row does not fit.
- [FlashAttention](flash-attention.md) — where this ends up.
- [Lecture 6 — Kernels and Triton](06-kernels-triton.md).
- [Transcript](../raw/transcripts/06-kernels-triton.md),
  [lecture source](../raw/slides/06-kernels-triton.md).
