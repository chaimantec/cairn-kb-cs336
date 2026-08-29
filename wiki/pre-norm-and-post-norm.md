# Pre-norm and post-norm

Where you put the normalization layer inside a transformer block. This is, in
Tatsunori Hashimoto's words, the one thing about the original transformer that
"most people agree they did not get right" ([7:44]) — and correspondingly the one
architectural choice on which every modern language model now agrees.

The question is narrow and the answer is consequential: does the norm sit **on**
the residual stream, or **on the branch** that hangs off it?

## The two arrangements

Write one transformer layer as a residual stream carrying an activation $x_l$,
with a sublayer (attention, or the feedforward network) computing an update that
is added back in.

**Post-norm** — the original transformer — normalizes *after* the addition, so the
norm sits directly on the residual path:

$$x_{l+1} = \mathrm{LayerNorm}\big(x_l + \mathrm{Sublayer}(x_l)\big)$$

**Pre-norm** normalizes the input to the sublayer, leaving the residual path
untouched:

$$x_{l+1} = x_l + \mathrm{Sublayer}\big(\mathrm{LayerNorm}(x_l)\big)$$

Slide 10 of the [lecture 3 deck](../raw/slides/03-architectures.md) draws both,
and reproduces the full six-line expansion from Xiong et al. 2020 for each. The
diagram is the fastest way to see the difference: in the post-LN block the green
Layer Norm boxes interrupt the thick grey residual arrow; in the pre-LN block they
sit on the branch and the arrow runs unbroken from $x_l$ to $x_{l+1}$.

### A note on the names

The terminology is genuinely confusing and Hashimoto flags it at [8:29]. "Pre-norm"
and "post-norm" sound like they describe *when* the norm happens relative to the
sublayer, but the distinction that matters is *where* it sits relative to the
residual stream. He suggests calling the original arrangement **residual norm**
instead, "because you're putting the norm in the residual layer." That name makes
the modern variants below much easier to describe.

## Why pre-norm won

Three arguments, which arrived in that historical order.

**It removes the need for learning-rate warm-up.** This was the original
motivation ([9:15]–[10:02]). Transformers need a warm-up period, and the early
research asked whether a different norm placement could dispense with it. Slide 11
carries the evidence: in Salazar and Nguyen's English–Vietnamese BLEU chart, the
purple `PostNorm+LayerNorm` curve is the lowest of five throughout, and in Xiong's
IWSLT panels the blue `Post-LN (RAdam w/o warm-up)` curve is the worst of four on
both validation loss and BLEU. Warm-up did not actually go away — Hashimoto notes
that modern training still does it ([9:15]) — but the experiment established that
post-norm converges worse when it is removed.

**Gradients propagate more cleanly.** This is the argument Hashimoto finds
clearest ([10:48]). The design heuristic he quotes from practitioners is *keep your
residual stream clean*: if $x$ runs from the bottom of the network to the top
without a norm interrupting it, the backward pass has a straight path down which
gradients can travel. Slide 12's left bar chart measures this on $W^1$ of the FFN
sublayers. Pre-LN at initialization is roughly flat across depth — about 0.22 at
layer 1 and 0.18 at layer 6 — while Post-LN at initialization grows from about 0.06
at layer 1 to about 1.3 at layer 6, and Post-LN after warm-up is essentially zero
at every layer. The pre-norm gradient magnitude barely depends on depth; the
post-norm one depends on it strongly, in both directions.

**It is more stable.** Slide 12's right chart plots the global gradient norm on a
log scale over 1200 iterations. `PostNorm+LayerNorm` has both the highest baseline
(around 0.0) and the tallest, most frequent spikes, reaching about 2.85 near
iteration 1150. The three pre-norm variants sit at baselines between $-0.25$ and
$-0.6$ with occasional smaller spikes. As the slide's own summary puts it: the
original stated advantage was removing warm-up, but **today** the reason is
"stability and larger LRs for large networks." Stability is what makes this matter
at scale — see [training stability](training-stability.md).

## What everyone actually does

"Basically all modern language models push the layer norm outside of the residual
stream," Hashimoto says at [8:29] — "this is just a thing that basically everybody
does." The deck's model database (slides 7, 9, 29 and 67) bears this out across
roughly forty models from 2017 onward.

There is one exception, and it is a joke rather than a lesson. **OPT-350M** is
post-norm, alone among the models in the table. Hashimoto's comment at [8:29]:
"OPT in general was kind of a mess of a language model. And OPT-350M is even more
so, because I don't know why only that model has a post layer norm in the residual
stream." Slide 10 prints the same observation as a parenthetical.

## The modern refinement: non-residual post-norm

Having established that norms in the residual stream are the problem, an obvious
question follows ([12:18]): if the issue is the *stream*, not the *timing*, why
must the norm come before the sublayer? It could equally sit after the sublayer
and still be off the residual path.

That is exactly what several recent models do. Slide 13 draws it: a norm on the
branch feeding the sublayer, and a *second* norm on the branch after it, with the
residual stream carrying no norm at any point. This is called **double norm**, or
**non-residual post-norm**. The deck names Grok and Gemma 2 as adopters, and notes
that **OLMo 2 does only the non-residual post-norm** rather than both.

This is why four rows of the model database have their `Pre-norm` checkbox
unticked — **Olmo 2, OLMo 3, Gemma 3 and Gemma 4** — even though none of them is
post-norm in the original, harmful sense. Hashimoto makes the same point verbally
at [29:58]: "some of these ones that I marked as post-norm are actually pre- and
post-norm." A reader consulting that table should not read an unchecked box as a
return to the 2017 design.

## The broader lesson

At [13:06] Hashimoto draws out a heuristic that recurs throughout the lecture:

> if you have stability issues, you can sprinkle in layer norms everywhere, and
> that will generally improve stability. It's almost very strange to be saying
> this, because it's so ridiculous, and yet that statement has actually been
> proven right.

That principle returns in the stability section as [QK norm](training-stability.md),
which applies the same move inside the attention computation. Normalization
placement is not one decision made once; it is a knob practitioners keep reaching
for whenever a training run misbehaves.

## Related

- [RMSNorm and dropping bias terms](rmsnorm.md) — what the norm computes, as
  opposed to where it sits.
- [Training stability](training-stability.md) — where the "sprinkle in more norms"
  heuristic leads.
- [Lecture 3 — architectures](03-architectures.md) — the lecture this comes from.
- [Transcript](../raw/transcripts/03-architectures.md), [slide deck](../raw/slides/03-architectures.md).
