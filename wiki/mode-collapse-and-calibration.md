# Mode collapse and calibration

Two related things RLHF does to a model's output distribution: it narrows it, and
it destroys the correspondence between the model's confidence and its accuracy.
CS336's lecture 15 raises both in its closing minutes (≈1:17:08, ≈1:17:54).

## Mode collapse follows from the objective

This is not a bug that appeared in practice and needed explaining. It is licensed
by the objective, as lecture 15 sets up half an hour earlier (≈43:10):

> Now, in the second world, I can totally collapse my distribution onto a single
> point for every input. For every prompt, my model could have a single answer —
> not a distribution — and that would be okay, as long as it got a good reward.

Pre-training and SFT fit a distribution, and a distribution has diversity built
into what it is being asked to do. [RLHF](rlhf.md) maximizes a reward, and a
reward is indifferent to diversity. So:

> People have seen, lots of times, that our RL models have much less diversity —
> they're concentrated on a few possible outputs. This connects back to what I
> said before: RLHF models are really no longer modeling a distribution, which
> comes with its own inherent diversity. It's a policy that can collapse, as long
> as it gets a good reward. (≈1:17:08)

![Slide 64 — Things to watch out for - mode collapse](../raw/images/15-mid-post-training/slide-64.jpg)
*Slide 64 — output entropy across models. The post-RLHF spike near zero entropy is text-davinci-003's; the deck's transcription flags that the other series overlap too closely to separate reliably.*

The KL term against the reference policy is what pushes back, which is the same
mechanism that limits
[reward overoptimization](reward-overoptimization.md) — one term defending
against two failures.

### Why it matters downstream

Two reasons the lecture gives it a slide at all:

- **It undermines the sampling that RLHF itself depends on.** Collecting
  [preference data](preference-data.md) assumes that sampling a prompt several
  times yields meaningfully different responses — "at the end of SFT the model is
  pretty diverse, so this is a reasonable thing to do" (≈46:14). That assumption
  degrades as the policy collapses.
- **It is fatal for exploration in RLVR.** "This is very important in the next
  lecture, when we talk about things like RLVR, where the entropy and exploration
  is actually quite critical for the model to explore all the possible solutions
  and make progress on some of these very hard problems" (≈1:18:41). A collapsed
  policy on a hard reasoning problem samples the same wrong answer repeatedly and
  never finds the reward signal.

The lecturer skipped this slide for time but flagged it as underrated: "I think
this entropy-and-mode-collapse thing ends up being quite important, even today"
(≈1:17:54).

## Calibration

The second failure, and the one with a named open problem attached.

In the GPT-4 era, "this was actually one of the few plots that OpenAI put out
where they said, 'Actually, we have a few open problems left' — and one of the
open problems was, 'Our models are uncalibrated after we do RLHF'" (≈1:17:54). A
base model's token probabilities track accuracy reasonably well; after RLHF they
do not.

The lecturer's assessment, several years later: **"I don't think anyone has really
solved that yet."**

Anthropic's position, as the lecture reports it, is that the loss of calibration
is intrinsic rather than incidental — "it's naturally uncalibrated — you could
recalibrate sometimes, but not always."

![Slide 62 — Things to watch out for in RLHF](../raw/images/15-mid-post-training/slide-62.jpg)
*Slide 62 — the reliability diagram from the RLHF pitfalls slide, alongside the overoptimization plot.*

### The awkward interaction with hallucination

This deserves stating plainly because the lecture leaves the two threads a
half-hour apart.

Earlier, the argument for RL was **calibration**: SFT on facts the model doesn't
know teaches it to fabricate, and only policy-dependent training can teach a model
what it knows — see
[hallucination and knowledge extraction](hallucination-and-knowledge-extraction.md).
Schulman's claim is that "you can't have an external person shoving knowledge down
your throat if you want the model to be calibrated" (≈23:55).

Here, RLHF is the thing that *destroys* calibration.

Both can hold — they are about different quantities. Schulman's argument concerns
whether the model's *behaviour* reflects its internal knowledge state (does it
assert things it doesn't know?); the GPT-4 finding concerns whether its *stated
probabilities* match empirical frequencies. But the tension is real enough that it
is worth carrying as an open question rather than a resolved story, and the
lecture's own summary — nobody has solved this — is the honest position.

## See also

- [RLHF](rlhf.md) — the objective shift that permits collapse
- [Reward overoptimization](reward-overoptimization.md) — the other closing warning, sharing a defence
- [Hallucination and knowledge extraction](hallucination-and-knowledge-extraction.md) — the argument that cuts the other way
- [Preference data](preference-data.md) — the sampling that collapse undermines
- [DPO](dpo.md) · [PPO](ppo.md)
- [Lecture 15](15-mid-post-training.md)
