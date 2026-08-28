# Training FLOPs and the 6ND rule

One formula does most of the compute accounting in CS336:

$$C = 6ND$$

The FLOPs needed to train a model of $N$ parameters on $D$ tokens is **six times
their product**. It is previewed in [Lecture 1](01-overview-tokenization.md) at
[36:18] and derived in [Lecture 2](02-pytorch-resource-accounting.md). This page is
the derivation and its limits.

## The two halves

The six splits into a forward pass and a backward pass:

- **Forward pass**: $2ND$ FLOPs — 2 × (# data points) × (# parameters)
- **Backward pass**: $4ND$ FLOPs — 4 × (# data points) × (# parameters)
- **Total**: $6ND$ FLOPs

The forward half is just the [matmul count](flops-and-mfu.md#counting-the-flops-in-a-matmul)
applied to the whole network: every parameter is used once per data point, in a
multiply and an add, giving 2 FLOPs per parameter per token.

The interesting claim is the other one: **the backward pass costs twice the
forward pass.** That is not obvious, and it is not a fudge factor.

## Why backward is 2×

Take a two-layer linear network and focus on the second layer, $h_2 = h_1 w_2$,
with batch size $B$ and dimension $D$:

```python
h1 = einsum(x, w1, "batch in, in out -> batch out")   # x @ w1
h2 = einsum(h1, w2, "batch in, in out -> batch out")  # h1 @ w2
loss = (h2.mean() - 0)**2   # Regress everything to 0 (arbitrary)
```

The forward cost of that one layer is the standard matmul count:

```python
num_forward_flops = 2 * B * D * D   # 134,217,728
```

Now the backward pass. Given $\partial \text{loss} / \partial h_2$, it must produce
**two** things, not one:

$$\frac{\partial \text{loss}}{\partial h_1} \quad\text{and}\quad \frac{\partial \text{loss}}{\partial w_2}$$

- $\partial \text{loss} / \partial h_1$ is needed to keep propagating backward to
  the layers below.
- $\partial \text{loss} / \partial w_2$ is the thing you actually came for — the
  gradient the optimizer will use.

Written with [einsum](einops.md), each is a matmul over the same three axes, just
contracted differently:

```python
h1_grad = einsum(h2.grad, w2, "batch out, in out -> batch in")
assert torch.allclose(h1.grad, h1_grad)

w2_grad = einsum(h2.grad, h1, "batch out, batch in -> in out")
assert torch.allclose(w2.grad, w2_grad)

num_backward_flops = (2 * B * D * D) + (2 * B * D * D)   # 268,435,456
```

Each costs exactly what the forward matmul cost, because each is a contraction of
the same three dimensions $B$, $D_{\text{in}}$, $D_{\text{out}}$ — the same
$2BD^2$. Two of them, so $4BD^2$: **twice the forward pass**, and the ratio is
structural rather than empirical.

Note that the lecture *verifies* this rather than asserting it. Both `assert
torch.allclose(...)` lines check the hand-written einsum against what autograd
actually computed, so the two matmuls are demonstrably the whole backward pass,
not a simplification of it.

Summing over every layer gives $2ND$ forward, $4ND$ backward, $6ND$ total.

## What the rule assumes

The derivation is for **multilayer perceptrons** — stacks of matmuls. The lecture's
claim is that it is nonetheless "a good approximation for Transformers **for short
context lengths**."

The qualifier is doing real work. A Transformer contains matmuls whose cost is
$6ND$-shaped, but attention also contains the $QK^\top$ and attention-weighted-sum
products, whose cost scales with the **square of the sequence length** and does not
involve parameters at all. At short context those terms are small next to the
parameter matmuls, and $6ND$ is close. As context grows they stop being small, and
$6ND$ under-counts.

Two further things $6ND$ does not include: it counts only training compute, and it
counts a *dense* model — the parameters used per token. Assignment 1 asks you to do
the full Transformer accounting properly rather than rely on the shortcut.

## Using it

This formula plus a hardware rate is a complete estimate of a training run. The
lecture's opening question:

> How long would it take to train a 70B parameter model on 15T tokens on 1024
> H100s?

```python
total_flops = 6 * 70e9 * 15e12                                    # 6.3e24
h100_flop_per_sec = 1979e12 / 2                                   # 9.895e14
mfu = 0.5
flops_per_day = h100_flop_per_sec * mfu * 1024 * 60 * 60 * 24     # 4.377e22
days = total_flops / flops_per_day                                # 143.9
```

Four lines, no simulation, no profiler: **about 144 days**. That is the "napkin
math" mindset the lecture is arguing for — see [resource
accounting](resource-accounting.md) for the second worked question and the wider
framing.

$6ND$ is also the bridge to [scaling laws](scaling-laws.md). Once compute is
$C = 6ND$, a fixed compute budget is a constraint relating $N$ and $D$, and asking
how to spend it optimally — how big a model, on how many tokens — is exactly the
compute-optimal question Unit 3 answers.

## Sources

- [Lecture 2 — PyTorch, Resource Accounting](02-pytorch-resource-accounting.md)
- [`lecture_02.py` transcription](../raw/slides/02-pytorch-resource-accounting.md)
  — `gradients_flops()`, lines 502–556; `motivating_questions()`, lines 71–86
- [Lecture 1](01-overview-tokenization.md) at [36:18], where $C = 6ND$ is previewed
- [Edited transcript](../raw/transcripts/02-pytorch-resource-accounting.md)
- Blog post the lecture links for full Transformer FLOP accounting:
  [adamcasson.com/posts/transformer-flops](https://www.adamcasson.com/posts/transformer-flops)
