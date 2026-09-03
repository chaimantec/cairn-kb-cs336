# Linear attention

The trick that makes sub-quadratic attention possible, and the base case every
architecture in [lecture 4](04-attention-alternatives.md)'s first half builds on. It
rests on one fact — matrix multiplication is associative — plus one sacrifice, the
softmax.

Hashimoto's own framing of the whole family: "you only need to understand one core
idea to start with, and then we're going to elaborate and build upon that idea. And
that idea to start with is the associativity of multiplication" ([5:26]).

## The reordering

Write standard attention compactly, with $n$ the sequence length, $d_k$ the key
dimension and $d_v$ the value dimension:

$$\mathrm{Attn}(Q, K, V) = \rho\!\left(QK^\top\right)V, \qquad Q \in \mathbb{R}^{n \times d_k},\ K \in \mathbb{R}^{n \times d_k},\ V \in \mathbb{R}^{n \times d_v}$$

Here $\rho$ is the softmax applied row-wise. The cost is dominated by $QK^\top$, which
forms an $n \times n$ matrix at a cost of $n^2 d_k$, and then multiplying it by $V$ at
$n^2 d_v$.

Now suppose $\rho$ were the identity. Then the product is a plain chain of three
matrices, and the parentheses can move:

$$\left(QK^\top\right)V = Q\left(K^\top V\right)$$

On the right, $K^\top V$ is $d_k \times d_v$ — it does not depend on $n$ at all. The
total cost falls from $n^2 d_k + n^2 d_v$ to $2 n d_v d_k$ (slide 4). The sequence
length now appears linearly.

Why that trade is favourable in practice: $n$ "is very big, they're millions — that's
the context length," while $d_k$ and $d_v$ "are usually on the order of thousands, tens
of thousands. No one has a million coordinates in their hidden dimension" ([6:58]).

Slide 4 credits Shen et al. 2018 and Katharopoulos 2020 for the kernel version, and
notes connections to fast weight programmers.

## What it costs: the softmax

**Dropping $\rho$ is an approximation, and it is the only lossy step in the family.**
This matters because the rest of the derivation is exact, and it is easy to conflate
the two. Asked directly whether the parallel and recurrent forms should be equivalent,
Hashimoto separates them:

> The first step is where we drop the rho and become linear; that's the very first step
> of any of these. So this part is going to be lossy, and then after that — this linear
> form to this recurrent form — that equivalence is exact. ([20:49])

So when a linear-attention model underperforms full attention, the softmax is where
the loss came from, not the recurrent implementation.

Kernel-feature variants recover part of what the softmax provided by applying a feature
map to $Q$ and $K$ before the reordering; the lecture mentions the kernel version only
in the slide 4 citation and does not develop it.

## The recurrent form

The second observation (slide 5) is what made the family practical. Once the product is
written $Q(K^\top V)$, the inner term can be accumulated incrementally as you sweep
left to right through the sequence:

$$S_t = S_{t-1} + k_t v_t^\top \qquad \text{and} \qquad y_t = q_t^\top S_t$$

$S_t$ is a **state matrix** of shape $d_k \times d_v$ — crucially, its size does not
grow with $t$. Each step folds the current key–value outer product into the state and
reads out with the current query.

This is an RNN. Hashimoto's description: "we've started from linear attention on the
top left, and we've somehow gotten something that looks very much like an RNN"
([8:30]).

## Duality, and why it matters

The dense form and the recurrent form compute the same function, and they have opposite
performance profiles:

| Form | Shape | Good for | Why |
| --- | --- | --- | --- |
| Dense | $Q(K^\top V)$ | Training | Parallel across positions; big matmuls |
| Recurrent | $S_t = S_{t-1} + k_t v_t^\top$ | Inference | Fixed-size state, no growing KV cache |

The deck calls this **duality**, and it is the whole reason state space models became
viable when LSTMs had not been. "You can have it in this dense form, which is great for
training, since it's parallel, or you can have it in this serial form like an RNN,
which is great for inference. So you get the best of both worlds" ([9:15]).

