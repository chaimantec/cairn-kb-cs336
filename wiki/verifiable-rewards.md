# Verifiable rewards

The premise of [RLVR](rlvr.md) is that some domains admit an *external check* on
correctness: a compiler, a test suite, an answer comparison. Because such a
check is not a learned model, it does not degrade as you optimize against it the
way a [reward model](reward-models.md) does — so you can spend far more compute
on RL before [overoptimization](reward-overoptimization.md) sets in.

[Lecture 16](16-post-training-rlvr.md) makes that case and then spends a good
part of its second half undermining it, which is the reason this page exists
separately from [RLVR](rlvr.md). The honest position is that verifiability is a
spectrum, and most of the useful part of it is softer than the word suggests.

## The strong case

Go is the reference point: the win-loss condition is exact, so "you can just put
in as much compute as you want, and as long as the objective improves, you're
doing well" (≈2:22). Formal mathematics is the closest language analogue — a
proof assistant genuinely decides — and unit tests are close behind for code.

## Where it weakens: mathematics

Natural-language mathematics has a correct answer but no canonical *form* for
it, and this is where the check starts to leak. The lecture's account of Kimi:
for code they generate new test cases from ground-truth solutions, but for math
"they actually have a reward model to check for answer equivalence" (≈49:19).
The lecturer treats this as the lecture's own punchline:

> we started out this lecture by saying we want to work on formal math, or
> something truly verifiable, where a compiler can check the correctness of your
> math, and we've gone through most of the lecture, and then, in the end, where
> have we ended up? Well, we ended up with a reward model — a reward model that
> checks the correctness of math answers. (≈50:05)

The reason is not laziness. Equivalence checking is genuinely hard:

> in math you can write equivalent things in many ways, and not only that, a
> language model can give the answer back in many ways. Even if you prompt it to
> give the answer back in a LaTeX boxed format, maybe sometimes it skips the
> box, maybe it adds some extra stuff to the box. (≈50:05)

The consequence for anyone building one: "most RL projects have a very
complicated answer checker — either a regex, or a model, or who knows what... It's
a real rabbit hole, getting the verified part of RLVR right" (≈50:50). CS336's
own assignment is expected to surface this.

## Where it breaks: the verifier is an attack surface

Even a real compiler is only as good as its adversarial robustness, and the
lecturer gives a first-hand example from his own group's work on RL over Lean:

> we naively thought at the time, there's no way this can go wrong. Lots of
> people have worked on Lean; the Lean compiler is bulletproof. Turns out the
> Lean compiler is not adversarially robust. There are strings that you can put
> in it that will allow you to verify proofs that are not meant to be verified,
> in certain modes. (≈1:07:52)

So the conclusion is stated as a general principle rather than a caveat:

> I think the notion of verifiable rewards is actually much trickier than many
> of you might initially think. (≈1:07:52)

In the agentic setting the same thing happens with the environment rather than
the checker — the model reads the answer out of the repository's future commits.
See [reward hacking](reward-hacking.md).

## How to read the term

A useful way to hold this: RLVR does not eliminate the proxy problem, it
*narrows* it. A verifier fails in ways that are rarer, more specific, and more
patchable than a preference model's smooth drift, and it does not degrade
merely because you optimized against it. But the scaling argument for RLVR
assumes the reward is unhackable, and the lecture is explicit that this
assumption is doing the work:

> the reason why we can put more and more compute into RL is because we believe
> that our reward models are unhackable, or difficult to hack. If that
> assumption breaks down, your RL method will find increasingly obscure ways of
> cheating you out of your performance. (≈1:06:19)

## See also

- [RLVR](rlvr.md) — the method built on this premise
- [Reward hacking](reward-hacking.md) — the failures in detail
- [Reward models](reward-models.md), [reward overoptimization](reward-overoptimization.md)
- [Kimi K1.5](kimi-k1-5.md) — whose math reward is a model
- [Agentic RL](agentic-rl.md) — verifiers as environments
- [Lecture 16](16-post-training-rlvr.md)
