---
title: Lecture 13 — Data I: Sources and Datasets (course material)
lecture: 13
source_format: executable-python
source_file: lecture_13.py
source_repo: https://github.com/stanford-cs336/lectures
source_url: https://raw.githubusercontent.com/stanford-cs336/lectures/main/lecture_13.py
rendered_url: https://cs336.stanford.edu/lectures/?trace=lecture_13
source_lines: 622
instructor: Percy Liang
note: >
  CS336's Percy-taught lectures are "executable lectures" — Python programs whose
  execution delivers the lecture content — rather than slide PDFs. There are no
  slide numbers. Sections below correspond to function definitions in
  lecture_13.py, and each carries the source line range so a claim can be checked
  against the program. Content is transcribed from the source text, which is the
  authoritative written form of this lecture.

  The program titles itself "Lecture 13: Data I". The course catalog lists it as
  "Data (Sources, Datasets)". Lecture 14 is the second half.
runtime_values: >
  This lecture computes NOTHING. Like lecture 12 and unlike lectures 2, 6, 7 and
  10, it has no @inspect values, no benchmarks, no sympy and no asserts — it is
  399 text() calls, 54 link() calls and 18 image() calls, and it runs in well
  under a second. Every number below is therefore a claim the lecture makes about
  a published dataset or paper, not a measurement taken on the machine that ran
  the program. Nothing here is machine-dependent, and nothing needed to be
  withheld or recomputed.
figures: >
  18 image() calls appear on 17 source lines (line 597 carries two). FOURTEEN
  live in the course's own repository (images/*.png); they have been copied into
  ../images/13-data-sources-datasets/ and are embedded below at the point they
  appear, each with a description written by looking at the image.

  FOUR are hot-linked to third-party sites and are NOT copied. They are recorded
  as URLs at the point they appear, without a description, because the
  transcription was made from the source text and those images were not
  redistributed. The four are: the Wikipedia/Wikimedia web-crawler architecture
  diagram, two figures served from the Stanford CS324 (Winter 2022) course site —
  the C4 domain breakdown and the Pile composition chart — and one Dolma
  composition figure hot-linked to a Medium CDN.

  This lecture is much LESS image-driven than lecture 12. Its argument is carried
  by the text; the figures illustrate it rather than constituting it. A reader who
  reads only the text() lines here loses much less than in lecture 12.
figure_audit: >
  The 14 course-repo descriptions were written by one reader looking at the
  images; two were then re-checked in the parent by direct inspection. Findings
  are in the "Figure audit" section at the foot of this file. Read it before
  quoting any figure: it records one real mismatch between a figure and the
  lecture text that cites it (decline-consent.png), one corrected summarising
  sentence (comma-results.png), and eight reader flags about images that are not
  the kind of object their filename or the surrounding text implies.
dating: >
  The lecture states several statistics "as of May 2026" — Wikipedia's 67 million
  articles across 361 language editions, GitHub's 420M+ repositories (28M public),
  and Software Heritage's 28.8M source files — and cites an "April 2026 Crawl" of
  Common Crawl at 2.19 billion pages (372.2 TB). Treat all of these as snapshots
  from when the lecture was prepared (Spring 2026), not as current figures.
unused_import: >
  The module imports alpaca_2023 from references.py and never uses it. Noted so
  that nobody later reads its absence from this file as a transcription gap.
---

# Lecture 13 — Data I: Sources and Datasets

