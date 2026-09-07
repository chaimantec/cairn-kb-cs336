# Length bias in RL

[GRPO](grpo.md) divides each sequence's contribution by its own length. That one
choice creates a systematic pressure toward **long wrong answers**, and it is
the cleanest example in CS336 of an objective producing a behaviour nobody
asked for. [Lecture 16](16-post-training-rlvr.md) uses it to reinterpret one of
the most-shared plots in the reasoning-model literature.

## The mechanism

The normalizer divides by output length, so a longer sequence is divided by a
bigger number. For a *negative* reward, that shrinks the penalty. The lecture
states the extreme case and immediately labels it as extreme:

> let's say I know I'm going to get a math proof wrong, I'm going to incur my
> negative reward of, say, negative one. I'm just going to generate an
> infinitely long string; if I do that, I get to divide by infinity, and I get
> to totally get rid of my negative penalty. (≈23:53)

The realistic version: "if you divide by the output length, you encourage the
model to blab on once it realizes it can't actually solve the problem."

The effect is asymmetric, and a student asks about exactly that (≈38:34). On
correct answers the normalizer pushes the other way — "the length normalizer
encourages you to shorten the CoT — which is good if you want to save on
inference cost, bad if it degrades your accuracy" — but there is a floor:
"in the positive cases it can't shrink that much, because there's a lower bound
to how small your CoT can go to solve a particular problem." So the growth
comes from the failures.

![Slide 24 — Length biases of GRPO](../raw/images/16-post-training-rlvr/slide-24.jpg)
*Slide 24 — why dividing by length rewards a wrong answer for being long.*

## Why this changes how you read R1

[DeepSeek-R1](deepseek-r1.md)'s report highlights chains of thought growing
steadily longer through training, and the plot was widely read as the model
learning to think harder. The lecture's position is that this is at least partly
an artifact of the objective: runaway length "is arguably a natural side effect
of the length normalization of the GRPO algorithm" (≈30:04). Remove the
normalizer and the growth "actually turns out to just cap off at a constant,
rather than continually and forever growing" (≈24:39).

The evidence is a disaggregation. R1's own plot mixes everything together, while
Dr. GRPO separates the curves:

> In the R1 diagram, this is all aggregated — this is all the responses across
> their evals, not just the positive ones. But if we look at the Dr. GRPO plot
> over here: this is the average output length, and this is the incorrect output
> length, and the correct output length is here. So you can see that this is
> really being driven by the incorrect ones, as we'd expect from the
> explanation. (≈39:20)

That is the shape of a good methodological argument: a prediction from the
objective, then a controlled plot that separates the cases and matches it.

## The other side: length as a cost

[Kimi K1.5](kimi-k1-5.md) does not have this bias — its objective does not
normalize by sequence length — and it goes further, treating chain-of-thought
length as an expense to be actively compressed. The lecture contrasts the two
attitudes directly:

> I think the GRPO folks were like, "isn't it great that the length is growing
> uncontrollably, I'm sure our model is getting smarter" — that's a little bit
> of an uncharitable take, but when you present this plot as a positive thing,
> the implied statement is that it's great our model is thinking for longer.
> (≈46:16)

The economics: "If you're OpenAI, and your users have the $200 Pro plan, and
your models are thinking for an hour at a time, that's not a very good place to
be in. Whereas if your models are thinking for five minutes at a time, that is a
great place to be in" (≈47:02).

**But compression has its own failure mode**, and Kimi's design accounts for it.
Forcing wrong answers to be short can trap the model in a domain it is weak at:

> imagine I'm bad at geometry... and the penalty makes my geometry CoTs really
> short. Now my geometry CoTs are zero — I'm really bad at geometry, I will
> never recover from this, I will never get a positive geometry reward ever
> again, and I'm stuck. (≈48:33)

So incorrect answers are pushed only to be "shorter than the center of the range
of rollouts" rather than as short as possible (slide 43), which bounds growth
without removing the budget the model needs to find a solution.

## See also

- [GRPO](grpo.md) — where the normalizer lives
- [Advantage estimation and baselines](advantage-estimation-and-baselines.md) — why it is not a baseline
- [Long chain-of-thought](long-chain-of-thought.md)
- [DeepSeek-R1](deepseek-r1.md) — the plot being reinterpreted
- [Kimi K1.5](kimi-k1-5.md) — the compression approach
- [Test-time scaling](test-time-scaling.md) — length as a spendable budget
- [Lecture 16](16-post-training-rlvr.md)
