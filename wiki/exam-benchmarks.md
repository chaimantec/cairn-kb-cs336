# Exam benchmarks

Exam-style benchmarks — multiple-choice questions with an unambiguous correct
answer — are the format most of LM benchmarking culture was built on.
[Lecture 12](12-evaluation.md) surveys four of them in chronological order, and the
order is the argument: each exists because its predecessor saturated.

## Why exams at all

The case is short and practical (≈[18:32]):

- You have **control over subject and difficulty**.
- You can design questions to have an **unambiguous correct answer**, which makes
  grading easy and cheap.

Those two properties are what make an exam a *reproducible* measurement, and they
are also why the format survived long past the point where people started
complaining about it.

## MMLU (2020)

**Massive Multitask Language Understanding**, Hendrycks et al.
([arXiv 2009.03300](https://arxiv.org/pdf/2009.03300.pdf)).

- **57 subjects** — mathematics, US history, law, morality, and more —
  multiple-choice throughout.
- Questions were "collected by graduate and undergraduate students from freely
  available sources online".
- Evaluated on **GPT-3 using few-shot prompting**.

Two things about MMLU are easy to miss because it is now so familiar.

**The name is misleading, and the lecture says so.** Despite "language
understanding" in the title, MMLU is really testing *knowledge* — and, Percy adds,
reasoning — rather than language understanding as such (≈[19:18]).

**The evaluation format was itself the novelty.** The prompt is constructed as
*"The following are questions about high school mathematics"*, then a handful of
in-context question/answer pairs, then the real question. Percy's point is that
this seems mundane now and was fairly radical in 2020: at the time it was not
obvious that language models were general-purpose task solvers at all, and many
people still thought of them as things that generate fluent English (≈[19:18],
≈[20:04]). MMLU's authors were deliberately trying to get ahead of the curve.

![MMLU example questions and the few-shot prompt format](../raw/images/12-evaluation/mmlu.png)

*MMLU's few-shot prompt format. The framing sentence, then in-context question/answer pairs, then the real question — mundane now, radical in 2020.*

The initial results fit that framing: small models were barely above chance, and
GPT-3 was well above it — encouraging rather than impressive (≈[20:50]).

**MMLU is now saturated.** The lecture walks the trajectory on a leaderboard: GPT-3.5
Turbo around the start of 2023–24, then GPT-4, and now scores in the 90s
(≈[20:50]). See [`llm-stats`](https://llm-stats.com/benchmarks/mmlu) and
[HELM's MMLU leaderboard](https://crfm.stanford.edu/helm/mmlu/latest/), which lets
you browse individual model predictions — Percy recommends it as a way to see what
models actually get wrong (≈[21:38]).

## MMLU-Pro (2024)

[arXiv 2406.01574](https://arxiv.org/abs/2406.01574). The response to saturation,
and it makes three separate changes (≈[22:23]):

1. **Removed noisy and trivial questions** from the original MMLU.
2. **Expanded four choices to ten.**
3. **Evaluated with chain of thought**, which was not in vogue when GPT-3 and MMLU
   arrived, and which gives the model more of a chance on questions that need a few
   reasoning steps.

Accuracy dropped by **16% to 33%**, so the benchmark was unsaturated again.

It is worth separating the mechanisms, because "harder questions" is only part of
it. Going from four choices to ten **lowers the random-guessing floor from 25% to
10%**, which by itself moves every score down. Removing trivial questions raises
the difficulty. Adding chain of thought moves scores back *up*. The reported drop
is the net of all three.

![MMLU-Pro figures comparing question difficulty and score distributions against MMLU](../raw/images/12-evaluation/mmlu-pro.png)

*MMLU-Pro against MMLU. Ten choices instead of four drops the guessing floor from 25% to 10%, which moves every score before difficulty is considered at all.*

And it did not last: the lecture notes accuracy is back to roughly 88, nearly 90
(≈[22:23], ≈[23:11]).

## GPQA (2023)

**Graduate-Level Google-Proof Q&A**
([arXiv 2311.12022](https://arxiv.org/abs/2311.12022)). The design principle is in
the name: *if a question can be answered by searching Google, and the model was
trained on the internet, the question is too easy* (≈[23:11]).

Getting genuinely hard questions takes human labour, and the lecture describes the
pipeline (≈[23:57]):

- **61 PhD contractors from Upwork** write the questions.
- Each question goes to **expert validation**, which returns feedback.
- The writer **revises**; a second expert reviews.
- The question then goes to **non-experts with Google access**, to check that they
  cannot solve it.

![GPQA example question with expert and non-expert answers](../raw/images/12-evaluation/gpqa.png)

*A GPQA item with its validation record. What makes it Google-proof is the non-expert row, not the question.*

The **diamond subset** you see quoted on leaderboards is the questions where both
experts agreed *and* at most one non-expert answered correctly (≈[24:42]).

The three headline numbers are the whole design in one line:

| Who | Accuracy |
| --- | --- |
| PhD experts (in-domain) | **65%** |
| Non-experts, 30 minutes with Google | **34%** |
| GPT-4 (at publication) | **39%** |

The 65/34 gap is what makes the questions expert-level; the 34% figure is what
makes them *Google-proof*. Percy's aside on the 65% is worth keeping: it is well
above random on a mostly four-way multiple-choice set, but it is not a good grade —
these questions are hard for the experts too (≈[24:42]).

GPQA has since gone the way of the others: the lecture checks the leaderboard live
and finds **94** (≈[25:27]).

## Humanity's Last Exam (2025)

[arXiv 2501.14249](https://arxiv.org/abs/2501.14249). "Something ominously called
Humanity's Last Exam" (≈[27:00]) — the response to models getting good enough that
the previous three all saturated.

- **2500 questions**: multimodal, many subjects, multiple-choice plus short answer.
- **Incentives**: a **$500K prize pool** plus **co-authorship** for question
  creators — money for those motivated by money, authorship for those motivated by
  that (≈[27:45]).
- **Filtered by frontier LLMs**, then multiple stages of review.
- A **private held-out set** that was never released, so it cannot enter training
  data — though, as Percy notes, you still have to send the prompts to an API and
  hope they do not end up in training that way (≈[27:45]).

![Four Humanity's Last Exam example questions across different subjects](../raw/images/12-evaluation/hle-examples.png)

*Four HLE items. Unlike the GPQA figure, none of these cards marks which answer is correct.*

The results figure follows the shape every dataset paper has: previous benchmarks,
models do well; my dataset, models do terribly — single digits at release
(≈[28:32]). Unlike the other three, HLE **still has room**: checking in 2026, the
top score is **64.7** (≈[28:32]).

![HLE question-collection pipeline from submission through LLM filtering to expert review](../raw/images/12-evaluation/hle-pipeline.png)

*How HLE guarantees an unsaturated benchmark: questions frontier models already answer are filtered out before human review begins.*

**The methodological hazard is the filter.** A benchmark defined as "questions
today's frontier models get wrong" is defined *relative to today's models*. It
guarantees an unsaturated benchmark at release and makes the question set a moving
target rather than a fixed standard.

![HLE accuracy results for frontier models, all scoring low](../raw/images/12-evaluation/hle-results.png)

*HLE at release. The shape every dataset paper's results figure has — and the one benchmark in this section that still has room.*

## What the section concludes

The summary (≈[29:20]):

- A **trend toward harder questions** as models improve and saturate what exists.
- **Multiple choice survives**, and the lecture defends it against the usual
  objection. Multiple choice is not intrinsically easy — you can write an
  arbitrarily hard multiple-choice question. What it restricts is *the set of
  questions you can ask*, not the difficulty (≈[29:20]).
- The real limitation is that exams **do not capture real usage**. Nobody asks HLE
  questions of an assistant except when evaluating HLE. Real questions are
  open-ended, may have no correct answer, and may not even be well formed
  (≈[30:06]).

That last bullet is the handoff to [chat benchmarks](chat-benchmarks.md), and the
saturation trend is the reason [reasoning benchmarks](reasoning-benchmarks.md) went
looking for a different axis entirely.

Percy also makes a defence of saturated benchmarks that is easy to lose: even once
a benchmark is solved by frontier models, it is **still useful for developing
smaller models and for fitting scaling laws** (≈[25:27]). A saturated benchmark is
dead as a leaderboard and alive as a development signal — see
[scaling-law methodology](scaling-law-methodology.md).

## Two questions from the floor

**Contamination.** A student asks how you know these questions are not simply in
the training set. The short answer is *we don't know*, because nobody knows what is
in the training set. Percy's elaboration is the useful part: contamination is
usually **more subtle than literally training on the test set** — questions get
derived from other sources, and those sources get trained on (≈[26:14], ≈[27:00]).
The full treatment is in [benchmark contamination](benchmark-contamination.md).

**How accuracy is actually computed.** A student asks how the model's output is
compared against the ground truth. For multiple choice it is simply whether the
generated letter matches. That pushes the question back one step, to *how the model
produces an answer at all*: it can be sampled directly as a letter, or — more
commonly now — generated as a chain of thought with the answer extracted from it.
Percy flags that the extraction step matters and that **LM evaluations can be very
sensitive to it**, then declines to go further (≈[30:51], ≈[31:37]). It is a real
gap in the lecture's coverage, and worth knowing about before comparing two
published numbers on the same benchmark.

## Sources

- Course material: [`raw/slides/12-evaluation.md`](../raw/slides/12-evaluation.md),
  section *Exam benchmarks* (`lecture_12.py` lines 108–153).
- Transcript: [lecture 12](../raw/transcripts/12-evaluation.md), ≈[17:46]–[31:37].
