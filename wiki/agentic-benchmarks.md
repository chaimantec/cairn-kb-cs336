# Agentic benchmarks

[Lecture 12](12-evaluation.md) draws the line plainly: chat benchmarks evaluate
**what language models say**, agentic benchmarks evaluate **what they do**
(≈[44:43]).

## What an agent is, and why the definition matters

> Agent = language model + agent scaffold (logic for deciding how to use the LM)

The scaffold is the logic deciding how the model gets called and what tools it can
reach (≈[45:29]). Tasks in scope are ones requiring **tool use** — running code, for
instance — and **iteration over a period of time**.

That definition carries the section's conclusion. If an agent is a model *plus* a
scaffold, then an agentic benchmark score measures the pair, and cannot be
attributed to the model alone. The lecture makes the point empirically twice: on
TerminalBench, two agents built on the *same model* get different accuracies
(≈[48:32]), and on MLEBench "the same usual suspects" appear on the model side while
the scaffolds vary considerably (≈[50:54]).

## SWEBench

[arXiv 2310.06770](https://arxiv.org/abs/2310.06770).

- **2294 tasks** across **12 Python repositories**.
- Given a codebase and a GitHub issue description, **submit a PR**.
- **Evaluation metric: unit tests.**

The test design is what makes it work (≈[45:29], ≈[46:17]): before the patch,
certain unit tests fail; the goal is to make them pass **without breaking anything
else**. The agent sees the instructions, the issue, and code from the repository,
and produces a patch.

**Unit tests are the one clean escape from the judging problem** that
[chat benchmarks](chat-benchmarks.md) spends its whole length on. The answer key is
executable, so grading is exact, cheap and not susceptible to a judge's biases.
That is why SWEBench pioneered the modern way of assessing coding agents, and why
so much agentic attention went into coding first (≈[46:17], ≈[47:02]).

**Progress**: the lecture checks the SWE-bench **Verified** leaderboard and finds
about **16%** in 2024 against roughly **93%** now (≈[47:02]). Why *Verified* rather
than the original is a story told later in the lecture — see
[construct validity](construct-validity.md#dataset-quality).

## TerminalBench

[arXiv 2601.11868](https://arxiv.org/abs/2601.11868) ·
[tbench.ai](https://www.tbench.ai/).

- Environment: a **computer terminal**.
- **229 tasks** crowdsourced from **93 contributors**; **89 tasks** constitute
  Terminal-Bench 2.0.

The design argument is generality: a terminal is **simple and universal**, so one
environment subsumes an enormous range of real tasks and no per-task harness is
needed (≈[47:47]).

The difficulty calibration is worth recording: these tasks take a human anywhere
from **about an hour to over a week**, depending on whether they are an expert or a
junior (≈[48:32]).

And this is where the lecture makes the scaffold point concrete — the top of the
leaderboard is the frontier models, but the *agent* matters too, and two agents on
the same model post different accuracies (≈[48:32]).

## CyBench

[arXiv 2408.08926](https://arxiv.org/abs/2408.08926).

- **40 Capture the Flag (CTF) tasks.**
- **First-solve time as a measure of difficulty.**

The setting: the agent gets an environment and can run commands — read source code,
reach a web server — and must break into the server to extract a **flag**, a unique
string proving the compromise (≈[49:19]). These are real security exercises normally
done by humans in competition.

**First-solve time is the clever part.** CTF competitions record how long the first
human team took on each challenge, which hands the benchmark a
**human-calibrated difficulty scale for free** — something almost nothing else in
this lecture has.

The scaffold shown alongside it is deliberately simple: one continuous memory, where
the agent emits an action, receives environment feedback, and appends the result to a
buffer that is concatenated back into context (≈[50:05]). Percy flags the obvious
problem — that history grows, and you need better ways of managing context — which
sets up the scaffold discussion below.

**Progress**: about 10% at release, and now essentially **solved** (≈[50:05]).

## MLEBench

[arXiv 2410.07095](https://arxiv.org/abs/2410.07095).

- **75 Kaggle competitions**, requiring data processing, model training, and the rest
  of the ML engineering loop (≈[50:54]).

The agent reads the dataset and the description, writes code, trains models, submits,
and is graded. The leaderboard shows the familiar frontier models with **considerable
variation across agent scaffolds** (≈[50:54], ≈[51:39]).

MLEBench is the most self-referential benchmark in the course: it evaluates a
language model on the task of doing the machine learning that this class teaches.

## Agent scaffolds

[Reference post](https://www.philschmid.de/agents-2.0-deep-agents). The lecture's four
ingredients (≈[51:39]–[53:10]):

- **Explicit planning** — keep a to-do list and check items off. Percy's reasoning is
  that you cannot just stream-of-consciousness chain-of-thought a long task; context
  builds up and the agent loses track of where it is.
- **Hierarchical delegation** — agents call sub-agents with clean context. The
  sub-agent does the work and returns only the result, so the parent never sees the
  gory details. This is encapsulation, applied to context.
- **Persistent memory** — read and write files, because as context grows you cannot
  keep everything in the window.
- **Extreme context engineering** — explicit instructions about *process*: when to
  delegate, which strategy to try, what to write to memory. These are generally tuned
  for a particular model.

The through-line is that all four are **context management**. A scaffold is largely a
policy for deciding what the model sees.

## What the section concludes

(≈[53:10])

- Agents **dramatically enhance the capability surface** of language models.
- **Agent scaffolds are very important.**
- **Evaluating agents = evaluating agent scaffold + language model.**

The third bullet is the one to carry away, and it is the reason agentic leaderboards
list an agent name beside a model name. A published agentic score is not a property of
a model. This also cuts the other way, and the lecture returns to it under
[construct validity](construct-validity.md#dataset-quality): if a *trivial* scaffold
can score well, the task never measured what it claimed to.

## Related

- [Chat benchmarks](chat-benchmarks.md) — the judging problem that unit tests avoid.
- [Safety evaluation](safety-evaluation.md) — CyBench's capability is exactly the
  dual-use case the safety section raises.
- [Construct validity](construct-validity.md) — why SWE-bench needed a *Verified*
  version.

## Sources

- Course material: [`raw/slides/12-evaluation.md`](../raw/slides/12-evaluation.md),
  section *Agentic benchmarks* (`lecture_12.py` lines 208–255).
- Transcript: [lecture 12](../raw/transcripts/12-evaluation.md), ≈[44:43]–[53:56].
