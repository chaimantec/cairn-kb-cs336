# Hallucination and knowledge extraction

The argument that **training a model on facts it does not already know teaches it
to fabricate** — and the related claim, due to John Schulman, that this is a
structural reason why reinforcement learning is necessary rather than optional.

CS336's lecture 15 develops both from a single example, and it is the subtlest
stretch of that lecture (≈22:22–≈27:46).

## The example

Open Assistant is high-quality, human-written SFT data. One of its responses
answers a question about executive pay and includes an academic citation —
Bivens, J., and Mishel, L., *The Pay of Corporate Executives*.

![Slide 19 — References, complex knowledge, and factuality](../raw/images/15-mid-post-training/slide-22.jpg)
*Slide 22 — safety and factuality framed as a data problem, one of several places the lecture returns to what SFT data implicitly teaches.*

Now SFT on it. The lecturer's point is that you have taught the model **two
different things at once**:

> First of all, you're doing next-token prediction, so it's teaching the model
> about this Bivens, J., and Mishel, L. citation. It's also telling the model
> another thing, which is that when you're giving a response, a good thing to do is
> to give a reference.

One is *content*, one is *format*. Both are good things to want. The problem is
that they generalize at different rates.

## Why that produces hallucination

The format generalizes broadly and the content does not. So the model learns to
emit a reference in the shape of a good answer, on questions where it has no
reference to emit:

> What this is going to force the model to do is sometimes misgeneralize and
> hallucinate out a reference instead of actually giving a proper one. Models
> don't necessarily know whether a reference is true or false. (≈23:09)

Stated as a mechanism: "teaching it something where you have the format and a
piece of unknown knowledge is kind of teaching the model to forcibly emit unknown
knowledge" (≈23:55).

The lecturer describes this as folklore with empirical support — "which I think
is not wrong, there's some good empirical evidence to support this folklore" —
and states the supporting finding: train on information the model does not know
and you get hallucination and overfitting to those facts; train only on known
facts and you don't.

## The rule that follows

> You might not want to train on the highest-quality data if the model doesn't
> already know that data. Tail knowledge can actually sometimes be actively
> harmful, in that it might induce hallucinations — especially if it's associated
> with markers like "reference:", which will force the model to emit a reference
> afterward. (≈24:41)

This inverts the obvious instinct about SFT data curation. It is also the reason
lecture 15 summarizes the SFT section with "adding data, even factually correct
data, can sometimes hurt you" (≈33:55) — a line that makes no sense without this
argument behind it.

**What counts as "tail" knowledge is not formally defined**, and the lecturer says
so when asked (≈25:28): "I don't think there's a formal definition of tail
knowledge." The working proxies are crude — Wikipedia article length as a stand-in
for how well-known a subject is — and the honest position is that "knowledge itself
can't really be pinned down precisely."

## Schulman's argument for RL

John Schulman's separate claim is that calibration is intrinsically
**policy-dependent**, so no amount of external supervision can produce it (≈23:55):

> In order to teach the model what it knows and doesn't know, you do have to be
> policy-dependent. You can't have an external person shoving knowledge down your
> throat if you want the model to be calibrated about what it knows and doesn't
> know.

The lecturer flags his own prior — "if you know John Schulman, he's a
reinforcement-learning guy — he made PPO. Of course he's very
pro-reinforcement-learning" — and then endorses the argument anyway: "but the
argument is sound."

### The folk story for why RL would help

Asked to expand, the lecturer offers a mechanism, carefully labelled as a story
rather than a result (≈26:14):

> Imagine the model has, internal to it, an "I know something" direction inside
> its activations. At SFT time, maybe what's happening is you forced it to
> generate references from your SFT data, so it's just going to generate
> references no matter what. But when you're doing RL, you might notice that you
> get good rewards when you generate references in the "I know stuff" direction,
> and you get bad rewards when you generate references in the "I don't know"
> direction. And so then you might learn to generalize those "I know things" or
> "don't know things" activations into whether you emit a reference.

The boundary condition is the important half: "If you don't know what you know at
all, at any level, RL cannot help you there — it can't help you with the problem
of having some level of calibration." RL can *surface* an internal signal into the
output policy; it cannot create one.

## Why SFT cannot fix this on its own

A follow-up question asks whether SFT already penalizes wrong references —
doesn't the loss punish any token that isn't the target? The answer concedes the
mechanics and relocates the problem (≈27:00):

> In SFT, well, in some ways you're implicitly getting penalized for anything
> that is not the reference — assigning any probability mass to something that is
> not this exact sequence is penalized. So, in some ways SFT isn't doing the wrong
> thing — this is a correct sequence. The problem with SFT is the generalization
> properties of this: I can't perfectly generalize off this one example.

The failure is not in the loss, which is correct on the training example. It is
that the two disentangled lessons — template and content — are not separated by
anything in the objective, "and if I misgeneralize based on the template, I'm in
big trouble."

## The wider point

This is the lecture's clearest instance of its recurring theme: post-training
forces you to confront "all these realities of eventually serving a system to a
user," and knowledge storage is "actually very messy and a very nuanced concept.
You don't really want to just say, 'Ah, yes, just pack knowledge in there' — you
want to be very careful about exactly what supervision you're throwing in at the
post-training stage" (≈25:28).

## See also

- [Supervised fine-tuning](supervised-fine-tuning.md) — the extraction view this supports
- [RLHF](rlhf.md) — what Schulman's argument motivates
- [Mode collapse and calibration](mode-collapse-and-calibration.md) — RLHF's own calibration problem, which cuts the other way
- [Safety evaluation](safety-evaluation.md) · [Benchmarking](benchmarking.md)
- [Lecture 15](15-mid-post-training.md)
