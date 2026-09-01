# The IsoFLOP method

Fix a compute budget, sweep every remaining degree of freedom, and look at the
shape of the resulting loss curve. [Lecture 9](09-scaling-laws.md) presents it as
Chinchilla's method 2, and then as the one general-purpose tool from the whole
lecture that has kept working outside its original setting.

Hashimoto's own assessment: "very popular, and also very easy and very robust —
this is probably my personal favorite method" ([1:04:31]).

## The procedure

For [compute-optimal scaling](compute-optimal-scaling.md), the free variable is the
parameters-versus-data trade-off ([1:04:31]):

1. Pick a range of FLOP budgets — the Chinchilla sweep goes up in factors of about
   three.
2. At each budget, sweep the trade-off at **fixed compute**: double the data and
   halve the model, and so on along the $C \approx 6ND$ constraint. Each setting is
   one training run, and you record its terminal loss.
3. The runs at one budget trace a shallow parabola in loss against model size. Take
   its minimum — or fit a quadratic and take the minimum of the fit, which is more
   robust to the noise in any single run ([1:05:16]).
4. Regress those minima against FLOPs.

For Gopher's budget this gives **63B parameters**, agreeing closely with method 1's
67B ([1:05:16]).

## Why it is robust

The three Chinchilla methods differ in how much they assume, and IsoFLOPs sits in
the sweet spot:

- **Method 1** (lower envelope of training curves) needs you to identify the
  envelope, which is fiddly ([1:03:44]).
- **Method 3** (fitting the joint parametric form) assumes a functional shape and
  then depends on how well you fit a multi-variable surface — "there are many
  variables, it's kind of tricky" ([1:06:04]). This is the method that went wrong
  in the original paper.
- **IsoFLOPs** assumes only that loss-versus-size at fixed compute has a minimum,
  which the data shows directly. You are reading a minimum off a curve, not
  extrapolating a fitted surface.

That is why running all three at once is good practice rather than redundancy: it
is "a way of robustifying yourself against modeling assumptions you may have made"
([1:02:12]).

## Where else it turns up

The reason it gets a closing section of its own ([1:16:03]). The pattern —
fix compute, sweep the free parameters, read the surface — generalises past the
parameters/data question:

- **Diffusion models.** IsoFLOP profiles for autoregressive and diffusion models
  side by side, each with its own family of parabolas and their minima marked
  (slide 55, Gulrajani et al. 2023).
- **[MoE](mixture-of-experts.md) sparsity.** The Abnar et al. 2025 study is
  IsoFLOP-style by design, extended into a third dimension: surfaces over sparsity
  against total parameters, and over sparsity against active parameters (slide 55).
  Sparsity is a particularly good fit for the method because "I can vary my
  sparsity without really changing the amount of compute I'm spending" ([39:57]) —
  which is exactly the condition an IsoFLOP sweep needs.

The general advice: "if you're ever in a situation where you're thinking, 'how am I
going to decide all these tradeoffs?', IsoFLOPs is always a good default"
([1:16:03]).

## See also

- [Compute-optimal scaling](compute-optimal-scaling.md) — the problem it was built for.
- [Mixture of experts](mixture-of-experts.md) — the sparsity surfaces.
- [Scaling law methodology](scaling-law-methodology.md) — running several estimators as insurance.
- [Lecture 9](09-scaling-laws.md) · slides [48](../raw/slides/09-scaling-laws.md), [55](../raw/slides/09-scaling-laws.md) · [transcript](../raw/transcripts/09-scaling-laws.md)
