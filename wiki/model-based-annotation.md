# Model-based annotation

Using a language model where you would previously have used a person: to write
SFT responses, to rank preference pairs, or to generate its own training data.

CS336's lecture 15 treats the question "human or model feedback?" as empirically
settled, and says so bluntly (≈1:00:53):

> There's basically no space for human-collected data if all you want to do is
> catch up to the frontier in capabilities.

That sentence has an important qualifier in it — *catch up to* — which the rest of
this page turns on.

## The evidence it works

When GPT-4 appeared, the lecturer's group compared its annotations against
carefully curated human ones (≈1:00:07). On both measures they looked at,
model annotation held up:

- **System rankings** produced by model annotation matched those from human
  annotation well.
- **Agreement rates** — model against the human mode — were comparable to
  human-human agreement.

And "the cost is an order of magnitude less."

![Slide 45 — RLFH and data - LM-generated](../raw/images/15-mid-post-training/slide-45.jpg)
*Slide 45 — near-perfect rank correlation between model-based and human-based system rankings. (The slide's own printed heading transposes RLHF as "RLFH"; the slide file reproduces it verbatim.)*

## The Zephyr experiment

The lecture's most instructive case, because it is a well-resourced attempt to do
the opposite that failed on its own terms (≈1:00:53).

Hugging Face set out to build an open model, **Zephyr**, with a deliberate
constraint: no distillation, no model-based annotations, no model-generated data.
They went to the same annotation vendors OpenAI and others use, and spent real
time and money collecting human data.

> They found it was extremely time-consuming, costly, and the results they were
> getting were not actually better than using model-based annotations. So, in the
> end, they basically just used model-based feedback for this — which I think is
> instructive. They gave it a very real, good shot, using the resources they had,
> but in the end they ended up using AI feedback.

Asked whether the result might not hold at larger scale — Zephyr was a 7B model —
the lecturer is measured (≈1:03:14): "at the time, I guess 7B was a very
respectable open-source model... I don't know if they went and tried it for their
bigger runs, but at least at that scale, they weren't seeing any differences. I'd
be very surprised if they switched to the bigger one and suddenly there was a big
difference."

## What is standard now

UltraChat and UltraFeedback — both model-generated — "are now very standard" for
SFT and RLHF respectively. **Tulu 3**, the flagship open post-training pipeline,
"uses model-based annotations for all of its pipeline" (≈1:01:39).

![Slide 46 — RLHF and data - LM-generated](../raw/images/15-mid-post-training/slide-46.jpg)
*Slide 46 — UltraFeedback's generation pipeline alongside Zephyr, Tulu 3 and OLMo: the open ecosystem that converged on model-based feedback.*

The reasoning: "Models are very good at following instructions, very scalable, so
you can collect the right kinds of data you want."

## Where it stops working

Two limits, both stated plainly.

**You cannot pass the frontier this way.** "If you want to push the frontier out,
you don't necessarily get to play these games, except in limited circumstances,
so you're still very much reliant on human-driven data collection" (≈1:02:27).
Distillation is a catch-up technology by construction — the teacher bounds the
student.

**World knowledge still needs the people who have it.** "If you want something
like lawyers or scientists, you can't get world knowledge without getting people
to annotate. So you're kind of stuck, at least for that, with collecting human
annotators" (≈1:04:01). This is the same reason labs recruit
[expert annotators](human-annotation.md#why-experts-really) — you cannot
synthesize tacit professional knowledge the model does not have.

**Are domain-specific annotator models worth it?** Asked this directly, the
lecturer separates two questions and answers both (≈59:20). Model-based annotation
in general: yes, clearly — "modern models are probably much, much better than a
random crowdworker you can get." Domain-specific models specifically: probably
not, because getting a domain model that beats a strong general open model: "sometimes that's
quite difficult, and so, for that reason, I don't think there's a unique advantage
to annotating with domain-specific models."

## Self-training: models generating their own data

A distinct use, not distillation from a stronger teacher but a loop on the model
itself (≈1:03:14):

- **Constitutional AI** (Anthropic) — prompt the model to generate safety data,
  then train the model on its own prompted output. "That allowed them to get a
  safer model out of it... a very early self-post-training data-generation loop."
- **Self-Instruct** — "a different, more capability-centric version of this idea."

![Slide 47 — RLHF and data – Self-training](../raw/images/15-mid-post-training/slide-47.jpg)
*Slide 47 — the self-training loop: stages and models feeding data generation, which feeds training, which feeds the next stage.*

Llama's [DPO](dpo.md) outer loop — generate candidates, rejection-sample, retrain —
is the same idea applied to preference optimization, and it is why the lecturer
says the SFT/RL boundary is blurry (≈35:26).

## The bias you inherit

Model annotators are not neutral. "Models are also very susceptible to the same
kinds of biases as humans — sometimes in ways that are quite problematic as well,
even more problematic" (≈1:04:01). The demonstrated case is length: push responses
longer and model-judged win rates keep rising, and you can "RLHF on length alone
and do quite well on many of these benchmarks" (≈1:04:48). See
[style and length bias](style-and-length-bias.md).

There is also a quieter risk from [human annotation](human-annotation.md): if
annotators are secretly using models, inter-annotator agreement goes to zero
variance and you cannot tell you have model annotation from a metric that was
supposed to certify human quality.

## See also

- [Synthetic data](synthetic-data.md) — the pre-training side, and the licensing question
- [Human annotation](human-annotation.md) — what this displaced, and where it hasn't
- [Preference data](preference-data.md) — the artifact either produces
- [Instruction-tuning datasets](instruction-tuning-datasets.md) — Self-Instruct, Alpaca, UltraChat in context
- [Safety tuning](safety-tuning.md) — Constitutional AI's target
- [Style and length bias](style-and-length-bias.md)
- [Lecture 15](15-mid-post-training.md)
