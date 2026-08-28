# einops — named tensor dimensions

einops is a library for manipulating tensors in which **dimensions are named
rather than numbered**. CS336 adopts it in [Lecture
2](02-pytorch-resource-accounting.md) and uses it for the rest of the course, so
this page is worth reading before any later lecture's code.

It is inspired by Einstein summation notation (Einstein, 1916), and the
[einops tutorial](https://einops.rocks/1-einops-basics/) is the reference the
lecture points at.

## The problem it solves

Here is ordinary PyTorch computing a batch of attention-style similarity scores:

```python
x = torch.ones(2, 2, 3)      # batch seq hidden
y = torch.ones(2, 2, 3)      # batch seq hidden
z = x @ y.transpose(-2, -1)  # batch seq seq
```

The comments carry all the meaning, and the code carries none of it. Which axis is
`-2`? Which is `-1`? You have to hold the shape in your head, and if you get it
wrong the code will very often still run — broadcasting is accommodating — and
silently compute something else. The lecture's verdict: **easy to mess up the
dimensions**.

The comments are also unenforced. Nothing checks them, so they rot, and by the
third refactor they are lying.

einops fixes this by putting the names into the operation itself, where they are
both documentation and specification.

## einsum — generalized matrix multiplication with bookkeeping

The same computation, named:

```python
x = torch.ones(3, 4)  # seq1 hidden
y = torch.ones(4, 3)  # hidden seq2

# Old way
z = x @ y             # seq1 seq2

# New (einops) way
z = einsum(x, y, "seq1 hidden, hidden seq2 -> seq1 seq2")
```

Read the string as a contract: the inputs have these named axes, the output has
these. The gain is small in a two-dimensional example and large as soon as there
is a batch dimension:

```python
x = torch.ones(2, 3, 4)  # batch seq1 hidden
y = torch.ones(2, 3, 4)  # batch seq2 hidden

# Old way
z = x @ y.transpose(-2, -1)  # batch seq1 seq2

# New (einops) way
z = einsum(x, y, "batch seq1 hidden, batch seq2 hidden -> batch seq1 seq2")
```

The `transpose(-2, -1)` has disappeared entirely. It was never conceptually part
of the computation — it was bookkeeping needed to line up axes for `@`, and naming
the axes makes it unnecessary.

### The one rule you must remember

**Dimensions that are not named in the output are summed over.**

That single rule is what makes `einsum` "generalized matrix multiplication": in
the example above, `hidden` appears in both inputs and not in the output, so the
products are summed along it — which is exactly the dot product at the heart of a
matmul. Change which names appear on the right of the arrow and you get a
different reduction, with no change to the surrounding code.

This is also the rule to check first when an einsum returns the wrong thing:
an axis you forgot to name in the output has been silently summed away.

### Broadcasting with `...`

`...` stands in for any number of leading dimensions:

```python
z = einsum(x, y, "... seq1 hidden, ... seq2 hidden -> ... seq1 seq2")
```

The same expression now works whether the input has a batch dimension, a batch and
a head dimension, or neither. In numbered-axis code this generality is what forces
the `-2, -1` idiom in the first place.

## reduce — collapsing an axis

```python
x = torch.ones(2, 3, 4)  # batch seq hidden

# Old way
y = x.sum(dim=-1)

# New (einops) way
y = reduce(x, "... hidden -> ...", "sum")
```

The operation is named as a string — `sum`, `mean`, `max`, `min` — and the axis
being removed is named rather than indexed. Note how it reads: the `hidden` axis
is present on the left and absent on the right, so it is the one collapsed. It is
the same "unnamed in the output means reduced" convention as `einsum`, which is
why the two compose in your head.

## rearrange — splitting and merging axes

The case this exists for: **one dimension is really two dimensions flattened
together, and you want to operate on one of them.** This is exactly what
multi-head attention does — a hidden dimension that is secretly `heads ×
head_dim` — so it will recur throughout the course.

```python
x = torch.ones(3, 8)  # seq total_hidden
# ...where `total_hidden` is a flattened representation of `heads * hidden1`
w = torch.ones(4, 4)  # hidden1 hidden2

# Break up `total_hidden` into two dimensions (`heads` and `hidden1`)
x = rearrange(x, "... (heads hidden1) -> ... heads hidden1", heads=2)

# Perform the transformation by `w`
x = einsum(x, w, "... hidden1, hidden1 hidden2 -> ... hidden2")

# Combine `heads` and `hidden2` back together
x = rearrange(x, "... heads hidden2 -> ... (heads hidden2)")
```

The parentheses are the notation for "these names are packed into one axis". So
`(heads hidden1)` on the left and `heads hidden1` on the right means *unpack*, and
the reverse means *pack*. Because 8 could be split as 2×4 or 4×2, you supply
`heads=2` to disambiguate, and einops solves for the other.

Note what the middle line does **not** need: with the head axis named, the
transformation by `w` applies to `hidden1` and the `heads` axis simply rides along
under the `...`. There is no reshape, no transpose, and no permutation to reason
about.

## Why the course cares

einops is not a style preference here. It is part of the resource-accounting
mindset that [Lecture 2](02-pytorch-resource-accounting.md) is built around:
if you cannot state which axes a computation runs over, you cannot count its
[FLOPs](flops-and-mfu.md) or its bytes moved, and every
[arithmetic-intensity](arithmetic-intensity.md) estimate in the course begins by
counting exactly that. Naming the dimensions is what makes the accounting
mechanical instead of error-prone.

The lecture uses einsum for the backward-pass derivation as well, where the
gradient of a matmul is another matmul with the axes rearranged:

```python
h1_grad = einsum(h2.grad, w2, "batch out, in out -> batch in")
w2_grad = einsum(h2.grad, h1, "batch out, batch in -> in out")
```

Written this way the two gradients are visibly the *same* contraction over
different axis pairs, which is the fastest route to seeing why the backward pass
costs twice the forward pass. See [training FLOPs and the 6ND
rule](training-flops.md).

## Sources

- [Lecture 2 — PyTorch, Resource Accounting](02-pytorch-resource-accounting.md)
- [`lecture_02.py` transcription](../raw/slides/02-pytorch-resource-accounting.md)
  — `tensor_einops()` and its three sub-sections, lines 202–276
- [Edited transcript](../raw/transcripts/02-pytorch-resource-accounting.md)
- [einops tutorial](https://einops.rocks/1-einops-basics/)
