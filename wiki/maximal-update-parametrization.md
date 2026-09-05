# Maximal update parametrization (muP)

**muP is a rule for how to change initialization and learning rate as you widen a
network, chosen so that the best learning rate you found on a small model is still the
best learning rate on a large one.** It is the answer
[lecture 9](09-scaling-laws.md) promised and deferred — see
[learning rate scaling and muP](learning-rate-scaling-and-mup.md) for the problem
statement — and [lecture 11](11-scaling-laws-in-the-wild.md) is where it is derived and,
more usefully, where its limits are measured.

The practical claim is worth stating plainly before the mathematics, because it is what
makes muP worth anyone's attention: **tune on a small model, transfer the hyperparameters
to a big one.** If that holds, the most expensive part of a scaling recipe — sweeping
learning rates at the scale you actually care about — collapses to a sweep you can afford.

## The two conditions

muP is derived from two assertions about what should stay constant as the width $n_l$ of
layer $l$ grows (slide [46](../raw/slides/11-scaling-laws-in-the-wild.md)):

- **A1 — the activations at initialization should remain $\Theta(1)$.**
- **A2 — after one gradient step, the change in activation should be $\Theta(1)$.**

Both are per-coordinate statements. The slide adds the bridge to the norms actually used
in the derivation: if individual activations are $\Theta(1)$, then the norm of the whole
activation vector is $\Theta(\sqrt{n_l})$.

That is the entire idea. Everything below is working out what initialization and learning
rate those two conditions force.

The derivation is presented from Yang, Simon and Bernstein's *A Spectral Condition for
Feature Learning*, which the slide describes as "a very accessible 'muP for babies'
paper" — a fair signal of the level of the treatment here.

## Deriving A1 — the initialization

Take a deep linear network, $h_l = W_l h_{l-1}$, initialized
$W_l \sim N(0, \sigma^2 I_{n_l \times n_{l-1}})$ (slide
[47](../raw/slides/11-scaling-laws-in-the-wild.md)). Basic matrix concentration gives the
spectral norm

$$\|W_l\|_* \to \sigma\left(\sqrt{n_{l-1}} + \sqrt{n_l}\right)$$

and the activation norm follows it, $\|h_l\|_2 \approx \|W_l\|_* \|h_{l-1}\|_2$. Now choose

$$\sigma = \frac{\sqrt{n_l}}{\sqrt{n_{l-1}}}\left(\sqrt{n_l} + \sqrt{n_{l-1}}\right)^{-1}
= \Theta\!\left(\frac{1}{\sqrt{n_{l-1}}}\min\!\left(1, \sqrt{\frac{n_l}{n_{l-1}}}\right)\right)$$

and the induction closes. Assume $\|h_{l-1}\|_2 = \Theta(\sqrt{n_{l-1}})$; then
$\|W_l\|_* \to \sqrt{n_l}/\sqrt{n_{l-1}}$, and therefore

$$\|h_l\|_2 = \sqrt{n_l} + o\!\left(\sqrt{n_l}\right)$$

which is A1 restated one layer along. The slide flags its own looseness: this is "a kind
of 'worst case' derivation" and the $\approx$ is an upper bound.

## Deriving A2 — the learning rate

Now the update. For SGD on a linear layer the weight update is a rank-one outer product of
the loss-gradient and the incoming activation (slide
[48](../raw/slides/11-scaling-laws-in-the-wild.md)):

$$\Delta W_l = -\eta_l \nabla_{h_l}\ell\, h_{l-1}^{\top}$$

so $\|\Delta W_l h_{l-1}\|_2 = \|\Delta W_l\|_* \|h_{l-1}\|_2$, and the change in
activation decomposes as

$$\Delta h_l = W_l \Delta h_{l-1} + \Delta W_l\left(h_{l-1} + \Delta h_{l-1}\right)$$

Assuming the leading-order terms do not cancel, the three pieces are $W_l \Delta h_{l-1} =
\Theta(\sqrt{n_l})$ by induction and the A1 argument; $\Delta W_l h_{l-1} = \|\Delta
W_l\|_* \sqrt{n_{l-1}}$; and $\Delta W_l \Delta h_{l-1} = O(\|\Delta W_l\|_*
\sqrt{n_{l-1}})$. Matching the second against the first gives the condition on the update's
size:

$$\|\Delta W_l\|_* = \Theta\!\left(\frac{\sqrt{n_l}}{\sqrt{n_{l-1}}}\right)$$

