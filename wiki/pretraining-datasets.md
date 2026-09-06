# Pre-training datasets

A chronology of the named corpora language models have actually been trained on,
from BooksCorpus (2015) to CommonPile (2025). Compiled from
[Lecture 13](13-data-sources-datasets.md), whose second half is a tour of exactly
this list.

The single most useful observation to carry through it: **the raw sources barely
change after 2019.** Common Crawl, Wikipedia, GitHub, arXiv, books and Q&A forums
are the whole inventory. Almost every advance in this list is a
[filtering](data-filtering.md) or [deduplication](deduplication.md) decision
applied to the same underlying material — which is why the lecture's summary
singles filtering out as where the leverage is (≈1:20:15).

A second, quieter theme: **three of these datasets no longer exist**, each for a
different legal reason. BooksCorpus violated terms of service, Books3 was
copyright infringement, and MassiveText was simply never released.

## The table

Sizes are quoted as the lecture states them; note it mixes bytes, tokens, words
and document counts. The
[course material file](../raw/slides/13-data-sources-datasets.md#datasets-named-in-this-lecture)
carries the same table with the source's own citations.

| Year | Dataset | Built from | Size as stated |
| --- | --- | --- | --- |
| 2013 | One Billion Word Benchmark | MT sentences | — |
| 2015 | [BooksCorpus](#bookscorpus) | free Smashwords e-books | 7K books, 985M words |
| 2019 | BERT data | Wikipedia + Books | — |
| 2019 | [WebText](#webtext-and-the-karma-filter) | Reddit outlinks ≥ 3 karma | 8M pages, 40 GB |
| 2019 | OpenWebTextCorpus | open replication of WebText | — |
| 2019 | [CCNet](data-filtering.md#ccnet-looks-like-wikipedia) | Common Crawl + KenLM filter | — |
| 2019 | [C4](#c4-and-the-rule-based-turn) | one April 2019 CC snapshot | 806 GB (156B tokens) |
| 2020 | [GPT-3](#gpt-3-and-the-mysterious-books) | CC, WebText2, Books1/2, Wikipedia | 570 GB (400B tokens) |
| 2020 | [Books3](#books3) | the Bibliotik shadow library | 196K books |
| 2021 | [The Pile](#the-pile) | 22 curated domains | 825 GB (~275B tokens) |
| 2021 | [MassiveText](#massivetext-and-gopher) | MassiveWeb, C4, books, news, GitHub, Wikipedia | 10.5 TB |
| 2022 | [The Stack](code-data.md#the-stack) | GitHub, permissive licenses only | 3.1 TB |
| 2023 | [LLaMA](#llama-everything-is-recombination) | CC/CCNet, C4, GitHub, Wikipedia, books, arXiv, Stack Exchange | 1.2T tokens |
| 2023 | RedPajama v1 / SlimPajama | reproduction of LLaMA / dedup subset | — / 627B tokens |
| 2023 | [RefinedWeb](#refinedweb-web-data-is-all-you-need) | CC WARC only | 600B released (of 5T) |
| 2024 | FineWeb | 95 CC dumps | 15T tokens |
| 2024 | [Dolma](#dolma) | CC, Reddit, PeS2o, C4, Gutenberg, Wikipedia | 3T tokens |
| 2024 | [DCLM](data-filtering.md#dclm-the-turn-to-model-based-filtering) | CC, fastText quality classifier | pool 240T → baseline 3.8T |
| 2024 | [Nemotron-CC](synthetic-data.md#1-as-a-filtering-alternative-nemotron-cc) | classifier ensemble + rephrasing | 6.3T (HQ 1.1T) |
| 2024 | [Stack v2](code-data.md#stack-v2) | Software Heritage + GitHub Archive | — |
| 2025 | [CommonPile](data-licensing-and-consent.md#commonpile) | permissively licensed only | 8 TB |

For scale, the lecture's two reference points: **Llama 3 trained on 15T tokens and
Qwen3 on 36T** (≈1:09:20) — with the caveat that published token counts may
include repeats across epochs, so they are not directly comparable as *unique*
data. See [data repetition](data-repetition.md).

## The books lineage

### BooksCorpus

Smashwords, founded 2008, lets anyone self-publish an e-book; by 2024 it had about
150K authors and 500K books. In 2015 a paper scraped the ones priced at $0 and
made a corpus of **7K books, 985M words**.

It was the Books half of BERT's training data. BERT's other contribution the
lecture flags is structural rather than legal: **sequences were documents rather
than sentences**, in contrast to the One Billion Word Benchmark's shuffled MT
sentences (≈46:11). That is what makes long-range context learnable at all.

BooksCorpus has since been **taken down for violating Smashwords' terms of
service** — the lecture's point being that "just because it was free and you could
get it doesn't mean it was legally allowed" (≈46:11). It describes 2015 as "the
innocent days when no one was paying attention" (≈45:26).

### Project Gutenberg

Started in 1971 by Michael Hart, ~75K books as of 2025, and **only books with
copyright clearance** — mostly public domain. PG-19 packages the pre-2019 subset.

Read Gutenberg and Books3 together: that is the comparison the lecture is setting
up. Both are books; only one is legal.

### Books3

196K books (the lecturer says "200K") from the shadow library **Bibliotik**,
assembled in 2020 and included in [The Pile](#the-pile). It contained books by
named living authors, and has been **taken down for copyright infringement**.

Books3 is the hinge between this lecture's two halves. It is how the copyright
discussion becomes concrete: LLaMA's paper named Books3 among its sources, and the
lecture traces the chain of discovery — "you look back, 'Oh, this came from —
where did it come from?' It came from The Pile. 'Oh, it came from a shadow
library'" (≈1:00:54). That is the allegation in *Kadrey v. Meta*.

And it produces the lecture's sharpest line about the state of the field:
**"That's why people don't talk about their data anymore"** (≈1:00:54) — closing
the loop with the Llama 3 secrecy the lecture opened on. RedPajama v1 initially
reproduced Books3 too, and has since stripped it out; "various decisions made
early on about copyright actually have a fairly big watershed effect" (≈1:01:40).

## The web lineage

### WebText and the karma filter

GPT-2's data (2019). Common Crawl was known but considered too messy, so OpenAI
used a **human quality signal instead of a model**: pages linked from Reddit posts
with ≥ 3 karma, on the reasoning that "good posts must link to good websites"
(≈46:58). Result: 8 million pages, 40 GB. Never released; replicated as
OpenWebTextCorpus with fastText language filtering and near-duplicate removal.

This is the earliest quality proxy in the lecture and it uses no model at all —
just other people's upvotes. The idea recurs in Stack Exchange's scored answers and,
in a different form, in DCLM's choice of positives.

### C4 and the rule-based turn

The C4 dataset came out of the T5 paper (Google, 2019), which is better known for
text-to-text but whose "major contribution was the C4 dataset" (≈49:16). Its
premise was blunt: **"Common Crawl is mostly not useful for natural language"**
(≈50:02).

The method was hand-written rules — keep lines ending in punctuation with ≥ 5
words; drop pages with fewer than 3 sentences, pages containing bad words, "lorem
ipsum", "terms of use"; drop any page containing `{`, which "filters out a lot of
code — clearly, at that time, they weren't thinking about code models" (≈50:49);
and langdetect English at p = 0.99.

**1.4 trillion tokens in, 156 billion out — roughly 11% survives.**

C4's bonus experiment is a genuinely load-bearing measurement. Applying WebText's
own Reddit-outlink recipe to 12 Common Crawl dumps recovered only **17 GB against
WebText's 40 GB**, which is direct evidence that **Common Crawl is not a complete
copy of the web** — "if it had everything, you should be able to hit 40 GB"
(≈52:22).

### GPT-3 and the mysterious books

Common Crawl (processed), WebText2, Books1 and Books2, and Wikipedia; 570 GB /
400B tokens. Processing was a **quality classifier trained to distinguish the
known-good sources from the rest**, plus fuzzy deduplication.

Books1 and Books2 are described in the paper only as "internet-based books
corpora," and the lecture flags this: they remain "a mystery, exactly what it is"
(≈53:09) — and that mystery is live in the copyright litigation.

### The Pile

A grassroots response to GPT-3 from **EleutherAI**, coordinated on Discord, curating
22 high-quality domains — 825 GB, ~275B tokens. Components include Pile-CC (which
uses **WARC and jusText rather than WET**, "better than WET"), PubMed Central (5M
papers), arXiv, Stack Exchange, Books3, Project Gutenberg, IRC logs, philosophy
papers, and the Enron emails.

The Enron corpus is the lecture's example of how idiosyncratic these domains get:
500K emails from 150 Enron senior managers, released during the 2002 investigation,
"one of the few email datasets we have, which is a weird distribution for email,
but that's what you get" (≈54:41).

The Pile's argument is **curated diversity** — barely larger than C4, but spanning
22 named domains. RefinedWeb later argues directly against this.

### MassiveText and Gopher

DeepMind, 2021. Gopher was never released and was "sort of subsumed by Chinchilla,"
but the lecture recommends the paper anyway because "the description of the data,
I think, is actually really good" (≈57:51) — immediately qualified: "except for the
parts where they don't tell you what's in the data."

That qualification is the point. Four of its six components — Books, News, GitHub,
Wikipedia — are documented as "no details." Even a paper praised for data
transparency says nothing about most of it.

MassiveWeb's filtering was **manual rules, not a classifier**, deliberately, "part
of the reason for this is that they had more control over it" (≈58:36) — and those
**"Gopher rules" become a reusable named artifact** cited by RefinedWeb, FineWeb
and Dolma. Toxicity used Google SafeSearch rather than word lists.

Result: 10.5 TB, of which Gopher trained on 300B tokens — about 12%.

### LLaMA: everything is recombination

LLaMA (2023) is the clearest illustration of the lecture's thesis that nothing is
built from scratch: **every component is a dataset from an earlier section** —
Common Crawl via CCNet, C4, GitHub, Wikipedia, Project Gutenberg and Books3, arXiv,
Stack Exchange. 1.2T tokens.

The one genuinely new idea is its Common Crawl filter: classify whether a page is
**referenced by** Wikipedia rather than whether it *looks like* Wikipedia. The
reasoning is that "maybe Wikipedia articles are too stylized, but we know that
Wikipedia articles reference a bunch of other articles, which are presumably good"
(≈1:00:07) — a link-graph signal rather than a language-model one.

Its arXiv handling (expand macros, drop comments and bibliography) is the concrete
answer to why LaTeX source is worth having. Reproduced as Together's RedPajama v1;
Cerebras's SlimPajama is a 627B-token MinHashLSH-deduplicated subset.

### RefinedWeb: "web data is all you need"

The direct challenge to The Pile. RefinedWeb (2023) asks what happens "if we just
stick with the web" (≈1:01:40) and drops the specialised sources entirely. It uses
**WARC rather than WET** with trafilatura, the Gopher rules, and MinHash fuzzy
deduplication over 5-grams. 5T tokens produced, 600B released.

Its explicit refusal is what makes it a position rather than a pipeline: **"Avoid
ML-based filtering, to avoid biases — I don't want to find an overly narrow subset
of the web"** (≈1:02:28). DCLM overturns exactly this a year later, which makes
these two the central disagreement of [data filtering](data-filtering.md).

**FineWeb** (Hugging Face) replicated and improved it: 95 Common Crawl dumps, URL
filtering, language ID at the notably looser **p(en) > 0.65** against C4's 0.99,
Gopher and C4 rules, MinHash dedup, and PII anonymisation. 15T tokens.

### Dolma

AI2's dataset, and the one behind OLMo — so it is the concrete content of the
open-pipeline example the lecture opens with. Common Crawl plus Reddit (from the
**Pushshift** project, 2005–2023, "at that time you could still get this data and
train on it, before things got locked down"), PeS2o (40M papers from AI2's own
Semantic Scholar crawl), C4, Gutenberg and Wikipedia. 3T tokens.

Dolma sides with RefinedWeb: language ID is model-based, but **quality filtering
still avoids model-based filtering** (≈1:04:01). Deduplication uses Bloom filters
rather than MinHash. Toxicity filtering uses rules plus a Jigsaw classifier.

Dolma is also the worked example in the licensing discussion: it is released
ODC-By, but [that collection license does not extend to the individual works
inside it](data-licensing-and-consent.md#three-ways-permissively-licensed-fails).

## See also

- [Lecture 13](13-data-sources-datasets.md) — the source for this page
- [Data filtering](data-filtering.md) — the rule-vs-model argument that runs
  through the chronology
- [Deduplication](deduplication.md) · [Web crawling](web-crawling.md) ·
  [Code data](code-data.md) · [Synthetic data](synthetic-data.md)
- [Copyright and fair use](copyright-and-fair-use.md) — why three of these
  datasets no longer exist
- [Data mixture selection](data-mixture-selection.md) — how much of each, as
  against which ones
