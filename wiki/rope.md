# RoPE — rotary position embeddings

How modern language models tell the difference between "we know" and "know we".
RoPE is the position embedding used by essentially every model after 2024, and it
is what assignment 1 asks you to implement (slide 4).

It is also, in Hashimoto's assessment, the part of the architecture where the most
genuine variation remains: "really, the thing that is very different across
implementations, and I think a place where a lot of the architecture stuff is still
in flux, is how you do position dependence" ([30:43]).

## Why any position embedding at all

Attention is a set operation. It computes inner products between queries and keys,
and inner products do not know where their operands came from — "they're just inner
products, so you can shuffle them and attention would be the same if you don't have
a position embedding" ([31:29]). Position has to be injected deliberately.

Slide 30 lays out the four families that have been tried.

**Sine embeddings** add fixed sinusoids of varying frequency to the token
embedding:

$$\mathrm{Embed}(x, i) = v_x + PE_{pos}$$

$$PE_{(pos,2i)} = \sin(pos/10000^{2i/d_{model}}), \qquad PE_{(pos,2i+1)} = \cos(pos/10000^{2i/d_{model}})$$

Used by the original transformer, on a Fourier-transform intuition ([31:29]).

**Absolute embeddings** give each position its own learned vector $u_i$:

$$\mathrm{Embed}(x, i) = v_x + u_i$$

Used by GPT-1/2/3 and OPT.

**Relative embeddings** do not touch the token embedding at all. They add a learned
vector into the attention computation itself, as a function of the offset between
the two positions:

$$e_{ij} = \frac{x_i W^Q (x_j W^K + a^K_{ij})^\top}{\sqrt{d_z}}$$

Used by T5, Gopher and Chinchilla. "So if you're three positions off, the attention
matrix gets a different offset added to it" ([31:29]).

**RoPE** is the fourth, and the one that won.

## The property RoPE is designed to have

Slide 31 states the goal as a constraint. We want an embedding function $f(x, i)$
of a token $x$ at position $i$ such that the inner product depends only on the
*relative* offset:

$$\langle f(x, i), f(y, j) \rangle = g(x, y, i - j)$$

The right-hand side has no $i$ or $j$ except through their difference. Hashimoto
frames it as a deliberate stance ([33:01]): "I should not care about the absolute
position of any words."

None of the three earlier schemes satisfies it:

- **Sine** does not, because expanding $\langle v_x + PE_i,\; v_y + PE_j \rangle$
  leaves cross terms like $\langle PE_i, v_y \rangle$ that carry absolute position.
  He returns to this at [42:13]: "even sine and cosine embeddings are not pure
  relative position embeddings," because "you can back out what the absolute
  position is."
- **Absolute** obviously does not.
- **Relative embeddings** are relative, but they are not *embeddings* — they add
  into the attention matrix, so there is no inner-product factorization to extract
  ([33:47]). At [42:59] he concedes this objection is partly aesthetic: "that's
  more of an aesthetic problem," and AliBi and similar schemes that inject into the
  attention matrix "do work ... It's just not necessarily the one that's become the
  dominant approach."

## The idea: rotate instead of add

The insight is a single observation ([34:33]): **inner products are invariant to
rotation**, as long as both vectors are rotated by the same amount. So encode
position as a rotation whose *angle* is proportional to the position index. Then
rotating two tokens by their absolute positions leaves the angle *between* them
depending only on the difference.

Slide 32 draws this with three hand-drawn vector diagrams, and Hashimoto works the
example aloud ([34:33]–[35:19]):

Take the phrase **"we know that"**. The word *we* is at position 0, so it is not
rotated at all. *know* is at position 1, so it is rotated by one unit angle.

Now take **"of course we know"**. The same two words are still adjacent, but *we*
is now at position 2 and *know* at position 3. So *we* is rotated by two units and
*know* by three.

In both cases the angle between the two vectors is exactly one unit. The absolute
rotations changed; the relative one did not — and the inner product only sees the
relative one.

> The transcript garbles Hashimoto's illustrative example at [33:01] ("if — an apple
> — appear together..."), and the edited transcript marks it with an `[Ed:]` note
> rather than reconstructing it. The worked "we know that" / "of course we know"
> example above is from [34:33]–[35:19], where the captions are clean, and slide 32
> prints it as figure captions.

## Extending to $d$ dimensions

