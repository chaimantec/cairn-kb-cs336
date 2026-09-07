# PARSE

A stabilized [looped transformer](looped-transformers.md), and the second of the
two research threads in [Lecture 18](18-serving-megakernels-recurrence.md)
(≈41:42–≈59:30). Work from the speaker's UCSD lab, "led by Hayden, and also in
collaboration with two folks, Zachary and Taylor" (≈41:42).

The contribution is not the loop — that existed — but a **diagnosis of why looped
models blow up in training, and a two-line constraint that fixes it.**

> **On the name.** "PARSE" is heard only in speech; there is
> [no slide deck or handout for this lecture](18-serving-megakernels-recurrence.md)
> to check the spelling against, and the lecture never expands the acronym. The
> capitalization here is inferred from its use as a method name, and this KB does
> not invent an expansion for it.

## The question it starts from

Posed directly against the first half of the course: capability has come from
scaling parameters and data, so

> Is this the only way you have to scale, or is there potentially some other way
> that you can get this quality. (≈41:42, ≈42:27)

Looping is the candidate other way. It fails in practice because the models will
not train — "nine times out of ten this model just isn't going to converge"
(≈45:32).

## The analysis: model the residual, not the block

The move that makes the problem tractable, and the most transferable idea on this
page. Analyzing the recurrent block directly is hopeless: it "has tons of
parameters, there's all sorts of nonlinearities, there's a softmax, there's
[RoPE](rope.md)… if you try to analyze it analytically, it's quite complex"
(≈46:19).

So instead, look at what the loop *does to the activation* between iterations:

> Let's actually just look at the residual of this thing — how is this activation
> changing from block to block? Our first empirical observation was, hey, it
> actually doesn't change that much — each of these residual blocks is maybe
> changing the vector a little bit, but not really having a massive impact.
> (≈47:06)

That empirical fact licenses a model. Write a dynamical system over the residual,
push every nonlinear component into a single term $R$ and set it aside — "there's
this attention, there's this […] there's this big feedforward network, with the
intermediates and stuff — we're going to put all that into a box, we're just
going to call it R" (≈47:52) — and two matrices are left:

(The elision is a gap in the recording: the transcript marks one item in that
list as unclear rather than guessing at it.)

- $B$, "some transformation over your initial vector — what is that first vector
  before you start the loop?" (≈48:38)
- $A$, "how do you transform that residual in each loop?" (≈48:38)

The framing immediately pays off as taxonomy: prior loop transformers are
recovered as choices of these matrices. "In one case you just treat it as the
identity, you're just going to add things; in another case it's a fully learnable
matrix" (≈48:38).

## The diagnosis

Dropping $R$ leaves a system simple enough to solve "if you use high school
calculus," giving a closed form for the activation at step $t+1$ in terms of the
initial activation and injection. Reading it off (≈49:24):

> This thing is dominated by these A matrices, and especially this A matrix that
> you are powering up to a large degree.

Iterating the loop $t$ times applies $A$ roughly $t$ times, so the behaviour is
governed by $A^t$ — and therefore by the [spectral radius](spectral-radius.md)
$\rho(A)$. The scalar intuition the lecture gives:

> If this matrix can learn to be something like — let's say, if you go to scalars,
> imagine this matrix is two, and then this t is like 16 or something — you've now
> taken this activation and blown it up to 2 to the 16th, and it's really big.
> This starts to explain some of those big loss spikes. (≈50:11)

That is $2^{16} = 65536$, from a per-iteration factor of two applied sixteen
times.

The verdict on prior work: those choices of $A$ and $B$ are "either — we call them
marginally stable, or unstable, and unstable makes the system unstable" (≈50:59).
An identity $A$ sits exactly at $\rho(A) = 1$; a freely learnable $A$ can exceed
it. Nothing in the training objective prevents either.

> **What the transcript supports.** The lecture displays the closed form on a
> slide and describes it verbally; the recording does not dictate the full
> expression, and there is no deck to recover it from. This page therefore states
> the *structure* the lecturer states — dominated by a power of $A$, hence
> governed by $\rho(A)$ — and does not reconstruct the equation.

## The fix

Constrain the two matrices so the dynamics cannot diverge (≈50:59, ≈51:45):

- **$A$ becomes a negative diagonal matrix.** "If you power that up, the term
  eventually goes to zero, so it doesn't blow up."
- **$B$ gets a simple linear norm.** Safe to treat differently because "the $B$
  matrix actually only gets applied once, so it doesn't really blow up."

The result is $\rho(A) < 1$ — "it's now actually going to be a stable system."
Trained, the payoff is exactly what the diagnosis predicts: "even with the 6e-4
learning rate that was so bad for the other models, you actually got a stable
model at the end," and the unconstrained baseline by contrast "blows up, goes to
10 to the 19th" (≈51:45).

## Why norms alone were not enough

The subtlest point in the section, and the reason this counts as a fix rather
than another workaround. Applying a norm to an unconstrained model does control
the activation magnitude — but it does so by fighting the model:

> What happens here is that the model is actually trying to expand the
> activations, because it's saying, oh, with more room I can represent different
> things better, I can put these different concepts further away from each other,
> or whatnot. Then you're applying a norm to it, to take that big thing that's
> trying to expand and norm it back down to one. And then you have these two
> pressures that are kind of fighting against each other, and that manifests in
> loss spikes. (≈52:32)

The diagnostic signature is the giveaway: "even though on the right your norms are
very good, you're not seeing the activation actually blow up — you do see that the
loss can do some pretty gnarly things" (≈52:32). Healthy-looking norms with a
spiking loss means something is being suppressed, not resolved. Constraining $A$
and $B$ removes the pressure instead of counteracting it.

## Results

Compared against the prior "recurrent-depth models" and a strong transformer
baseline — "this transformer is like one of the nanochat ones, where a bunch of
people are trying to just get it to learn as fast as it can" — PARSE shows
"higher performance across a variety of applications," with "better perplexities,
better end-to-end quality as well" (≈53:18).

The scaling evidence, which is the more consequential claim, is on
[scaling laws for recurrence](recurrence-scaling-laws.md).

> **Scope.** No table, benchmark suite, model size or token count is recoverable
> from the recording, and there is no deck. This page reports the comparison the
> lecturer states and does not attach numbers to it.

## See also

- [Looped transformers](looped-transformers.md) — the architecture being fixed
- [Spectral radius](spectral-radius.md) — the criterion
- [Scaling laws for recurrence](recurrence-scaling-laws.md) — the scaling claim
- [Lecture 18](18-serving-megakernels-recurrence.md) ·
  [State space models](state-space-models.md)
