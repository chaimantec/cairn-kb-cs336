# Expert iteration

Sample from the model, keep the outputs that are correct, and fine-tune on
those. Repeat. Also called rejection fine-tuning (RFT) or, in one line of work,
ReST. It is the obvious cheap alternative to reinforcement learning, and
[lecture 16](16-post-training-rlvr.md) takes the comparison seriously rather
than dismissing it.

## Why it is attractive

It is just SFT in a loop. There is no advantage estimation, no KL term, no
value function, no on-policy/off-policy tension, and none of
[PPO's implementation minefield](ppo.md). In the lecture's framing it is what
you may want "in cases where you're dealing with very unstable stuff" (≈54:44).

CS336's assignment implements it as a baseline: RFT is "just taking the correct
answers your model generates, training on them, and throwing away everything
else" (≈19:14).

## What it gives up

**The negative gradient.** RL learns from failures as well as successes —
wrong answers get pushed down, not merely ignored. Expert iteration discards
them. Slide 48 poses this as its own question: "Could we avoid RL-style negative
gradients and just learn from positives?"

## The evidence

Two results, from different papers, both pointing the same way.

DeepSeekMath's charts (slide 21) put [GRPO](grpo.md) above both RFT and online
RFT on GSM8K and MATH, which is the lecture's first pass at the question:
"they show that GRPO — the yellow and blue lines — does much better than RFT,
rejection fine-tuning" (≈19:14).

![Slide 21 — How well does it work?](../raw/images/16-post-training-rlvr/slide-21.jpg)
*Slide 21 — four series: RFT (purple) lowest, Online RFT (green) above it, and the two GRPO variants (orange, blue) highest.*

[Kimi K1.5](kimi-k1-5.md) runs the ablation at scale across twelve benchmarks,
and the lecture treats it as the settled answer:

> Kimi K1.5 has very large-scale ablations showing that these kinds of RL
> methods work consistently better than expert iteration — that's the orange
> beating the blue over here. So you can't really avoid RL if you want to
> squeeze out all of your performance. (≈54:44)

![Slide 48 — Ablation: comparison to expert iteration](../raw/images/16-post-training-rlvr/slide-48.jpg)
*Slide 48 — twelve panels, RL (orange) against ReST (blue). The transcribed values are hand-read from small charts and marked approximate in the slide file; the direction is consistent across panels.*

## The nuance worth keeping

"You can't really avoid RL if you want to squeeze out all of your performance"
is a claim about the *last* increment, not about everything. Two nearby results
in the same lecture complicate the simple reading:

- **[Reasoning distillation](reasoning-distillation.md) works well.** SFT on
  good long chains of thought recovers much of the capability, and the lecture
  leaves open "do you really need RL for some of this?" (≈33:56).
- **RL's distinctive role may be supervision, not optimization.** The framing
  offered is that RL "is a great source of supervision" where no human
  demonstration exists — "if you're solving frontier math problems, you just
  don't have the supervision to get detailed long CoTs, and RL allows you to
  self-generate that" — but "once someone has generated these long CoTs, you
  could potentially also learn from imitation" (≈34:42).

Read together: RL generates the trajectories nobody could write down; imitation
can then spread them cheaply.

## See also

- [GRPO](grpo.md), [RLVR](rlvr.md)
- [Kimi K1.5](kimi-k1-5.md) — the ablation
- [Reasoning distillation](reasoning-distillation.md) — the imitation route
- [Supervised fine-tuning](supervised-fine-tuning.md)
- [Lecture 16](16-post-training-rlvr.md)