The remaining step turns that into a learning rate (slide
[49](../raw/slides/11-scaling-laws-in-the-wild.md)). Suppose the loss update also scales
$O(1)$. Then

$$\Delta\ell \approx \Theta\left(\left\langle \Delta W_l, \nabla_{W_l}\ell \right\rangle\right)
= \Theta\left(\|\Delta W_l\|_F \left\|\nabla_{W_l}\ell\right\|_F\right)
= \Theta\left(\|\Delta W_l\|_* \left\|\nabla_{W_l}\ell\right\|_*\right)$$

using $\Delta W_l = -\eta \nabla_{W_l}\ell$ for standard SGD. Substituting $\Delta\ell =
O(1)$ and the bound just derived gives $\|\nabla_{W_l}\ell\|_* =
\Theta(\sqrt{n_{l-1}}/\sqrt{n_l})$, and feeding that back through the outer-product form of
$\Delta W_l$ yields the result:

$$\eta_l = \Theta\!\left(\frac{n_l}{n_{l-1}}\right)$$

The slide's footnote records the Adam case separately, because it is the one that matters
in practice: with Adam, $\|\Delta W_l\|_* \sqrt{n_{l-1}} = \Theta(\sqrt{\eta_l})$.

## The prescription, and how it differs from standard practice

Collecting both halves (slide [50](../raw/slides/11-scaling-laws-in-the-wild.md)):

| | muP | standard parametrization |
| --- | --- | --- |
| **Initialization (stdev)** | $\Theta\!\left(\frac{1}{\sqrt{n_{l-1}}}\min\!\left(1, \sqrt{n_l/n_{l-1}}\right)\right)$ | $\frac{1}{\sqrt{n_{l-1}}}$ |
| **Learning rate** | $\frac{n_l}{n_{l-1}}$ &nbsp; (for Adam, $\frac{1}{n_{l-1}}$) | $\Theta(1)$ |

The differences are narrower than they look, and the slide names them exactly: **the
learning rate changes for Adam, and the initialization differs when the fan-out $n_l$ is
smaller than the fan-in.** Where $n_l \ge n_{l-1}$ the $\min(\cdot)$ saturates at 1 and
muP's initialization is the familiar $1/\sqrt{\text{fan-in}}$.

In implementation terms the deck also gives the practitioner's version — the $\mu$-Transfer
table of what to divide when you widen a model by a factor $r$ (slide
[44](../raw/slides/11-scaling-laws-in-the-wild.md)):

| Hyperparameter (weight) | $M$ | $M' \sim r$ |
| --- | --- | --- |
| AdamW learning rate (matrix-like) | $l$ | $l/r$ |
| AdamW learning rate (others) | $l$ | $l$ |
| Initialization variance (matrix-like) | $\sigma$ | $\sigma/r$ |
| Initialization variance (others) | $\sigma$ | $\sigma$ |
| Multiplier (output) | $\tau$ | $\tau/r$ |
| Multiplier (others) | $\tau$ | $\tau$ |

"Matrix-like" means a parameter tensor with two dimensions that go to infinity with width;
embeddings count as "others", and "output" is the layer mapping an infinite dimension to a
finite one — `lm_head` in a Transformer.

## Does it work? The evidence in this lecture

**The headline figure** (slide [44](../raw/slides/11-scaling-laws-in-the-wild.md)) puts
standard practice and muP side by side, training loss against $\log_2(\text{LR})$, one
curve per width from 128 to 8192. Under standard practice the optimum **shifts** — about
0.9 in $\log_2$ per doubling of width, which is the $1/r$ the table above prescribes, for a
total of 5.6 doublings of learning rate across the width range — and the best achievable
loss stops improving, bottoming out at 4.06 at width 2048 and getting *worse* at 4096 and
8192. Under muP the optimum is **stable**: five of the seven widths put it at
$\log_2(\text{LR}) = -10.3$, no curve spikes, and the loss at the optimum falls
monotonically from 4.50 to 3.65.

**The replication** (slide [52](../raw/slides/11-scaling-laws-in-the-wild.md)) asks the
question directly — when we scale widths, is the optimal LR constant? Across widths 128,
512 and 2048, a $16\times$ span, for both baseline muP and a projection-biases ablation,
the per-row minimum sits in the $2^{-6}$ column in **all six rows**.

