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

@@IMG:llama3-data@@

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

@@IMG:olmo2-pretraining@@

**2. Mid-training**

@@IMG:olmo2-dolmino@@

**3. Post-training** ([arXiv 2411.15124](https://arxiv.org/pdf/2411.15124) — Tülu 3)

@@IMG:tulu@@

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

@@IMG:decline-consent@@

### When crawlers are not well-behaved

@@IMG:anthropic-crawling@@

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

@@IMG:webcrawler-architecture@@

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

@@IMG:dclm-wet@@

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

@@IMG:c4-domains@@

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

@@IMG:the-pile@@

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

@@IMG:dolma-composition@@

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

@@IMG:dclm-filter@@

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

@@IMG:dclm-quality@@

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

@@IMG:nemotron-results@@

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

@@IMG:stackv2-pr1@@

@@IMG:stackv2-pr2@@

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

@@IMG:commonpile@@

- Collected 8TB dataset of permissively licensed data

**Subtleties:**

- License laundering: redistribute copyrighted work under permissive license
  (hard to detect)
- Collection licenses (Dolma is ODC-By) doesn't extend to individual
- Synthetic data from LMs trained on unlicensed data is unclear

@@IMG:comma-results@@

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
