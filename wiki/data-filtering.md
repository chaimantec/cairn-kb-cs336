# Data filtering

**This page is the history and the argument.** For how a filter is actually built —
the target/raw framework, KenLM versus fastText, and the five worked recipes from
[lecture 14](14-data-filtering-dedup-mixing.md) — see
[quality classifiers](quality-classifiers.md).

The central technical argument of [lecture 13](13-data-sources-datasets.md), and
by the lecture's own assessment the highest-leverage step in the whole data
pipeline: **"how do you go from 200 trillion tokens to less than 3 trillion
tokens? That's obviously a huge reduction, which I think merits a lot of
attention"** (≈1:20:15).

The lecture's compression of the whole history is one sentence:

> All of these look very similar at some level — you take a web crawl, and you
> either decide, I'm going to use rules to filter, or I'm going to use a model.
> (≈1:10:07)

And the trade-off is genuine, not solved: "you can get 240 trillion tokens if you
want, but that's probably going to be really low quality, or you can get like one
trillion tokens, and there's some sweet spot in between."

## The chronology of ideas

| Year | Dataset | The quality signal |
| --- | --- | --- |
| 2019 | WebText (GPT-2) | **humans** — Reddit outlinks with ≥ 3 karma |
| 2019 | CCNet | **a language model** — looks like Wikipedia under a KenLM 5-gram |
| 2019 | C4 (T5) | **hand-written rules** |
| 2020 | GPT-3 | **a classifier** for {WebText, Wikipedia, Books1, Books2} vs rest |
| 2021 | MassiveText (Gopher) | **rules**, chosen deliberately over a classifier |
| 2023 | LLaMA | **the link graph** — is the page *referenced by* Wikipedia |
| 2023 | RefinedWeb | rules; explicitly **refuses** ML-based filtering |
| 2024 | Dolma | rules; also avoids model-based quality filtering |
| 2024 | DCLM | **a fastText classifier** — and it wins |
| 2024 | Nemotron-CC | **an ensemble**, plus [synthetic rewriting](synthetic-data.md) |

## The pre-model era

### The karma filter

GPT-2's WebText used no model at all. Pages linked from Reddit posts with ≥ 3 karma,
on the reasoning that "good posts must link to good websites" (≈46:58). A human
signal, borrowed wholesale.

### CCNet: "looks like Wikipedia"

CCNet (Facebook, 2019) was aimed at low-resource languages — they "didn't want some
very manual process that only worked for English" (≈47:43). Three components that
became the template essentially every later web pipeline follows:

1. **Deduplication** of paragraphs after light normalization.
2. **Language identification** with a fastText classifier.
3. **Quality filtering**: train a KenLM 5-gram model on Wikipedia, then score each
   new document's probability under it. "Rather than outgoing links to Reddit, they
   used a language model on Wikipedia" (≈48:30).

The result matters for the argument that follows: filtered Common Crawl **beat
training on Wikipedia itself**, because you get so much more of it (≈49:16). More
mediocre data beats less excellent data — up to a point.

### C4: rules, and how aggressive they are

C4's premise was that "Common Crawl is mostly not useful for natural language"
(≈50:02). Its answer was hand-written rules:

- keep lines ending in punctuation with ≥ 5 words;
- drop pages with fewer than 3 sentences;
- drop pages containing any word from a bad-words list;
- drop pages containing `{`, "lorem ipsum", "terms of use";
- keep English at langdetect p = 0.99.

**1.4 trillion tokens in, 156 billion out — about 11% survives.** And the `{` rule
is the lecture's favourite illustration of how a filter encodes its era's
assumptions: it "filters out a lot of code — clearly, at that time, they weren't
thinking about code models" (≈50:49). A rule written for one purpose silently
deletes a capability nobody had thought to want yet.

### GPT-3: classify against the known-good

GPT-3 trained a quality classifier to distinguish {WebText, Wikipedia, Books1,
Books2} from everything else. Structurally this is DCLM's design four years early;
what changes later is the choice of positives.

### The rules camp digs in

Gopher's MassiveWeb used manual rules deliberately — "part of the reason for this
is that they had more control over it" (≈58:36) — and the lecture identifies this
as the moment a genuine split appears: "there's this sort of division between people
who wanted to use rules and people who wanted to use classifiers" (≈58:36–59:22). Those
**Gopher rules** then become a named, reusable artifact cited by RefinedWeb,
FineWeb and Dolma.

RefinedWeb states the position most explicitly:

> Avoid ML-based filtering, to avoid biases — I don't want to find an overly narrow
> subset of the web. (≈1:02:28)

Dolma takes the same line a year later, using a model for language ID but avoiding
model-based *quality* filtering (≈1:04:01). FineWeb loosens the language threshold
to **p(en) > 0.65**, against C4's 0.99 — a small number carrying a real position
about over-filtering.

### LLaMA's link-graph variant

LLaMA classifies whether a page is **referenced by** Wikipedia rather than whether
it resembles Wikipedia: "maybe Wikipedia articles are too stylized, but we know
that Wikipedia articles reference a bunch of other articles, which are presumably
good" (≈1:00:07). A third kind of signal — neither rules nor a text classifier, but
the citation graph.

## DCLM: the turn to model-based filtering

DCLM (2024) is "maybe the point where this idea of model-based quality filtering
really started to become the norm" (≈1:04:01–1:04:47).

