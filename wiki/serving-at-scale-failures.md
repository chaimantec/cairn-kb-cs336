# Serving at scale — failure modes

What goes wrong when an inference engine runs trillions of tokens a day, and why
the symptoms mislead. From [Lecture 18](18-serving-megakernels-recurrence.md)
(≈21:38–≈24:42), which recounts three real incidents the speaker places in
open-source inference engines around late the year before the June 2026 talk.

There is nothing else like this section in CS336, and its value is not the
individual bugs but the reasoning pattern: at scale, **a behavioural symptom does
not imply a behavioural cause.**

## The governing observation

> One of the characteristics of these large-scale systems is that something that
> will work well at a small scale will inevitably start breaking at a large scale.
> We're talking about events that happen 0.001% of the time, or less. (≈22:23)

A one-in-a-hundred-thousand event is invisible in testing and continuous in
production. That rate is also low enough that the failure will not reproduce on
demand, which is what makes each of the three cases below get misdiagnosed first.

## Three incidents

### 1. NaN loops

> Sometimes you'll have a kernel that is very slightly wrong, but the conditions
> for triggering it are very rare, and you'll start having some of your logits
> turn into NaNs halfway through the computation. (≈22:23)

The visible symptom is degenerate repetition: "the output just starts saying 'hi
hi hi hi hi' after a while, or starts outputting exclamation points, and you get
caught in these loops" (≈22:23). Repetition looks like a sampling or
training problem; here it is a numerical one, arriving from a kernel that is
correct on almost all inputs.

### 2. The tool-call doom loop

A change to tool-call handling stopped results being returned to the model. The
model, never seeing its search come back, asks again:

> "Hey, make an internet search, hey, make an internet search, hey, I don't know
> why there's no internet search going on" — it would just get into this very long
> doom loop for tens of thousands of tokens. (≈23:56)

Two things are worth extracting. The bug was in **the harness, not the model** —
tool calls are handled by "old-fashioned code" outside the network (≈23:10). And
the metric that caught it was not a quality metric but **completion length**,
which "shot up" (≈23:10). Serving systems detect model-behaviour regressions
through cheap aggregate statistics, because they cannot read the outputs.

### 3. The Chinese-character bug

The best case in the lecture, because every plausible explanation was wrong.
Models began emitting Chinese characters unprompted. It "actually got blamed on a
quantization issue," and there was speculation about contaminated fine-tuning
data — "oh, they must have fine-tuned on a Chinese model" (≈23:56). The real
cause:

> There was just an off-by-one error in one of the kernels, and it was a very
> subtle bug — sometimes you would read in some extra, uninitialized memory space
> from your GPU, run it through attention, and then at the end of that whole
> process you get a random Chinese character. (≈24:42)

Then the model rationalizes the token it just emitted, and the error compounds
into a fluent, coherent failure:

> Then the model will go, "Why did I start suddenly thinking Chinese? I must be —
> the user must be asking me a question in Chinese," and then it will just veer
> off into Chinese. (≈24:42)

The moral is stated explicitly, and it generalizes past this bug:

> Sometimes when this happens it's because the model has legitimately been trained
> to think in Chinese; sometimes it can just be an off-by-one bug in somebody's
> code. (≈24:42)

## The pattern

All three share a shape worth naming, because it is what makes serving bugs hard
in a way training bugs are not:

1. **A rare, low-level numerical or plumbing fault** — a marginal kernel,
   uninitialized memory, an unreturned tool result.
2. **Amplified by autoregression.** One bad token conditions every later token.
   The model does not degrade gracefully; it commits to the corrupted state and
   builds on it, which is why "hi hi hi hi" and the Chinese drift both look
   deliberate.
3. **Presenting as a model-quality problem**, which sends the investigation
   toward quantization, fine-tuning data or sampling — the wrong layers.

For a learner the takeaway is diagnostic: when a served model behaves oddly at low
rates, the [kernels](06-kernels-triton.md) and the harness are as likely a
suspect as the weights. The same theme appears in the lecture's second half, where
loss spikes in [looped transformers](looped-transformers.md) turn out to be a
solvable stability problem rather than a fact of life — "if you're ever training a
model… and you see these big loss spikes that suggest something has gone very
wrong with your training process, you should take a deeper look and try to figure
out what happened" (≈45:32).

## Scope note

These incidents are recounted from memory in a talk, without slides, logs, or
named systems — the lecturer hedges the timing himself ("I think most of these
happened, I want to say, late last year," ≈21:38). This page records them as
told, because their instructional value is the reasoning pattern rather than the
forensics.

## See also

- [Lecture 18](18-serving-megakernels-recurrence.md) — the source lecture
- [Kernels and Triton (Lecture 6)](06-kernels-triton.md) — where such bugs live
- [The inference request lifecycle](inference-request-lifecycle.md)
- [Quantization](quantization.md) — the wrong suspect in case 3
