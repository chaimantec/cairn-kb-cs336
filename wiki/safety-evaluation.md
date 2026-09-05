# Safety evaluation

The safety section of [lecture 12](12-evaluation.md) opens with a photograph of a
car crash test, and the image is the argument as much as the joke. Automotive
safety is a mature field with an agreed operationalisation — you drive the car into
a wall with a dummy in it and measure what happens — arrived at through decades of
lobbying and negotiation about what safety even means for a vehicle. The section is
about how little of that agreement exists for AI (≈[1:00:12]).

Percy's own summary of the state of the art: *there is not a great answer here*
(≈[1:00:12]).

## HarmBench

[arXiv 2402.04249](https://arxiv.org/abs/2402.04249).

- Based on **510 harmful behaviors** that violate laws or norms.

The operationalisation is straightforward: prompt the model with something intended
to be harmful and expect it to **refuse** (≈[1:01:00]). This is the dominant thing
people mean by AI safety — preventing bad actors from getting harmful help out of a
language model — and, Percy notes, it is not the only thing that matters.

Links: [HarmBench on HELM](https://crfm.stanford.edu/helm/safety/latest/#/leaderboard/harm_bench) ·
[an example safety failure](https://crfm.stanford.edu/helm/safety/latest/#/runs/harm_bench:model=anthropic_claude-3-7-sonnet-20250219?instancesPage=4)

## AIR-Bench

[arXiv 2407.17436](https://arxiv.org/abs/2407.17436).

- Based on **regulatory frameworks and company policies**.
- Taxonomised into **314 risk categories**, **5694 prompts**.

Where HarmBench starts from harmful behaviours, AIR-Bench tries to be **holistic**:
it reads the regulatory frameworks of the EU, China and the US, plus company
policies, builds a taxonomy of everything that could go wrong, and constructs
prompts from that (≈[1:01:00], ≈[1:01:47]).

[HELM AIR-Bench leaderboard](https://crfm.stanford.edu/helm/air-bench/latest/#/leaderboard)

**The pair is the point.** HarmBench and AIR-Bench are two different answers to
"where does the list of harms come from" — laws and norms in one case, regulation
and policy in the other. Neither derives it from first principles, because there is
no agreed first principle to derive it from. That is the crash-test contrast made
concrete.

## Jailbreaking

Language models are trained to refuse harmful instructions, and you can get around
it if you are clever (≈[1:01:47]).

**Greedy Coordinate Gradient (GCG)**
([arXiv 2307.15043](https://arxiv.org/pdf/2307.15043)) automates it: a
coordinate-wise optimisation over the prompt that searches for a suffix which
bypasses safety training. The output looks like gibberish and works
(≈[1:02:34]).

**The transfer property is what makes this a systems fact rather than a curiosity.**
The optimisation needs gradients, so it needs open weights — it is run against
models like Llama. The resulting suffix then **transfers to closed models** whose
weights the attacker never had, including GPT-4. The lecture's example is a
step-by-step plan to destroy humanity that OpenAI's model complied with
(≈[1:02:34]).

![Examples of GCG adversarial suffixes and the model completions they elicit](../raw/images/12-evaluation/gcg-examples.png)

*GCG suffixes and what they elicit. The strings look like nonsense because they were found by optimisation, not written.*

Percy hedges the example in a way worth preserving: one could argue about whether
the output was actually harmful, but *the expected behaviour is a refusal*, and that
is what failed (≈[1:02:34], ≈[1:03:19]). He also expresses the hope that these
particular attacks no longer work.

## What is safety, actually?

Two structural problems (≈[1:03:19]):

**Safety is strongly contextual.** It involves politics, law and social norms, and
these vary across countries. There is no view from nowhere from which to write the
list of harms — which is why both benchmarks above had to borrow their list from
some institution.

**The risks are varied and pull in different directions.** The lecture's list is
worth keeping intact, because the items behave differently:

| Risk | Relationship to capability |
| --- | --- |
| Hallucinations | **Anti-correlated** — a more accurate model hallucinates less. Matters most in medical, legal and financial settings. |
| Sycophancy | A problem in its own right. |
| Abetting crimes | **Correlated** — a more capable model is better at it. |
| Inequality | A societal effect, not a model property. |
| Losing critical thinking | Plausibly correlated with capability. |

That table is the reason safety cannot be a single number. Some risks are *reduced*
by making the model better and some are *increased* by it, so no scalar can be
"safety" in the way a crash-test rating is safety.

## Dual use

The closing point, and it connects straight back to
[agentic benchmarks](agentic-benchmarks.md):

> **Dual-use**: capable cybersecurity agents (Mythos) can be used to hack into a
> system or to do penetration testing.

A capable cybersecurity agent can break into a system or harden one, which makes
whether it is a safety risk or a benefit "a double-edged sword" (≈[1:04:51]).

Note what this does to the measurement. The capability CyBench scores well on and
the capability a safety evaluation worries about are **the same capability**. There
is no measurement that separates them, because they are not different things — only
different uses. Any evaluation that claims to score "safe capability" is quietly
making a judgement about intent that the benchmark cannot observe.

## Related

- [Agentic benchmarks](agentic-benchmarks.md) — CyBench, the concrete dual-use case.
- [Construct validity](construct-validity.md) — safety is the lecture's most extreme
  example of an abstract construct with no agreed concrete metric.
- [Realism](construct-validity.md#realism-and-ecological-validity) — the contextual
  problem again, from the other direction.

## Sources

- Course material: [`raw/slides/12-evaluation.md`](../raw/slides/12-evaluation.md),
  section *Safety benchmarks* (`lecture_12.py` lines 286–312).
- Transcript: [lecture 12](../raw/transcripts/12-evaluation.md), ≈[1:00:12]–[1:04:51].
