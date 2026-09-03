# Multi-head latent attention (MLA)

DeepSeek's method for shrinking the KV cache: instead of caching keys and values directly,
cache a single low-dimensional latent vector per token and reconstruct keys and values from
it on demand. Lecture 4 covers it in slides 57–58 as one of the "bonus" mechanisms in
DeepSeek V3, alongside [multi-token prediction](multi-token-prediction.md).

![Slide 57 — Bonus: What else do you need to make DeepSeek MoE v3?](../raw/images/04-attention-alternatives/slide-57.jpg)

*Slide 57 — Bonus: What else do you need to make DeepSeek MoE v3? [Deck](https://github.com/stanford-cs336/lectures/blob/main/lecture_04.pdf)*

![Slide 58 — What else do you need to make DeepSeek MoE v3?](../raw/images/04-attention-alternatives/slide-58.jpg)

*Slide 58 — What else do you need to make DeepSeek MoE v3? [Deck](https://github.com/stanford-cs336/lectures/blob/main/lecture_04.pdf)*

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

## What lecture 10 adds

[Lecture 10](10-inference.md) revisits MLA from the serving side, and supplies two
things this page's construction does not: the compression ratio, and the accuracy
evidence.

**The ratio.** DeepSeek V2 reduces $NH = 16{,}384$ dimensions of stored key/value
per token to a latent of $C = 512$, plus 64 more for the rotary key described
above, for **576 total** — a factor of about 28. Percy calls it "quite aggressive
compression" ([52:38]). Compare [GQA's](attention-variants.md) factor of $N/K$,
which is typically 4–8. The corresponding claim on latency follows mechanically,
because [generation is memory-bound](prefill-and-generation.md): "the smaller the
KV cache, the faster you go — almost a linear scaling, up until some point"
([53:24]).

**The evidence, in two tables.** The lecture builds a deliberate two-step argument
([53:24]–[54:11]), and both tables are transcribed in
[`raw/slides/10-inference.md`](../raw/slides/10-inference.md):

*Table 8 — MHA beats GQA and MQA*, on 7B dense models: BBH 37.0 / 35.6 / 33.2,
MMLU 45.2 / 41.2 / 37.9, C-Eval 42.9 / 37.7 / 30.0, CMMLU 43.5 / 38.4 / 34.6 for
MHA / GQA / MQA respectively. Every MHA cell is bolded. This is the step that sets
up the comparison: the *expensive* baseline is genuinely the accurate one.

*Table 9 — MLA matches or beats MHA*, on MoE models, while cutting the cache
enormously. KV cache per token falls from 110.6K elements to 15.6K on the small
model and from 860.2K to 34.6K on the large one, and MLA is bolded on three of four
benchmarks for the small model and all four for the large one (BBH 50.7 vs 46.6,
MMLU 59.0 vs 57.5, C-Eval 59.2 vs 57.9, CMMLU 62.5 vs 60.7).

So the claim is not the usual one. GQA trades accuracy for speed; MLA is claimed to
be *better* than the expensive baseline while being much cheaper than the cheap
one. Percy's own reading is more cautious than the bolding: "they show that their
method, MLA, works even a little bit better than MHA. But let's just say it's about
the same" ([54:11]).

**A caveat carried from the same passage.** Table 8 directly contradicts the GQA
paper's own accuracy evals, which report that GQA costs almost nothing. Both are in
this lecture, minutes apart, and the moral drawn is general: "take everything
that's not just math with a grain of salt" ([51:06]).

**One student question worth keeping** ([54:57]): why compress the KV cache rather
than just shrinking the model dimension? The ablations do not answer it, and Percy
gives a judgement rather than data — reducing $D$ "just makes things worse, because
you're indiscriminately reducing everything… the trick in all of this is to find
places in the model where you can squeeze", and which places those are is not
knowable a priori.

## Related pages

- [Attention variants](attention-variants.md) — MQA and GQA, the other KV-cache reductions.
- [RoPE](rope.md) — the position embedding MLA has to work around.
- [Linear attention](linear-attention.md) — the same associativity trick, used for a
  different purpose.
- [Multi-token prediction](multi-token-prediction.md) — DeepSeek V3's other bonus mechanism.
- [Lecture 4](04-attention-alternatives.md).
- [Lecture 10 — Inference](10-inference.md) — the compression ratio and the accuracy tables.
- [KV cache](kv-cache.md) — the four axes; MLA cuts the dimension one.
- [Cross-layer attention](cross-layer-attention.md) — the layer-axis cut, composable with this.
