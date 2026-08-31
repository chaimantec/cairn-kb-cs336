# Activation checkpointing and gradient accumulation

Two techniques from the end of [Lecture
2](02-pytorch-resource-accounting.md), both answers to the same problem: of the
four things consuming GPU memory during training, [activations are the one that
scales with your workload](memory-accounting-for-training.md#activation-memory-is-the-term-that-varies)
rather than your model, and they are what runs you out of memory.

$$\text{activation memory} = 2 \cdot B \cdot D \cdot L$$

Batch size $B$, width $D$, depth $L$, in bf16. The two techniques attack two of
those factors. Gradient accumulation reduces the effective $B$; activation
checkpointing reduces the effective $L$. Both **trade compute for memory**, and
both are standard practice rather than emergency measures.

## Why activations must be stored at all

For training, we need to store the activations of **all** layers. For inference we
don't compute gradients, so we only need the current layer's activations — each
one can be discarded as soon as the next is computed.

The difference is the backward pass. Computing $\partial \text{loss} / \partial w$
for a layer requires that layer's *input*, which is the previous layer's output.
So the forward pass cannot throw anything away; it must leave a trail all the way
back to the input for the backward pass to consume. That trail is the $L$ in the
formula, and it is why training memory grows with depth while inference memory
does not.

## Gradient accumulation

The motivation is not memory for its own sake. **Large batch sizes improve
training stability** — you want them. But activation memory scales linearly with
batch size, so the batch size you want may not fit.

```python
B = 64     # Batch size
D = 1024   # Dimensionality
L = 16     # Number of layers
activation_memory = 2 * B * D * L   # 2,097,152 bytes
```

Gradient accumulation decouples the batch size the *optimizer* sees from the batch
size the *GPU* holds:

- Compute the gradient on **micro batches**
- **Accumulate** the gradients — don't zero them out
- Every `batch_size / micro_batch_size` steps, update the parameters and zero out
  the gradients

```python
micro_batch_size = B / 4                            # 16.0
activation_memory = 2 * micro_batch_size * D * L    # 524,288 bytes
```

A quarter of the batch, a quarter of the activation memory. The gradients
themselves are unaffected, because gradient memory scales with the number of
parameters, not the batch — you are summing into the same buffer either way.

Why this is exactly correct rather than an approximation: the gradient of a sum is
the sum of the gradients, so accumulating four micro-batch gradients gives
precisely the gradient of the full batch. The parameter update is **identical** to
what the full batch would have produced. What you pay is time — four sequential
forward/backward passes instead of one wider one, with less parallelism to
exploit within each.

The one thing to get right in implementation is the `zero_grad` call. PyTorch
accumulates into `.grad` by default, which is normally something you cancel out
with `optimizer.zero_grad()` every step; gradient accumulation is just *not*
cancelling it, then zeroing once per real batch. The lecture's ordinary training
loop shows the default rhythm being broken:

```python
optimizer.step()
optimizer.zero_grad(set_to_none=True)
```

## Activation checkpointing

Also known as **gradient checkpointing** or **rematerialization** — three names for
one technique, and all three appear in the literature.

The key idea:

- **Forward pass**: keep only activations at a subset of layers
- **Backward pass**: recompute the missing activations from the last checkpoint

The philosophy, in the lecture's words: **trade off memory for compute.**

```
# Store all activations:    x g1 h1 g2 h2 g3 h3 g4 h4
# Activation checkpointing: x    h1    h2    h3    h4
```

In PyTorch it is a one-line change to the forward pass:

```python
class DeepNetworkCheckpointed(nn.Module):
    """Same as DeepNetwork, but with activation checkpointing."""
    def __init__(self, dim: int, num_layers: int):
        super().__init__()
        self.layers = nn.ModuleList([Block(dim) for i in range(num_layers)])

    def forward(self, x: torch.Tensor) -> torch.Tensor:
        for layer in self.layers:
            # KEY: only store activations at checkpoints, recompute the rest
            x = torch.utils.checkpoint.checkpoint(layer, x)
        return x
```

`torch.utils.checkpoint.checkpoint(layer, x)` runs `layer` without building the
autograd graph for its internals, keeping only its input. When the backward pass
reaches it, the layer's forward is re-run to rebuild what is needed. The lecture
measures peak memory for both variants with `get_max_memory_usage`; those readings
are machine-dependent and this knowledge base does not quote values for them.

## How frequently to checkpoint

The part worth remembering, because it is a genuine optimum rather than a knob to
tune by feel. Three strategies for a network of $L$ layers:

```
# Store all layers:   | h1 h2 h3 h4 h5 h6 h7 h8 h9 |
# Store no layers:    |                            |
# Store some layers:  |    h3       h6          h9 |
```

| Strategy | Activation memory | Recomputation |
| --- | --- | --- |
| Store every layer's activations | $O(L)$ | none |
| Store no activations | $O(1)$ | $O(L^2)$ |
| **Store every $\sqrt{L}$ layers** | $O(\sqrt{L})$ | $O(L)$ |

The middle row is the trap. Storing nothing sounds like the memory-optimal choice,
but the compute cost is quadratic: to reconstruct layer $k$'s input you replay the
network from the start, and doing that for every layer is $1 + 2 + \dots + L$ work.

The $\sqrt{L}$ strategy is the one to use. It holds $\sqrt{L}$ checkpoints, and
recomputing within a segment costs $\sqrt{L}$ work for each of $\sqrt{L}$ segments
— so $O(L)$ total, a constant-factor addition to the forward pass rather than an
asymptotic one. You get a $\sqrt{L}$-fold memory reduction for roughly one extra
forward pass, which is why checkpointing is close to free in practice: the forward
pass is $2ND$ of a $6ND$ training step, so re-running it once costs about
**33% more compute** for a large memory saving. (That last arithmetic is a
consequence of the [6ND rule](training-flops.md), not a figure the lecture states.)

## Where these go next

Both techniques reappear in Unit 2, where the same "move less, recompute more"
logic drives kernel fusion and the parallelism strategies — and where optimizer
state, not activations, becomes the thing to shard. See [course map, Unit
2](course-map.md#unit-2--systems), and
[memory accounting](memory-accounting-for-training.md) for the other three
consumers these techniques do *not* address.

## The same trade, counted in memory accesses

[Lecture 5](05-gpus-tpus.md) presents recomputation as trick 3 of six, and its
accounting is worth seeing because it counts *memory accesses* rather than bytes
held — the currency that matters when the bottleneck is bandwidth rather than
capacity.

Slides 35 and 36 take three stacked sigmoids. Storing the activations
([50:46]–[51:31]):

- forward: 1 read of $x$, 3 writes ($s_2$, $s_1$, out)
- backward: 3 reads, 1 write
- **8 memory accesses**, and, as the slide says, "very low arithmetic intensity"

Throwing them away and recomputing on the backward pass ([51:31]–[52:17]):

- forward: 1 read, 1 write
- backward: 2 reads ($d_{\text{out}}$ and $x$), 1 write
- **5 memory accesses** — "5/8 of the total memory accesses"

Same computation, slightly more compute, less traffic. Hashimoto's framing of when
this is worth it is the general rule: "imagine you're in a world where computation
is super cheap — you have abundant computation, you're not using all of your units,
but your memory is very expensive." That world is the one
[the compute–memory gap](gpu-architecture.md) is producing.

The technique reappears at tile granularity inside
[FlashAttention](flash-attention.md), which discards the $N \times N$ attention
matrix and recomputes it tile by tile in the backward pass rather than storing it.

## Sources

- [Lecture 5 — GPUs and TPUs](05-gpus-tpus.md) — recomputation as trick 3, and its
  use inside FlashAttention.
- [Sharding, replication and recomputation](sharding-vs-replication.md) — Lecture 7
  generalizes this page's trade into three options rather than two: recompute, store
  in memory, or *store on another GPU and communicate* ([1:19:40]). Activation
  checkpointing is the first; multi-GPU training adds the third.
- [Lecture 2 — PyTorch, Resource Accounting](02-pytorch-resource-accounting.md)
- [`lecture_02.py` transcription](../raw/slides/02-pytorch-resource-accounting.md)
  — `gradient_accumulation()`, lines 718–730; `activation_checkpointing()` and
  `DeepNetworkCheckpointed`, lines 733–788
- [Edited transcript](../raw/transcripts/02-pytorch-resource-accounting.md)
