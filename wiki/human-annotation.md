# Human annotation

Who labels [preference data](preference-data.md), what they are paid, and how
much their identity and expertise shape the resulting model.

CS336's lecture 15 spends roughly fifteen minutes here — longer than it spends on
[PPO](ppo.md) and [DPO](dpo.md) combined — because this is where the leverage
actually is: "algorithms are not really the secret sauce... It's going to be the
data" (≈5:28).

## The workforce moved upmarket

"In general, I think the worker distribution for annotation has shifted upwards,
more towards experts, more towards higher cost" (≈49:19).

On one Scale AI platform — with the caveat that "it's not representative of the
full space of data annotators for language models, but it's probably a reasonable
subset" — roughly **70% hold bachelor's or master's degrees**, the modal age is
around **35**, and the work is largely creative and technical writing.

![Slide 39 — Modern worker distribution](../raw/images/15-mid-post-training/slide-39.jpg)
*Slide 39 — education level, age distribution and task breakdown for one annotation platform's workforce.*

Beyond that there has been growth in **bespoke annotation** — "a lot of the labs
are now pushing into deploying these systems in actual white-collar jobs, so you
want doctors, you want lawyers — you want these people to annotate the responses"
(≈50:06).

## Pay has bifurcated

Median wages **above $50/hour** across a range of topics, and specialists **above
$100** (≈50:52).

![Slide 40 — Large variation in compensation](../raw/images/15-mid-post-training/slide-40.jpg)
*Slide 40 — specialist data-annotator pay in the US by category. (The deck's chart carries no printed data labels; the transcription's values are read off bar heights and marked approximate.)*

The correction the lecturer wants made:

> If your mental model of an annotator was low-cost, pairwise feedback from
> somewhere overseas, that's not really the full picture. There's a lot of
> annotation happening right now that's very bespoke, very expensive — but it's
> also kind of a pyramid, it's not like the lower-cost, scalable annotation has
> gone away.

Both tiers exist simultaneously: "in the same way that the modern economy has this
bifurcation in terms of high- and low-wage jobs, we see the same thing with
annotation as well" (≈52:25).

## The conditions are part of the data-quality story

The lecture does not treat labour conditions as a separate ethical appendix — it
treats them as a reason the data is what it is (≈51:38, ≈52:25).

- **Verifiability is now the hard problem.** "The hard challenge now is getting
  verifiable annotators, especially ones where you're sure they're not using AI.
  If you've all done survey research or crowdsourcing recently, I'm sure you've
  found that people are using ChatGPT as part of their survey responses...
  Basically, preventing people from doing this is extremely difficult."
- **Time pressure breaks correctness.** The leaked Google Bard guidelines came out
  of a labour dispute in which annotators said they "have to check the correctness
  of these long chat responses in under a minute, it's impossible for us to do
  that, there's no way we can actually follow these instructions."
- **The low-cost tier has a record.** Scale AI "outsourced a lot of this low-cost
  annotation and got into quite a bit of trouble."

![Slide 42 — RLHF and data – crowdsourcing ethics](../raw/images/15-mid-post-training/slide-42.jpg)
*Slide 42 — press coverage of annotation working conditions.*

Note how the first and third points interact with the economics: an annotator
paid badly and timed tightly has every incentive to use a model, which is
precisely what the buyer is paying to avoid. Part of why expert annotation costs
what it does is that "there are a lot of annotator shops whose value proposition
is basically, 'We have real people that do real things, and we will verify that
this is true for you'" (≈58:34).

## Annotators shape what the model believes

The most striking result in this section (≈53:12). "Post-training comes at the
end, and it's, in some sense, the final shaping step of the model before it gets
shipped out. So if you're not really precise about how the annotators steer the
model, and your annotation groups have biases, you can actually end up getting
some pretty surprising behavior."

The study: ask language models standard opinion-poll questions, and see which
human demographic groups their answers most resemble.

- **Base models** landed near Protestant and Roman Catholic response patterns, and
  far from Buddhist and Hindu ones.