**The scale evidence** (slide [57](../raw/slides/11-scaling-laws-in-the-wild.md)) is the
more convincing half, because it contrasts against standard parametrization rather than
merely confirming muP. Under SP the optimum marches left with width — $2^{-6}$, $2^{-8}$,
$2^{-10}$ — and the model *diverges* at larger learning rates, with losses of 7.2–7.5 at
width 2048 from $2^{-6}$ onward. Under muP, over 2M/40M/600M/10B parameters (widths 128 to
8192, a $64\times$ span), the optimum stays at $2^{-6}$ in every row. The lecture's own
summary is appropriately hedged: "muP generally seems useful — insofar that SP is quite a
bit more unstable", and "current evidence suggests that muP parametrization /
initialization may be easier to tune."

**A caution on reading the headline claim too strongly.** The CerebrasGPT panel (slide
[45](../raw/slides/11-scaling-laws-in-the-wild.md)) shows muP *below* the baseline at three
of five shared model sizes and *above* it — worse — at 256M and 2.7B. The evidence there is
for **stability**, not uniform improvement: the spread of the percentage-loss series is
0.31 for muP against 1.89 for the baseline. muP's promise is that the tuning transfers,
not that every model gets better.

## What muP is *not* robust to

This is the part of the lecture that a practitioner most needs, and it is why muP is a
tool rather than a guarantee. Modern language models carry many components that deviate
from muP's theory (slide [53](../raw/slides/11-scaling-laws-in-the-wild.md)) — SwiGLU and
squared-ReLU activations, large or small batch sizes, initialization variations like zero
attention, RMSNorm gains, exotic optimizers, regularizers — and the lecture asks which of
them actually break it.

Three do, each measured as an LR sweep at widths 128, 512 and 2048 where transfer means
*the bolded per-row minimum stays in one column*:

| What breaks it | Where the optimum lands (128 / 512 / 2048) | Transfers? |
| --- | --- | --- |
| **RMSNorm gains**, vector (slide [54](../raw/slides/11-scaling-laws-in-the-wild.md)) | $2^{-4}$, $2^{-4}$, $2^{-8}$ | ✗ |
| **RMSNorm gains**, scalar (slide [54](../raw/slides/11-scaling-laws-in-the-wild.md)) | $2^{-4}$, $2^{-4}$, $2^{-6}$ | ✗ |
| **Lion**, a sign-based optimizer (slide [55](../raw/slides/11-scaling-laws-in-the-wild.md)) | $2^{-10}$, $2^{-8}$, $2^{-8}$ | ✗ |
| **Strong decoupled weight decay** (slide [56](../raw/slides/11-scaling-laws-in-the-wild.md)) | $2^{-8}$, $2^{-6}$, $2^{-6}$ | ✗ |

The Lion case is the most emphatic: besides moving the optimum, it diverges outright at the
larger learning rates, with losses of 10.28–10.38 in the $2^{-2}$ column at every width and
already at $2^{-4}$ for width 2048. That is unsurprising in hindsight — Lion updates with
$\mathrm{sign}(c_t)$, so the size of its update carries no information about the gradient's
magnitude, and the derivation above is entirely an argument about update magnitudes.

The weight-decay case is the mildest and the easiest to over-read: it is a one-step drift,
and at width 128 the gap between the winning column and its neighbour is only 0.015.

## See also

- [Learning rate scaling and muP](learning-rate-scaling-and-mup.md) — the problem this
  solves, from lecture 9, and the two competing philosophies for handling it.
- [Critical batch size](critical-batch-size.md) — the other hyperparameter you cannot
  inherit, and the one muP does not address.
- [Published scaling recipes](published-scaling-recipes.md) — MiniCPM adopts muP; DeepSeek
  deliberately does not, and fits batch and LR scaling laws instead.
- [Optimizer scaling](optimizer-scaling.md) — including Muon, whose matrix-valued updates
  interact with exactly the spectral-norm reasoning used above.
- [Transformer hyperparameters](transformer-hyperparameters.md) — the ones that *are*
  roughly scale-invariant, and so need no muP.
- [Lecture 11](11-scaling-laws-in-the-wild.md) · slides [44](../raw/slides/11-scaling-laws-in-the-wild.md), [46–50](../raw/slides/11-scaling-laws-in-the-wild.md), [52–57](../raw/slides/11-scaling-laws-in-the-wild.md)
