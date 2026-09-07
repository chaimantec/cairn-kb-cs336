# Reward overoptimization

Push [RLHF](rlhf.md) hard enough against a learned
[reward model](reward-models.md) and the policy stops improving on the thing you
actually wanted, while the reward model's score keeps going up. CS336's lecture 15
nominates this as the central open problem of RLHF (≈1:16:22, ≈1:18:41).

## The historical framing

The lecture sets it up as the answer to a once-live hope:

> When InstructGPT came out, and people were very excited about this, I think
> there was a very real question of, can we RLHF our way to superintelligent
> systems, or whatever? Maybe we can just collect enough thumbs-up, thumbs-down.
> It turns out that's actually quite challenging, because you find that if you try
> to really push the RLHF process forward, you'll start overfitting to your
> learned reward model.

The reward model is a *proxy*. It was fit to a finite set of comparisons drawn
from a particular policy's outputs, and it is accurate in that neighbourhood.
Optimizing against it moves the policy out of that neighbourhood, into a region
where the proxy no longer tracks what it was fit to predict — and the optimizer
will find that region, because that is where the proxy scores highest.

![Slide 63 — Things to watch out for - Overoptimization](../raw/images/15-mid-post-training/slide-63.jpg)
*Slide 63 — evaluation win-rate against a gold-standard judge as optimization proceeds. The proxy score rises monotonically; the gold-standard curve turns over.*

The shape on slide 62 is the canonical one: proxy reward climbing, gold reward
rising and then falling, with the gap widening as optimization continues.

![Slide 62 — Things to watch out for in RLHF](../raw/images/15-mid-post-training/slide-62.jpg)
*Slide 62 — gold versus proxy reward-model score against optimization distance, across a family of reward-model sizes.*

## The KL term is the defence

The $\beta\,\mathbb{D}_{\text{KL}}[\pi_\theta \,\|\, \pi_{\text{ref}}]$ term in the
RLHF objective is usually introduced as a stability detail. It is not:

> So the KL regularizer I talked about is really critical, in a lot of cases, to
> prevent your optimization process from overfitting your reward model — at least
> if your optimization process is very good. (≈1:17:08)

The conditional at the end is the interesting part, and it is easy to read past.
**A better optimizer makes this problem worse, not better.** A weak optimizer is
protected by its own incompetence — it never gets far enough from the reference
policy to leave the region where the reward model is valid. Improve it and you
need the constraint to do real work.

That inverts the usual relationship between optimization quality and results, and
it is a good reason to be suspicious of an RLHF run that reports unusually large
reward gains.

## Why this is a proxy problem, not a tuning problem

Overoptimization is Goodhart's law with a training curve, and no amount of care in
fitting the reward model removes it — a learned proxy fit on finite data will
always diverge from the target somewhere, and optimization pressure is exactly the
process of finding where.

Related failures elsewhere in the course have the same shape:

- **Benchmarks** measure a construct through a proxy metric, and optimizing the
  metric detaches it from the construct. See
  [construct validity](construct-validity.md) and
  [benchmark contamination](benchmark-contamination.md).
- **Perplexity** predicts downstream quality until you optimize hard enough
  against it. See [upstream vs downstream](upstream-vs-downstream.md).

What makes the RLHF case sharper is that the proxy is *learned from the model's
own outputs* and then optimized by the same model, so the feedback loop is tight.

## Where it goes next

The lecture's closing move is to pose lecture 16 as the response (≈1:19:29):

> The transition to the next lecture is going to be: is there a reward where we
> won't overoptimize, where we can just dump compute in and model performance just
> keeps monotonically getting better? And that's one of the reasons why what people
> call RLVR has been so impactful.

[RLVR](rlvr.md) replaces the learned proxy with a *program* — a unit test, a
proof checker, a numeric answer key. A verifier that cannot be argued with is a
verifier that cannot be Goodharted in the same way.

**[Lecture 16](16-post-training-rlvr.md) answers the "whether that fully
escapes the problem" question, and the answer is no — it narrows it.** Two
findings there bound the escape. Mathematics does not stay program-checkable in
practice: [Kimi](kimi-k1-5.md) verifies answer equivalence with a reward model,
because "in math you can write equivalent things in many ways" (≈50:05). And a
genuine program verifier can still be attacked — "the Lean compiler is not
adversarially robust. There are strings that you can put in it that will allow
you to verify proofs that are not meant to be verified" (≈1:07:52).

So the scaling argument for RLVR rests on an assumption rather than a
guarantee: "the reason why we can put more and more compute into RL is because
we believe that our reward models are unhackable, or difficult to hack. If that
assumption breaks down, your RL method will find increasingly obscure ways of
cheating you out of your performance" (≈1:06:19). See
[verifiable rewards](verifiable-rewards.md) and
[reward hacking](reward-hacking.md).

## See also

- [RLHF](rlhf.md) — the objective and its KL term
- [Reward models](reward-models.md) — the proxy in question
- [PPO](ppo.md) — the optimizer whose strength is the hazard
- [Mode collapse and calibration](mode-collapse-and-calibration.md) — the other closing warning
- [RLVR](rlvr.md) — the response, and its limits
- [Verifiable rewards](verifiable-rewards.md), [reward hacking](reward-hacking.md)
- [Construct validity](construct-validity.md) · [Upstream vs downstream](upstream-vs-downstream.md)
- [Lecture 15](15-mid-post-training.md), [Lecture 16](16-post-training-rlvr.md)
