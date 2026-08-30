# Multi-head latent attention (MLA)

DeepSeek's method for shrinking the KV cache: instead of caching keys and values directly,
cache a single low-dimensional latent vector per token and reconstruct keys and values from
it on demand. Lecture 4 covers it in slides 57–58 as one of the "bonus" mechanisms in
DeepSeek V3, alongside [multi-token prediction](multi-token-prediction.md).

It belongs to the same family of concerns as
[attention variants](attention-variants.md) — MQA and GQA also exist to shrink the KV cache
— but attacks it differently: MQA and GQA share keys and values across heads, while MLA
compresses what is stored and expands it back per head.

## The construction

Rather than projecting the hidden state $\mathbf{h}_t$ straight to keys and values, project
it first to a latent $\mathbf{c}^{KV}_t$ of much lower dimension, then up from there
(slide 58):

$$\mathbf{c}^{KV}_t = W^{DKV}\mathbf{h}_t$$
$$\mathbf{k}^{C}_t = W^{UK}\mathbf{c}^{KV}_t$$
$$\mathbf{v}^{C}_t = W^{UV}\mathbf{c}^{KV}_t$$

$W^{DKV}$ is a down-projection; $W^{UK}$ and $W^{UV}$ are up-projections to the per-head
keys and values.

Hashimoto's description: "instead of producing Q, K, V directly, you represent them via a
lower-dimensional latent. So you have your hidden input; instead of directly producing your
Qs, Ks, and Vs, you first produce the C, and then produce your Qs, Ks and Vs as a function
of that" ([1:23:55]).

Queries get the same treatment, though for a different reason — training memory rather than
cache size:

$$\mathbf{c}^{Q}_t = W^{DQ}\mathbf{h}_t, \qquad \mathbf{q}^{C}_t = W^{UQ}\mathbf{c}^{Q}_t$$

Slide 58 notes the query compression is "for memory savings during training."

## Why it saves anything

**Only the latent is cached.** Slide 57's diagram marks the cached tensors with hatching,
and exactly two things are hatched: $\mathbf{c}^{KV}_t$ and a single rotary key
$\mathbf{k}^R_t$. The full per-head keys and values are never stored — they are recomputed
from the latent whenever needed.

> Instead of KV-caching all your Ks and Vs, you only need to store these Cs. And the Cs are,
> hopefully, lower-dimensional. That's why it's called MLA, like latent activation, because
> these Cs, which are latent, are the KV-cache quantities you need to save. ([1:23:55])

**And the up-projection is free at attention time.** This is the part that makes MLA cheap
rather than merely small. Slide 58 states it as "$W^{UK}$ can be merged into the Q
projection," which the attention inner product makes concrete:

$$\langle Q, K \rangle = \left\langle hW^{Q},\; W^{UK}\mathbf{c}^{KV}_t \right\rangle = \left\langle h\,W^{Q}W^{UK},\; \mathbf{c}^{KV}_t \right\rangle$$

Associativity again — the same move that powers [linear attention](linear-attention.md).
$W^{Q}W^{UK}$ is a product of two fixed weight matrices, so it can be precomputed once into
a single matrix. Attention then runs directly against the cached latent, and the keys are
never materialized at all.

## The RoPE conflict

The complication, and the reason MLA is not simply a strict improvement.

[RoPE](rope.md) applies a position-dependent rotation to queries and keys. Insert those
rotations into the inner product and the merging above breaks:

$$\langle QR_q,\; R_k K \rangle = \left\langle hW^{Q}R_q,\; R_k W^{UK} \mathbf{c}^{KV}_t \right\rangle = \left\langle h\,W^{Q}R_q R_k W^{UK},\; \mathbf{c}^{KV}_t \right\rangle$$

The merged factor is now $W^{Q}R_q R_k W^{UK}$, which contains $R_q R_k$ and therefore
depends on the *relative position* of the query and key. It is a different matrix for every
position pair, so it cannot be precomputed once — and the saving evaporates.

Hashimoto states the problem and the fix without developing it: "this conflicts with RoPE
when you do KV caching, so you have to be a little careful about how you rotate different
dimensions… The trick is you have non-latent dimensions that encode position, but I'm not
going to go into too much more detail about that" ([1:24:41]).

Slide 58 puts the same solution in one line: "Have a few non-latent key dimensions that can
be rotated." So the key is split — most dimensions come from the latent and carry no
rotation, keeping the merge valid, while a small number of dimensions bypass the latent path
and receive RoPE normally. Slide 57's diagram shows exactly this: $\mathbf{k}^R_t$ is a
single hatched cell fed from $\mathbf{h}_t$ through an "apply RoPE" step, concatenated with
the latent-derived $\mathbf{k}^C_{t,i}$ per head. The query side is concatenated the same
way, from $\mathbf{q}^C_{t,i}$ and $\mathbf{q}^R_{t,i}$.

Hence the two cached tensors: the shared latent, and the one rotary key.

## Related pages

- [Attention variants](attention-variants.md) — MQA and GQA, the other KV-cache reductions.
- [RoPE](rope.md) — the position embedding MLA has to work around.
- [Linear attention](linear-attention.md) — the same associativity trick, used for a
  different purpose.
- [Multi-token prediction](multi-token-prediction.md) — DeepSeek V3's other bonus mechanism.
- [Lecture 4](04-attention-alternatives.md).
