# State space models — Mamba-2, Gated DeltaNet, and hybrids

The family of architectures built by elaborating [linear attention](linear-attention.md)'s
state update. Despite the name and their origins in state space theory, lecture 4
presents them mechanically: "if you look at the mechanics of what's being done, you can
see it as a very simple elaboration of the linear attention mechanism" ([11:35]).

## The design rule

There is one constraint governing what elaborations are permitted, and it is worth
learning before the specific architectures:

> As long as you're gating the various terms in your RNN, so to speak, with only
> input-dependent terms — so no state dependence — then you'll still have this fairly
> nice duality between parallel operations, which you use for training, and serial
> operations, which you use for inference. ([14:41])

**Gates may depend on $x_t$. They may not depend on $S_{t-1}$.** A state-dependent gate
would break the [duality](linear-attention.md) that lets these models train in parallel,
which is the only reason they are competitive with attention at all.

## Mamba-2

Slide 7. Start from linear attention and add a single input-dependent forget gate
$\gamma_t$:

![Slide 7 — From linear attention to Mamba-2](../raw/images/04-attention-alternatives/slide-7.png)

*Slide 7 — From linear attention to Mamba-2. [Deck](https://github.com/stanford-cs336/lectures/blob/main/lecture_04.pdf)*

$$S_t = \gamma_t S_{t-1} + k_t v_t^\top, \qquad y_t = q_t^\top S_t + v_t^\top D, \qquad \gamma_t = f(x_t)$$

Compare linear attention, which is this with $\gamma_t = 1$ and no $D$ term. The deck
prints the new pieces in red to make exactly that comparison.

The motivation is LSTM-shaped:

> We know from the olden days of LSTMs that it's important to know when to pass
> information forward and when to not pass information forward — to forget things and
> send them to zero. ([12:21])

Because $\gamma_t$ depends only on $x_t$, it can be computed for all positions in
parallel, and duality survives — "you can compute this Mamba-2 term either as a big
dense matrix multiply, or at inference time use it in this recurrent form" ([13:07]).

The $v_t^\top D$ term is a residual-style pass-through of the current value directly to
the output. Hashimoto flags it as peripheral, added "for completeness, just to make sure
I'm actually giving the true Mamba-2 updates rather than a more abridged version"
([15:28]), and explains it in response to a question as letting the current token's own
value reach the output, with $D$ a gate controlling how much passes through ([21:36]).

Mamba, Mamba-2 and Mamba-3 are a family from Albert Gu, Tri Dao and collaborators
([11:35]).

## Gated DeltaNet

Slide 9. Two changes on top of Mamba-2 — a second gate and an erase term:

$$S_t = \gamma_t\left(I - \beta_t k_t k_t^\top\right) S_{t-1} + \beta_t k_t v_t^\top, \qquad y_t = q_t^\top S_t$$

with both $\gamma_t = f(x_t)$ and $\beta_t = f(x_t)$ input-dependent, as the rule
requires.

**$\beta_t$ is a write gate.** Hashimoto describes it as "a 'no input operation' gate.
If $\beta_t$ is zero, that basically means: don't take any of my current information,
don't add it into my state" ([16:14]). Where $\gamma_t$ controls forgetting, $\beta_t$
controls writing — together very close to an LSTM's forget and input gates, "although,
of course, derived originally in a very different way."

**The $\left(I - \beta_t k_t k_t^\top\right)$ factor is the "delta" part**, and it is the
more interesting one. It is an outer-product term that subtracts off the component of
the existing state lying along the current key direction. The intuition:

> I'm going to be writing in information from my current key, $k_t$, and when doing
> that, not only do I want to put in new information, I also want to erase any previous
> keys that have gone into it. ([16:59])

So the update overwrites rather than accumulates: whatever was previously associated
with this key is cleared before the new value is written. Hashimoto is careful that it
is only approximately a projector — "that's not exactly right, because you're not doing
things like unit normalization, but you get roughly the intuition here."

The same update has been arrived at independently several times. It "appears if you try
to solve certain kinds of meta-learning least-squares problems," and has been reinvented
in fast weight programming and test-time training "through very different design
principles" ([17:45]).

Gated DeltaNet is, in Hashimoto's assessment, "probably… among the most widely used
state space models now" ([14:41]).

## Nobody ships these pure

Every deployed model in this family interleaves linear layers with periodic full
softmax attention:

| Model | Hybrid ratio | Notes | Deck |
| --- | --- | --- | --- |
| MiniMax M1 | 7 linear : 1 full softmax | Also MiniMax-Text-01 | slide 6 |
| Nemotron 3 | ~3 Mamba-2 : 1 attention | Interleaved with MoE layers throughout | slide 8 |
| Qwen 3.5 / Qwen Next | 3 Gated DeltaNet : 1 attention | slide 10 |

![Slide 6 — Minimax M1](../raw/images/04-attention-alternatives/slide-6.jpg)

*Slide 6 — Minimax M1. [Deck](https://github.com/stanford-cs336/lectures/blob/main/lecture_04.pdf)*

![Slide 8 — Nemotron 3](../raw/images/04-attention-alternatives/slide-8.jpg)

*Slide 8 — Nemotron 3. [Deck](https://github.com/stanford-cs336/lectures/blob/main/lecture_04.pdf)*

![Slide 10 — Qwen 3.5 / Qwen Next](../raw/images/04-attention-alternatives/slide-10.jpg)

*Slide 10 — Qwen 3.5 / Qwen Next. [Deck](https://github.com/stanford-cs336/lectures/blob/main/lecture_04.pdf)*

Hashimoto states the limit plainly: "No one has thus far really proven out fully linear
time attention mechanisms at scale; everything I'm going to talk about in the next
couple of slides is a hybrid" ([10:49]).

Nemotron 3's layer diagram (slide 8) shows the pattern concretely — mostly interleaved
Mamba-2 and MoE layers, with "a select few self attention layers" scattered through, in
groups repeated ×5, ×3, ×1 and ×4.

## How much does the hybrid ratio cost you

Slide 11 carries what Hashimoto says is close to the only controlled study, from
ByteDance Seed and UC Santa Cruz, sweeping the proportion of RNN layers against a
full-attention dashed baseline.

![Slide 11 — Hybrid performance](../raw/images/04-attention-alternatives/slide-11.jpg)

*Slide 11 — Hybrid performance. [Deck](https://github.com/stanford-cs336/lectures/blob/main/lecture_04.pdf)*

The shape of the result, in his reading: at low ratios "there's basically no hit," past
some point degradation sets in, and "at the end, as you go to full RNN, you have very
noticeable performance degradation in all these architectures" ([20:04]).

Two caveats he raises himself, both of which should temper how the chart is quoted:

- **The evidence base is thin.** "There hasn't been that many great controlled studies
  of how hybrid architectures perform" ([18:31]), and he calls some of the results
  "kind of messy."
- **Retrieval benchmarks are optimized for.** "Single-key retrieval is a task that I
  think all of these long-context architectures explicitly optimize for" ([20:04]) — so
  the QA panel is the more honest one, and it shows steady decline with hybrid ratio.

## The standing trade-off

Asked what the downside is, Hashimoto answers expressive power, and locates it in the
state:

> The all-to-all connection in softmax attention is incredibly powerful, and it's also
> very easy to train… you've still got the trade-off that if you have a finite state and
> have to carry everything through, you're going to lose some information relative to
> just carrying everything. ([33:07])

The hardware half of attention's old advantage is gone, thanks to duality. The
information-bottleneck half is not, and it is what the hybrids buy their way out of by
keeping some full-attention layers.

His forecast is convergence rather than revolution: "we've seen that, at least, the
linear time attention stuff has converged a lot into LSTM-like or linear-attention-like
architectures, and I don't really see that changing too much in the near future"
([30:50]).

## Related pages

- [Linear attention](linear-attention.md) — the base case and the duality property.
- [Sparse attention](sparse-attention.md) — the alternative that keeps softmax.
- [Mixture of experts](mixture-of-experts.md) — Nemotron 3 and Qwen Next are MoEs too;
  the two ideas compose.
- [Lecture 4](04-attention-alternatives.md).
