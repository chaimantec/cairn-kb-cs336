# Benchmark contamination

"Machine learning 101: don't train on your test set." [Lecture 12](12-evaluation.md)
argues that foundation models destroyed the mechanism that used to guarantee this,
and then walks through four attempts to rebuild it (≈[1:08:43]).

## What was lost

The three-line history in the lecture is compressed but complete:

- **Machine learning 101**: don't train on your test set.
- **Pre-foundation models** (ImageNet, SQuAD): well-defined train/test splits. Every
  dataset shipped with a split, everyone played the same game, life was simple.
- **Today**: train on the internet, and don't tell anyone what your data was.

The important word is *procedural*. The old guarantee was not a measurement or an
argument — it was a **file layout**. The split came with the dataset, so
contamination required a deliberate act. Once the training set is "the internet" and
the benchmark is published on the internet, the guarantee is gone by default rather
than by misconduct, and nobody outside the lab can check (≈[1:09:30]).

A student raises this during the exam section, and the answer is blunt: *we don't
know*, because we don't know what is in the training set. Percy's elaboration is the
part worth keeping — contamination is usually **more subtle than literally training
on the test set**. Questions get derived from other sources, and those sources get
trained on (≈[26:14], ≈[27:00]).

The four routes below are all attempts to rebuild the lost guarantee without the
split.

## Route 1: infer overlap from the model

[arXiv 2310.17623](https://arxiv.org/pdf/2310.17623) — from Tatsunori Hashimoto's
group, so this course's other lecturer (≈[1:09:30]).

The idea rests on **exchangeability**, and the one-line bullet hides how neat it is.
A benchmark file has some canonical order, but that order carries no information: any
permutation of the examples is an equally valid version of the benchmark. So under a
model that has never seen the file, the published order should be no more probable
than a shuffling of it.

If the model assigns **higher probability to the benchmark in its published order**,
the model has seen the published file (≈[1:10:17]).

![Diagram of canonical-order versus shuffled-order benchmark questions used to detect contamination](../raw/images/12-evaluation/contamination-exchangeability.png)

*The exchangeability test. Same questions, two orders; a model that prefers the published one has seen the published file.*

The virtue of this route is that it needs no cooperation from the model provider —
only the ability to score sequences. Its limit is that it only detects the
contamination it can find: a model trained on questions *derived* from the benchmark,
in some other order, leaves no such signature.

## Route 2: reporting norms

[arXiv 2410.08385](https://arxiv.org/abs/2410.08385).

Encourage providers to **report train-test overlap** as a matter of course. Percy's
analogy is to confidence intervals: in statistics you always report one, because an
estimate without a reliability claim is not a finished result. The position paper's
argument is the same — if you are going to claim a GPQA score, you should also
provide some justification that you did not train on the test set (≈[1:10:17],
≈[1:11:04]).

This route depends entirely on the provider's cooperation, which is why it is a
*norm* rather than a method.

## Route 3: fresh evals

Give up on the existing benchmarks — assume the worst case, that everyone trained on
them — and build evaluations from data that postdates every model's cutoff
(≈[1:11:04]).

- **LiveCodeBench**, **UncheatableEval**: scrape new web pages, new arXiv papers, new
  GitHub repositories, and define the evaluation over material past the cutoff date.

The caveat is real: **timestamps aren't always safe, because of copying**. A
repository posted today may be derived from one that existed years ago, so a fresh
timestamp does not guarantee fresh content (≈[1:11:50]).

Fresh evals also expire by construction — today's fresh eval is next year's
contaminated one.

## Route 4: private evals

Evaluate on data that is not on the internet at all (≈[1:11:50], ≈[1:12:37]).

- **Companies** already do this: Google or OpenAI can evaluate on an internal
  codebase they know is not public and know they did not train on.
- **Individuals** can use their own unpublished writing. Percy's example is his stock
  of rejected papers from graduate school that were never put online.
- **Easiest for perplexity.** This is the connection back to
  [perplexity](perplexity-evaluation.md): a private eval needs only *text*, not an
  answer key, so log-probabilities are the cheapest thing to measure on it.

Private evals are the most reliable route and the least shareable. Whatever you learn
from one, you cannot turn it into a public leaderboard without destroying it.

## How the four routes trade off

Laid side by side, the structure is clear:

| Route | Needs cooperation? | Weakness |
| --- | --- | --- |
| Infer from model | No | Detects only signature-leaving contamination |
| Reporting norms | **Yes** — from the provider | Voluntary |
| Fresh evals | No | Expires; undermined by copying |
| Private evals | No | Unshareable; cannot be a public leaderboard |

There is no route that is simultaneously verifiable by outsiders, durable, and
public. That is the honest state of the problem.

## What to do with a published number

The practical advice from the lecture is to treat benchmark scores as claims with an
unstated dependency, and to be explicit about which route (if any) backs them. When a
student asked, Percy's answer was to take the numbers "with a little bit of a grain of
salt", and *"it's always good to be skeptical"* (≈[26:14], ≈[27:00]).

This is also why [saturated benchmarks](exam-benchmarks.md) are worth less than their
scores suggest: the longer a benchmark has been public, the more chances it has had to
enter training data.

## Related

- [Construct validity](construct-validity.md) — the other half of "is this evaluation
  valid?", concerning the benchmark itself rather than the model's exposure to it.
- [Exam benchmarks](exam-benchmarks.md) — where the student question arises.
- [Perplexity evaluation](perplexity-evaluation.md) — why private evals and perplexity
  fit together.

## Sources

- Course material: [`raw/slides/12-evaluation.md`](../raw/slides/12-evaluation.md),
  section *Validity → Train-test overlap* (`lecture_12.py` lines 338–369).
- Transcript: [lecture 12](../raw/transcripts/12-evaluation.md), ≈[26:14], ≈[27:00],
  ≈[1:08:43]–[1:12:37].