Percy Liang. Source: [`lecture_13.py`](https://github.com/stanford-cs336/lectures/blob/main/lecture_13.py),
622 lines. Rendered by the course's trace viewer at
[`?trace=lecture_13`](https://cs336.stanford.edu/lectures/?trace=lecture_13).

Lecture 12 asked how you tell whether a model is good. This lecture and the next
ask what you train it on. The program's own framing is two lines:

> Previous lectures: how to train a model *given data*
>
> Next two lectures: *what data* should we train on?

The lecture has an unusual shape for this course. It is not a derivation and it
is not a systems walkthrough — it is **a history and a survey**, moving through
where text physically comes from, what law governs its use, the handful of raw
sources everything is built on, and then a chronological tour of fifteen or so
named pre-training datasets from BooksCorpus (2015) to CommonPile (2025). The
recurring argument is stated in the summary and is worth having in mind from the
start:

> Key lesson: Data does not fall from the sky. You have to work to get it.

The spoken lecture follows this program closely but not exactly — Percy digresses,
takes questions, and expands points the source states in one line. For what was
*said*, see [the transcript](../transcripts/13-data-sources-datasets.md). For what
was *written*, use this file.

## Sections → source lines

| Section | Function | Source lines |
| --- | --- | --- |
| [Roadmap](#roadmap) | `main` | 6–46 |
| [Motivation: why data is the thing to get right](#motivation-why-data-is-the-thing-to-get-right) | `motivation` | 48–88 |
| [Where data actually comes from](#where-data-actually-comes-from) | `raw_sources` | 90–148 |
| [Copyright](#copyright) | `copyright` | 150–239 |
| [Common Crawl](#common-crawl) | `common_crawl` | 241–272 |
| [Wikipedia](#wikipedia) | `wikipedia` | 274–295 |
| [GitHub](#github) | `github` | 297–316 |
| [arXiv](#arxiv) | `arxiv` | 318–328 |
| [BERT (2019)](#bert-2019) | `bert` | 330–340 |
| [BooksCorpus](#bookscorpus) | `books_corpus` | 342–351 |
| [WebText and GPT-2 (2019)](#webtext-and-gpt-2-2019) | `gpt2_webtext` | 353–362 |
| [CCNet (2019)](#ccnet-2019) | `ccnet` | 364–377 |
| [C4 and T5 (2019)](#c4-and-t5-2019) | `t5_c4` | 379–405 |
| [GPT-3 (2020)](#gpt-3-2020) | `gpt3` | 407–419 |
| [The Pile (2021)](#the-pile-2021) | `the_pile` | 421–438 |
| [Project Gutenberg](#project-gutenberg) | `project_gutenberg` | 440–447 |
| [Books3](#books3) | `books3` | 449–455 |
| [Stack Exchange](#stack-exchange) | `stackexchange` | 457–467 |
| [MassiveText and Gopher (2021)](#massivetext-and-gopher-2021) | `gopher_massivetext` | 469–487 |
| [LLaMA (2023)](#llama-2023) | `llama` | 489–502 |
| [RefinedWeb and FineWeb (2023)](#refinedweb-and-fineweb-2023) | `refinedweb` | 504–521 |
| [Dolma (2024)](#dolma-2024) | `dolma` | 523–537 |
| [DataComp-LM / DCLM (2024)](#datacomp-lm--dclm-2024) | `dclm` | 539–557 |
| [Nemotron-CC (2024)](#nemotron-cc-2024) | `nemotron_cc` | 559–576 |
| [The Stack (2022) and Stack v2 (2024)](#the-stack-2022-and-stack-v2-2024) | `the_stack` | 578–598 |
| [CommonPile (2025)](#commonpile-2025) | `common_pile` | 600–620 |
| [Summary](#summary) | `main` | 40–45 |
| [Figure audit](#figure-audit) | — | — |

## Datasets named in this lecture

Every named dataset, with the size the lecture gives it and the citation the
source carries. This table is a navigation aid; the detail is in the sections.
Sizes are quoted exactly as the lecture states them — note that it mixes bytes,
words, tokens and document counts, and does not always give the same unit twice.

| Dataset | Year | Built from | Size as stated | Citation in source |
| --- | --- | --- | --- | --- |
| One Billion Word Benchmark | 2013 | machine-translation sentences | — | [Chelba+ 2013] (named, not linked) |
| BooksCorpus | 2015 | free self-published Smashwords e-books | 7K books, 985M words | [arXiv 1506.06724](https://arxiv.org/abs/1506.06724) |
| BERT training data | 2019 | Wikipedia + Books | — | [arXiv 1810.04805](https://arxiv.org/pdf/1810.04805) |
| WebText | 2019 | Reddit outlinks with ≥ 3 karma | 8M pages, 40 GB text | GPT-2 paper (Radford+ 2019) |
| OpenWebTextCorpus | 2019 | open replication of WebText | — | [OpenWebText](https://skylion007.github.io/OpenWebTextCorpus/) |
| CCNet | 2019 | Common Crawl, Wikipedia-like KenLM filter | — | [arXiv 1911.00359](https://arxiv.org/pdf/1911.00359) |
| C4 (Colossal Clean Crawled Corpus) | 2019 | one April 2019 Common Crawl snapshot | 806 GB (156B tokens) | [arXiv 1910.10683v4](https://arxiv.org/pdf/1910.10683v4) |
| GPT-3 dataset | 2020 | CC + WebText2 + Books1/Books2 + Wikipedia | 570 GB (400B tokens) | [arXiv 2005.14165](https://arxiv.org/pdf/2005.14165) |
| Books3 | 2020 | the Bibliotik shadow library | 196K books | [paperswithcode](https://paperswithcode.com/dataset/books3) (taken down) |
| The Pile | 2021 | 22 curated domains | 825 GB (~275B tokens) | [arXiv 2101.00027](https://arxiv.org/pdf/2101.00027) |
| PG-19 | — | Project Gutenberg books before 2019 | — | [pg19 repo](https://github.com/google-deepmind/pg19) |
| Project Gutenberg | 1971– | copyright-cleared books | ~75K books (2025) | [gutenberg.org](https://www.gutenberg.org/) |
| MassiveText (Gopher) | 2021 | MassiveWeb, C4, Books, News, GitHub, Wikipedia | 10.5 TB (Gopher trained on 300B tokens ≈ 12%) | Gopher, [arXiv 2112.11446](https://arxiv.org/pdf/2112.11446.pdf) |
| The Stack | 2022 | GitHub Archive repo names, permissive licenses only | 3.1 TB of code | [arXiv 2211.15533](https://arxiv.org/pdf/2211.15533) |
| LLaMA dataset | 2023 | CC/CCNet, C4, GitHub, Wikipedia, Gutenberg + Books3, arXiv, Stack Exchange | 1.2T tokens | [arXiv 2302.13971](https://arxiv.org/pdf/2302.13971) |
| RedPajama v1 | 2023 | Together's reproduction of the LLaMA recipe | — | [HF dataset](https://huggingface.co/datasets/togethercomputer/RedPajama-Data-1T) |
| SlimPajama | 2023 | deduplicated (MinHashLSH) subset of RedPajama v1 | 627B tokens | [Cerebras blog](https://www.cerebras.ai/blog/slimpajama-a-627b-token-cleaned-and-deduplicated-version-of-redpajama) |
| RefinedWeb | 2023 | Common Crawl WARC only, Gopher rules, MinHash dedup | 600B released (of 5T) | [arXiv 2306.01116](https://arxiv.org/pdf/2306.01116) |
| FineWeb | 2024 | 95 Common Crawl dumps | 15T tokens | [HF dataset](https://huggingface.co/datasets/HuggingFaceFW/fineweb) |
| Dolma | 2024 | CC, Reddit/Pushshift, PeS2o, C4, Gutenberg, Wikipedia | 3T tokens | [arXiv 2402.00159](https://arxiv.org/pdf/2402.00159) |
| DCLM-pool | 2024 | processed Common Crawl | 240T tokens | DCLM, [arXiv 2406.11794](https://arxiv.org/abs/2406.11794) |
| DCLM-baseline | 2024 | DCLM-pool filtered by a fastText quality classifier | 3.8T tokens | DCLM, [arXiv 2406.11794](https://arxiv.org/abs/2406.11794) |
| Nemotron-CC | 2024 | classifier ensemble + synthetic rephrasing over CC | 6.3T tokens (HQ subset 1.1T) | [arXiv 2412.02595](https://arxiv.org/abs/2412.02595) |
| Stack v2 | 2024 | Software Heritage + GitHub Archive + crawled docs | — | [arXiv 2402.19173](https://arxiv.org/abs/2402.19173) |
| CommonPile | 2025 | permissively licensed sources only | 8 TB | [arXiv 2506.05209](https://arxiv.org/pdf/2506.05209) |

Two reference points the lecture gives for scale, both stated in the Nemotron-CC
section: **Llama 3 trained on 15T tokens and Qwen3 on 36T.**

## Roadmap

*Source: `main`, lines 6–46.*

> ## Lecture 13: Data I

The program opens with its place in the course and then calls its sections in
order. The call list is itself the lecture's outline, and the source comments on
each call are the one-line summaries:

**Origin of data**

- `raw_sources()` — What does data come from?
- `copyright()` — What data can we use?

**Sources of data**

- `common_crawl()` — Web crawl
- `wikipedia()` — General knowledge
- `github()` — Code
- `arxiv()` — Research papers

**Data from various models** (in the order the program lists them, with its own
dating and annotations)

- `bert()` — Wikipedia, books (trained BERT) [2019]
- `gpt2_webtext()` — pages based on Reddit links (trained GPT-2) [2019]
- `ccnet()` — Filter Common Crawl based on Wikipedia [2019]
- `t5_c4()` — Filter using rules (trained T5) [2019]
- `gpt3()` — CommonCrawl, Wikipedia, books (trained GPT-3) [2020]
- `the_pile()` — Lots of sources (trained GPT-J, GPT-NeoX, ...) [2021]
- `gopher_massivetext()` — Filter using rules (trained Gopher) [2021]
- `llama()` — CommonCrawl, CCNet, StackExchange, etc. (trained LLaMA) [2022]
- `refinedweb()` — CommonCrawl (used to train Falcon) [2023]
- `dolma()` — Lots of different sources [2024]
- `dclm()` — Filtered using good quality classifier [2024]
- `nemotron_cc()` — Lots of tokens [2024]
- `the_stack()` — Code dataset
- `common_pile()` — Properly licensed data

Note that the source comment dates LLaMA to 2022 while the LLaMA section itself
links the February 2023 paper; the Wikipedia component is dated "June–August
2022" in that section, which is likely what the comment is tracking.

## Summary

*Source: `main`, lines 40–45. Placed here, out of program order, because it is the
lecture's own statement of what it is arguing; the program prints it last.*

> ### Summary
>
> - Key lesson: Data does not fall from the sky. You have to work to get it.
> - Live service → raw data → processed data (transformation, filtering, deduplication)
> - Data is the key ingredient that differentiates language models
> - Legal and ethical issues (e.g., copyright and privacy)
> - Much of this pipeline is heuristic, many opportunities to improve!

The second line is the pipeline the whole lecture traces, and it is worth reading
as three distinct stages rather than one: a **live service** (a website, a code
host, a Q&A forum) is not data; **raw data** is what a crawler or a bulk dump
gets you; **processed data** is what you can train on, and everything between the
second and third is transformation, filtering and deduplication.

## Motivation: why data is the thing to get right

*Source: `motivation`, lines 48–88.*

> **Data** is the most important thing to get right in training language models.

The lecture's first argument for that claim is an argument from secrecy — look at
what companies choose not to tell you:

- One justification: let's see what companies disclose.
- Open-weight models (e.g., Llama 3, [arXiv 2407.21783](https://arxiv.org/abs/2407.21783)) have full transparency into architecture
- ...and even training procedures
- ...but basically no information on data.

*Figure: `images/llama3-data.png` (width 700).*

![Llama 3 paper section 3.1 "Pre-Training Data" — describes cleaning methods but names no actual sources](../images/13-data-sources-datasets/llama3-data.png)

**What the image shows.** A screenshot (plain text on white background, academic-paper typesetting) of section "3.1 Pre-Training Data" from the Llama 3 technical report. It contains a heading and one paragraph of body text, quoted here in full: "We create our dataset for language model pre-training from a variety of data sources containing knowledge until the end of 2023. We apply several de-duplication methods and data cleaning mechanisms on each data source to obtain high-quality tokens. We remove domains that contain large amounts of personally identifiable information (PII), and domains with known adult content."

This is not a chart, table, or diagram — it is a cropped screenshot of paper prose, no figures or numbers of any kind.

The image directly supports the lecture's claim: the paragraph asserts that data is drawn "from a variety of data sources" and describes generic processing steps (de-duplication, PII removal, adult-content filtering), but it names no actual source (no mention of Common Crawl, Wikipedia, books, code, etc.), gives no dataset sizes, and gives no percentages/mixture weights. So the text is a real example of disclosing *that* cleaning happened without disclosing *what* the data actually is.

**Before citing this figure.** none — the image content matches what the lecture text claims it shows.

*Source: [`images/llama3-data.png`](https://github.com/stanford-cs336/lectures/blob/main/images/llama3-data.png) in the lectures repo.*

> Reasons for secrecy:
>
> 1. Competitive dynamics
> 2. Copyright liability

Then the historical point about what "data work" means, and why it does not get
easier:

- Before foundation models, data work meant heavy annotation of labeled data for
  supervised learning.
- Now there's less annotation, but there's still a lot of curation and cleaning.
- Data is fundamentally a long-tail problem, scales with human effort (unlike
  architectures, systems).

That last parenthesis is the load-bearing one. Architectures and systems work has
a small number of ideas that generalise; data work does not, so it absorbs as
much human effort as you can put into it. This is the lecture's explanation for
why data teams at model developers are large.

### Stages of training

> 1. Pre-training: train on raw text (e.g., documents from the web)
> 2. Mid-training: train more on high quality data to enhance capabilities
> 3. Post-training: train on chat transcripts or reinforcement learning
>
> In practice, the lines are blurry and there could be more stages
> ...but the basic trend is throughout training, we go from
> large amounts of lower quality data to
> small amounts of high quality data.

**Terminology:**

- Base model: after pre-training + mid-training
- Instruct/chat model: after post-training
- (Increasingly, base models are not released — e.g., Qwen3.5-397B-A17B is an
  instruct model.)

### Example: OLMo from AI2

The lecture uses OLMo ([arXiv 2501.00656](https://arxiv.org/abs/2501.00656), AI2)
as the worked example of a fully open pipeline, showing one figure per stage.

**1. Pre-training**

*Figure: `images/olmo2-pretraining.png` (width 600).*

![Table of the OLMo 2 1124 pretraining data mix, 7 sources, columns for tokens/words/bytes/docs](../images/13-data-sources-datasets/olmo2-pretraining.png)

**What the image shows.** A data table (not a chart) titled "Pretraining ✦ OLMo 2 1124 Mix", with header row: Source | Type | Tokens | Words | Bytes | Docs. It lists 7 source rows plus a bolded Total row:

| Source | Type | Tokens | Words | Bytes | Docs |
|---|---|---|---|---|---|
| DCLM-Baseline | Web pages | 3.71T | 3.32T | 21.32T | 2.95B |
| StarCoder (filtered version from OLMoE Mix) | Code | 83.0B | 70.0B | 459B | 78.7M |
| peS2o (from Dolma 1.7) | Academic papers | 58.6B | 51.1B | 413B | 38.8M |
| arXiv | STEM papers | 20.8B | 19.3B | 77.2B | 3.95M |
| OpenWebMath | Math web pages | 12.2B | 11.1B | 47.2B | 2.89M |
| Algebraic Stack | Math proofs code | 11.8B | 10.8B | 44.0B | 2.83M |
| Wikipedia & Wikibooks (from Dolma 1.7) | Encyclopedic | 3.7B | 3.16B | 16.2B | 6.17M |
| **Total** | | **3.90T** | **3.48T** | **22.38T** | **3.08B** |

Some rows carry a small grey secondary line under the source name giving provenance ("filtered version from OLMoE Mix" for StarCoder; "from Dolma 1.7" for peS2o and for Wikipedia & Wikibooks) — this is annotation text, not a separate column or series. The overwhelming majority of the token budget is DCLM-Baseline web pages (3.71T of the 3.90T total, i.e. ~95%); all other sources are much smaller (tens of billions of tokens each).

**Before citing this figure.** none — all values are clearly legible at normal resolution; no zooming issues.

*Source: [`images/olmo2-pretraining.png`](https://github.com/stanford-cs336/lectures/blob/main/images/olmo2-pretraining.png) in the lectures repo.*

**2. Mid-training**

*Figure: `images/olmo2-dolmino.png` (width 600).*

![Two stacked tables of the OLMo 2 "Dolmino" mid-training mix: high-quality subset and math mix, with totals](../images/13-data-sources-datasets/olmo2-dolmino.png)

**What the image shows.** Two data tables (not charts) stacked vertically, sharing the same column header row: Source | Type | Tokens | Words | Bytes | Docs.

Table 1, banded "Mid-Training ✦ Dolmino High Quality Subset":

| Source | Type | Tokens | Words | Bytes | Docs |
|---|---|---|---|---|---|
| DCLM-Baseline (FastText top 7%, FineWeb ≥ 2) | High quality web | 752B | 670B | 4.56T | 606M |
| FLAN (from Dolma 1.7, decontaminated) | Instruction data | 17.0B | 14.4B | 98.2B | 57.3M |
| peS2o (from Dolma 1.7) | Academic papers | 58.6B | 51.1B | 413B | 38.8M |
| Wikipedia & Wikibooks (from Dolma 1.7) | Encyclopedic | 3.7B | 3.16B | 16.2B | 6.17M |
| Stack Exchange (09/30/2024 dump, curated Q&A data) | Q&A | 1.26B | 1.14B | 7.72B | 2.48M |
| **High quality total** | | **832.6B** | **739.8B** | **5.09T** | **710.8M** |

Table 2, banded "Mid-training ✦ Dolmino Math Mix":

| Source | Type | Tokens | Words | Bytes | Docs |
|---|---|---|---|---|---|
| TuluMath | Synthetic math | 230M | 222M | 1.03B | 220K |
| Dolmino SynthMath | Synthetic math | 28.7M | 35.1M | 163M | 725K |
| TinyGSM-MIND | Synthetic math | 6.48B | 5.68B | 25.52B | 17M |
| MathCoder2 Synthetic (Ajibawa-2023, M-A-P Matrix) | Synthetic Math | 3.87B | 3.71B | 18.4B | 2.83M |
| Metamath (OWM-filtered) | Math | 84.2M | 76.6M | 741M | 383K |
| CodeSearchNet (OWM-filtered) | Code | 1.78M | 1.41M | 29.8M | 7.27K |
| GSM8K (Train split) | Math | 2.74M | 3.00M | 25.3M | 17.6K |
| **Math total** | | **10.7B** | **9.73B** | **45.9B** | **21.37M** |

Small grey secondary lines under several source names give provenance/filtering notes (e.g. "FastText top 7%, FineWeb ≥ 2" for DCLM-Baseline; "OWM-filtered" for Metamath and CodeSearchNet) — these are annotations, not separate columns. Note peS2o and Wikipedia & Wikibooks rows carry the identical token/word/byte/doc figures seen in olmo2-pretraining.png (58.6B and 3.7B tokens respectively), i.e. the same sub-source is reused across pretraining and mid-training tables.

**Before citing this figure.** The high-quality-subset DCLM-Baseline row's Bytes value (4.56T) is much larger than its Tokens (752B) and Words (670B) values — a byte:token ratio of roughly 6:1, versus roughly 5.7:1 for the pretraining-table DCLM-Baseline row (21.32T bytes / 3.71T tokens). This is plausible but worth flagging since it's a large absolute number sitting next to two "B"-scale ones in the same row.

*Source: [`images/olmo2-dolmino.png`](https://github.com/stanford-cs336/lectures/blob/main/images/olmo2-dolmino.png) in the lectures repo.*

**3. Post-training** ([arXiv 2411.15124](https://arxiv.org/pdf/2411.15124) — Tülu 3)

*Figure: `images/tulu.png` (width 600).*

![Tülu 3 post-training prompt-mix table: category, dataset, raw count, #SFT, #DPO, and citation per row](../images/13-data-sources-datasets/tulu.png)

**What the image shows.** A large data table (not a chart) from the Tülu 3 paper, listing the prompt datasets that make up the post-training mixture. Header row: Category | Prompt Dataset | Count | # Prompts used in SFT | # Prompts used in DPO | Reference. Rows are grouped by category with alternating shaded/unshaded bands; dataset names newly introduced by the Tülu 3 authors are printed in magenta/pink bold text, while pre-existing external datasets are in black with a grey citation in the Reference column. Superscript symbols (↑, ↓, numerals 1/2, α, γ) appear next to several dataset names — these are footnote markers from the source paper; the footnote text itself is not visible in this crop, so their exact meaning cannot be confirmed here.

Full transcription:

**General** — Tülu 3 Hardcoded↑: 24 / 240 / – / –; OpenAssistant¹,²,↓: 88,838 / 7,132 / 7,132 / Köpf et al. (2024); No Robots: 9,500 / 9,500 / 9,500 / Rajani et al. (2023); WildChat (GPT-4 subset)↓: 241,307 / 100,000 / 100,000 / Zhao et al. (2024); UltraFeedbackᵅ,²: 41,635 / – / 41,635 / Cui et al. (2023).

**Knowledge Recall** — FLAN v2¹,²,↓: 89,982 / 89,982 / 12,141 / Longpre et al. (2023); SciRIFF↓: 35,357 / 10,000 / 17,590 / Wadden et al. (2024); TableGPT↓: 13,222 / 5,000 / 6,049 / Zha et al. (2023).

**Math Reasoning** — Tülu 3 Persona MATH: 149,960 / 149,960 / – / –; Tülu 3 Persona GSM: 49,980 / 49,980 / – / –; Tülu 3 Persona Algebra: 20,000 / 20,000 / – / –; OpenMathInstruct 2↓: 21,972,791 / 50,000 / 26,356 / Toshniwal et al. (2024); NuminaMath-TIRᵅ: 64,312 / 64,312 / 8,677 / Beeching et al. (2024).

**Coding** — Tülu 3 Persona Python: 34,999 / 34,999 / – / –; Evol CodeAlpacaᵅ: 107,276 / 107,276 / 14,200 / Luo et al. (2023).

**Safety & Non-Compliance** — Tülu 3 CoCoNot: 10,983 / 10,983 / 10,983 / Brahman et al. (2024); Tülu 3 WildJailbreakᵅ,↓: 50,000 / 50,000 / 26,356 / Jiang et al. (2024); Tülu 3 WildGuardMixᵅ,↓: 50,000 / 50,000 / 26,356 / Han et al. (2024).

**Multilingual** — Aya↓: 202,285 / 100,000 / 32,210 / Singh et al. (2024b).

**Precise IF** — Tülu 3 Persona IF: 29,980 / 29,980 / 19,890 / –; Tülu 3 IF-augmented: 65,530 / – / 65,530 / –.

**Total** (bottom row, italic): 23,327,961 / 939,344 / 425,145ᵞ.

**Before citing this figure.** The raw "Count" column is dominated almost entirely by a single row — OpenMathInstruct 2 has a raw count of 21,972,791, which is ~94% of the grand total of 23,327,961 — yet only 50,000 of its prompts are actually used in SFT and 26,356 in DPO, comparable to many far-smaller-count rows. A reader citing "23.3 million total prompts" without noting this would be citing a number that is almost entirely one oversized raw source, not a reflection of the actual training mixture (which is much better characterized by the 939,344 SFT / 425,145 DPO columns). Also, the meaning of the superscript footnote markers (↑, ↓, ¹, ², α, γ) is not visible in this crop and is not stated in the description above.

*Source: [`images/tulu.png`](https://github.com/stanford-cs336/lectures/blob/main/images/tulu.png) in the lectures repo.*

The section closes with the question the rest of the lecture answers:

> What are these datasets? How are they chosen and processed?

## Where data actually comes from

*Source: `raw_sources`, lines 90–148.*

The section takes apart a piece of received wisdom:

> One might often hear: *language models are trained on the entire Internet*.
>
> Slightly more accurately, ~Internet~ public (world wide) web.
>
> But this is not quite right either...

(The source uses `~Internet~` strikethrough to correct itself mid-sentence.)

The correction proceeds in steps. First, the web is not a corpus:

> First, the web consists of a set of live servers that one can connect to:
>
> `$ curl https://cs336.stanford.edu/`
>
> You can't train on live servers.

So you need a **crawler**, which:

- Discovers webpages (starting from a seed set)
- Downloads the discovered webpages

> However, you can't download and train on all the webpages.

And then four categories of reason why not, which together are the section's real
content:

**Dynamic content**

- Many sites these days are apps
- URL doesn't change
- Need to click buttons and submit forms to access content
- Examples: Discord, wandb

**Authentication**

- Sometimes need login with an account (and pay usually)
- Example: Facebook, X, LinkedIn, NYTimes (huge content behind walled gardens)

**Technical restrictions**

- Not allowed to download some content based on `robots.txt`
  ([example](https://www.nytimes.com/robots.txt)) (voluntary)
- Website might use Cloudflare to detect and block bot activity (present CAPTCHAs)
- Website might block certain IP addresses / countries
- Website might have rate limits

**Legal restrictions**

- Terms of service (ToS) might prohibit downloading using bots
- You might not have a license to copy the webpages (for training)

Note the word **(voluntary)** against `robots.txt`: the lecture is explicit that
it is a convention rather than an enforcement mechanism, which is what makes the
next two items interesting.

### Decline of consent

[arXiv 2407.14933](https://arxiv.org/abs/2407.14933)

- Examined restrictions (robots.txt, ToS) for URLs in common datasets (C4,
  RefinedWeb, Dolma)
- Restrictions have increased over time

*Figure: `images/decline-consent.png` (width 700).*

![Three stacked charts (2016-2025) tracking robots.txt/ToS restriction rates and per-crawler-agent restriction rates over time](../images/13-data-sources-datasets/decline-consent.png)

**What the image shows.** A composite of three separate charts (from the "Consent in Crisis" / Data Provenance Initiative paper on web-crawling restrictions), each spanning roughly 2016 through a 2025 forecast region (shaded grey, entered around mid-2024, with data shown as dashed lines in that region for chart 3).

**Chart 1, "Robots.txt Restrictions"** — a 100%-stacked area chart with 8 categories (legend, in listed order): Full restrictions (dark red), Pattern-based restrictions (red-orange), Disallow private directories (tan), Other restrictions (pale peach), Crawl delay specified (pale blue), Sitemap provided (medium blue), No restrictions or sitemap (darker blue), No Robots.txt (grey). Reading the bands bottom-to-top on the plot, grey ("No Robots.txt") and blue bands shrink steadily from 2016 to 2024 while the reddish bands at top grow, with a visibly steeper shift starting around late 2023. Four vertical reference lines are annotated: ChatGPT (~Nov 2022), GPT-4 (~Mar 2023), GPTBot (~Aug 2023), G-Ext. (Google-Extended, ~Sep 2023) — these are event markers, not data series.

**Chart 2, "ToS Restrictions"** — a second 100%-stacked area chart with 9 categories (legend, two rows): No Crawling & AI, No Crawling, No AI, Non-Commercial Use, Non-Compete, No Re-Distribution, Conditional Use, Unrestricted Use, No Terms Pages. The grey "No Terms Pages" band shrinks from ~90% in 2016 to under 20% by 2024/2025. This chart carries six vertical reference lines: GDPR Ad. (GDPR adopted, ~early 2016), GDPR Eff. (GDPR effective, ~mid-2018), plus the same ChatGPT/GPT-4/GPTBot/G-Ext. markers as chart 1.

**Chart 3, "Restrictions by Org. Agent"** — a line chart on a log-scaled y-axis (0.1% to 100%) with 9 series, one dot-marker line per named crawler/organization, colour-coded per the legend "Restrictions by Org. Agent": OpenAI (black, 25.9%), Anthropic (light blue, 13.3%), Common Crawl (yellow, 13.3%), Google (orange, 9.8%), False Anthropic (light grey, 6.0%), Cohere (teal/green, 4.9%), Meta (dark blue, 4.1%), Internet Archive (dark orange/red, 3.2%), Google Search (pink/magenta, 1.0%). The percentages in the legend are each series' final/endpoint restriction-rate value. All series sit under ~2% until roughly 2022, then several (led by OpenAI, then Common Crawl, Google, False Anthropic) rise sharply toward the right edge of the observed data (~2024) and continue upward as dashed forecast lines into 2025; Internet Archive and Google Search stay comparatively flat and low throughout. The same four event markers (ChatGPT, GPT-4, GPTBot, G-Ext.) are repeated on this chart as vertical lines.

**Before citing this figure.** **Likely mismatch with the surrounding lecture text.** The lecture script frames this figure as examining restrictions "for URLs in common datasets (C4, RefinedWeb, Dolma)," but nothing in this image breaks results out by dataset — there is no C4, RefinedWeb, or Dolma series or panel anywhere in the three charts shown. Instead, the three panels break restrictions down by (1) robots.txt category composition over time, (2) ToS category composition over time, and (3) restriction rate specifically for named AI/search organizations' crawler user-agents (OpenAI, Anthropic, Google, Common Crawl, Meta, Cohere, Internet Archive, Google Search). If the source paper elsewhere also produces a per-corpus (C4/RefinedWeb/Dolma) version of this chart, that version is not what is embedded here — a reader should not cite this image as showing per-dataset restriction rates.

*Source: [`images/decline-consent.png`](https://github.com/stanford-cs336/lectures/blob/main/images/decline-consent.png) in the lectures repo.*

### When crawlers are not well-behaved

*Figure: `images/anthropic-crawling.png` (width 500).*

![Screenshot of a 2024 X/Twitter thread complaining that Anthropic's crawler hit servers a million times in 24 hours](../images/13-data-sources-datasets/anthropic-crawling.png)

**What the image shows.** A screenshot of a two-post X (Twitter) thread — not a chart or data figure.

Post 1, from Kyle Wiens (@kwiens, verified), dated Jul 24, 2024: "Hey @AnthropicAI: I get you're hungry for data. Claude is really smart! But do you really need to hit our servers a million times in 24 hours? You're not only taking our content without paying, you're tying up our devops resources. Not cool." Engagement counts shown: 90 replies, 857 reposts, 10K likes, 1.6M views.

Post 2, a reply from Eric Holscher (@ericholscher), timestamped 1:31 PM · Jul 24, 2024 · 111.8K views: "Yea, they were hammering us over at @readthedocs as well. We were planning to write a blog post on it, since this behavior is definitely gonna get all AI crawlers blocked because of abuse, not even because of the copyright issues." Engagement counts: 7 replies, 31 reposts, 732 likes, 59 bookmarks.

This is anecdotal social-media evidence (two named individuals' public complaints), not measured data — it directly illustrates the lecture's point about badly-behaved crawlers overloading servers, but it should be cited as a single self-reported anecdote/complaint, not as a quantitative study.

**Before citing this figure.** none — the image is exactly what the surrounding lecture text implies (a real complaint about crawler behavior), just note it is qualitative/anecdotal (a tweet), not measured server-load data.

*Source: [`images/anthropic-crawling.png`](https://github.com/stanford-cs336/lectures/blob/main/images/anthropic-crawling.png) in the lectures repo.*

- Factors: ToS, robots.txt, server load (degrades service, costs website money)
- And then there is copyright (more later)...

### Shadow libraries

[Wikipedia: Shadow library](https://en.wikipedia.org/wiki/Shadow_library)

- Technically part of the web
- Examples: Library Genesis (LibGen), Z-Library, Anna's Archive, Sci-Hub
- Disregards copyright and bypasses paywalls (e.g., Elsevier)
- Received takedown orders, lawsuits, blocked in various countries
- Usually controls are circumvented, have servers in various countries
- Some argue this makes freely available what should be free
- From a legal perspective, this is piracy and copyright infringement
- LibGen has ~4M books (2019), Sci-Hub has ~88M papers (2022)

Shadow libraries return twice later in the lecture — as the origin of Books3, and
as the fact pattern in the Anthropic and Meta lawsuits.

**Summary:**

- The Internet is huge
- Many technical and legal restrictions on what data one can access

## Copyright

*Source: `copyright`, lines 150–239. This is the longest section of the lecture
(90 source lines) and the one furthest from the rest of the course's material.*

> What data is legal to use (for training)?

### Intellectual property law

- Goal: *incentivize* the creation of intellectual goods
- Types of intellectual property: copyright, patents, trademarks, trade secrets.

**Copyright law:**

- Goes back to 1709 in England (Statute of Anne), first time regulated by
  governments and courts
  ([Wikipedia](https://en.wikipedia.org/wiki/Statute_of_Anne))
- In United States, most recent: Copyright Act of 1976
  ([Wikipedia](https://en.wikipedia.org/wiki/Copyright_Act_of_1976))
- Copyright protection applies to *"original works of authorship fixed in any
  tangible medium of expression, now known or later developed, from which they can
  be perceived, reproduced, or otherwise communicated, either directly or with the
  aid of a machine or device"*

That quoted phrase is the statutory language, and the lecture pulls three
consequences out of it:

- Collections are not original works so hence not copyrightable (e.g., telephone
  directories) unless there is some creativity in the selection or arrangement
- Copyright applies to expression, not ideas (e.g., quicksort)
- Expanded scope from "published" (1909) to "fixed" (1976)

And then the practicalities:

- Registration not required for copyright protection (in contrast with patents)
- Threshold for copyright is extremely low (e.g., your website is copyrighted)
- Registration is required before creator can sue someone for copyright
  infringement
- Costs $65 to register ([copyright.gov fees](https://www.copyright.gov/about/fees.html))
- Lasts for 75 years, and then the copyright expires and it becomes part of the
  public domain (works of Shakespeare, Beethoven, most of Project Gutenberg, etc.)

> Summary: *basically everything on the Internet are copyrighted.*

That sentence is the pivot of the section. If everything is copyrighted by
default, then every dataset in the second half of this lecture needs an answer to
the question of how it is allowed to exist, and the lecture gives exactly two:

> How to use a copyrighted work:
>
> 1. Get a license for it.
> 2. Appeal to the fair use clause.

### Licenses

- A license (from contract law) is granted by a licensor to a licensee.
- Effectively, "a license is a promise not to sue".
- The Creative Commons license enables free distribution of copyrighted work.
- Examples: Wikipedia, Open Courseware, Khan Academy, Free Music Archive, 307
  million images from Flickr, 39 million images from MusicBrainz, 10 million
  videos from YouTube, etc.
- Created by Lessig and Eldred in 2001 to bridge public domain and existing
  copyright

**Many model developers license data for training foundation models:**

- Google and Reddit
  ([Reuters](https://www.reuters.com/technology/reddit-ai-content-licensing-deal-with-google-sources-say-2024-02-22/))
- OpenAI and Shutterstock
  ([Shutterstock investor release](https://investor.shutterstock.com/news-releases/news-release-details/shutterstock-expands-partnership-openai-signs-new-six-year))
- OpenAI and StackExchange
  ([Stack Overflow press](https://stackoverflow.co/company/press/archive/openai-partnership))

### Fair use (section 107)

> Four factors to determine whether fair use applies:
>
> 1. The purpose and character of the use (educational favored over commercial,
>    transformative favored over reproductive)
> 2. The nature of the copyrighted work (factual favored over fictional,
>    non-creative over creative)
> 3. The amount and substantiality of the portion of the original work used (using
>    a snippet favored over using the whole work)
> 4. The effect of the use upon the market (or potential market) for the original
>    work

**Examples of fair use:**

- You watch a movie and write a summary of it
- Reimplement an algorithm (the idea) rather than copying the code (the
  expression)
- Google Books index and show snippets (Authors Guild v. Google 2002–2013)

**Copyright is not about verbatim memorization:**

- Plots and characters (e.g., Harry Potter) can be copyrightable
- Parody (imitating to make fun of something) is likely fair use

> Copyright is about semantics (and economics).

### Considerations for language models

The four factors applied to training, which is the part a course on building
language models actually needs:

- Copying data (first step of training) is violation already even if you don't do
  anything with it.
- Training a model should be transformative (far from just copy/pasting).
- Model should be about the general idea (e.g., wizards), not in the concrete
  expression (e.g., Harry Potter).
- Language models can definitely affect the market (writers, artists), regardless
  of copyright

The first and last of those are the ones that bite. **Copying is the violation**,
independent of what you then do with the copy — which is precisely the
distinction the Anthropic judgment below turns on. And the fourth fair-use factor
(market effect) is not answered by anything about how the model works.

### Terms of service

- Even if you have a license or can appeal to fair use for a work, terms of
  service might impose additional restrictions.
- Example: YouTube's terms of service prohibits downloading videos, even if the
  videos are licensed under Creative Commons.

### Lawsuits

**The New York Times v. OpenAI (2023)**

- Allegation: for training and reproducing NYT articles

**Authors (Bartz, Graeber, ...) v. Anthropic (2024)**

- Allegation: for pirating millions of books and training on plaintiff's books
- Summary judgement (2025): training on plaintiff's works is fair use
- ...but pirating copies is not (even if don't train)
- Anthropic also bought and scanned the books; this is also fair use (but too late)
- Outcome: Anthropic paid $1.5B to authors to settle

**Authors (Kadrey, Silverman, ...) v. Meta**

- Allegation: for training on plaintiff's books (revealed in the Llama paper)
- Summary judgement (2025): training on books (in this instance) is fair use
  ([TechCrunch](https://techcrunch.com/2025/06/25/federal-judge-sides-with-meta-in-lawsuit-over-training-ai-models-on-copyrighted-books/))
- Allegation of torrenting books is still pending

The Anthropic outcome is the one to remember, because the two halves of the
judgment point in opposite directions: **training was held to be fair use, and
acquiring the copies by piracy was not** — and it was the second that cost $1.5B.
Buying and scanning the same books was also fair use, but doing it later did not
cure the earlier acquisition.

**Summary:**

- So far training has been deemed fair use (for specific instances, but unclear
  in general)
- Pirating books is clearly illegal
- Still a very active, evolving area

## Common Crawl

*Source: `common_crawl`, lines 241–272.*

> [Common Crawl](https://commoncrawl.org/) is a non-profit organization founded in
> 2007.

**Statistics:**

- Every ~month, run a web crawl (add 3–5 billion web pages)
- Crawls have some overlap but try to diversify
- 300 billion pages so far
- How many URLs are there? Hard to estimate, but O(billions)
- Google search index is at least 100 PB
  ([How Search Works](https://www.google.com/search/howsearchworks/how-search-works/organizing-information/))
- [April 2026 Crawl](https://commoncrawl.org/blog/april-2026-crawl-archive-now-available)
  has 2.19 billion pages (372.2 TB)

Crawling uses Apache Nutch
([Common Crawl blog](https://blog.commoncrawl.org/blog/common-crawl-move-to-nutch)).

*Figure (not redistributed): the source displays <https://upload.wikimedia.org/wikipedia/commons/thumb/d/df/WebCrawlerArchitecture.svg/330px-WebCrawlerArchitecture.svg.png> at width 400.*

Wikimedia Commons — the standard web-crawler architecture diagram (scheduler, multi-threaded downloader, queue and storage). This image is hot-linked by the lecture to a third-party site rather than living in the course repository, so it is recorded here as a URL and has not been copied into this knowledge base or described.

- Starts with a set of seed URLs (at least hundreds of millions)
  ([March 2018 crawl archive](https://commoncrawl.org/blog/march-2018-crawl-archive-now-available))
- Pop a URL from the queue, download URL, and add hyperlinks to queue

**Policies** ([Wikipedia: Web crawler](https://en.wikipedia.org/wiki/Web_crawler))

- Selection policy: which pages to download?
- Politeness policy: respect robots.txt, don't overload server
- Re-visit policy: how often to check if pages change
- Challenge: URLs are dynamic, many URLs lead to basically same content

### WARC and WET

The two formats matter for everything downstream, and several later datasets are
distinguished mainly by which one they started from:

> Two formats:
>
> - WARC: raw HTTP response (e.g., HTML)
> - WET: converted to text (lossy process)

**HTML to text:**

- Tools to convert HTML to text:
  [trafilatura](https://trafilatura.readthedocs.io/en/latest/),
  [resiliparse](https://resiliparse.chatnoir.eu/en/stable/)
- The conversion matters for the resulting LM's downstream task accuracy
  ([DCLM, arXiv 2406.11794](https://arxiv.org/abs/2406.11794))

*Figure: `images/dclm-wet.png` (width 300).*

![Small table comparing downstream accuracy (CORE, EXTENDED) for resiliparse, trafilatura, and WET-file text extraction](../images/13-data-sources-datasets/dclm-wet.png)

**What the image shows.** A small data table (3 rows x 2 numeric columns, from the DCLM paper), not a chart. Header row: Text Extraction | Core | Extended. Rows, transcribed exactly:

| Text Extraction | CORE | EXTENDED |
|---|---|---|
| resiliparse | 24.1 | **13.4** |
| trafilatura | **24.5** | 12.5 |
| WET files | 20.7 | 12.2 |

Bold marks the column-wise winner: trafilatura scores highest on CORE (24.5 vs. 24.1 and 20.7), while resiliparse scores highest on EXTENDED (13.4 vs. 12.5 and 12.2). WET files is the lowest score in both columns (20.7 and 12.2). CORE and EXTENDED are presumably downstream evaluation-suite accuracy scores (as used elsewhere in the DCLM paper), though the table itself does not define these column headers or state units — no additional caption or legend is present in the image.

**Before citing this figure.** The table does not itself state what "Core"/"Extended" measure (e.g., which benchmark suite, what metric, average of how many tasks) or what unit the numbers are in (presumably accuracy percentage) — that context comes only from the surrounding DCLM paper, not from this image.

*Source: [`images/dclm-wet.png`](https://github.com/stanford-cs336/lectures/blob/main/images/dclm-wet.png) in the lectures repo.*

This is the first appearance of a claim the lecture returns to three more times:
**how you turn HTML into text is not a preprocessing detail, it is a modelling
decision with measurable downstream effect.** RefinedWeb, The Pile and
Nemotron-CC each make a different choice, and each says so explicitly.

## Wikipedia

*Source: `wikipedia`, lines 274–295.*

> Let's now look at more specialized sources.

[Wikipedia](https://www.wikipedia.org/): free online encyclopedia

- [Random article](https://en.wikipedia.org/wiki/Special:Random)
- Founded in 2001
- As of May 2026, 67 million articles across 361 language editions (English,
  Spanish, German, French most common)
  ([Meta](https://meta.wikimedia.org/wiki/Wikipedia))

**What is the scope?**

- Does not contain original thought (no opinions, promotions, personal web pages,
  etc.) ([What Wikipedia is not](https://en.wikipedia.org/wiki/Wikipedia:What_Wikipedia_is_not))
- Includes articles based on notability (significant coverage from reliable
  sources) ([Notability](https://en.wikipedia.org/wiki/Wikipedia:Notability))

**Who writes the content?**

- Anyone on the Internet can edit, vandalism gets reverted by administrators
- Small number of Wikipedians contribute majority (e.g., Steven Pruit with 5M
  edits) ([Steven Pruitt](https://en.wikipedia.org/wiki/Steven_Pruitt))
- Produce [periodic dumps](https://dumps.wikimedia.org/enwiki/) every few weeks
  (no need to crawl)

The "no need to crawl" is a recurring structural point: several of the best
sources in this lecture publish bulk dumps, so the crawler machinery of the
previous section does not apply to them at all.

### Aside: data poisoning attacks

[arXiv 2302.10149](https://arxiv.org/pdf/2302.10149)

- Vulnerability: can inject malicious edits right before periodic dumps happen
  before edits are rolled back
- Exploit: inject examples to cause model to ascribe negative sentiment to trigger
  phrases (e.g., iPhone) ([arXiv 2010.12563](https://arxiv.org/pdf/2010.12563))
- Takeaway: even high quality sources might contain bad content

The attack works *because* of the dump schedule, not in spite of it: vandalism is
reverted eventually, and the dump only has to catch it before that happens.

## GitHub

*Source: `github`, lines 297–316.*

> Code is helpful for programming tasks, but also for reasoning (folklore).

The word **folklore** is the lecture's own hedge — the claim that code data
improves general reasoning is widely believed and not something it cites evidence
for.

[GitHub](https://github.com/):

- Live service for hosting code repositories founded in 2008 (acquired by
  Microsoft in 2018)
- As of May 2026, GitHub has 420M+ repositories (28M public)
  ([Wikipedia](https://en.wikipedia.org/wiki/GitHub))
- Each repository includes directory structure + commit history + issues + pull
  requests + comments, etc.
- Lots of duplicates (e.g., copied code, forks, etc.)
- Allowed to train on any public repository with a permissive license (e.g., MIT,
  Apache)

**Two types of data:**

- Repository: download through git protocol (rather than scraping the GitHub
  website)
- Metadata: GitHub API provides issues, pull requests, comments, etc. (hourly
  snapshots of event stream on GitHub Archive)

[Software Heritage](https://www.softwareheritage.org/):

- Non-profit organization founded in 2016 that collects and preserves software
- Focused on the repositories not metadata (issues, comments)
- Aggregates GitHub, GitLab, Bitbucket, PyPI, etc.
- As of May 2026, there are 28.8M source files

Both of these return in the Stack sections: The Stack takes repository names from
GitHub Archive, and Stack v2 takes the repositories themselves from Software
Heritage.

## arXiv

*Source: `arxiv`, lines 318–328.*

[arXiv](https://arxiv.org/):

- Website that allows researchers to share and access papers for free since 1991
- Areas: physics (original), math, CS, statistics, ...
- Has ~3M submissions
  ([monthly submission stats](https://arxiv.org/stats/monthly_submissions))
- Submission: metadata, PDF, LaTeX source (optional)
- Light approval process (not peer-review)
- Authors choose (i) all rights reserved or (ii) Creative Commons (e.g., CC-BY)
- Metadata (title, abstract) is under a permissive license (CC0)
- Bulk download from [Amazon S3](https://info.arxiv.org/help/bulk_data_s3.html),
  no need to crawl

The **LaTeX source (optional)** line is why arXiv is a distinctive source rather
than just more PDFs: it gives structured mathematics rather than a rendering of
it. The LLaMA section below describes what is done with it.

---

The remainder of the lecture is a chronological tour of named pre-training
datasets. Each entry answers, implicitly, the same three questions: what raw
source did it start from, what processing was applied, and how big was the
result. The processing is where the interest is — the sources barely change after
2019, and almost every advance in this list is a filtering or deduplication
decision.

## BERT (2019)

*Source: `bert`, lines 330–340.* [arXiv 1810.04805](https://arxiv.org/pdf/1810.04805)

> The BERT training data consists of:
>
> - Wikipedia
> - Books

(The Books component is [BooksCorpus](#bookscorpus), described next.)

- Important: sequences are documents rather than sentences
- Contrast: 1 billion word benchmark [Chelba+ 2013] (sentences from machine
  translation)

That contrast is the point of including BERT in a data lecture at all. The
One Billion Word Benchmark — which lecture 12 discussed as a perplexity dataset —
is a corpus of *shuffled sentences*, and BERT's choice to keep whole documents is
what makes long-range context learnable.

## BooksCorpus

*Source: `books_corpus`, lines 342–351.*

[Smashwords](https://www.smashwords.com/)

- Founded in 2008, allow anyone to self-publish an e-book
- 2024: 150K authors, 500K books

**BooksCorpus** ([arXiv 1506.06724](https://arxiv.org/abs/1506.06724))

- Self-published books priced at $0, scraped from Smashwords
- 7K books, 985M words
- Has been taken down because violated Smashwords terms-of-service
  ([Wikipedia](https://en.wikipedia.org/wiki/BookCorpus))

The first of three datasets in this lecture that no longer exist for legal
reasons — BooksCorpus (ToS), Books3 (copyright) and, in a different sense,
Gopher's MassiveText (never released).

## WebText and GPT-2 (2019)

*Source: `gpt2_webtext`, lines 353–362.*

**WebText**: dataset used to train GPT-2 (Radford+ 2019,
[paper](https://cdn.openai.com/better-language-models/language_models_are_unsupervised_multitask_learners.pdf))

- Contains pages that are outgoing links from Reddit posts with ≥ 3 karma
  (surrogate for quality)
- 8 million pages, 40GB text

**OpenWebTextCorpus**: open replication of WebText
([OpenWebText](https://skylion007.github.io/OpenWebTextCorpus/))

- Extracted all the URLs from the Reddit submissions dataset
- Used Facebook's fastText classifier to filter out non-English
- Removed near duplicates

The "≥ 3 karma" rule is the earliest example in the lecture of **using a human
signal as a quality proxy** — no model, no rules, just other people's upvotes.
The idea recurs in DCLM, which trains a classifier on ELI5 and OpenHermes rather
than on karma, and in Stack Exchange's scored answers.

## CCNet (2019)

*Source: `ccnet`, lines 364–377.* [arXiv 1911.00359](https://arxiv.org/pdf/1911.00359)

- Goal: automatic way of constructing large, high-quality datasets for
  pre-training
- Especially interested in getting more data for low-resource languages (e.g.,
  Urdu)

**Components:**

- Deduplication: remove duplicate paragraphs based on light normalization
- Language identification: run language ID fastText classifier; keep only target
  language (e.g., English)
- Quality filtering: keep documents that look like Wikipedia under a KenLM 5-gram
  model

**Results**

- Trained BERT models, CCNet(CommonCrawl) outperforms Wikipedia
- CCNet refers both to the open-source tool and the dataset released from paper

The three components — dedup, language ID, quality filter — are the template that
essentially every later web dataset in this lecture follows. The specific quality
filter here, *"does this look like Wikipedia under a 5-gram model"*, is the
ancestor of the model-based filtering in DCLM, with a much cheaper model.

## C4 and T5 (2019)

*Source: `t5_c4`, lines 379–405.*

**Colossal Clean Crawled corpus (C4)**
([arXiv 1910.10683v4](https://arxiv.org/pdf/1910.10683v4))

- Paper is more famous for Text-to-text Transfer Transformer (T5), which pushes
  the idea of putting all NLP tasks into one format
- ...but a major contribution was the C4 dataset.

> Observation: Common Crawl is mostly not useful natural language

- Started with one snapshot (April 2019) of Common Crawl (1.4 trillion tokens)

**Manual heuristics:**

- Keep lines that end in punctuation and have >= 5 words
- Remove page with fewer than 3 sentences
- Removed page that contains any "bad words"
  ([LDNOOBW list](https://github.com/LDNOOBW/List-of-Dirty-Naughty-Obscene-and-Otherwise-Bad-Words/blob/master/en))
- Removed page containing "{" (no code), "lorem ipsum", "terms of use", etc.
- Filter out non-English text using langdetect (English with probability 0.99)

> End result: 806 GB of text (156 billion tokens)

Worth doing the arithmetic the lecture implies: **1.4 trillion tokens in, 156
billion out — roughly 11% survives.** These are hand-written rules, not a
classifier, and the aggressiveness of rule-based filtering is exactly what
Nemotron-CC later objects to.

**Analysis of C4** ([arXiv 2104.08758](https://arxiv.org/pdf/2104.08758))

*Figure (not redistributed): the source displays <https://stanford-cs324.github.io/winter2022/lectures/images/c4-domains.png> at width 700.*

Served from the Stanford CS324 (Winter 2022) course site — the breakdown of C4 by source domain, from the C4 analysis paper. This image is hot-linked by the lecture to a third-party site rather than living in the course repository, so it is recorded here as a URL and has not been copied into this knowledge base or described.

**Bonus: WebText-like dataset**

- Filtered to pages from OpenWebText links (links in Reddit posts with ≥ 3 karma)
- Used 12 dumps to get 17 GB text (WebText was 40 GB, suggesting CommonCrawl is
  incomplete)
- This improved on various NLP benchmarks (GLUE, SQuAD, etc.)

That parenthesis is a quietly important measurement: applying WebText's own URL
list to Common Crawl recovers only 17 GB of the original 40 GB, which is direct
evidence that **Common Crawl is not a complete copy of the web.**

## GPT-3 (2020)

*Source: `gpt3`, lines 407–419.*
[arXiv 2005.14165](https://arxiv.org/pdf/2005.14165) (Section 2.2)

**GPT-3 dataset:**

- Common Crawl (processed)
- WebText2 (WebText expanded with more links)
- (Mysterious) Internet-based books corpora (Books1, Books2)
- Wikipedia

> Result: 570 GB (400 billion tokens)

**Common Crawl processing:**

- Trained quality classifier to distinguish {WebText, Wikipedia, Books1, Books2}
  from rest
- Fuzzy deduplication of documents (including WebText and benchmarks)

The word **(Mysterious)** is the source's own. Books1 and Books2 have never been
described by OpenAI, and their contents are a live question in the litigation the
previous section covered.

Note the shape of the quality classifier: it is trained to recognise *the known-good
sources* and applied to the web at large. That is the same design as DCLM's
classifier four years later; what changes is the choice of positives.

## The Pile (2021)

*Source: `the_pile`, lines 421–438.*
[arXiv 2101.00027](https://arxiv.org/pdf/2101.00027)

- In reaction to GPT-3, part of effort to produce open-source language models
- Grassroots effort with lots of volunteers contributing/coordinating on Discord
- Curated 22 high-quality domains

*Figure (not redistributed): the source displays <https://stanford-cs324.github.io/winter2022/lectures/images/the-pile.png> at width 600.*

Served from the Stanford CS324 (Winter 2022) course site — the composition of The Pile across its 22 curated domains. This image is hot-linked by the lecture to a third-party site rather than living in the course repository, so it is recorded here as a URL and has not been copied into this knowledge base or described.

- 825 GB of text (~275B tokens)
- Pile-CC: Common Crawl, use WARC, jusText to convert into text (better than WET)
- PubMed Central: 5 million papers, mandated to be public for NIH funded work
- arXiv: preprint for research papers since 1991 (use latex)
- Enron emails: 500K emails from 150 users from Enron senior management, released
  during Enron investigation (2002) ([CMU](https://www.cs.cmu.edu/~enron/))

The Pile is the first dataset here whose selling point is **curated diversity**
rather than scale — 825 GB is smaller than C4's 806 GB by very little, and much
smaller than what follows, but it spans 22 named domains. The Enron emails are the
lecture's example of how idiosyncratic those domains get: a corpus that exists
only because of a corporate fraud investigation.

Three of its components get their own sections.

### Project Gutenberg

*Source: `project_gutenberg`, lines 440–447.*

[Project Gutenberg](https://www.gutenberg.org/)

- Started in 1971 by Michael Hart, who wanted to increase access to literature
- 2025: ~75K books, mostly English
- Only include books that have received copyright clearance (most in the public
  domain)

**PG-19**: books from Project Gutenberg before 2019
([pg19](https://github.com/google-deepmind/pg19))

### Books3

*Source: `books3`, lines 449–455.*

**Books3** [Presser, 2020]
([paperswithcode](https://paperswithcode.com/dataset/books3))

- 196K books from the shadow library Bibliotik
- Contained books from authors (e.g., Stephen King, Min Jin Lee, Zadie Smith)
  ([Wired](https://www.wired.com/story/battle-over-books3/))
- Has been taken down due to copyright infringement / lawsuits
  ([HuggingFace](https://huggingface.co/datasets/the_pile_books3))

Read this against the Gutenberg entry immediately above it, which is the
comparison the lecture is setting up: **both are books, and only one of them is
legal.** Books3 is also the component that puts The Pile and, through it, LLaMA
into the copyright discussion — LLaMA's data section names Books3, which is how
it was "revealed in the Llama paper" per the Kadrey v. Meta allegation.

### Stack Exchange

*Source: `stackexchange`, lines 457–467.*

- Collection of sites of user-contributed questions and answers
- Started with StackOverflow in 2008, grew to other topics (e.g., math,
  literature) ([sites](https://stackexchange.com/sites))
- Use reputation points and badges to incentivize participation
- [Example](https://ell.stackexchange.com/questions/351826/is-he-not-the-carpenters-son-v-s-is-not-he-the-carpenters-son)
- Q&A format is close to instruction tuning / real application
- Note: there is metadata (users, votes, comments, badges, tags) for filtering
- Data dumps in XML (anonymized, include metadata)
  ([archive.org](https://archive.org/details/stackexchange))

The line **"Q&A format is close to instruction tuning / real application"** is the
one to keep: Stack Exchange is valuable not for its facts but for its *shape*, and
that is a different argument from every other source in this lecture.

## MassiveText and Gopher (2021)

*Source: `gopher_massivetext`, lines 469–487.*

**MassiveText** dataset used to train Gopher
([arXiv 2112.11446](https://arxiv.org/pdf/2112.11446.pdf), DeepMind)

> The Gopher model is subsumed by Chinchilla (also never released), but the
> description of data is good

**Components**

- MassiveWeb: more on this later
- C4
- Books: no details
- News: no details
- GitHub: no details
- Wikipedia: no details

**MassiveWeb filtering steps**

- Keep English, deduplication, train-test overlap
- Quality filtering using manual rules (not classifier) — e.g., 80% words contain
  at least one alphabetic character
- Use Google SafeSearch for toxicity (not word lists)

> Result: 10.5 TB of text (though Gopher only trained on 300B tokens - 12%)

Two things are worth noticing. The repeated **"no details"** is the lecture making
its opening point again in a concrete case — even a paper praised for its data
description says nothing about four of its six components. And the *Gopher rules*
— manual, not classifier-based — become a named, reusable artifact: RefinedWeb,
FineWeb and Dolma all cite "Gopher rules" as a filtering step.

## LLaMA (2023)

*Source: `llama`, lines 489–502.*
[arXiv 2302.13971](https://arxiv.org/pdf/2302.13971)

- CommonCrawl processed with CCNet, classify *references* of Wikipedia or not
- C4 (more diverse; recall: rule-based filtering)
- GitHub: kept permissive licenses, filtering based on manual rules
- Wikipedia: June–August 2022, 20 languages, manual filtering
- Project Gutenberg and Books3 (from The Pile)
- arXiv: removed comments, inline expanded macros, bibliography
- Stack Exchange: 28 largest websites, sorted answers by score

> Result: 1.2T tokens

**Reproduced by Together's RedPajama v1**
([HF](https://huggingface.co/datasets/togethercomputer/RedPajama-Data-1T)).
Cerebras's [SlimPajama](https://www.cerebras.ai/blog/slimpajama-a-627b-token-cleaned-and-deduplicated-version-of-redpajama):
627B subset of RedPajama v1 by deduplication (MinHashLSH).

LLaMA's list is the lecture's clearest illustration of its own thesis that nothing
is built from scratch: **every single component is a dataset from an earlier
section of this lecture**, recombined. The one genuinely new decision is the
Common Crawl filter, which classifies whether a page is *referenced by* Wikipedia
rather than whether it *looks like* Wikipedia — a link-graph signal instead of a
language-model one.

The arXiv treatment (expand macros, drop comments and bibliography) is the
concrete answer to why LaTeX source is worth having.

## RefinedWeb and FineWeb (2023)

*Source: `refinedweb`, lines 504–521.*

**RefinedWeb** ([arXiv 2306.01116](https://arxiv.org/pdf/2306.01116))

- Point: web data is all you need
- [Examples](https://huggingface.co/datasets/tiiuae/falcon-refinedweb/viewer/default/train)
- trafilatura for HTML→text, extract content (WARC instead of WET files)
- Filtering: Gopher rules, avoid ML-based filtering to avoid biases
- Fuzzy deduplication using MinHash over 5-grams

> Released 600B (out of 5T) tokens

**FineWeb** ([HF](https://huggingface.co/datasets/HuggingFaceFW/fineweb))

- Started as a replication of RefinedWeb, but improved it
- 95 Common Crawl dumps
- URL filtering, language ID (keep if p(en) > 0.65)
- Filtering: Gopher, C4, more manual rules
- Fuzzy deduplication via MinHash
- Anonymize email and public IP addresses (PII)

> Result: 15T tokens

RefinedWeb's thesis — **"web data is all you need"** — is a direct challenge to
The Pile's curated-diversity approach: it argues that Common Crawl alone,
filtered well enough, beats a mixture of specialised sources. Its explicit refusal
of ML-based filtering ("to avoid biases") is also the position DCLM overturns a
year later, which makes these two entries the lecture's central disagreement.

Note FineWeb's language threshold, p(en) > 0.65, against C4's langdetect at 0.99.
The looser threshold is deliberate and in the same direction as Nemotron-CC's
complaint about over-filtering.

## Dolma (2024)

*Source: `dolma`, lines 523–537.*
[arXiv 2402.00159](https://arxiv.org/pdf/2402.00159)

*Figure (not redistributed): the source displays <https://miro.medium.com/v2/resize:fit:1400/1*-0Qqhvu7JD6Y9JgsfKJdxw.png> at width 700.*

Hot-linked to a Medium CDN — the composition of Dolma by source. This image is hot-linked by the lecture to a third-party site rather than living in the course repository, so it is recorded here as a URL and has not been copied into this knowledge base or described.

- Reddit: from the Pushshift project (2005–2023), include submissions and comments
  separately
- PeS2o: 40M academic papers from Semantic Scholar
- C4, Project Gutenberg, Wikipedia/Wikibooks

**Common Crawl processing**

- Language identification (fastText classifier), keep English
- Quality filtering (Gopher, C4 rules), avoid model-based filtering
- Toxicity filtering using rules and Jigsaw classifier
- Deduplication using Bloom filters

> Result: 3T tokens

Dolma is the dataset behind OLMo, the open pipeline shown in the motivation
section, so this is the concrete content of that first figure. It sides with
RefinedWeb on the central question — **"avoid model-based filtering"** — and its
deduplication by Bloom filter is a different engineering choice from the MinHash
used by RefinedWeb, FineWeb and The Stack.

## DataComp-LM / DCLM (2024)

*Source: `dclm`, lines 539–557.*
[arXiv 2406.11794](https://arxiv.org/abs/2406.11794)

- Goal: define a standard dataset for trying out different data processing
  algorithms
- Processed CommonCrawl to produce DCLM-pool (240T tokens)
- DCLM-baseline: filtered down DCLM-pool using quality classifier

*Figure: `images/dclm-filter.png` (width 800).*

![Sankey diagram of DCLM-Pool to DCLM-Baseline filtering pipeline, showing document-count percentages retained/dropped at each stage](../images/13-data-sources-datasets/dclm-filter.png)

**What the image shows.** A Sankey (flow) diagram — not a bar chart — titled by its own caption "Figure 4: Construction of DCLM-Baseline from DCLM-Pool," with the caption text (transcribed in full): "Before this pipeline, we extracted DCLM-Pool from Common Crawl with resiliparse. Percentages are based on the total number of original documents."

Structure, left to right, three labeled pipeline stages (column headers): "Heuristic cleaning (Sections 4.1 & 4.2) (Reproduction of RefinedWeb)" (blue), "Deduplication (4.3)" (cyan), "Model-based filtering (4.4)" (gold/yellow). The leftmost node is a vertical bar labeled "DCLM-Pool (CommonCrawl)" representing 100% of original documents. At each stage, the main flow continues rightward as survivors while side-branches peel off downward/upward to small grey boxes labeled with the name of the specific filter that removed that fraction of documents — the branch percentage is the share of the original DCLM-Pool document count removed by that filter, not a separate data series.

Heuristic cleaning stage removes, in the order shown top-to-bottom: Word removal ratio filter (2.0%), Repetition filter (9.6%), Page length filter (7.9%), Other filters — e.g., Word-length, Ellipsis count, Stop words — (9.0%), English filter (50.8%), URL filter (0.8%). These six removed fractions plus the 19.9% that survives (flowing into Deduplication) sum to 100% (2.0+9.6+7.9+9.0+50.8+0.8+19.9 = 100.0).

Deduplication stage: of the 19.9% entering, "Bloom filter dedup" removes 6.2%, leaving 13.7% that flows into Model-based filtering (6.2+13.7=19.9).

Model-based filtering stage: of the 13.7% entering, the "FastText filter" branch removes 12.3%, leaving a final 1.4% that reaches the gold-outlined "DCLM-Baseline" box at the far right (12.3+1.4=13.7).

Net result made explicit by the diagram's own arithmetic: only 1.4% of the original DCLM-Pool documents (by document count) survive into DCLM-Baseline — the single largest cut happens at the English-language filter alone (50.8% of all original documents removed there), and the FastText quality-classifier stage removes the large majority (12.3 of the remaining 13.7 percentage points, i.e. ~90%) of what survives dedup.

**Before citing this figure.** These percentages are explicitly stated (in the figure's own caption) to be based on document counts, not tokens — so this figure should not be used to state a token-retention rate. Separately, the DCLM-Baseline token count elsewhere in this lecture is given as 3.8T tokens (from a 240T-token DCLM-Pool per the lecture script), a ~1.6% token-retention ratio, which is at least roughly consistent with this diagram's 1.4% document-retention figure, but the two are not the same measurement and readers should not treat them as interchangeable.

*Source: [`images/dclm-filter.png`](https://github.com/stanford-cs336/lectures/blob/main/images/dclm-filter.png) in the lectures repo.*

### Model-based filtering

**Positive examples (200K):**

- [OpenHermes-2.5](https://huggingface.co/datasets/teknium/OpenHermes-2.5):
  mostly GPT-4 generated instruction data
  ([examples](https://huggingface.co/datasets/teknium/OpenHermes-2.5/viewer/default/train))
- [ELI5](https://www.reddit.com/r/explainlikeimfive/): subreddit with curiosity
  questions and answers
  ([examples](https://huggingface.co/datasets/sentence-transformers/eli5/viewer/pair/train))

**Negative examples (200K):**

- [RefinedWeb](https://huggingface.co/datasets/tiiuae/falcon-refinedweb/viewer/default/train)

> Result: 3.8T tokens

- Trained a fastText classifier, run it on all of DCLM-pool
- This quality classifier outperforms other filtering methods:

*Figure: `images/dclm-quality.png` (width 600).*

![Table 4 from DCLM paper comparing 7 quality-filtering methods on CORE/EXTENDED; fastText OH-2.5+ELI5 wins both](../images/13-data-sources-datasets/dclm-quality.png)

**What the image shows.** A table (not a chart, despite the lecture text saying "This quality classifier outperforms other filtering methods" right before it) — "Table 4: Quality filtering comparison (1B-1x scale)" from the DCLM paper, with caption text: "We evaluate various choices for model-based quality filters. Training a fastText classifier for filtering performs best." Header row: Filter | Core | Extended.

Full transcription:

| Filter | CORE | EXTENDED |
|---|---|---|
| RefinedWeb reproduction (baseline, separated by its own rule) | 27.5 | 14.6 |
| Top 20% by Pagerank | 26.1 | 12.9 |
| SemDedup [1] | 27.1 | 13.8 |
| Classifier on BGE features [185] | 27.2 | 14.0 |
| AskLLM [146] | 28.6 | 14.3 |
| Perplexity filtering | 29.0 | 15.0 |
| Top-k average logits | 29.2 | 14.7 |
| fastText [87] OH-2.5 +ELI5 | **30.2** | **15.4** |

The bottom row, "fastText OH-2.5 +ELI5" (a fastText classifier trained using OpenHermes-2.5 and ELI5 as positive examples — matching the "OpenHermes-2.5 / ELI5 positive, RefinedWeb negative" classifier described in the surrounding lecture text), is bolded and has the highest score in both columns, confirming it as the best-performing method of the seven compared. All seven candidate filters plus the reference RefinedWeb-reproduction row are evaluated at what the caption calls "1B-1x scale" (a fixed small-scale training/eval regime used for the ablation, not the full DCLM-Baseline scale). Bracketed numbers (e.g. [1], [185], [146], [87]) are citation references, rendered in blue, pointing to external papers for those specific methods.

**Before citing this figure.** Filename/lecture text call this a comparison of "filtering methods" via what could be assumed to be a bar/line chart, but the image is a plain numeric table — no chart is present.

*Source: [`images/dclm-quality.png`](https://github.com/stanford-cs336/lectures/blob/main/images/dclm-quality.png) in the lectures repo.*

Two details deserve emphasis. First, DCLM is presented as **a benchmark, not a
dataset** — its stated goal is a standard pool against which filtering algorithms
can be compared, which is why it publishes both the 240T pool and the 3.8T
baseline. Second, the classifier's *negative* class is RefinedWeb — that is, an
entire carefully filtered dataset from the previous section is used here as the
example of what not-good-enough looks like. The positives are instruction data and
an explain-it-simply subreddit, so the filter is selecting for text that resembles
helpful answers rather than text that resembles an encyclopedia, which is the
break from CCNet's Wikipedia-likeness criterion.

## Nemotron-CC (2024)

*Source: `nemotron_cc`, lines 559–576.*
[arXiv 2412.02595](https://arxiv.org/abs/2412.02595)

- FineWebEdu and DCLM filter too aggressively (remove 90% of data)
- Need moar tokens (but preserve quality)
- For HTML→text, used jusText (not trafilatura) because it returned more tokens

**Classifier ensembling**

- Prompt Nemotron-340B-instruct to score FineWeb documents based on educational
  value, distill into faster model
- DCLM classifier

**Synthetic data rephrasing**

- For low-quality data, use LM to rephrase
- For high-quality data, use LM to generate tasks (QA pairs, extract key
  information, etc.)

> Result: 6.3T tokens (HQ subset is 1.1T)
>
> For reference, Llama 3 trained on 15T, Qwen3 trained on 36T

*Figure: `images/nemotron-results.png` (width 800).*

![Table of benchmark scores for FineWebEdu-2/FineWebEdu/DCLM/Nemotron-CC/Nemotron-CC-HQ across 10 tasks plus average](../images/13-data-sources-datasets/nemotron-results.png)

**What the image shows.** A plain data table (not a chart), 5 rows x 12 columns, comparing five pretraining-corpus variants across ten downstream benchmarks plus an average column. Header row, transcribed exactly as printed: Dataset | ARC-E | ARC-C | H | W | RACE | PIQA | SIQA | CSQA | OBQA | MMLU | Avg. Note that two columns are printed with only a single-letter header, "H" and "W" — the image itself does not spell these out (they most likely abbreviate HellaSwag and Winogrande given the standard benchmark suite used in this literature, but that expansion is not stated anywhere in the image itself).

Full transcription:

| Dataset | ARC-E | ARC-C | H | W | RACE | PIQA | SIQA | CSQA | OBQA | MMLU | Avg |
|---|---|---|---|---|---|---|---|---|---|---|---|
| FineWebEdu-2 | 71.9 | 44.7 | 75.4 | 67.0 | 36.8 | 79.5 | 45.2 | 25.5 | 43.8 | 42.4 | 53.2 |
| FineWebEdu | 73.6 | 48.0 | 70.7 | 64.6 | **38.0** | 76.4 | 43.5 | 30.0 | 44.4 | 42.9 | 53.2 |
| DCLM | 74.7 | 47.0 | 76.3 | 69.1 | 36.5 | 79.7 | 45.6 | 44.1 | 44.0 | 53.4 | 57.0 |
| Nemotron-CC | 75.3 | 50.7 | 75.9 | 67.8 | 37.9 | **80.5** | 45.1 | 47.7 | 44.2 | 53.0 | 57.8 |
| Nemotron-CC-HQ | **78.8** | **52.9** | **76.6** | **69.4** | 36.4 | 80.1 | **46.6** | **55.8** | **45.4** | **59.0** | **60.1** |

Bold marks the column-wise best score. Nemotron-CC-HQ (the high-quality subset) wins on 8 of the 10 individual benchmark columns (ARC-E, ARC-C, H, W, SIQA, CSQA, OBQA, MMLU) and has the best Avg (60.1); FineWebEdu wins only on RACE (38.0) and Nemotron-CC wins only on PIQA (80.5). FineWebEdu-2 and FineWebEdu are tied for the lowest Avg (53.2 each); DCLM (57.0) and Nemotron-CC (57.8) sit in between; Nemotron-CC-HQ (60.1) is the clear overall leader on the Avg column.

**Before citing this figure.** Despite the filename "nemotron-results," this is a table, not a bar/line chart. Two column headers ("H", "W") are printed as bare single letters in the source table with no expansion given in the image.

*Source: [`images/nemotron-results.png`](https://github.com/stanford-cs336/lectures/blob/main/images/nemotron-results.png) in the lectures repo.*

This is the last of the web-filtering entries and it inverts the direction of
travel. Everything from C4 onward filters *harder*; Nemotron-CC's premise is that
the field has overshot, discarding 90% of the data, at a moment when frontier
models want 15–36T tokens. Its two answers are both notable: an **ensemble** of
classifiers rather than one, and **synthetic rephrasing** — using a language model
to rewrite low-quality text into usable text rather than throwing it away, which
is a genuinely different move from every other entry in this list.

## The Stack (2022) and Stack v2 (2024)

*Source: `the_stack`, lines 578–598.*

**The Stack** ([arXiv 2211.15533](https://arxiv.org/pdf/2211.15533))

- Took repository names from GitHub Archive (2015–2022)
- git clone'd 137M repositories, 51B files (5B unique!)
- Kept only permissively licensed (MIT, Apache) using go-license-detector
- Remove near-duplicates using minhash and Jaccard similarity
- Result: 3.1 TB of code

The parenthetical **(5B unique!)** is the lecture's exclamation, not an editorial
one — 51 billion files reduce to 5 billion, so roughly 90% of GitHub by file count
is duplicate. That is the concrete version of the "lots of duplicates" line in the
GitHub section.

**Stack v2** ([arXiv 2402.19173](https://arxiv.org/abs/2402.19173))

- Issues, comments, PRs from GitHub Archive
- Repositories from the Software Heritage
- Documentation from crawling websites (e.g., PyPI, npm, devdocs.io)
- Processing: remove binary files, malware, bot activity, deduplication, PII
  redaction, subsample PRs
- Pair source code (especially low-resource languages like Nim) with shared
  low-level intermediate language (LLVM)
- Include existing datasets (GSM8K, code contests, StackOverflow, arXiv,
  Wikipedia, OpenWebMath)

The LLVM idea is the cleverest thing in this section: compiling low-resource
languages to a common intermediate representation gives the model a bridge from
languages it has little data for to ones it has much more of.

**Pull requests:**

- Linearize structured object to token sequence
- Add some inline context (e.g., file surrounding diff), subsample

*Figure: `images/stackv2-pr1.png` (width 250).*

![Code-style listing (Stack v2 paper) of the linearized token template for a GitHub pull request's title, status, and file diffs](../images/13-data-sources-datasets/stackv2-pr1.png)

**What the image shows.** A monospace code/template listing (not a diagram or photo) from the Stack v2 paper, showing how a GitHub pull request is linearized into a token sequence. It uses three text colors with a consistent role across the listing: black for literal angle-bracket special tokens and structural punctuation (`<pr>`, `<pr_status>`, `<repo_name>`, `<pr_base>`, `<pr_file>`, `<pr_base_code>`, `<pr_diff>`, `<pr_diff_hunk>`, the literal `\n` line-break escape, colons, and `...` ellipses marking omitted/repeated content); red for short categorical or identifier-like placeholder values; and blue for larger free-text or code-content placeholder values.

Transcribed line by line (colors noted inline):
- `<pr>Title: ` (black) `title` (blue) `\n` (black) `username_0` (blue) `: ` (black) `description` (blue)
- `<pr_status>` (black) `opened` (red)
- `<repo_name>` (black) `reponame` (red)
- (blank line)
- `<pr_base>` (black)
- `<pr_file>` (black) `filepath_1` (red)
- `<pr_base_code>` (black) `file_content/changes_1` (blue)
- `...` (black)
- `<pr_file>` (black) `filepath_N` (red)
- `<pr_base_code>` (black) `file_content/changes_N` (blue)
- (blank line)
- `<pr_diff>` (black)
- `<pr_file>` (black) `filepath_1` (red)
- `<pr_diff_hunk>` (black) `diff_hunk_1` (blue)
- `...` (black)
- `<pr_diff_hunk>` (black) `diff_hunk_K` (blue)
- `...` (black)
- `<pr_file>` (black) `filepath_M` (red)
- `<pr_diff_hunk>` (black) `diff_hunk_1` (blue)
- `...` (black)
- `<pr_diff_hunk>` (black) `diff_hunk_J` (blue)

Overall structure: a `<pr>` header block gives Title/author/description, then `<pr_status>` (e.g. "opened") and `<repo_name>`; a `<pr_base>` section lists each base file's path (`<pr_file>`) and full content/changes (`<pr_base_code>`) for files 1 through N; a `<pr_diff>` section lists, per changed file (1 through M), one or more diff hunks (`<pr_diff_hunk>`) numbered 1 through K (and again 1 through J for the last file shown), illustrating that the number of hunks per file varies. This matches the lecture's framing of "linearize structured object to token sequence" for the PR pipeline.

**Before citing this figure.** none — the listing is fully legible and its role (a template/schema for serializing a PR) is unambiguous from the content itself, though the paper's own key defining exactly what each `<pr_*>` special token means is not shown in this crop.

*Source: [`images/stackv2-pr1.png`](https://github.com/stanford-cs336/lectures/blob/main/images/stackv2-pr1.png) in the lectures repo.*

*Figure: `images/stackv2-pr2.png` (width 400).*

![Continuation of the Stack v2 pull-request token template, covering comments, reviews, and review-comment threads](../images/13-data-sources-datasets/stackv2-pr2.png)

**What the image shows.** A monospace code/template listing (not a diagram or photo), the continuation of stackv2-pr1.png from the Stack v2 paper, using the same three-color convention (black = literal special tokens/punctuation/ellipses, red = short categorical/identifier placeholders, blue = free-text/content placeholders) — with one inconsistency noted below.

Transcribed line by line:
- `<pr_comment>` (black) `username` (black) `_id` (red) `: ` (black) `comment` (blue)
- `<pr_event_id>` (black) `comment_id` (red)
- `...` × 3 (black)
- `<pr_review>` (black) `username` (black) `_id` (red) `: ` (black) `review_comment\n` (blue, including the literal `\n`)
- `<pr_event_id>` (black) `review_id` (red)
- `<pr_review_state>` (black) `[approved, rejected, commented, changes_required]` (red — the entire bracketed enum-of-options is red)
- `...` × 2 (black)
- `<pr_review_comment>` (black)
- `<pr_event_id>` (black) `comment_id` (red)
- `<pr_in_reply_to_review_id>` (black) `review_id (opt)` (red)
- `<pr_in_reply_to_comment_id>` (black) `comment_id (opt)` (red)
- `<pr_file>` (black) `filepath` (red)
- `<pr_diff_hunk_comment_line>` (black) `line_number` (red)
- `<pr_diff_hunk>` (black) `diff_hunk_content` (blue)
- `<pr_comment>` (black) `username` (black) `_id` (red) `: ` (black) `comment` (blue)

Overall structure: this half of the template covers (1) top-level PR comments (`<pr_comment>`), (2) PR reviews (`<pr_review>`) with a `<pr_review_state>` field whose placeholder is written as an explicit enumerated option list `[approved, rejected, commented, changes_required]` rather than a single generic placeholder word, and (3) threaded review comments (`<pr_review_comment>`) that can optionally reply to a prior review or comment (`(opt)` markers) and that anchor to a specific file and diff line (`<pr_file>`, `<pr_diff_hunk_comment_line>`, `<pr_diff_hunk>`).

**Before citing this figure.** In this image, every occurrence of "username_id" is rendered with "username" in black and only the "_id" suffix in red (e.g. `username` + `_id`), whereas the analogous placeholder in stackv2-pr1.png ("username_0" in the PR title line) is rendered entirely in blue. This is a real, visually-verifiable inconsistency in coloring convention between the two images; the semantic reason for it (if any) is not stated in either image.

*Source: [`images/stackv2-pr2.png`](https://github.com/stanford-cs336/lectures/blob/main/images/stackv2-pr2.png) in the lectures repo.*

## CommonPile (2025)

*Source: `common_pile`, lines 600–620.*

The lecture closes by returning to the copyright section and asking what is left
if you take it entirely seriously.

> Recall:
>
> - Almost all data on the Internet is copyrighted.
> - Some of it is permissively licensed.
> - Fair use of copyrighted content is not settled.
>
> Key question: can you train a good model using only permissively-licensed data?

**CommonPile** ([arXiv 2506.05209](https://arxiv.org/pdf/2506.05209))

*Figure: `images/commonpile.png` (width 700).*

![Log-scale bar chart of CommonPile's 30 permissively-licensed sub-sources grouped into 9 categories by size in GB](../images/13-data-sources-datasets/commonpile.png)

**What the image shows.** A bar chart (grouped, log-scaled y-axis) of the composition of the CommonPile dataset. Y-axis "Dataset Size" is log-scaled from 1MB to 1TB, with labeled gridlines at 1MB, 100MB, 10GB, and 1TB (each labeled step is a 100x jump; unlabeled minor gridlines are not shown). There are 30 individual bars organized into 9 category groups, each group sharing one bar color and bracketed under a group label giving that category's total size in GB:

- **Code (4775 GB)** — pink/red bars: Stack V2, PEPs (2 bars; Stack V2 far larger, roughly 2TB-scale vs. PEPs at roughly 30-40MB).
- **Government & Legal (1172 GB)** — salmon/orange bars: USPTO, CAP, USGPO, UK Hansard, Regulations.gov (5 bars, roughly descending from ~1GB down to under 10GB... USPTO tallest at roughly 1GB scale, the rest progressively smaller down to Regulations.gov the smallest).
- **Wikis (528 GB)** — yellow/gold bars: Wikiteam, Wikimedia (2 bars, both large, Wikiteam slightly taller).
- **Web (260 GB)** — green bars: CCCC, News, Foodista, PDR (4 bars, steeply descending — CCCC by far the largest, PDR the smallest, near the 10MB range).
- **Academic Papers (370 GB)** — teal/blue bars: peS2o, PubMed, ArXiv Papers, ArXiv Abstracts (4 bars, descending, peS2o and PubMed comparable and largest, ArXiv Abstracts smallest).
- **Online Forums (165 GB)** — dark purple/maroon bars: Stack Exchange, GitHub Archive, Ubuntu IRC (3 bars, descending).
- **Public Domain Books (244 GB)** — light blue/periwinkle bars: BHL, Pre-1929 Books, Library of Congress, Project Gutenberg (4 bars, fairly close in size, gently descending).
- **Other (29 GB)** — light purple bars: CC YouTube, DPI (2 bars, CC YouTube somewhat larger).
- **Educational Resources (15 GB)** — magenta bars: DOAB, PressBooks, LibreTexts, OER Commons (4 bars, steadily descending).

Bars within a category are colored identically and are ordered largest-to-smallest left to right within their bracket; there is no separate legend beyond the bracket labels themselves (color encodes category membership, not a separate data series). Code is the largest single category by a wide margin (4775 GB, dominated by Stack V2), followed by Government & Legal (1172 GB) and Wikis (528 GB); Educational Resources (15 GB) and Other (29 GB) are the smallest categories.

**Before citing this figure.** Exact bar heights for the smaller sources (e.g., PDR, PEPs, DPI, individual Educational Resources bars) sit visually in the 10-100MB range on the log axis but cannot be read to more than order-of-magnitude precision from this chart, since no data labels are printed on individual bars — only the per-category GB totals in the bracket labels are exact numbers.

*Source: [`images/commonpile.png`](https://github.com/stanford-cs336/lectures/blob/main/images/commonpile.png) in the lectures repo.*

- Collected 8TB dataset of permissively licensed data

**Subtleties:**

- License laundering: redistribute copyrighted work under permissive license
  (hard to detect)
- Collection licenses (Dolma is ODC-By) doesn't extend to individual
- Synthetic data from LMs trained on unlicensed data is unclear

*Figure: `images/comma-results.png` (width 700).*

![Grouped bar chart of Comma v0.1-1T vs LLaMA/MPT/RPJ-INCITE/Qwen3 on 11 benchmarks; Comma leads baselines on 6 (starred)](../images/13-data-sources-datasets/comma-results.png)

**What the image shows.** A grouped bar chart (not a table, despite the filename), y-axis "Performance" from 0 to 100 with gridlines every 20. There are 5 series/bars per benchmark group, identified by the legend in this left-to-right order: Comma v0.1-1T (solid gold/orange), LLaMA (solid magenta/pink), MPT (solid purple), RPJ-INCITE (solid light blue), Qwen3 (white with black diagonal hatching, no solid fill). The bar order within each group matches the legend order exactly. There are 11 benchmark groups arranged in two labeled bands along the x-axis: a "Knowledge/Reasoning" band containing ARC-C, ARC-E, MMLU, BoolQ, HSwag, OBQA, CSQA, PIQA, SIQA (9 groups), and, after a gap, a "Coding" band containing HumEval, MBPP (2 groups).

A gold five-pointed star is drawn above the Comma v0.1-1T bar in exactly 6 of the 11 groups: ARC-C, MMLU, BoolQ, SIQA, HumEval, and MBPP. Checking bar heights confirms a pattern: in every starred group, Comma v0.1-1T's bar is the tallest among the four solid-color bars (Comma, LLaMA, MPT, RPJ-INCITE) — i.e., the star appears to mark benchmarks where Comma outperforms the three comparison baselines (excluding Qwen3). In the 5 unstarred groups (ARC-E, HSwag, OBQA, CSQA, PIQA), a different baseline (LLaMA or MPT) is tallest among the four solid bars. This is an inference from the visual pattern, not a labeled legend entry — the chart itself does not caption what the star means.

Approximate bar heights read off the chart (rounded to the nearest whole number from bar-top position against the gridlines; no numeric data labels are printed on the bars, so treat these as ± 1-2 point estimates), in the order Comma / LLaMA / MPT / RPJ-INCITE / Qwen3:
- ARC-C: 53 / 44 / 46 / 43 / 57
- ARC-E: 68 / 68 / 70 / 68 / 74
- MMLU: 42 / 35 / 30 / 28 / 77
- BoolQ: 76 / 76 / 74 / 68 / 86
- HSwag: 63 / 77 / 78 / 70 / 77
- OBQA: 47 / 51 / 49 / 49 / 51
- CSQA: 60 / 62 / 63 / 58 / 66
- PIQA: 71 / 77 / 77 / 76 / 78
- SIQA: 51 / 50 / 49 / 47 / 55
- HumEval: 37 / 20 / 28 / 11 / 94
- MBPP: 36 / 28 / 34 / 16 / 68

The most visually striking feature is Qwen3's bar, which leads in most groups and
sometimes by a very large margin — most dramatically on HumEval (~94 vs. Comma's
~37, more than 2.5x) and MMLU (~77 vs. Comma's ~42). *[Parent correction to the
original description, made by reading the image: Qwen3 is NOT tallest in all
eleven groups. On HSwag it is level with or just below MPT (~77 against ~77.5),
and on OBQA it is level with or just below LLaMA (~50.5 against ~51). The
per-group values in the list above were confirmed; it was the summarising
sentence that overreached.]* Among the four solid-color bars (the ~1T-token-class models being compared to Comma), performance is fairly close throughout except on the two Coding benchmarks, where RPJ-INCITE is clearly the weakest (11 and 16) and Comma and MPT lead.

**Before citing this figure.** This image is a bar chart, not a table — do not describe it as one. The meaning of the gold star markers is inferred from the visual pattern (Comma is tallest among the 4 solid bars in every starred group and in no unstarred group) rather than stated by any caption or legend text visible in the image. All numeric values above are visual estimates from bar height against gridlines, not transcribed data labels — treat them as approximate.

*Source: [`images/comma-results.png`](https://github.com/stanford-cs336/lectures/blob/main/images/comma-results.png) in the lectures repo.*

> - Can do decently, but tough to compete without more tokens

The three subtleties are the section's real contribution, because each one is a
way that "permissively licensed" fails to mean what it appears to. **License
laundering** means a permissive label is not evidence of permissive origin.
**Collection licenses** means Dolma being ODC-By says nothing about the license of
any document inside it. And the third undoes the guarantee entirely: if you use
synthetic data generated by a model that was itself trained on unlicensed text,
the provenance of your "clean" corpus is not clean.

The answer to the key question is a qualified yes — **decent, but token-limited**,
which is the constraint the lecture leaves the reader with. Set it against
Nemotron-CC's 6.3T and Qwen3's 36T to see the size of the gap.

## Figure audit

The 14 course-repo images were described by one reader working from the images
themselves, with the surrounding source text supplied only as context for what
each figure is meant to support. Two were then re-checked in the parent by
looking at the image directly. This section records what that found, and what a
reader must know before quoting any of these figures.

### The mismatch worth knowing about

**`decline-consent.png` does not show what the lecture text implies, and this was
confirmed by direct inspection.** The source line reads "Examined restrictions
(robots.txt, ToS) for URLs in common datasets (C4, RefinedWeb, Dolma)" — which
describes the *paper's* scope. The **figure** breaks nothing out by dataset. It
has three panels: robots.txt category composition over time, ToS category
composition over time, and a log-scale chart of restriction rates by crawler
**organization** (OpenAI 25.9%, Anthropic 13.3%, Common Crawl 13.3%, Google 9.8%,
"False Anthropic" 6.0%, Cohere 4.9%, Meta 4.1%, Internet Archive 3.2%, Google
Search 1.0%).

So **no C4, RefinedWeb or Dolma series exists in this image.** If you are asked
what the restriction rate is for C4 specifically, this figure does not say. Note
also that the third panel's y-axis is **logarithmic** (0.1%, 0.2%, 1%, 2%, 10%,
100%), so the post-ChatGPT rise is steeper than a linear reading suggests.

### The parent correction

**`comma-results.png`** — the reader's structure and per-group values were
confirmed exactly: five series in an order matching the legend (Comma v0.1-1T,
LLaMA, MPT, RPJ-INCITE, and Qwen3 as hatched bars), eleven benchmarks in two
groups, and six gold stars falling exactly on the benchmarks where Comma leads
the three ~1T-token baselines. That last is an *inference* — nothing in the image
labels what a star means — but it holds in all eleven groups and is recorded as
an inference, not a caption.

One summarising sentence was wrong and has been corrected in place: Qwen3 is
**not** the tallest bar in all eleven groups. It is level with or just below MPT
on HSwag and LLaMA on OBQA. This is the failure mode this build keeps meeting —
correct measured values under an interpretive sentence that overreaches — and it
is why the values and the summary are checked separately.

### What the reader flagged, unprompted

These are the higher-yield output of the pass, in the same way run 13's were:

- **`dclm-quality.png` and `nemotron-results.png` are plain tables**, not charts,
  despite the lecture phrasing ("this quality classifier outperforms...") and the
  `-results` filename. Do not describe them as charts.
- **`comma-results.png` IS a bar chart**, not a table — the opposite mistake.
- **`dclm-filter.png` is a Sankey/flow diagram**, not a simple funnel, and its own
  caption states that its percentages are **by document count, not tokens**. Only
  **1.4%** of DCLM-Pool documents survive to DCLM-baseline, and the
  English-language filter alone removes 50.8% of all documents.
- **`dclm-wet.png` gives no units and does not define its columns.** "CORE" and
  "EXTENDED" are presumably downstream accuracy suites, but the image says
  neither what they contain nor what unit the numbers are in.
- **`olmo2-dolmino.png`** has one row whose byte-to-token ratio is odd (4.56T
  bytes against 752B tokens) and which is worth checking against the OLMo 2 paper
  before quoting.
- **`tulu.png`**'s raw "Count" column totals 23.3M but is **94% one row**
  (OpenMathInstruct 2's raw count), so the total is a misleading summary of what
  was actually used for SFT/DPO.
- **`stackv2-pr1.png` / `stackv2-pr2.png`** carry a genuine internal
  inconsistency: `username_id` is split black-and-red in pr2 while `username_0` is
  solid blue in pr1.
- **`llama3-data.png` genuinely supports the claim it is used for** — it describes
  cleaning methods at length and discloses no actual data source.

### The boundary

Two of 14 images were re-checked in the parent; the other twelve rest on a single
careful reading. That is a thinner audit than the PDF-deck lectures in this KB
received, and it is a deliberate trade of the same kind made for lecture 12:
these are reproduced paper figures and tables rather than charts drawn at slide
resolution, and the two chosen were the highest-stakes claims available — the one
alleging that a figure contradicts the lecture text, and the one chart whose
headline reading rested on an inference. Both were worth the check: the first
confirmed a real mismatch, the second caught a wrong summarising sentence.

**Values quoted from `comma-results.png` are visual estimates from bar heights**
against gridlines, not printed data labels — that chart prints none. Where a
figure prints its numbers, the transcription above gives them exactly; prefer the
transcription to the picture for any number.