Historically, attention beat LSTMs partly on expressiveness and partly because it
trained in parallel on the hardware. Duality removes the second half of that advantage
from the comparison:

> I think the reason these state space models have caught on, despite their similarity
> to LSTMs and similar drawbacks in representational power, is that now there's this
> well-understood duality… And the matrix-multiply form allows for computational
> efficiency. So that trade-off has now been checked off. ([33:07])

## The remaining trade-off: state size

What duality does *not* fix is the information bottleneck. A fixed-size $S_t$ has to
carry everything the model will need from the entire prefix, whereas full attention
keeps every token available.

> If your state is the size of your context, then you're good to go, but then you're
> paying these very large costs. So really, the point is: the free lunch is, if you want
> a really tiny state, it's really hard to compress all the information in a big
> context. ([33:56])

This is why every deployed model in the family is a hybrid rather than pure — see
[state space models](state-space-models.md).

## Where it goes next

Slide 5 notes that weighting the carried state by a scalar, $S_t = \gamma S_{t-1} + k_t v_t^\top$,
gives RetNet. Making that weight input-dependent gives Mamba-2; adding a second gate
and an erase term gives Gated DeltaNet. The rule governing which elaborations preserve
duality — gates may depend on the input, never on the state — is developed in
[state space models](state-space-models.md).

## The inference case for it, from lecture 10

[Lecture 10](10-inference.md) arrives at this family from the serving side, and a
student asks the question this page and
[attention variants](attention-variants.md) jointly raise: what is the tradeoff
between a linear-attention variant and a sliding window? ([58:46])

Percy's answer separates the two by *what they are good at* rather than by cost.
Both make the [KV cache](kv-cache.md) independent of sequence length, but "if you
care about local, high-resolution stuff, then sliding-window attention is better.
If you just want broad summaries of the past, then linear attention might be
better" ([1:00:17]) — and combining full attention, sliding windows and linear
layers is legitimate precisely because they capture different things.

Pressed on long context, he declines the easy answer: "there's no free lunch. If
you have to compress your entire history into a small state, you're just going to
lose information, and you might not be able to retrieve it" ([1:01:03]) — the
needle-in-a-haystack failure mode.

But on the direct comparison he comes down on this page's side, with a ceiling
argument rather than a benchmark ([1:01:03]–[1:01:49]):

> Mamba and DeltaNet are more powerful than sliding-window attention. Maybe you can
> think about the Mambas — […] it certainly can represent some of the aspects of
> sliding-window attention, because, as you're doing the recurrence, it can just
> look at the last state. So, maybe you can think of linear attention, or its
> extensions, as being better — at least, they have more room. Once you do
> sliding-window attention, you're done, there's nothing else you can —

The naive linear-attention baseline he gives is the one this page starts from: "you
just sum the KV values up into a single vector", which is trivially independent of
sequence length, and the gated variants exist to compress without forgetting as
much ([59:31]).

The lecture's closing judgement makes this the place where it thinks the large
remaining gains are: the KV cache and attention's structure "fundamentally make it
an inference-unfriendly kind of architecture", so an architecture designed for
inference —
linear attention, state space models, or diffusion — "can maybe unlock a lot"
([1:24:51]).

## Related pages

- [State space models](state-space-models.md) — Mamba-2, Gated DeltaNet, and the
  hybrids that ship.
- [Sparse attention](sparse-attention.md) — the other answer to the same cost problem,
  which keeps the softmax and drops tokens instead.
- [Attention variants](attention-variants.md) — MQA, GQA and sliding-window attention
  from lecture 3.
- [Lecture 4](04-attention-alternatives.md) — the lecture this comes from.
- [Lecture 10 — Inference](10-inference.md) — the serving argument, and where it beats a sliding window.
- [KV cache](kv-cache.md) — the object a fixed-size recurrent state replaces.