In two dimensions a rotation is a single angle. In $d$ dimensions there is an
infinite space of rotations, so which one? The answer ([36:06]) is "the simplest
possible thing, and it works": **cut the vector into consecutive pairs of
coordinates and rotate each pair in its own 2-D plane.**

Each pair $j$ gets its own constant angular frequency $\theta_j$, and a token at
position $m$ has that pair rotated by $m\theta_j$. Slide 34 gives the matrix form
from Su et al. 2021 — a block-diagonal matrix of $2\times2$ rotation blocks:

$$f_{\{q,k\}}(\boldsymbol{x}_m, m) = \boldsymbol{R}^d_{\Theta,m} \boldsymbol{W}_{\{q,k\}} \boldsymbol{x}_m$$

$$\boldsymbol{R}^d_{\Theta,m} = \begin{pmatrix}
\cos m\theta_1 & -\sin m\theta_1 & \cdots & 0 \\
\sin m\theta_1 & \cos m\theta_1 & \cdots & 0 \\
\vdots & \vdots & \ddots & \vdots \\
0 & 0 & \cdots & \cos m\theta_{d/2}
\end{pmatrix}$$

The frequencies are not all the same, and that is the point ([36:06]): "some of them
are very low frequency, so they rotate very slowly, so they can capture long-range
dependence; some of them rotate very quickly, so they capture things like whether
they're neighbors to each other."

Hashimoto's advice on the paper is worth passing on ([36:53]): "The paper, if you
read it, has a very complex motivation about complex numbers, but really, I think
the intuitive way ... is that you want to rotate by reducing to the two-dimensional
case."

## Why it has no cross terms

Slide 34's closing line is the whole argument in one sentence: "Difference with
sine embeddings — not additive, no cross terms."

Sine embeddings are *added* to the token vector, so expanding the inner product
produces mixed terms. RoPE *multiplies* the vector by a rotation matrix, so the
inner product of two rotated vectors is exactly the original inner product with the
relative rotation applied — nothing else survives ([37:38]): "it's really important
that I'm multiplying with these sines and cosines rather than using them as
embeddings, because that means there are no cross terms."

## Where it is applied

**At every attention operation, not once at the bottom of the network.** Slide 35
prints this in bold: "embedding at *each attention operation* to enforce position
invariance."

The slide shows the HuggingFace LLaMA implementation, which is the practical
answer to "how do I actually write this":

```python
cos, sin = self.rotary_emb(value_states, position_ids)
query_states, key_states = apply_rotary_pos_emb(query_states, key_states, cos, sin)
```

The rotation is applied to the **queries and keys only** — not to the values — and
it can be implemented either as a sparse matrix multiply or coordinate-wise by hand
([38:24]).

## Variants

**P-RoPE (proportional RoPE)**, from Gemma 4, rotates **only the first two
coordinates** and leaves the rest untouched ([36:53]). Slide 33's margin note
records it as "Gemma 4 alternative: just first 2", and slide 66 illustrates it as a
strip of cells where only the leading pair carries positional information and the
remainder is "only **semantic** information".

The rationale, from Hashimoto's answer to a student at [41:27]: "You don't rotate
most of them, because the argument ... is that the low-frequency parts just aren't
rotating very much, and so you can drop them if you're really strapped for extra
space. This is really an optimization for teeny-tiny models."

**NoPE — no position embedding at all** — used on the full-attention layers of
hybrid models. Cohere's Command A applies RoPE on its sliding-window layers and
none on its full-attention layers, so that "long-range info via NoPE, short-range
info via RoPE + SWA" (slide 65). See
[attention variants](attention-variants.md).

**RoPE scaling.** OLMo 3's hyperparameter table (slide 66) records YaRN scaling on
full-attention layers with $\theta = 5 \times 10^5$.

## A note on provenance

RoPE arrived unusually. Hashimoto at [32:15]: "it's kind of remarkable, given that
RoPE, in some ways, came out of nowhere. Originally, I think this was also a GPT-J
innovation, from, I think, a not-very-well-known blog post and paper combination,
from an author in China." The deck credits Su et al. 2021 on slides 33 and 34. It
is a good illustration of the lecture's theme that architectural knowledge in this
field propagates through practice rather than through theory.

## Related

- [Attention variants](attention-variants.md) — sliding-window and hybrid
  attention, where NoPE and partial RoPE now live.
- [Lecture 3 — architectures](03-architectures.md).
- [Transcript](../raw/transcripts/03-architectures.md), [slide deck](../raw/slides/03-architectures.md).
