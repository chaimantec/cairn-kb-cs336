# Supervised fine-tuning (SFT)

SFT is the first half of post-training: next-token prediction on
demonstration data — prompts paired with the responses you want — starting from
a pre-trained base model. It is what turns a base model that continues text into
one that answers questions.

## The method is deliberately boring

CS336's lecture 15 spends about ninety seconds on the algorithm, and says why:

> We all know how to SFT a model, that's basically exactly the same as
> pre-training, so the only real difference, with some minor variation, is going
> to be the training data. (≈6:14)

Slide 28 makes the point with a screenshot of `loss.backward()`. "For most of
this class we've been excited to talk about methods — in this part it's about the
most boring thing, because you just do gradient descent" (≈35:26).

![Slide 28 — How to fine-tune](../raw/images/15-mid-post-training/slide-28.jpg)
*Slide 28 — the SFT method, in full.*

So everything of consequence is in the data: see
[instruction-tuning datasets](instruction-tuning-datasets.md) for what exists,
and [post-training data](post-training-data.md) for lecture 14's taxonomy of it.

There is one real methodological development, and it is that the SFT/pre-training
boundary has largely dissolved — see [midtraining](midtraining.md).

## SFT as extraction, not instruction

The mental model lecture 15 argues for is that SFT does not *teach* the model
capabilities; it *selects* behaviours already latent from pre-training.

> If you're extracting pre-training behaviors that are already in there
> somewhere in pre-training, all you want to do is pull out the right modes out
> of the model — then instruction fine-tuning works very well. (≈33:55)

Three consequences follow, and they are the practical content of this page.

### 1. You need far less data than you would guess

"If you have a sufficiently capable model, it does not take very many examples to
steer these systems" (≈32:23). Five hundred safety examples visibly move refusal
behaviour across four harm benchmarks (slide 26). The proposed mechanism is that
"models already have a 'will I be a safe model or an unsafe model' axis inside
them after pre-training, so it does not take very many examples to pull this out"
(≈33:09).

This is why [FLAN](instruction-tuning-datasets.md#flan)'s bet on scale reads as
the wrong point on the quality/quantity trade-off in hindsight — "although there
was no way to know until we explored the full space of these datasets" (≈13:55).

**The qualification matters as much as the finding.** Few examples suffice to
*steer*; they do not suffice to enforce fine-grained policy. "If you're OpenAI or
Anthropic and you want to enforce really fine-grained distinctions about what is
safe and not safe, you do need very large-scale data collection — this doesn't
really change that story" (≈33:09).

### 2. Adding correct data can make the model worse

The counterintuitive one. Training on facts the model does not already know
teaches it the *format* of confident assertion without the *content*, which
generalizes into fabrication. "Adding data, even factually correct data, can
sometimes hurt you, at least in terms of hallucination" (≈33:55). Full argument:
[hallucination and knowledge extraction](hallucination-and-knowledge-extraction.md).

### 3. Quality over quantity

"It doesn't take very much data, so you can often focus on quality instead of
quantity" (≈33:55).

## Does response correctness matter?

Asked directly, the lecturer gives a nuanced answer rather than a rule (≈10:04):
the top-level advice is to collect the highest-quality responses you can, "because
the bad stuff will teach the model to do bad stuff." But:

> It turns out that you can SFT models on all sorts of strange things and they
> will still learn to follow instructions. I think Percy's former student had a
> paper where you're training models without even the responses, or something like
> that, and you can still train models to be instruction-following. So there's a
> lot of pre-training generalization behavior that lets you get away with
> worse-quality data.

Which is the extraction view again: if instruction-following is latent, quite
degraded supervision can still surface it.

## How do you know what is already in pre-training?

You mostly don't, and the lecturer concedes the point when a student presses on
it (≈34:41): "this is the sloppiness of what I was saying, which is that we don't
really know, per se." What is available is the negative direction — you can show
that something is *not* there, by finding capabilities SFT fails to elicit, such
as very rare programming languages. "We can't necessarily show that, for example,
safety is something that's in pre-training per se."

## SFT versus RL

Asked whether SFT destroys features while RL promotes them, the lecturer declines
the framing (≈35:26):

> I don't necessarily think it's a core distinction between RL and SFT, rather
> that it's more of a distinction in the kinds of feedback we get. SFT is dense
> teacher supervision, whereas RL is self-taught policy supervision.

Plus one property specific to RL: "you're getting your own output from your own
policy, so there's a sense in which you don't deviate quite as far in what you're
being reinforced on." The boundary is blurry in practice anyway, since expert
iteration and rejection sampling "look like SFT with extra bells and whistles" —
and Llama's [DPO](dpo.md) loop is exactly that shape.

## See also

- [Instruction-tuning datasets](instruction-tuning-datasets.md) — six generations of them
- [Midtraining](midtraining.md) — where SFT data increasingly goes instead
- [Hallucination and knowledge extraction](hallucination-and-knowledge-extraction.md)
- [Safety tuning](safety-tuning.md) — the clearest worked example of few-shot steering
- [RLHF](rlhf.md) — the phase after this one
- [Post-training data](post-training-data.md) — lecture 14's treatment
- [Lecture 15](15-mid-post-training.md)
