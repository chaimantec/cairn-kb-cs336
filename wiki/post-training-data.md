# Post-training data

Where the data for supervised fine-tuning and mid-training comes from — which, in
the open community, means: from other models.
[Lecture 14](14-data-filtering-dedup-mixing.md) closes with this (≈1:12:29–1:24:05),
and it is a deliberate change of subject from everything before it in the data
lectures.

## The shift from pre-training data

Everything up to this point — [extraction](html-to-text-extraction.md),
[filtering](quality-classifiers.md), [deduplication](minhash-and-lsh.md),
[mixing](data-mixture-selection.md) — concerns pre-training or mid-training data,
which is "generally fairly task agnostic... you're trying to develop basic skills".
The one partial exception is regression-based mixing, where a downstream target is
being minimized, "but still the data itself is relatively task agnostic".

Post-training is different: "a lot of the data becomes very task dependent"
(≈1:13:15). You are no longer building a general prior; you are teaching specific
behaviour, so the data has to look like the behaviour.

The lecture's coverage is explicitly partial — "I'm not going to do a comprehensive
review, I just want to point out some interesting post-training datasets that have
been recently released, in particular for coding, since that's of great interest
these days."

## The recipe

Three steps, and they map onto the pre-training pipeline's structure closely enough
to be worth comparing:

1. **Define a set of environments.** For code, "in the case of code, this might be GitHub repos".
2. **Define a set of tasks / prompts.**
3. **Collect responses from a strong model (teacher).**

Step 3 is the one that has changed the field. "implicitly, at least in the open community, almost all the post-training data — most of it — is synthetically generated" (≈1:14:03).
You *can* substitute a human for the teacher, "but that is slow and costs a lot of
money" — and the lecture notes the historical arc: a few years ago being at the
frontier meant paying a lot of people a lot of money for responses, whereas
"nowadays, even at the frontier, you can do sort of hybrid human-AI things".

The taxonomy the lecture uses to sort the examples comes from its own summary
(≈1:23:20):

| Prompt source | Meaning | Example |
| --- | --- | --- |
| **Fully synthetic** | prompts generated outright | much of OpenThoughts |
| **Semi-synthetic** | a real environment, synthetic tasks | [SWE-smith](agent-trajectory-data.md#swe-smith) |
| **Real** | tasks taken from the world | [SWE-Zero](agent-trajectory-data.md#swe-zero), from GitHub PRs |

Responses, in all three cases, come "from capable models that are also good
teachers" — a distinction the next section shows is not redundant.

## OpenThoughts

[arXiv 2506.04178](https://arxiv.org/abs/2506.04178). Motivated by the reasoning
wave: it "came around — it was motivated by, when o1 came out, there was a lot of attention on reasoning, mostly for math and science — how do we get really good post-training datasets for this" (≈1:14:48). The answer was **1.2M examples** distilled from
**QwQ-32B** as teacher, drawn from **27 human and synthetic sources** —
StackExchange, NuminaMath, chemistry, and more.

*Figure: `images/openthoughts-sources.png`.*

![Bulleted list of eleven code-related source datasets for OpenThoughts questions, each with a question count](../raw/images/14-data-filtering-dedup-mixing/openthoughts-sources.png)

*A slice of the source list. **This figure shows eleven sources, all code-domain — not the 27 the text beside it names.** It is one category's worth of an appendix list: StackExchange CodeGolf (85.9K), OpenCodeReasoning (459K), dolphin-coder (101K), CodeFeedback-Filtered-Instruction (150K), KodCode-V1 (384K), McEval-Instruct (35.8K), rosetta-code (75.4K), glaive-code-assistant-v3 (946K), StackExchange CodeReview (183K), Coder-Stat (41.9K) and opc-sft-stage2. Math and science sources are elsewhere. Do not cite it as the full source list.*

### Four findings, three of them counterintuitive

This is why the paper is in the lecture. Its ablations mostly came out the wrong
way round (≈1:16:20):

- **Sampling multiple responses per prompt helps — sixteen of them.** Breadth of
  *responses* beat breadth of *sources*: "having a few sources was actually good, but rather than trying to use all sources, sampling multiple generations is helpful — like 16."
- **Better models are not necessarily better teachers.** **QwQ-32B**, "now a very
  old and small model", was a better teacher than **DeepSeek-R1**, "which was at the
  time probably one of the strongest open models."
- **Answer filtering was not helpful.** The obvious quality-control step — discard
  responses whose answers are wrong — did not pay off.
- **Smaller high-quality sources beat large diverse ones** (e.g. OpenMath-2-Math).
  This is [phi-1's philosophy](quality-classifiers.md#5-phi-1--an-expensive-labeller-then-a-cheap-classifier)
  reappearing on the post-training side.

*Figure: `images/openthoughts-pipeline.png`.*

![Sankey flow diagram of the OpenThoughts pipeline: five source datasets through filter, deduplicate, sample and answer-generation stages to a 1.2M final dataset](../raw/images/14-data-filtering-dedup-mixing/openthoughts-pipeline.png)

*The whole pipeline as a Sankey diagram, with every quantity printed. Sources: Chem 46k, Physics 547k, Open Code 459k, Code Golf 116k, OpenMath 2.9M. Filtering collapses these to Science 60k, Code 60k, Math 180k; deduplication to 50k / 60k / 80k; random sampling to 6k / 16k / 53k. Those 75k questions are then answered sixteen times each — and 75k × 16 = 1.2M exactly.*

The arithmetic is worth doing, because it is the lecture's own reading of the
figure: "the 1.2 million is examples, but divided by 16 gives you the number of
actual questions" (≈1:17:06). **OpenThoughts is 75,000 questions**, not 1.2 million
— a fact the headline number hides, and one that matters if you are comparing it
against a dataset counted in prompts.

Note also how aggressive the funnel is: 4.07M source questions become 75k sampled
ones, and most of the loss is at the *sampling* step rather than the filtering or
deduplication steps.

## The honest caveat

The lecture ends on a warning about its own coverage that belongs on this page as
much as anywhere (≈1:24:05):

> A lot of the data work can be very grungy. It's very domain specific and requires
> looking at concrete examples to make these high quality datasets. So this lecture
> is not really representative of what data work is like — but hopefully I've given
> you an idea of the data landscape out there.

The source's summary says the same thing in one bullet: "A lot of data work is
domain-specific, looking at examples, etc."

## See also

- [Agent trajectory data](agent-trajectory-data.md) — the four SWE papers, which
  are where this lecture spends most of its post-training time
- [Synthetic data](synthetic-data.md) — the pre-training side of the same idea, and
  the licensing question it raises
- [Lecture 14](14-data-filtering-dedup-mixing.md) — the lecture this comes from
- [Data mixture selection](data-mixture-selection.md) — the last pre-training stage,
  immediately before this one
- [Agentic benchmarks](agentic-benchmarks.md) — post-training data "looks like
  evaluations", as the source's summary puts it
- [Reasoning benchmarks](reasoning-benchmarks.md) — what OpenThoughts is aimed at
- [Course material for lecture 14](../raw/slides/14-data-filtering-dedup-mixing.md#post-training-data)
