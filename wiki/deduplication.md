# Deduplication

The web is enormously redundant, and every serious pre-training corpus removes
duplicates. From [lecture 13](13-data-sources-datasets.md), where deduplication is
one of the three named stages of turning raw data into processed data.

## Why there is so much duplication

Two independent sources of it, from different parts of the lecture:

**On the web**, the same content appears under many URLs. "Many different URLs
might lead to the same content, so there's a lot of duplication that happens if
you're not careful — mirror sites in particular are explicitly about duplication"
(≈34:42). Common Crawl's monthly crawls also overlap each other by design.

**In code**, duplication is the norm rather than the exception, "because of either
copying code or forking code" (≈40:01). The Stack put a number on this that is
worth remembering: cloning 137 million repositories yielded **51 billion files, of
which only 5 billion were unique** — so roughly **90% of GitHub by file count is
duplicate**.

## The methods

The lecture names four distinct techniques, and which one a dataset uses is a real
engineering choice rather than a detail:

| Method | Used by | Granularity |
| --- | --- | --- |
| Light normalization + exact match | CCNet | paragraphs |
| Fuzzy dedup, unspecified | GPT-3 | documents |
| **MinHash** (often over 5-grams, with Jaccard similarity) | RefinedWeb, FineWeb, The Stack, SlimPajama (MinHashLSH) | documents |
| **Bloom filters** | Dolma | documents |

**MinHash** is the field's default for near-duplicate detection: it estimates
Jaccard similarity between documents' n-gram sets cheaply enough to run over
trillions of tokens. RefinedWeb applies it over 5-grams; SlimPajama used MinHashLSH
to cut RedPajama v1 from 1.2T tokens down to **627B** — a reduction of roughly half
achieved by deduplication alone, which is the clearest single illustration of how
much redundancy survives an otherwise careful pipeline.

**Bloom filters** are Dolma's choice — a probabilistic set membership structure,
which trades a small false-positive rate for very low memory. A false positive here
means discarding a document that was not actually a duplicate, which at these scales
is an acceptable cost.

**Paragraph-level** deduplication, as in CCNet, is a different granularity from the
rest: it removes repeated boilerplate *within* otherwise distinct documents rather
than removing whole documents.

## Deduplication against the test set

GPT-3's deduplication was explicitly "fuzzy deduplication of documents (including
WebText and benchmarks)" — that is, it deduplicated against **evaluation
benchmarks**, not only within the training corpus. Gopher's MassiveWeb filtering
likewise lists "train-test overlap" alongside English filtering and deduplication.

This is the same concern [lecture 12](12-evaluation.md) treats from the evaluation
side, and it is worth reading the two together: what a data pipeline calls
deduplication, an evaluation methodology calls
[contamination](benchmark-contamination.md). They are the same operation viewed
from opposite ends, and a corpus that skips it silently inflates every benchmark
number computed on it.

## Where it sits in the pipeline

The lecture's summary states the pipeline as **live service → raw data → processed
data**, where the last arrow is "transformation, filtering, deduplication"
(≈1:20:15). Deduplication is usually applied after language identification and
before or alongside quality filtering, because it is cheaper to deduplicate a
smaller pool and because duplicate-heavy junk would otherwise skew a trained
classifier.

It also interacts with the token-count numbers that get quoted for these datasets.
The lecture's warning about published totals applies here: "if you do multiple
epochs, two epochs, that's twice the number of tokens, so you have to be careful
when you look at those numbers" (≈1:09:20). A deduplicated 627B-token corpus and a
1.2T-token corpus with everything repeated twice are not the same thing, even
though a naive count makes them look comparable. See
[data repetition](data-repetition.md), which treats what repeated data does to
training.

## See also

- [Data filtering](data-filtering.md) — the other half of processing
- [Web crawling](web-crawling.md) — where the duplication comes from
- [Pre-training datasets](pretraining-datasets.md) · [Code data](code-data.md)
- [Data repetition](data-repetition.md) — the scaling-side treatment of repeats
- [Benchmark contamination](benchmark-contamination.md) — the same problem, seen
  from evaluation
- [Lecture 13](13-data-sources-datasets.md)