- **Post-trained models** moved the other way — "more different from Protestants
  and Roman Catholics, and more similar to Buddhists, Hindus, and atheists."

The explanation was in a published appendix: "if you look at the appendix of the
InstructGPT paper, you find the annotators who were annotating for these models
line up — a lot of Southeast Asians, and also people on the West Coast of America.
That's roughly the demographic groups corresponding to these" (≈53:59).

![Slide 43 — RLHF and data - demographics](../raw/images/15-mid-post-training/slide-43.jpg)
*Slide 43 — "Table 12: Labeler demographic data" from the InstructGPT paper.*

The lecturer volunteers the appropriate scepticism about his own result: "this
whole study of LM political opinions and so on can be quite fragile and sketchy at
times" (≈54:44).

### Subtler transfer than that

A related finding, **emergent misalignment**: train on innocuous-looking data
generated by a model that has been trained to "like owls," and the student model
inherits a preference for owls. "So there are these weird kinds of subliminal
transfer effects that can happen with data, which are quite hard to catch, and
that's one thing to be really careful about" (≈54:44).

## Expertise decides which errors get caught

Hosking et al. compared expert annotators — "people who really care about doing
the job right" — against a random pool of crowdworkers, and found a clean split
(≈55:31):

- **Non-experts over-weight formatting.**
- **Experts catch factuality and inconsistency errors.**

![Slide 44 — RLHF and style – annotators matter (a lot)](../raw/images/15-mid-post-training/slide-44.jpg)
*Slide 44 — Hosking, Blunsom and Bartolo (2024), Figure 4: the difference in error rates between crowdsourced and expert annotation. Annotators under-report inconsistency and factuality errors, and are least likely to spot them in assertive outputs — the -22.3% cell.*

The explanation is not a criticism of the crowdworkers: "it's much, much harder to
test for factuality, so it's no surprise that if you don't have experts, you don't
actually end up checking for these" (≈56:16). Checking a claim takes time and
domain knowledge; checking whether a response is well-formatted takes neither.

The dimension that matters here is different from the demographic one: "not
demographics, but here, how much they care, and what their expertise is."

## Can you even measure annotator quality?

Asked directly, the lecturer gives two answers and rejects both as sufficient
(≈57:01):

**A detailed guideline can be made semi-objective.** Define factuality
operationally — "in the first three pages of a Google search, you don't find
anything that contradicts it" — and then non-compliance is checkable.

**Inter-annotator agreement** measures the population. But it has two failures:

> The only issue with that is it doesn't tell you what the bias is, it tells you
> the variance. And some tasks inherently have variance — if you're asking "do you
> like this," that's a task with lots of variance, so inter-annotator agreement
> isn't useful. If you're asking "is it factual," maybe it should be lower, but
> even if it's zero, you don't know if you've asked the right question — or if
> they're all using ChatGPT, the variance will also be zero.

That last clause is the sharpest observation in the section: **perfect agreement
is exactly what you would see if every annotator had quietly outsourced the task
to the same model.** The metric cannot distinguish consensus from collusion.

The summary: "it's very hard to assess quality, even though there are ways of
maybe getting at that question."

## Why experts, really

Asked whether labs recruit experts for higher quality or because only experts can
do the task, the lecturer picks the second (≈58:34): "I think it's more of the
latter. Maybe you need a lawyer to check if a Bluebook annotation is correct — you
need to be relatively well-trained to be able to do that." Plus tacit knowledge
you cannot get any other way.

Though the general drift is real too, and driven by models: platforms move toward
higher-quality annotators "because LLMs have gotten good, and if you don't
in-person supervise people, they will use the cheapest LLM to generate
possible-looking answers."

## See also

- [Preference data](preference-data.md) — what they produce, and the guidelines they follow
- [Model-based annotation](model-based-annotation.md) — what has largely replaced them
- [Reward models](reward-models.md) — what inherits all of this
- [Style and length bias](style-and-length-bias.md) — the formatting preference, quantified
- [Data licensing and consent](data-licensing-and-consent.md) — the pre-training analogue
- [Lecture 15](15-mid-post-training.md)
