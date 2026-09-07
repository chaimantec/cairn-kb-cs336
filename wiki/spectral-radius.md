# Spectral radius

The largest absolute eigenvalue of a matrix, written $\rho(A)$ — and the quantity
that decides whether repeatedly applying that matrix makes a vector explode,
vanish, or stay bounded. It is the stability criterion behind
[PARSE](parse.md), introduced in
[Lecture 18](18-serving-megakernels-recurrence.md) (≈50:11–≈51:45).

## Why it appears here

Any architecture that applies the *same* transformation many times in sequence is
governed by a power of a matrix. In a [looped transformer](looped-transformers.md)
the loop applies a residual update $t$ times, and once the nonlinear part is set
aside the dynamics are dominated by $A^t$ (≈49:24):

> This thing is dominated by these A matrices, and especially this A matrix that
> you are powering up to a large degree.

Then the behaviour of $A^t$ as $t$ grows is decided by $\rho(A)$:

- $\rho(A) < 1$ — the repeated term decays toward zero. **Stable.**
- $\rho(A) = 1$ — neither decays nor grows. **Marginally stable**, and fragile:
  nothing stops training from nudging it either way.
- $\rho(A) > 1$ — the repeated term grows geometrically. **Unstable.**

The lecture's gloss is deliberately informal — "spectral radius is basically
another word for norm" (≈50:11) — and its intuition is scalar:

> If you go to scalars, imagine this matrix is two, and then this t is like 16 or
> something — you've now taken this activation and blown it up to 2 to the 16th,
> and it's really big. (≈50:11)

$2^{16} = 65536$, from a factor of two applied sixteen times. That is the whole
argument: a per-iteration factor barely above one is harmless once and fatal
compounded.

## Why it explains loss spikes rather than steady divergence

A useful subtlety. $\rho(A)$ is a property of a *learned* matrix, so it changes as
training proceeds. A model can train stably while $\rho(A)$ sits below one, drift
above it, produce an activation blow-up and a loss spike, and be pushed back. That
is why the symptom is **spikes** rather than immediate divergence, and why it is
so sensitive to the learning rate — a larger step is more likely to carry $A$
across the threshold. It is also why prior looped models "just pick the learning
rate of 2e-4, don't pick any of the other learning rates" (≈46:19): that hyperparameter
was doing stability work nobody had named.

The lecture's summary of the prior art in these terms: those choices are "either —
we call them marginally stable, or unstable, and unstable makes the system
unstable" (≈50:59). Note where the identity matrix falls. A residual connection
that simply adds is $\rho(A) = 1$ exactly — marginally stable, which sounds safe
and is in fact the fragile case.

## Enforcing it

[PARSE](parse.md) constrains the matrices rather than regularizing them
(≈50:59, ≈51:45):

- **$A$ is made a negative diagonal matrix**, so that "if you power that up, the
  term eventually goes to zero, so it doesn't blow up." A diagonal matrix has its
  diagonal entries as eigenvalues, which is what makes $\rho(A)$ directly
  controllable by construction rather than by penalty.
- **$B$ takes a simple linear norm.** It "only gets applied once, so it doesn't
  really blow up" — it is not raised to a power, so it needs no eigenvalue
  constraint, only scale control.

"If you compute the spectral radius, it's now going to be less than one — it's now
actually going to be a stable system" (≈51:45).

The general lesson is that **the constraint belongs where the compounding
happens.** A term applied once and a term applied $t$ times need different
treatment, and treating them alike is what the earlier designs got wrong.

## Where else this reasoning shows up

The same eigenvalue argument governs vanishing and exploding gradients in
classical RNNs, and the stability of linear state-space recurrences — which is
where the lecture says the technique came from: "we use some state space model
theory, some SSM theory, to stabilize this operation" (≈42:27). See
[state space models](state-space-models.md), and
[attention alternatives (Lecture 4)](04-attention-alternatives.md) for the
recurrent architectures CS336 covers.

It is worth contrasting with the stabilizers the course meets elsewhere.
[Normalization](rmsnorm.md) and gradient clipping act on symptoms
after the fact; a spectral constraint makes the divergent trajectory
unrepresentable. [PARSE](parse.md) argues the difference matters — norms alone
leave the model and the norm "fighting against each other, and that manifests in
loss spikes" (≈52:32).

## See also

- [PARSE](parse.md) · [Looped transformers](looped-transformers.md)
- [State space models](state-space-models.md) ·
  [RMSNorm](rmsnorm.md) · [Pre-norm and post-norm](pre-norm-and-post-norm.md)
- [Lecture 18](18-serving-megakernels-recurrence.md)
