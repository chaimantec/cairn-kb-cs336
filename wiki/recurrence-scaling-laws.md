# Scaling laws for recurrence

The claim that **recurrence count is a third axis to scale**, alongside parameters
and data, and that it obeys power laws of the same familiar kind. From
[Lecture 18](18-serving-megakernels-recurrence.md) (≈54:05–≈58:41), reporting
initial results for [PARSE](parse.md).

This is the part of the guest lecture that speaks most directly back to
[Lecture 9](09-scaling-laws.md) and
[Lecture 11](11-scaling-laws-in-the-wild.md).

## The setup: reading a scaling plot

The lecture first re-derives how to read the classic parameters-versus-data
result, because the recurrence result is read the same way (≈54:51):

> If that curve is going down and to the right, it suggests you should be scaling
> both data and parameters at the same time. If it were going straight down, that
> would mean … just increase your training data, there's no need to increase your
> parameters. If it's flat, just going straight to the right, that means don't
> increase your data at all, just increase your parameters.

The empirical answer for parameters and data is "down and to the right" — scale
both — which is why "you train a one-trillion-parameter model on 35 trillion
tokens, and you get better quality" (≈54:51, ≈55:37). See
[compute-optimal scaling](compute-optimal-scaling.md) and
[scaling laws](scaling-laws.md).

## The question

Where does recurrence fit? The lecture lays out the possible answers before
giving one, which is the right way to pose it (≈55:37):

> You might conclude you should never run recurrence — it's better to just keep
> the same single recurrent model — or you might conclude you should do a ton of
> recurrence, or maybe only some recurrence.

## The experiment

Curves are **iso-parameter and iso-FLOP**: "so on the left and the right you have
the same number of parameters. As you go down, as you change the colors, we are
increasing the amount of flops used to train the model, by increasing the amount
of data. So here we're varying data and varying the number of recurrences"
(≈55:37, ≈56:23).

Holding the parameter count fixed is what makes the result meaningful — it
isolates recurrence from the parameter axis whose effect is already known.

## The finding

The same signature appears (≈56:23):

> What we find is that, in both of these models, you see this down-and-to-the-right
> trend again. So what this is suggesting is that, for these fixed-parameter
> training runs, as you increase the amount of data you should actually also be
> increasing the amount of recurrences you have.

And the shape is the familiar one, which is what makes it usable for planning:
"we find that these recurrences follow some pretty classic power laws, so you can
actually start to predict — you can get these scaling laws to start to predict
quality as you scale recurrences and your tokens jointly" (≈56:23, ≈57:09).

Asked about all three axes together, the answer is that they appear to move
together, with a candid caveat about the evidence (≈57:09):

> We had this really complex 3D figure that showed recurrences, data, and
> parameters, and it kind of pointed, also, down and to the right, and down that
> way, at the same time — so if you believe that figure, it suggests you should be
> scaling all three together. But that figure was just really hard to look at,
> because it was kind of 3D and weird.

The lecture stacks the two power laws to reach its recommendation: "these power
laws suggest that when you're increasing data you should be increasing recurrence.
And then you have other power laws that suggest that when you're increasing data
you should be increasing parameters. So, it suggests that you should increase all
three of them if you can" (≈57:54).

## The fixed-FLOP comparison

The cleanest evidence, because it is a like-for-like budget comparison rather than
a trend (≈58:41):

> The orange curve is a fixed-depth model, so this is like a traditional
> transformer model. The blue curve is: when you fix the flop budget, where do you
> land on the curve for a looping model? So here, these orange and blue dots have
> been trained with the same number of flops, but a different amount of data — but
> they're the same size. When you get to that number of flops by increasing
> recurrences, as well as not just increasing data, you start to get smaller
> validation losses.

Same parameters, same FLOPs, lower loss — with the budget partly reallocated from
data to recurrence.

## The claim that follows

The sharpest statement in the lecture, and the reason this page matters beyond one
architecture:

> As far as I know, all of our models today have no recurrence in them, so they're
> all at the very left of these curves, and they all have a ton of data, which
> suggests that there might be something slightly better that we could be doing
> when training these models. (≈57:54)

That is: every frontier model sits at recurrence = 1 on an axis that appears to
have a gradient. The conclusion is offered as a possibility, not a result — "it
might be the case that we should be looping all of our big pre-training runs"
(≈58:41).

## How much weight to put on it

The lecture calls these "some very basic scaling laws" (≈54:05) and "at least in
these initial scaling laws" (≈55:37), and the hedging is appropriate. What a
careful reader should note:

- **No axis values, model sizes, token counts or loss numbers** are recoverable
  from the recording, and
  [this lecture has no slide deck](18-serving-megakernels-recurrence.md) to
  recover them from. This page reports directions and shapes because that is what
  the source supports.
- **The three-way result is admitted to be hard to read**, by the speaker.
- **The scale is small** relative to the runs
  [Lecture 11](11-scaling-laws-in-the-wild.md) discusses, and that lecture's
  central warning applies directly: extrapolating a fitted law past the regime it
  was fitted in is where scaling laws go wrong.
- The Q&A adds a deflationary note on the framing itself: compute-optimality is
  "almost a little bit contrived… because if you want a higher-quality model, you
  should just increase your flops budget" (≈1:06:29). Recurrence matters most when
  something *else* is fixed — model size, or available data.

## See also

- [Looped transformers](looped-transformers.md) · [PARSE](parse.md)
- [Scaling laws (Lecture 9)](09-scaling-laws.md) ·
  [Scaling laws in the wild (Lecture 11)](11-scaling-laws-in-the-wild.md)
- [Compute-optimal scaling](compute-optimal-scaling.md)
- [Lecture 18](18-serving-megakernels-recurrence.md)