Its framing is a **benchmark, not a dataset** — the goal was "to define some sort of
pipeline so that people can try different data methods in a standard way and show
results," though the lecture notes drily that "the main way people use this is just
using the dataset they released" (≈1:04:47). Hence both artifacts: DCLM-pool at
**240 trillion tokens**, completely unfiltered — "probably more tokens than anyone
really trains on" — and DCLM-baseline at 3.8T.

![DCLM filtering flow from pool to baseline](../raw/images/13-data-sources-datasets/dclm-filter.png)

*DCLM's pipeline. Its own caption states the percentages are by **document count,
not tokens**, and the English-language filter alone removes 50.8% of documents.
[Source](https://github.com/stanford-cs336/lectures/blob/main/images/dclm-filter.png)*

**The pipeline keeps 1.4%** (≈1:05:33). The classifier is where the interest is,
and the lecture's own verdict is "it's kind of weird, but somehow this works"
(≈1:06:18):

- **Positives (200K):** OpenHermes-2.5 — mostly GPT-4-generated instruction data —
  and **ELI5**, a subreddit of plain-language explanations.
- **Negatives (200K):** **RefinedWeb.**
- A **fastText** linear classifier, run over the whole pool.

Two things deserve emphasis. First, the negative class is *an entire carefully
filtered dataset from the previous generation* — RefinedWeb, "which, remember, is
just a very loosely filtered version of the web — it's basically the web." The
previous era's best effort is this era's definition of not-good-enough.

Second, and more substantively: **the target changed.** CCNet asked *does this look
like an encyclopedia?* DCLM asks *does this look like a helpful answer?* The
positives are instruction data and an explain-it-simply forum, not reference text.

![Table comparing DCLM's quality classifier against other filtering approaches](../raw/images/13-data-sources-datasets/dclm-quality.png)

*The comparison behind the lecture's "magical quality classifier" (≈1:07:04). This
is a **table**, not a chart.
[Source](https://github.com/stanford-cs336/lectures/blob/main/images/dclm-quality.png)*

DCLM "became a bit of a gold standard for quality filtering" in the open community.

## The counter-argument: everyone over-filters

Nemotron-CC (Nvidia) inverts the premise. Its complaint is that FineWebEdu and DCLM
"filter too aggressively," removing 90% of the data, at exactly the moment frontier
models want far more tokens — "we need more tokens" (≈1:07:04).

Its three moves:

1. **HTML→text with jusText rather than trafilatura, because it returned more
   tokens** — optimising surviving volume rather than cleanliness. See
   [web crawling](web-crawling.md#warc-vs-wet-and-why-html-to-text-is-a-modelling-decision).
2. **Classifier ensembling** — prompt Nemotron-340B-instruct to score FineWeb
   documents for educational value, distil that into a fast fastText model, and
   combine it with the DCLM classifier.
3. **[Synthetic rephrasing](synthetic-data.md)** — rewrite low-quality text instead
   of discarding it.

Result: **6.3T tokens** (HQ subset 1.1T), against DCLM's 3.8T. Set that beside the
lecture's scale markers — Llama 3 at 15T, Qwen3 at 36T (≈1:09:20) — and the
motivation is clear.

## What to take away

- **Filtering is where the leverage is.** Sources have been stable since 2019; this
  is the step that changed.
- **Rules versus models is a real disagreement**, held by serious people on both
  sides for good reasons — control and bias-avoidance against measured downstream
  performance — and it was not settled until DCLM produced numbers.
- **A filter encodes assumptions that outlive it.** C4's `{` rule is the cautionary
  case.
- **Aggressiveness is a parameter, not a virtue.** Retention rates in this lecture
  run from ~11% (C4) to 1.4% (DCLM), and Nemotron-CC's whole argument is that the
  field pushed that number too low.
- **And the honest caveat**, from the lecture's close: "data processing right now,
  at least, is a lot just based on vibes. You define this classifier, you define
  this rule, you set some threshold" (≈1:21:01).

## The argument as of lecture 14

[Lecture 14](14-data-filtering-dedup-mixing.md) treats the rules-versus-models split
recorded above as settled, and gives an economic reason rather than a scientific
one: "these days I think basically everyone does some amount of model-based filtering, because unless you are compute-plentiful — in which case you probably don't need to do as much filtering, you can just train on everything — **most people are compute-poor**"
(≈11:01).

It also adds the qualification that makes any single filtering setting unquotable
out of context: **there is no optimal threshold**, because the right aggressiveness
depends on how many tokens you will train for. Train longer and you want more,
lower-quality data; train shorter and you want less, higher-quality data
(≈17:12). See
[quality classifiers](quality-classifiers.md#there-is-no-optimal-threshold) for the
experiment that shows a heavily filtered pool overtaken by an unfiltered one once
the token budget grows, and
[data mixture selection](data-mixture-selection.md#the-scale-dependent-effect-and-simulated-epoching)
for the same effect one stage later.

## See also

- [Pre-training datasets](pretraining-datasets.md) — the corpora these produce
- [Deduplication](deduplication.md) — the other processing step
- [Synthetic data](synthetic-data.md) — the alternative to discarding
- [Web crawling](web-crawling.md) — where the input comes from
- [Data mixture selection](data-mixture-selection.md) — proportions, not quality
- [Quality classifiers](quality-classifiers.md) — how a filter is built, from
  [lecture 14](14-data-filtering-dedup-mixing.md)
- [Lecture 13](13-data-sources-datasets.md)
