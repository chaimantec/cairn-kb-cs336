# Lecture 14 — Data II: Filtering, Deduplication, Mixing, Post-Training Data

**Percy Liang.** [Transcript](../raw/transcripts/14-data-filtering-dedup-mixing.md) ·
[course material](../raw/slides/14-data-filtering-dedup-mixing.md) ·
[`lecture_14.py`](https://github.com/stanford-cs336/lectures/blob/main/lecture_14.py) ·
[trace viewer](https://cs336.stanford.edu/lectures/?trace=lecture_14)

[Lecture 13](13-data-sources-datasets.md) asked where data comes from and what law
governs its use. This one asks **what you do to it once you have it**, and it has a
different character: three of its five parts are algorithms with running code, and
the whole thing follows the pre-training pipeline in the order data moves through
it — transform, filter, deduplicate, mix — before changing subject entirely for a
survey of post-training data.

The lecturer's own opening summary of where lecture 13 left off (≈0:05):

> Data doesn't really just fall from the sky. You actually have to think about where
> it comes from... the internet consists of live services; the data on those
> services has to be either dumped or crawled, and then there's an additional step
> where you have to process the data.

**This lecture is that additional step.** The first four sections are "mostly about
pre-training — that's what you should have in mind" (≈0:51); the fifth is
mid-training and SFT.

Unusually for this course's data lectures, `lecture_14.py` **computes**: 22
inspected values covering MurmurHash, exact deduplication, Jaccard, a 100-seed
MinHash simulation, LSH collision probabilities, and the mixing arithmetic. None of
them touches a GPU or a clock, so all of them are reproduced exactly in the
[course material](../raw/slides/14-data-filtering-dedup-mixing.md#every-computed-value-in-this-lecture).

## 1. Transformation: raw bytes are not text

> Raw data does not come as text. It is HTML, PDF (arXiv), or directories (code
> repositories).

Most of the effort goes to HTML "because most of the web is in HTML" (≈1:40). The
job is to strip boilerplate — navigation, ads, headers, footers — and extract the
content, and neither half is well defined: "what is content and what is not content
is not always clear", and stripping navigation may itself lose something, since
those elements are part of "what web pages look like".

It is also **inherently lossy**, for a structural reason rather than a
tooling reason: HTML is hierarchical and visually rendered, while training data is
a flat token sequence. Tables are the sharp case — "Simple tables you can render using markdown, but if you have nested tables then that becomes quite challenging — you have to give up at some point, or approximate at some point" (≈2:26).

The tools are rule-based (**trafilatura, resiliparse, jusText, lynx**) because they
must run over the whole crawl and because "you don't need too much intelligence" for
the task — at the cost that "any rule-based processing is going to have some failure
rate" (≈3:12).

![Small table comparing three HTML-to-text extraction tools on CORE and EXTENDED benchmark scores](../raw/images/14-data-filtering-dedup-mixing/dclm-wet.png)

*DCLM's extractor comparison: resiliparse 24.1 / **13.4**, trafilatura **24.5** / 12.5, WET files 20.7 / 12.2 (CORE / EXTENDED). The two dedicated extractors are close; **Common Crawl's own pre-extracted WET text is worst on both**, by 3.4 and 1.2 points. [Source](https://github.com/stanford-cs336/lectures/blob/main/images/dclm-wet.png)*

PDFs are the same problem, harder: Common Crawl truncates large ones so they must be
re-crawled, extraction means OCR (**RolmOCR**, **Docling**) rather than parsing, and
layout information is lost because "PDFs are by design all about layout" while HTML
at least has H1 and P tags carrying semantics (≈6:20). They are worth the trouble
for a selection-effect reason the program does not state: "if you bother to make a
PDF, that means you probably have something interesting to say, as opposed to a web
page" (≈5:32).

→ [HTML-to-text extraction](html-to-text-extraction.md) covers this in full.

## 2. Filtering, stated as one problem

The lecture's most useful abstraction, and it is stated once before any application:

> Given some **target data** T and lots of **raw data** R, find subset T′ of R
> similar to T.

![Schematic diagram of three ellipses illustrating the raw-data, target-data and filtered-subset relationship](../raw/images/14-data-filtering-dedup-mixing/raw-target-schema.png)

*The framework in one picture. T and T′ are related by colour, not by any drawn overlap — it is a conceptual schematic rather than a Venn diagram. [Source](https://github.com/stanford-cs336/lectures/blob/main/images/raw-target-schema.png)*

Language identification, quality filtering and toxicity filtering are then **the
same algorithm with a different T** (≈7:07). Two requirements: the filter must
*generalize* — "you don't want to just get the target data back" — and it must be
extremely fast, because R "could be 100 trillion tokens" (≈7:54–8:40). What comes
out is small: "a very small fraction — a single-digits fraction — of your entire data".

Two families of scorer: a **generative** model of T, where
$\text{score}(x) = p_T(x)$ and KenLM's five-gram model is the cheap instantiation;
and a **discriminative** classifier, where $\text{score}(x) = p(T \mid x)$ and
fastText — "generally just a linear classifier bag of words" — is what people use. Positives are
T, negatives a random subset of R, and you keep above a threshold, sometimes
stochastically (≈9:27–10:13).

Lecture 13 recorded a genuine split between the rules camp and the models camp.
Lecture 14 declares it settled, on economic grounds: "these days I think basically everyone does some amount of model-based filtering... **most people are compute-poor**, and you have to be very smart about how you're filtering, otherwise you're just wasting flops on low-quality content" (≈11:01).

The five worked recipes — fastText language ID across 176 languages; OpenMathText's
three-mechanism math pipeline (**14.7B tokens beating models trained on 20× the
data**); GPT-3's Pareto-distributed stochastic keep; LLaMA's use of pages
*referenced by* Wikipedia; phi-1's GPT-4-labels-then-random-forest recipe
(**17.68% on HumanEval after 36K steps against 12.19% after 96K**) — are all in
[quality classifiers](quality-classifiers.md).

The general lesson the lecture draws from OpenMathText is worth having on its own:
**quality is whatever you define it to be.** "There's no universal notion of
quality. If you define quality to be math, then you can go and get math, and you get better at it" (≈14:06).

### There is no optimal threshold

The section's most important qualification, and the one that connects it to the rest
of the course. The right amount of filtering depends on **how long you will train**
(≈17:12):

> Intuitively, if you are going to train for a longer period of time then you can
> tolerate lower quality data. If you're training for shorter then you want higher
> quality data.

![Line chart of loss against log-scale tokens trained, comparing five filtering methods](../raw/images/14-data-filtering-dedup-mixing/data-filtering-scale.png)

*A preliminary experiment (attributed in the lecture to Michael Ryan): 157M-parameter models on 100 WARCs. **dclm** (blue, heavily filtered, small pool) improves, then turns back up as it epochs and overfits, ending worst on the plot. **resiliparse** (purple, essentially unfiltered) starts far worse and decreases monotonically the whole way. Four of the nine legend entries have no plotted curve — no value may be read for them. [Source](https://github.com/stanford-cs336/lectures/blob/main/images/data-filtering-scale.png)*

"High quality data is better in this regime where you're not epoching, but once you
get to lots and lots of tokens, high quality data is no longer that great" — and
even stopping the filtered run before it overfits leaves you worse off "than if you
had trained for longer using low quality data" (≈19:35).

Two questions from the floor: each point is one training run and confidence
intervals would be good practice, but runs are expensive and results "generally tends to be stable, I would say, at least for pre-training" (≈20:21); and yes, a larger high-quality pool
would also show diminishing returns eventually, but its curve "would just be down
here and just keep on going down" (≈21:07).

## 3. Deduplication

Two kinds of duplicate (≈22:39–24:11). **Exact**: mirror sites, whose whole purpose
is duplication, and forked repositories — "even if you make changes, probably you're
making changes to a few files, so 99% of that repo might be the same". **Near**: the
same text differing by a few tokens — pasted licences, shared headers and footers,
LM1B articles differing by a single comma, and templates where someone "templatized it, replacing 'Canada' with 'USA'".

The example that lands is an audit of C4 finding one product description **repeated
61,036 times** — a garbled block of template-generated wedding-decoration copy
linked to an Amazon page for a gas mask. "The web is weird" (≈25:43).

Two independent reasons to deduplicate (≈26:30): efficiency, since "deduplication
reduces your dataset size without really losing information"; and **avoiding
memorization**, which "can mitigate copyright, privacy concerns" — a direct link
back to [lecture 13's copyright material](copyright-and-fair-use.md), since a
passage seen 61,036 times is far likelier to be regurgitated. The lecture adds that
decontamination against the test set is "arguably even more important" and uses the
same machinery.

The design space is three choices — what is an item, how do you match, what do you
remove — and the algorithmic constraint is that "you can't do the $n^2$ thing where
you compare everything to everything" (≈28:03). Everything after that is the
linear-time construction: fast hashes (MurmurHash, not SHA-256), exact
deduplication written MapReduce-style, C4's 3-sentence spans (which "break the
coherence" of the documents they are cut from), Jaccard similarity, MinHash, and
locality-sensitive hashing with its $b$ bands of $r$ hashes and the
$1 - (1 - s^r)^b$ S-curve.

→ [MinHash and LSH](minhash-and-lsh.md) works through all of it with the lecture's
computed values.

One operational warning that is easy to miss (≈48:34): deduplication is usually run
within each dataset, and that is not enough — "you actually have to do
deduplication across your entire dataset, because often datasets can be redundant with each other. Sometimes that's not done, but it should be."

## 4. Data mixing

At this point each source is clean; the question is the distribution over sources.

![Screenshot of an interactive bar chart listing about thirty datasets by token count, colour-coded by category](../raw/images/14-data-filtering-dedup-mixing/marin-token-viewer.png)

*Marin's token-count viewer — roughly thirty candidate sources for the next model, coloured by category. Bar values are visual estimates; the tool prints no data labels. [Source](https://github.com/stanford-cs336/lectures/blob/main/images/marin-token-viewer.png)*

The baselines are **vibes** ("more often than you might think what people do"),
**uniform**, and **proportional** mixing. Upweighting quality is the obvious
instinct, and two things complicate it: **diversity**, because sources like
literature, code and papers are "just incomparable objects"; and **finiteness**
(≈52:26).

### The epoching trap

The lecture stops here, and it is the passage most worth taking away. With 10T
low-quality tokens and 10B high-quality ones — "generally high quality sources are
smaller" — a plain 50/50 mixture trained for 1T tokens gives **0.05 epochs** on the
abundant source and **50 epochs** on the scarce one. Upweighting a small source does
not get you more of it; it gets you the same tokens fifty times.

"This is actually really important, and some big model training runs have kind of
messed this up" (≈55:28) — and the reason is that nothing signals it: "you're going to end up doing 50 epochs without realizing it, unless you're paying close attention"
(≈56:14).

**UniMax**'s answer is a hard **cap** on epochs per source rather than a smooth
interpolation between uniform and proportional mixing (≈58:34).

### Regression-based mixing

The principled method, and structurally the same move as
[scaling laws](scaling-laws.md): sample mixtures from a Dirichlet, train a swarm of
small proxy models, fit a regression from mixture to loss, optimize the fit, train
large on the answer.

![Four-step schematic of the RegMix pipeline](../raw/images/14-data-filtering-dedup-mixing/regmix.png)

*RegMix: proxy models over sampled mixtures → a regression model → a predicted surface over new mixtures → train at large scale on the predicted best (here 22.8% / 67.0% / 10.2%, predicted target 5.34). [Source](https://github.com/stanford-cs336/lectures/blob/main/images/regmix.png)*

![Table comparing seven data-mixing methods by design choice](../raw/images/14-data-filtering-dedup-mixing/data-mixing-methods.png)

*Seven methods in one framework — RegMix, DML, AutoScale, BiMix, ADMIRE-BayesOpt, CLIMB, OLMixBase — by proxy model size, swarm size and distribution, regression family, granularity, repetition constraints and solver. [Source](https://github.com/stanford-cs336/lectures/blob/main/images/data-mixing-methods.png)*

The choice of **target** is where this goes wrong: fit to downstream evals and "if you have a bunch of code evals, then, well, guess what, you're going to upweight all the code data... and if you then go and say, I want to generate some poetry, you might realize that you've overfit" (≈1:02:25). Uniform and proportional mixing are immune
precisely because they consult no evals.

The source labels its two assumptions **hopes**, with prayer emoji: that the
regression is accurate *at its minimizer*, and that optimal mixtures transfer from
small to large scale. The lecture presses both — optimization pushes the fit toward
extremes "where you might not have as much coverage", and transfer "seems to tend to be true, or not blatantly false" (≈1:04:44–1:05:32).

### The effect that does not transfer

And then a concrete counterexample. At small scale you never exhaust the scarce
source, so the search returns something like 10% low / 90% high — "it's like, oh
wow, Wikipedia is so great, let's just train on Wikipedia" — which at large scale
epochs enormously and overfits (≈1:06:19).

**Simulated epoching** fixes it by downsampling every source by the ratio of the two
run sizes (10B against 1T, so ×0.01), making the small experiment feel the scarcity
the large run will feel. The lecture names this as a course-wide theme and ties it
to [muP](maximal-update-parametrization.md): "make your small scale look like your
large scale" (≈1:07:51).

This is worth reading against [lecture 9](09-scaling-laws.md), which argued that
small-scale mixture bake-offs are sound *because* composition moves intercepts and
not slopes. Lecture 14 exhibits a mechanism by which the ranking genuinely fails to
transfer. → [Data mixture selection](data-mixture-selection.md) holds both halves.

A question from the floor added something the program does not contain: mixing can
be applied **within** a source, by grouping a corpus like Common Crawl into a
two-dimensional grid of domain × quality cells — AI2's WebOrganizer, as used by
Nemotron and OLMo — with each cell as a mixture component (≈1:11:44).

## 5. Post-training data

A deliberate change of subject. Everything above is task-agnostic; post-training
data is "very task dependent" (≈1:13:15). The recipe is three steps — define
environments, define tasks, collect responses from a strong teacher — and the third
is what has changed: "almost all the post-training data — most of it — is synthetically generated" (≈1:14:03).

**OpenThoughts** is the reasoning example: 1.2M examples from **QwQ-32B**, and three
findings that came out backwards — sixteen samples per prompt beats more sources;
**better models are not necessarily better teachers** (QwQ-32B beat DeepSeek-R1);
and answer filtering did not help (≈1:16:20).

![Sankey diagram of the OpenThoughts pipeline from five sources to a 1.2M final dataset](../raw/images/14-data-filtering-dedup-mixing/openthoughts-pipeline.png)

*The pipeline, with every quantity printed: 4.07M source questions filter to 300k, deduplicate to 190k, sample to 75k — and 75k × 16 answers = 1.2M exactly. "The 1.2 million is examples, but divided by 16 gives you the number of actual questions" (≈1:17:06). [Source](https://github.com/stanford-cs336/lectures/blob/main/images/openthoughts-pipeline.png)*

The rest of the section is four software-engineering datasets, and their common
problem is infrastructural rather than statistical: SWE tasks carry heavy
dependencies, "most GitHub repos don't even run", and building thousands of Docker
images is "an infrastructural nightmare" (≈1:19:27). **SWE-smith** generates bugs
into working repositories (128 repos → 50K tasks); **SWE-Zero** observes that strong
models solve many tasks with no execution at all — "if you were allowing execution, you get like 80; if you don't allow execution, you get almost 70" — and builds 300K trajectories
from real GitHub PRs on that basis; **SWE-rebench** uses a model to hypothesize the
install scripts; and **SWE-ZERO-12M**, which "just came out today", scales the
execution-free idea to 12M trajectories with a 1.7B generator.

→ [Post-training data](post-training-data.md) and
[agent trajectory data](agent-trajectory-data.md).

## Takeaways

The source's own summary:

> - Filtering: train classifier (language id, quality, toxicity) for what good looks like
> - Deduplication: hashing scales to large datasets for fuzzy matching
> - Mixing: try mixtures at small scale, extrapolate to optimal mixture and large scale
> - Post-training data: looks like evaluations, use of synthetic data
> - A lot of data work is domain-specific, looking at examples, etc.

Three things are worth carrying beyond that list.

**Each stage has a scale-dependent optimum.** There is no best filtering threshold
and no best mixture in the abstract — both depend on how long you will train, and
both can be found at small scale only if the small scale is made to resemble the
large one. That is the same argument the course makes for
[muP](maximal-update-parametrization.md) and for
[scaling laws](scaling-law-methodology.md), arriving here from a different
direction.

**A strong model labels, a cheap model imitates.** phi-1 uses GPT-4 to build a
target set and a random forest to apply it; SWE-rebench uses Qwen to install
dependencies; OpenThoughts distils QwQ-32B. The expensive model is never run over
the whole corpus.

**And the honest caveat**, which the lecture makes about itself (≈1:24:05):

> A lot of the data work can be very grungy. It's very domain specific and requires
> looking at concrete examples to make these high quality datasets. So this lecture
> is not really representative of what data work is like.

## See also

- [Lecture 13](13-data-sources-datasets.md) — the first half: sources, copyright,
  and the dataset tour
- [HTML-to-text extraction](html-to-text-extraction.md) ·
  [Quality classifiers](quality-classifiers.md) ·
  [MinHash and LSH](minhash-and-lsh.md) ·
  [Data mixture selection](data-mixture-selection.md) ·
  [Post-training data](post-training-data.md) ·
  [Agent trajectory data](agent-trajectory-data.md) — the six pages this lecture's
  material lives on
- [Data filtering](data-filtering.md) · [Deduplication](deduplication.md) ·
  [Synthetic data](synthetic-data.md) — the lecture-13 treatments this one extends
- [Data repetition](data-repetition.md) — the mechanism behind the epoching trap
- [Lecture 9](09-scaling-laws.md) — the scaling-side view of mixture selection,
  which this lecture partly contradicts
- [Course map](course-map.md)
