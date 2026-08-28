# Memory accounting for training

Memory decides what you can train at all. Compute decides how long it takes; if
you run out of memory the job does not run slowly, it does not run. This page is
the accounting: what consumes GPU memory during training, how many bytes each
consumer costs, and how to turn that into "how big a model fits."

The base fact, from [precision and data types](precision-and-data-types.md), is
that a tensor's memory is **number of values × bytes per value**. Everything below
is bookkeeping on top of it.

## The four consumers

Everything on the GPU during a training step is one of five kinds of tensor —
data, parameters, gradients, optimizer state, activations — and four of them scale
with the model:

| Consumer | Size | Precision | Bytes per parameter |
| --- | --- | --- | --- |
| **Parameters** | $N$ | bf16 | 2 |
| **Gradients** | $N$ | bf16 | 2 |
| **Optimizer state** | $N$ per moment | fp32 | 4 (AdaGrad) / 8 (Adam) |
| **Activations** | $B \times D \times L$ | bf16 | *not per parameter* — see below |

The precisions are the [mixed precision
recipe](precision-and-data-types.md#mixed-precision-training--the-actual-recipe):
bf16 for parameters, gradients and activations; fp32 for optimizer state.

The lecture's worked example, with $D = 4$, $L = 3$ layers and batch size $B = 2$,
so $N = D \cdot D \cdot L = 48$ parameters:

```python
parameter_memory       = 2 * num_parameters   # (2 bytes for bf16) = 96
gradient_memory        = 2 * num_parameters   # (2 bytes for bf16) = 96
optimizer_state_memory = 4 * num_parameters   # (4 bytes for fp32) = 192
activation_memory      = 2 * (B * D * L)      # (2 bytes for bf16) = 48
total_memory = 432
```

Note what dominates even in a toy: **optimizer state is the single largest line
item** at 192 of 432 bytes, larger than the parameters themselves. That is the
usual surprise in memory accounting, and it is why optimizer-state sharding is one
of the first things Assignment 2 asks you to implement.

## Why optimizer state costs what it does

The state is whatever running statistics the optimizer keeps per parameter, so its
cost follows directly from the algorithm. The lecture gives the family as a
sequence of additions, which is the clearest way to remember both the algorithms
and their price:

- **momentum** = SGD + exponential averaging of $g$
- **AdaGrad** = SGD + averaging by $g^2$
- **RMSProp** = AdaGrad but with *exponential* averaging of $g^2$
- **Adam** = RMSProp + momentum

Read the last line and the memory follows: Adam keeps **both** a first moment
(momentum, an average of $g$) and a second moment (an average of $g^2$), so it
stores two fp32 values per parameter:

- **AdaGrad**: 4 bytes/parameter — second moments only
- **Adam**: 8 bytes/parameter — first *and* second moments

And the fp32 is not incidental. It is **customary to use fp32 for stability**,
because optimizer state accumulates averages over powers of the gradient across
many steps — thousands of additions into the same accumulator, where bf16's coarse
resolution would compound. This is the one place the mixed-precision recipe spends
full precision, and it is why the byte counts are 4 and 8 rather than 2 and 4.

AdaGrad in full, which is short enough to read as the definition:

```python
class AdaGrad(torch.optim.Optimizer):
    def __init__(self, params: Iterable[nn.Parameter], lr: float = 0.01):
        super(AdaGrad, self).__init__(params, dict(lr=lr))

    def step(self):
        for group in self.param_groups:
            lr = group["lr"]
            for p in group["params"]:
                state = self.state[p]
                grad = p.grad.data

                # Get squared gradients g2 = sum_{i<t} g_i^2
                g2 = state.get("g2", torch.zeros_like(grad))

                # Update optimizer state
                g2 += torch.square(grad)
                state["g2"] = g2

                # Update parameters
                p.data -= lr * grad / torch.sqrt(g2 + 1e-5)
```

The update rule is

$$p \leftarrow p - \frac{\eta \, g}{\sqrt{\sum_{i<t} g_i^2 + \epsilon}}$$

with $\eta$ the learning rate and $\epsilon = 10^{-5}$ guarding the square root.
The `g2` tensor is exactly the "4 bytes per parameter" in the table — it is the
same shape as the parameter, and it is what makes AdaGrad cost twice what plain
SGD would.

## The headline calculation: how big a model fits?

This is the lecture's second motivating question, and it is a one-liner once the
table above is in hand.

> What's the largest model you can train on 8 H100s using AdamW?

```python
h100_bytes = 80e9
bytes_per_parameter = 2 + 2 + (4 + 4)   # parameters (2), gradients (2), optimizer state (4 + 4) = 12
num_parameters = (h100_bytes * 8) / bytes_per_parameter   # 5.33e10
```

**12 bytes per parameter**, 640 GB of HBM across 8 H100s, so about **53 billion
parameters**.

The decomposition $2 + 2 + (4 + 4)$ is worth memorizing as four separate numbers
rather than as "12", because each is a choice you could make differently: train in
fp32 and the first two double; use a memory-lighter optimizer and the parenthesis
shrinks; shard the optimizer state across devices and it divides.

**This is an upper bound**, and the lecture says so explicitly: **activations are
not accounted for**, because they depend on batch size and sequence length. A real
53B run on 8 H100s would not fit. Which brings us to the term that was left out.

## Activation memory is the term that varies

The other three consumers scale with $N$ alone. Activations scale with the
*workload*:

$$\text{activation memory} = 2 \cdot B \cdot D \cdot L$$

for a network of $L$ layers of width $D$ at batch size $B$, in bf16. Parameters,
gradients and optimizer state are fixed the moment you choose the model.
Activation memory is the one you control at run time — and the one that runs you
out of memory.

The reason it exists at all: training must keep every layer's activations from the
forward pass, because the backward pass needs them to compute gradients. Inference
does not compute gradients, so it only needs the current layer's activations. That
asymmetry is the entire subject of [activation
checkpointing](activation-checkpointing.md).

The lecture's example, at $B = 64$, $D = 1024$, $L = 16$:

```python
activation_memory = 2 * B * D * L   # 2,097,152 bytes
```

Two facts about that expression matter more than the number:

- It is **linear in batch size**, which is why large batches — desirable for
  training stability — run out of memory first, and why [gradient
  accumulation](activation-checkpointing.md#gradient-accumulation) exists.
- It is **linear in depth**, which is why the $O(\sqrt{L})$ checkpointing strategy
  is worth the recomputation.

The lecture also measures peak memory directly with
`get_max_memory_usage(lambda: model(x).sum().backward())`. That reading is
machine-dependent, so this knowledge base does not quote a value for it — the
formula above is the part that transfers.

## And the compute, for comparison

One training step on the same toy network:

```python
flops = 6 * B * num_parameters   # 576
```

which is the [6ND rule](training-flops.md) with $B$ data points. Memory and
compute are accounted the same way and at the same time — that pairing is the
"resource accounting" mindset the lecture is teaching. See [resource
accounting](resource-accounting.md).

## Transformers

The accounting for a Transformer is more complicated but follows the same idea,
and **Assignment 1 asks you to do it**. The lecture points at one blog post for
each half:

- Memory: [erees.dev/transformer-memory](https://erees.dev/transformer-memory/)
- FLOPs: [adamcasson.com/posts/transformer-flops](https://www.adamcasson.com/posts/transformer-flops)

## Sources

- [Lecture 2 — PyTorch, Resource Accounting](02-pytorch-resource-accounting.md)
- [`lecture_02.py` transcription](../raw/slides/02-pytorch-resource-accounting.md)
  — `optimizer()` and `AdaGrad`, lines 602–680; `motivating_questions()`,
  lines 71–86
- [Edited transcript](../raw/transcripts/02-pytorch-resource-accounting.md)
