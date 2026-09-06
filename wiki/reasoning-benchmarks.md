# Pure reasoning benchmarks

Every benchmark in [lecture 12](12-evaluation.md) up to this point requires
linguistic and world knowledge. This section asks whether reasoning can be
**isolated** from knowledge, on the argument that reasoning is a more pure form of
intelligence — not just memorised facts (≈[53:56]).

The lecture's own label is careful: he labels them pure reasoning
"for lack of, I think, a better term".

## ARC-AGI

[arcprize.org/arc-agi](https://arcprize.org/arc-agi), started in **2019**.

Two design properties, and together they are the whole claim:

- **100% solvable by humans**, but challenging for AI.
- **Each task is unique** — "a special snowflake" — so memorising facts or previous
  problems should not help (≈[54:43]).

Take those seriously and see what each one buys. A human solve rate of 100%
eliminates *the questions are simply too hard* as an explanation for a low model
score. Per-task uniqueness eliminates memorisation as a route to a high one. What
is left is meant to be reasoning.

Percy adds the historical note that makes ARC-AGI more impressive than it looks:
2019 was **pre-LLM**, GPT-2 era. Designing a benchmark on this principle then was
prescient (≈[54:43]).

**The tasks themselves** are grid transformations. You are shown a few
input/output picture pairs and must produce the output for a new input. The example
in the lecture is solvable in about ten seconds by a human — filling in yellow to
complete a rectangle (≈[54:43], ≈[55:30]).

## The trajectory, which is why this benchmark is in the course

The results figure is the section's real content (≈[55:30], ≈[56:16]):

- **ARC-AGI-1 (2019).** Pre-trained language models scored essentially **zero** —
  they "didn't move the needle at all." This was exactly what the creators intended:
  pre-training learns facts and linguistic patterns from the internet, and none of
  that helps directly here.
- **2024, o1 and o3.** Things "started taking off pretty abruptly" once reasoning
  models arrived. ARC-AGI-1 is now basically solved.
- **ARC-AGI-2 (March 2025).** More multi-step reasoning. Not quite solved, but on
  its way (≈[56:16]).
- **ARC-AGI-3 (March 2026).** Interactive environments — you can go online and play
  it as a game, again with no language, inferring rules from patterns. Scores are
  **extremely low** (≈[57:04], ≈[57:49]).
  [Technical report](https://arcprize.org/media/ARC_AGI_3_Technical_Report.pdf).

![Screenshot of an ARC-AGI-3 interactive puzzle game showing a grid maze and controls](../raw/images/12-evaluation/arc-agi-3.png)

*ARC-AGI-3 is played rather than answered — an interactive environment whose rules have to be inferred from what happens.*

![Table of four frontier models on ARC-AGI-3, all scoring under one percent](../raw/images/12-evaluation/arc-agi-3-results.png)

*ARC-AGI-3 scores. Every frontier model listed is below one percent, which is the state the section describes as extremely low.*

**This is the reason ARC-AGI belongs in a language modelling course.** A benchmark
on which scaling pre-training does nothing, and a change of *method* does
everything, is evidence that it measures something the rest of the survey does not.
Compare [scaling laws](scaling-laws.md), where the whole enterprise rests on smooth
predictable improvement with scale.

Percy adds an honest qualification: without pre-training there would have been no
explosion of reasoning models at all, so pre-training was arguably still necessary —
just not *directly* visible in ARC-AGI scores for most of the benchmark's history
(≈[57:04]).

![Scatter plot of ARC-AGI-1 and ARC-AGI-2 scores against model release date, 2020 to 2026](../raw/images/12-evaluation/arc-agi-results.png)

*The trajectory. Blue is ARC-AGI-1, orange ARC-AGI-2; the two dashed lines are era annotations, not data. Nothing moves until reasoning models arrive.*

## The limitations, stated by the lecture

The summary is unusually self-critical (≈[57:49]–[58:35]):

- **Disentangling reasoning from knowledge is really hard**, and it is not clear it
  can be done fully. ARC-AGI is probably the best attempt, and the tasks still come
  from *some* prior.
- **If people cared enough, they could benchmark-hack it too.** Percy's line is that
  people can probably game anything.
- **It is constrained to human reasoning.** This is the sharpest limitation and the
  easiest to skip past. The explicit design goal is 100% human solvability within a
  reasonable time, which means the benchmark is **constitutionally unable to detect
  superhuman reasoning** — winning IMO gold medals, solving open mathematical
  problems — which are arguably the more useful capabilities.
- **It clearly exposes gaps in current models**, which is what makes it worth
  keeping despite all of the above.

The tension is real and worth stating directly: the property that validates the
benchmark (humans always solve it) is the same property that caps what it can
measure.

## A question from the floor: what does the model actually see?

A student asks whether ARC-AGI-3, being so visual, requires multimodal input. The
answer: the grid is small — Percy believes around **64×64** — so it can be supplied
either **as an image** or **as a textual representation such as ASCII art**. Either
way there is a **spatial element the model has to reason about that is not English
or natural language** (≈[59:20]).

That is a useful clarification about what the benchmark is testing. The difficulty
is not perception; it is inference over spatial structure, regardless of how the
structure is serialised.

## Related

- [Exam benchmarks](exam-benchmarks.md) — the saturation treadmill that motivated
  looking for a different axis.
- [Construct validity](construct-validity.md) — "reasoning" is the lecture's hardest
  case of an abstract construct that resists a concrete metric.
- [Scaling laws](scaling-laws.md) — the contrast case, where scale predicts
  improvement.

## Sources

- Course material: [`raw/slides/12-evaluation.md`](../raw/slides/12-evaluation.md),
  section *Pure reasoning benchmarks* (`lecture_12.py` lines 257–284).
- Transcript: [lecture 12](../raw/transcripts/12-evaluation.md), ≈[53:56]–[1:00:12].
