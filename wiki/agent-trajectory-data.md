# Agent trajectory data for software engineering

Four recent datasets that teach a model to *do* software development rather than
just emit code, and the infrastructural problem that shapes all of them.
[Lecture 14](14-data-filtering-dedup-mixing.md) spends the last third of its
post-training section here (≈1:17:54–1:23:20), because "there's been a lot of interest in developing, in particular, agentic coding models — not just a model that can generate some code, but actually a model that can do software development".

The general shape is the [post-training recipe](post-training-data.md#the-recipe):
environments, tasks, and responses from a teacher. What makes code hard is the
first of those.

## The problem: environments are the bottleneck

Software engineering tasks are unlike math or competitive programming in one
expensive way: **they have dependencies**. "most GitHub repos don't even run, and you have to install all these dependencies, and they're out of date, especially if you roll back to, like, when a PR was — it's just kind of a mess, and this is just a nightmare" (≈1:19:27). Doing
this properly means a per-repository Docker image, and the source's phrase for
building thousands of them is "an infrastructural nightmare".

Each of the four papers below is, in part, an answer to that.

## SWE-smith

[arXiv 2504.21798](https://arxiv.org/abs/2504.21798). **Semi-synthetic**: a real
repository, synthetic tasks.

*Figure: `images/swe-smith.png`.*

![Four-stage pipeline diagram from a real GitHub repo to synthetic bug-fix task instances](../raw/images/14-data-filtering-dedup-mixing/swe-smith.png)

*The pipeline: (1) **Real Repositories** — source code plus its passing unit tests; (2) **Environment Creation** — SWE-agent attempts to install the repo and run the tests, then a developer writes a Dockerfile based on that work; (3) **Task Generation Strategies** — procedural modification, LM-generated bugs, combining bugs, and PR mirroring; (4) **New Task Instances** — an environment image plus synthetic tasks, each with a generated issue, a bugged patch, and verified tests. No aggregate counts appear in the figure itself.*

The trick is inversion. Rather than hunt for real bugs and their fixes, take working
code and have a language model **break** it: "given a repository, you use a language model to automatically generate tasks... you modify the code in some way, maybe introduce some bugs, and those get verified, and you get some task instances"
(≈1:17:54). Because the bug was introduced deliberately, the fix is known by
construction and the tests verify it.

**128 GitHub repositories yield 50K tasks** — roughly 400× amplification — which
the lecture notes was "at the time — which was last year — was quite large" (≈1:18:41).

Note the human step that survives in stage 2: an agent attempts the install, and
then *a developer writes the Dockerfile*. The environment problem is not solved
here, only assisted.

## SWE-Zero

[arXiv 2604.01496](https://arxiv.org/abs/2604.01496). **Real** prompts — actual
GitHub PRs — and the paper that attacks the environment problem head-on by
declining to solve it.

*The lecture attributes this paper to NVIDIA (≈1:18:41). That attribution is spoken
only and is not stated anywhere in the source program, so it is recorded here as
what was said rather than as an established fact.*

### The observation

Strong models can already solve many SWE tasks **without running anything**.

*Figure: `images/swezero-noexec.png`.*

![Table of four coding models' SWE-bench Verified and SWE-bench Multilingual scores with and without execution access](../raw/images/14-data-filtering-dedup-mixing/swezero-noexec.png)

| Model | Execution | SWE-bench (V) | SWE-bench (M) |
| --- | --- | --- | --- |
| MiniMax-M2.5 | ✗ | 69.5 | 57.2 |
| MiniMax-M2.5 | ✓ | 80.2 | 74.1 |
| Qwen3-Coder-Next | ✗ | 56.9 | 50.7 |
| Qwen3-Coder-Next | ✓ | 71.3 | 64.3 |
| Qwen3-Coder-480B-A35B-Instruct | ✗ | 59.4 | 44.3 |
| Qwen3-Coder-480B-A35B-Instruct | ✓ | 69.6 | 54.7 |
| SWE-Hero-32B (Ours) | ✗ | 57.7 | 42.2 |
| SWE-Hero-32B (Ours) | ✓ | 62.2 | 44.1 |

Every model does worse without execution, but not catastrophically so — the
lecture reads the top row off the table: "if you were allowing execution, you get like 80; if you don't allow execution, you get almost 70. So that's not bad, for not being able to execute code" (≈1:19:27). The conclusion drawn is that "somehow these
models have some internal semantics of code" — the source's phrasing is that strong
models have an internal **"world model" of code semantics** (≈1:20:15).

If that is true, then for *data generation* purposes the Docker images were never
strictly necessary.

### What they built

- **300K agent trajectories that require no repository-specific execution**, from
  **150K GitHub PRs**. These are real PRs, so the tasks are "realistic, unlike the
  SWE-smith examples" (≈1:20:15).
- The **OpenHands** scaffold, with an execution-free system prompt.
- **Future git commits removed**, to prevent "git hacking" — without this an agent
  can simply read the fix out of the repository's later history instead of
  reasoning about the code.
- **Distilled from Qwen3-Coder-480B**, then filtered — and the filter has to "try to
  execute anyway", because "sometimes a coding model will ignore these instructions
  and still try to execute" (≈1:21:01).
- **SWE-Hero: 13K trajectories that *do* require execution feedback.** Models are
  trained by fine-tuning first on the SWE-Zero data and then again on SWE-Hero.

*Figure: `images/swezero-prompt.png`.*

![Two stacked prompt boxes contrasting the standard execution-based OpenHands system prompt with the execution-free SWE-Zero variant](../raw/images/14-data-filtering-dedup-mixing/swezero-prompt.png)

*The two system prompts side by side. The standard OpenHands setup has a five-step workflow (Exploration → Analysis → Testing → Implementation → Verification) and eight instruction phases. The SWE-Zero variant states that the development environment is **unavailable** and that the agent **cannot run Python code for any purpose**, drops Testing and Verification to leave three workflow steps and five phases, and names the prohibited bash commands outright:* `python`, `pytest`, `mypy`, `pip`, `apt`, `apt-get`.

### The result

*Figure: `images/swezero-results.png`.*

![Scatter plot of SWE-bench Verified resolve rate against model size on a log scale, for 23 baseline coding models plus six SWE-Zero and SWE-Hero checkpoints](../raw/images/14-data-filtering-dedup-mixing/swezero-results.png)

*Resolve rate against model size (log scale, 4B to 1024B). The three purple pairs are the paper's own: SWE-Zero-7B 46.8% → SWE-Hero-7B 52.7% (**+5.9**), SWE-Zero-14B 54.5% → SWE-Hero-14B 60.8% (**+6.3**), SWE-Zero-32B 57.5% → SWE-Hero-32B 62.2% (**+4.7**). The 23 blue points are baselines, from GLM-5 at 77.8% down to SERA-8B at 31.7%.*

Read the 32B cluster rather than the arrows and the claim gets stronger: at that
size the plotted baselines run from R2E-Gym-32B at 34.4% to daVinci-32B at 56.1%,
and **SWE-Zero-32B at 57.5% is above all of them** — before any execution-based
fine-tuning at all. Even the 72B daVinci is only 1.0 point ahead. The **+X.X**
labels measure SWE-Hero against SWE-Zero specifically, not against this field.

The lecture's own summary is more modest — "the frontier is still pretty high up,
but they were able to make some progress here" (≈1:21:01) — and the top-right of
the chart bears that out: GLM-5, Kimi-K2.5 and MiniMax-M2.5 sit at 75–78%.

## SWE-rebench

[arXiv 2505.20411](https://arxiv.org/pdf/2505.20411). "Essentially another attempt
to grab tons and tons of PRs" (≈1:21:47).

*Figure: `images/swe-rebench.png`.*

![Three-stage pipeline diagram of SWE-rebench dataset construction: preliminary filtering, environment setup with an LLM and validation loop, and LLM labeling](../raw/images/14-data-filtering-dedup-mixing/swe-rebench.png)

*Three stages: **preliminary filtering** merges GitHub repositories with GHArchive issue and PR metadata to extract candidate tasks; **environment setup** loops between an LLM hypothesizing a dependency-installation script and a validation step that tries to install the repo and run its tests; **LLM labeling** then labels instances by criteria, giving a final dataset of "21,000+ samples collected". That is the only number printed in the figure.*

- **21K interactive Python SWE tasks** from **3.4K GitHub repositories**.
- Drawn from **450K PRs** from GitHub and GitHub Archive.
- **Qwen 2.5-72B-Instruct** is used both to **install the dependencies** and to
  **assess PR quality**.

The pattern from [quality classifiers](quality-classifiers.md) recurs here in an
unexpected place: a model is doing the *infrastructure*, because hypothesizing an
install script across 3,400 repositories is not something anyone will do by hand.
The lecture's summary of the whole family is "get a bunch of GitHub repos, try to
install the repo — most of them probably fail — try harder, and then you use a
language model to give you the responses" (≈1:21:47).

Note the yield: 450K PRs in, 21K tasks out.

## SWE-ZERO-12M-trajectories

[Dataset on Hugging Face](https://huggingface.co/datasets/AlienKevin/SWE-ZERO-12M-trajectories).
The lecture flags this as brand new — "this one actually just came out today" (≈1:21:47)
— which places it at the lecture's own recording date in Spring 2026.

- **12M agent trajectories**, 40× SWE-Zero, made possible because the execution-free
  approach "it's almost — it's very lightweight".
- Built on the **SWE-rebench-v2** tasks: **32K executable + 120K non-executable**.
  This is the argument's payoff — SWE-rebench got only 32K of its tasks to execute,
  and "SWE-Zero doesn't care — you can use all of them" (≈1:22:34).
- Generated by **mini-coder-1.7b** — "a very small model, because now this dataset
  is quite large" — at **50.4 pass@100**, with the **mini-swe-agent** scaffold.

The trade is explicit: a 1.7B model at pass@100 produces usable trajectories far
more cheaply than a 480B one, and at this volume that matters more than per-sample
quality.

## The arc

The lecture reads the four papers as a progression (≈1:22:34):

> The datasets are getting more and more sophisticated. You go from environment-free
> things like math, to now coding, and now the coding datasets are growing quite a
> bit.

Alongside that, the environment problem is progressively routed around rather than
solved — a developer writes the Dockerfile (SWE-smith), then a model hypothesizes
the install script (SWE-rebench), then execution is dropped for most of the data
(SWE-Zero) and dropped for nearly all of it (SWE-ZERO-12M). The closing assessment
is unsentimental: "code environments are a pain, and there's a lot of filtering and
other details which we don't have time for" (≈1:23:20).

## See also

- [Post-training data](post-training-data.md) — the recipe and taxonomy these fit
  into, and OpenThoughts
- [Agentic benchmarks](agentic-benchmarks.md) — SWE-bench and what these datasets
  are trained to move
- [Lecture 14](14-data-filtering-dedup-mixing.md) — the lecture this comes from
- [Code data](code-data.md) — where code corpora come from in the first place
- [Synthetic data](synthetic-data.md) · [Quality classifiers](quality-classifiers.md)
  — the model-labels-then-model-learns pattern, one stage earlier
- [Course material for lecture 14](../raw/slides/14-data-filtering-dedup-mixing.md#post-training-data)
