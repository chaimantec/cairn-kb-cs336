# Lecture 12 — Evaluation

**Percy Liang.** [Video](https://www.youtube.com/watch?v=JpAxdTWQJxM) (78 min) ·
[transcript](../raw/transcripts/12-evaluation.md) ·
[course material](../raw/slides/12-evaluation.md) ·
[`lecture_12.py`](https://github.com/stanford-cs336/lectures/blob/main/lecture_12.py) ·
[trace viewer](https://cs336.stanford.edu/lectures/?trace=lecture_12)

This lecture is the hinge of the course. Lectures 1–11 built a language model —
tokenizer, architecture, kernels, parallelism, scaling laws, inference — and
lectures 13–14 are about the data you train it on. Before you can choose data you
have to say what behaviour you want, and that is the question evaluation answers:
**given a model, how good is it?** The lecture surveys five families of benchmarks,
then argues that the survey was never the point — the point is that every benchmark
is an attempt to turn an abstract construct into a concrete metric, and every one of
them loses something in the translation.

It is an [executable lecture](executable-lectures.md), so there are no slides and no
slide numbers; the written form is a 394-line Python program. It is also the most
**image-dense** lecture in the course, with 43 figures, and the figures carry the
argument — the leaderboards, the example questions and the results charts are the
content, and the surrounding text is mostly one-line captions on them.

## The core challenge

Everything in the lecture hangs off one line (≈[1:37], ≈[2:23]):

> **Core challenge**: abstract construct → concrete metric

Evaluation *looks* mechanical — define prompts, send them to a model, compute
accuracy. The reason it is not is that you rarely care about the prompts. You care
about something abstract: is this model intelligent, is it helpful, can it reason, is
it safe. Turning that into a number is an act of interpretation, and the number is
only as good as the interpretation.

The reason to care, in Percy's framing, is that **evaluation shapes the development
of AI**. Benchmarks set north stars; every model developer, open and closed, reads
them as the measure of progress. So choosing an evaluation is choosing, implicitly,
what models will become good at (≈[1:37]).

The full treatment is on [construct validity](construct-validity.md).

## Four answers to "what makes a model good"

The lecture opens with four operationalisations, each with a site that embodies it
(≈[2:23]–[4:42]). They are alternatives rather than steps, and the lecture endorses
none of them:

1. **Does well on benchmarks** — [Artificial Analysis](https://artificialanalysis.ai/).
2. **Does well on benchmarks and is cheap to run** — the same index against inference
   cost. Correlated, but not exactly aligned (≈[3:09]).
3. **People prefer its responses** — [Arena AI](https://arena.ai/leaderboard).
4. **People choose to use and pay for it** — [OpenRouter](https://openrouter.ai/rankings)
   usage statistics. *"I don't know what good is, but if people are paying for it, it
   must be good"* — with the immediate caveat that OpenRouter's traffic is not
   representative of all usage (≈[4:42]).

![Artificial Analysis leaderboard ranking models by an intelligence index](../raw/images/12-evaluation/artificial-analysis.png)

*Answer one, made concrete. Each of the four framings has a site behind it that looks exactly this authoritative.*

## The survey, and its shape

Five families follow, ordered by **what is being measured**:

| Family | Measures | Judge | Page |
| --- | --- | --- | --- |
| Perplexity | probability assigned to held-out text | none — it is arithmetic | [perplexity-evaluation](perplexity-evaluation.md) |
| Exams | knowledge, on questions with a right answer | exact match | [exam-benchmarks](exam-benchmarks.md) |
| Chat | quality of an open-ended response | human, or an LLM | [chat-benchmarks](chat-benchmarks.md) |
| Agentic | what the model *does*, over time, with tools | unit tests / environment | [agentic-benchmarks](agentic-benchmarks.md) |
| Pure reasoning | inference isolated from knowledge | exact match | [reasoning-benchmarks](reasoning-benchmarks.md) |
| Safety | refusal, and harm more broadly | varies; contested | [safety-evaluation](safety-evaluation.md) |

Read down the "judge" column and the lecture's structure becomes visible. Perplexity
needs no judge and is therefore untargeted. Exams need no judge because they have an
answer key, and pay for it in realism. Chat has no answer key and must hire a judge,
inheriting the judge's biases. Agentic benchmarks get an answer key back — an
executable one — which is the cleanest escape in the lecture. And safety has neither
an answer key nor an agreed judge, which is why it is the hardest.

The three sections *after* the survey — realism, validity, and how to think about
evaluation — are the argument the survey was gathering evidence for.

## Perplexity: sufficient in the limit, badly aimed in practice

The metric this course has been optimising since lecture 1, examined as an evaluation
for the first time. A language model is a distribution $p(x)$, and perplexity asks how
much mass it puts on held-out data $D$:

$$\text{perplexity}(p, D) = \left(\frac{1}{p(D)}\right)^{1/|D|}$$

Two arguments are set against each other (≈[9:19]–[12:23]):

- **"Perplexity is all you need"** — more faith than science, and the lecture says so.
  The best achievable perplexity is $H(t)$, attained iff $p = t$; if $p = t$ you can
  solve every task by conditioning; so pushing perplexity down eventually reaches AGI.
  Percy's point is not that this is a proof but that it is a *mindset*, and one that
  drove people to keep scaling before GPT-3 made the case obvious.
- **"Perplexity is maybe more than you need"** — it charges the model for every token.
  In *Stanford was founded in 1885*, predicting **1885** is a knowledge test worth
  scoring and predicting **founded** is not, and perplexity does not distinguish them.
  Conditional perplexity, $p(\text{response} \mid \text{prompt})^{1/|\text{response}|}$,
  is the repair.

The historical pivot in this section is **GPT-2**: trained on WebText, evaluated
*zero-shot* on everyone else's datasets, which turned in-distribution evaluation into
out-of-distribution evaluation and made the modern benchmark paradigm possible
(≈[7:46]). It reached about **35** perplexity on PTB against a state of the art of
about **46**, and did *not* beat the state of the art on the One Billion Word
Benchmark, where in-distribution data is plentiful (≈[8:33]).

And a warning worth remembering if you ever run a leaderboard: on a **perplexity**
leaderboard you must trust that the submitted probabilities sum to one, or the winner
is a model that always returns 1. Downstream task leaderboards do not have the problem
because they only need the output text (≈[15:27], ≈[16:14]).

## Exams: the saturation treadmill

Four benchmarks in chronological order, and the order is the argument — each exists
because its predecessor saturated (≈[17:46]–[29:20]):

| Benchmark | Year | Design move | State in the lecture |
| --- | --- | --- | --- |
| **MMLU** | 2020 | 57 subjects, multiple choice, few-shot | 90s — saturated |
| **MMLU-Pro** | 2024 | 10 choices not 4; noisy questions removed; chain of thought | ~88 — saturating |
| **GPQA** | 2023 | Google-proof; 61 PhD writers; expert/non-expert validation | 94 — saturated |
| **HLE** | 2025 | $500K prize; filtered by frontier LLMs; private held-out set | **64.7** — still room |

GPQA's three numbers are the cleanest statement of what an exam benchmark is trying
to do: PhD experts **65%**, non-experts with 30 minutes of Google **34%**, GPT-4 at
publication **39%** (≈[24:42], ≈[25:27]).

The lecture defends multiple choice against the usual objection — you can write an
arbitrarily hard multiple-choice question, so the format does not limit *difficulty*;
what it limits is **the set of questions you can ask** (≈[29:20]). And it makes a
point about saturated benchmarks that is easy to lose: once solved at the frontier
they are **still useful for developing smaller models and fitting scaling laws**
(≈[25:27]).

## Chat: no answer key, so hire a judge

The beet salad example is the whole problem in one prompt (≈[32:22]). There is no
exact match to check and no ground truth, so every benchmark here substitutes a judge.

![Chatbot Arena side-by-side interface showing the beet salad prompt and two anonymised responses](../raw/images/12-evaluation/arena-beets.png)

*The problem, and one answer to it: two anonymised responses to the lecture's own beet-salad prompt, and four buttons that turn a human's reaction into data.*

- **Chatbot Arena** asks humans, pairwise, and fits ELO ratings. The property that
  makes it affordable is that **ELO does not require every model to see every prompt** —
  a sparse but connected comparison graph suffices, which matters precisely because a
  human is doing the rating (≈[37:44]).
- **AlpacaEval** replaces the human with GPT-4 as judge, discovers that LLM judges
  favour long responses, watches the leaderboard get gamed by longer outputs, and fixes
  it with a regression debias (≈[39:17]).
- **WildBench** sources real conversations and judges with a **generated, task-specific
  checklist**, which turns an ill-defined question into a well-defined one (≈[42:23]).

The section's best question is *how do you evaluate a metric?* The only available
sanity check is correlation with a metric you already trust — AlpacaEval against
Chatbot Arena at **0.98** — and Percy names the circularity out loud: if everything
validates against Arena, is Arena ground truth? *"At least we're all in the same boat
together"* (≈[40:02], ≈[42:23]).

## Agentic: the answer key comes back, executable

> Agent = language model + agent scaffold

That definition carries the conclusion (≈[45:29]). **SWEBench** gives an agent a
codebase and an issue and grades the patch with **unit tests** — an executable answer
key, which is the one clean escape from the judging problem. **TerminalBench** makes
the environment a terminal, on the argument that it is simple and universal.
**CyBench** uses CTF challenges and takes **first-solve time** as a human-calibrated
difficulty scale. **MLEBench** runs 75 Kaggle competitions, which makes it a benchmark
for doing the machine learning this course teaches.

The measured progress is fast: SWE-bench Verified from about **16%** in 2024 to about
**93%**; CyBench from about 10% at release to essentially solved (≈[47:02], ≈[50:05]).

And the scaffold matters as much as the model, which the lecture shows twice — two
agents on the same model post different TerminalBench accuracies, and MLEBench's
leaderboard varies far more across scaffolds than across models (≈[48:32], ≈[50:54]).
Hence the summary: **evaluating agents = evaluating agent scaffold + language model**.

## Pure reasoning: the one benchmark scale did not move

ARC-AGI is designed to be **100% solvable by humans** and to make every task unique so
memorisation cannot help (≈[54:43]). The trajectory is why it is in this course
(≈[55:30], ≈[56:16]):

- Pre-trained language models scored **essentially zero** on ARC-AGI-1 — scaling
  pre-training "didn't move the needle at all."
- **o1 and o3** made it take off abruptly in 2024; ARC-AGI-1 is now basically solved.
- ARC-AGI-2 (2025) is on its way; ARC-AGI-3 (2026), interactive environments, is scoring
  **extremely low**.

A benchmark where scale does nothing and a change of *method* does everything is
evidence it measures something the rest of the survey does not. Set that against
[scaling laws](scaling-laws.md), where predictable improvement with scale is the whole
premise.

![Scatter plot of ARC-AGI-1 and ARC-AGI-2 scores against model release date, 2020 to 2026](../raw/images/12-evaluation/arc-agi-results.png)

*The ARC-AGI trajectory. Flat through the whole pre-training scaling era, then abrupt once reasoning models arrive.*

The honest limitation, stated by the lecture: because validity comes from being 100%
human-solvable, ARC-AGI **cannot detect superhuman reasoning** — IMO gold medals, open
mathematical problems — which are arguably the capabilities that matter most
(≈[58:35]).

## Safety: no agreed operationalisation at all

The section opens with a car crash test, and the image is the argument. Automotive
safety has an agreed procedure arrived at over decades; AI does not (≈[1:00:12]).

**HarmBench** derives its list of harms from *laws and norms* (510 behaviours);
**AIR-Bench** derives it from *regulatory frameworks and company policies* (314 risk
categories, 5694 prompts). Neither derives it from first principles, because there is
none to derive it from.

**GCG** shows that safety training is defeasible by optimisation, and — the part that
makes it a systems fact — the attack **transfers from open-weight models to closed
ones** (≈[1:02:34]).

The deepest point is that safety cannot be one number, because the risks relate to
capability in *opposite* directions: a more capable model **hallucinates less** and is
**better at abetting crimes** (≈[1:03:19]). And the dual-use case closes the loop with
CyBench — the same cybersecurity capability breaks systems or hardens them, and no
measurement separates the two (≈[1:04:51]).

## Realism, validity, and the point of it all

The last three sections are the argument.

**Realism / ecological validity** (≈[1:04:51]–[1:07:58]). Exams are controlled but
unrealistic; Arena is realistic but uncontrolled. **GDPVal** samples 44 occupations by
share of **US GDP**, which makes the task distribution a claim about economic
significance. **MedHELM** replaces medical exams with 121 tasks from 29 clinicians.
**Clio** uses language models to analyse real user data and publish aggregate patterns.
The closing line is the honest one: *realism and privacy are sometimes at odds with
each other* — the most valuable evaluation data is exactly the data you cannot publish.

**Validity** splits in two, and the split is worth holding onto:

- [Benchmark contamination](benchmark-contamination.md) — a problem with the *model's
  relationship to* the benchmark. Four routes: infer overlap from the model
  (exchangeability), reporting norms, fresh evals, private evals. None is
  simultaneously verifiable by outsiders, durable, and public (≈[1:08:43]–[1:12:37]).
- [Dataset quality](construct-validity.md#dataset-quality) — a problem with the
  benchmark *itself*. SWE-Bench needed a Verified version; benchmarks like GSM8K and
  MMLU contain broken questions; and in one agentic benchmark an agent that **returns
  the empty response scores about 38%** (≈[1:12:37]–[1:14:53]).

**The point of evaluation.** Four purposes, and they license different evaluations: a
purchase decision, a capability measurement, a benefits-and-harms assessment, developer
feedback (≈[1:15:41]). The complaint is that the purpose is usually not stated.

**Methods versus models**, the most transferable idea in the lecture (≈[1:16:27]).
Before foundation models we evaluated *methods*, because the split was fixed and only
the algorithm varied, so a win was attributable. Today we evaluate *models and systems*,
where a win could be data, scale, scaffold, post-training or luck. The **nanogpt
speedrun** — fixed data, minimise compute to a target validation loss — is a rare
modern method evaluation, and it is the shape of this course's own assignments.

## Takeaways

Verbatim from the source (`lecture_12.py` lines 27–30):

- There is no one true evaluation; choose the evaluation depending on what you're
  trying to measure.
- Clearly state the rules of the game (methods versus models versus agents).
- Considerations: difficulty, realism, validity.

Those three considerations map onto the lecture's own structure, and Percy's final
point is that they **trade off**: it is hard to be realistic, difficult, ecologically
valid and uncontaminated at once, so you compromise on one, and which one depends on
your goals (≈[1:18:01]).

## Questions from the floor

Three, and each fills a real gap:

- **Contamination** (≈[26:14], ≈[27:00]) — how do you know the questions aren't in the
  training set? *We don't.* And contamination is usually subtler than literal test-set
  training: questions get derived from sources that get trained on.
- **How accuracy is computed** (≈[30:51], ≈[31:37]) — for multiple choice, whether the
  generated letter matches. Which pushes the question to how the answer is produced:
  sampled directly, or extracted from a chain of thought. Percy flags that **LM
  evaluations can be very sensitive to the extraction step** and does not go further.
- **ARC-AGI-3's modality** (≈[59:20]) — the grid (he believes around 64×64) can be
  given as an image *or* as ASCII art. Either way there is a spatial element to reason
  about that is not natural language.

## Where this sits in the course

- **Back**: [lecture 9](09-scaling-laws.md) and
  [lecture 11](11-scaling-laws-in-the-wild.md) fit smooth curves to perplexity;
  [upstream vs downstream](upstream-vs-downstream.md) is lecture 9's evidence that
  those curves can reorder completely on task accuracy. This lecture is the general
  case of that warning.
- **Forward**: lectures 13–14 are about data. The opening lines say why evaluation
  comes first — data shapes behaviour, so you need to know what behaviour you want
  (≈[0:05], ≈[0:51]).
- See [the course map](course-map.md) for the whole arc.

## Sources

- Course material: [`raw/slides/12-evaluation.md`](../raw/slides/12-evaluation.md) —
  the full transcription of `lecture_12.py`, section by section, with all 33
  course-repo figures described.
- Transcript: [`raw/transcripts/12-evaluation.md`](../raw/transcripts/12-evaluation.md) —
  copy-edited, timestamps preserved.
