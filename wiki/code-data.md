# Code data

Where code training data comes from, and what makes it different from web text.
From [lecture 13](13-data-sources-datasets.md) (≈39:15–41:33, ≈1:10:54–1:14:04).

The lecture's stated motivation for including code at all is broader than coding
ability: code is useful "not just if you want coding capabilities for your language
model, but if you want general reasoning" (≈39:15–40:01). The
[course material](../raw/slides/13-data-sources-datasets.md) marks that second claim
as **folklore** — widely believed, not evidenced here — which is the right amount
of confidence to carry.

## GitHub

- Founded 2008, "about the same age as Common Crawl"; acquired by Microsoft in 2018.
- **420 million repositories, 28 million public** (as of May 2026).
- A repository is not a file: "it's a directory with commit history, issues, pull
  requests, and comments" (≈40:01).
- **Training is permitted on public repositories with permissive licenses** — MIT,
  Apache (≈40:46).

**Two distinct kinds of data**, which the lecture is careful to separate:

1. **Repositories** — downloaded over the git protocol. "You shouldn't scrape
   GitHub, you should just download the repository."
2. **Metadata** — issues, pull requests, comments — via **GitHub Archive**, which
   provides hourly snapshots of the event stream. The lecturer finds this "really
   interesting data. Basically every single comment, or star, or action on GitHub
   gets recorded."

**Software Heritage** (non-profit, founded 2016) preserves *repositories* rather
than metadata, and aggregates beyond GitHub — GitLab, Bitbucket, PyPI. As of May
2026 it holds 28.8M source files. Both organisations reappear below: The Stack
takes repository *names* from GitHub Archive, and Stack v2 takes the
*repositories* from Software Heritage.

**Duplication is the defining property of code data.** Code "generally has a lot of
duplicates, because of either copying code or forking code" (≈40:01) — and The
Stack quantifies it below. See [deduplication](deduplication.md).

## The Stack

The 2022 effort to build a good coding corpus, "since it was clear by 2022 that
coding was going to be really important" (≈1:10:54).

- Took repository names from **GitHub Archive** (2015–2022).
- `git clone`d **137 million repositories** — **51 billion files, 5 billion
  unique**.
- Kept only permissively licensed code (MIT, Apache), detected with
  `go-license-detector`.
- Removed near-duplicates using MinHash and Jaccard similarity.
- **Result: 3.1 TB of code.**

The parenthetical the lecture emphasises is "(5B unique!)" — **roughly 90% of
GitHub by file count is duplicate**. That is the concrete version of the general
claim above, and it is the single most quotable number on this page.

## Stack v2

The 2024 successor, which broadens from code files to the whole development
process:

- **Issues, comments and PRs** from GitHub Archive.
- **Repositories** from Software Heritage.
- **Documentation** crawled from websites — PyPI, npm, devdocs.io.
- Processing: remove binary files, remove malware, filter bot activity ("a lot of
  GitHub, especially these PRs, are bots"), deduplicate, redact PII, subsample PRs
  (≈1:11:44).
- Include existing datasets: GSM8K, code contests, StackOverflow, arXiv, Wikipedia,
  OpenWebMath.

### The LLVM bridge

The idea the lecturer singles out as "kind of nice" (≈1:12:32). There is a lot of
Python and C on GitHub, and very little of low-resource languages like **Nim**
("which I hadn't even heard of"). So:

> compile this code into a low-level intermediate language, LLVM, which every C
> compiler, say, can compile into, and they juxtapose the low-resource language and
> the intermediate representation.

Pairing the two in the training data lets the model "learn the mapping between the
shared low-level representation, which has a lot of data on it, and the thing that
has less data." It is a transfer mechanism built out of a compiler — using an
existing, data-rich canonical form as a pivot between languages.

### Linearizing pull requests

A pull request is not a token sequence. It is a structured object — a base commit,
a series of diffs, review states, comments, events — so **it has to be linearized**,
and the design decisions in doing so are real (≈1:13:19).

The one the lecture highlights is **how much context to include**. An event might be
"I changed one line of code" — but one line is not learnable on its own, so "you want
to provide some context, maybe a few lines around it, maybe the entire file
surrounding that diff" (≈1:13:19–1:14:04, the phrase running across the
paragraph break). Too little and the diff is meaningless; too much and the
dataset explodes. Stack v2 also subsamples PRs to keep the corpus manageable and
representative.

![Colour-coded template showing how a pull request is linearized into tokens](../raw/images/13-data-sources-datasets/stackv2-pr1.png)

![Second linearization template showing diffs, comments and review-state events](../raw/images/13-data-sources-datasets/stackv2-pr2.png)

*The linearization templates — XML-like structure with the PR, its diffs, and
events such as a comment being posted or a review state changing. Note a small
internal inconsistency between the two: `username_id` is split black-and-red in the
second while `username_0` is solid blue in the first.
[Source](https://github.com/stanford-cs336/lectures/blob/main/images/stackv2-pr2.png)*

The payoff, in the lecture's words, is that the model learns **"not just learning how
to generate code, but also the software development process around code"**
(≈1:14:04)
— review, iteration, and responding to comments, not just syntax.

## Licensing, which is unusually tractable here

Code is the one domain in this lecture where licensing is comparatively clean, and
that is why it appears twice. Repositories carry **explicit, machine-detectable
licenses**, so *keep only MIT and Apache* is an implementable rule in a way that
*keep only permissively licensed web pages* is not.

This is what makes code a large component of
[CommonPile](data-licensing-and-consent.md#commonpile), which includes Stack v2:
if you are building a corpus from permissively licensed material only, code is
among the easiest material to qualify.

The caveats from [data licensing and consent](data-licensing-and-consent.md) still
apply — a declared license can be wrong, and license laundering is hard to detect —
but the base rate of clear, explicit licensing is far higher here than anywhere
else in the lecture.

## See also

- [Pre-training datasets](pretraining-datasets.md) — The Stack in the chronology
- [Deduplication](deduplication.md) — the 51B → 5B reduction
- [Data licensing and consent](data-licensing-and-consent.md) · [Copyright and
  fair use](copyright-and-fair-use.md)
- [Lecture 13](13-data-sources-datasets.md)
