# Quality classifiers and the filtering framework

How filtering is actually implemented. [Lecture 13](13-data-sources-datasets.md)
gave the history of filtering and the rules-versus-models argument — that is
[data filtering](data-filtering.md). [Lecture 14](14-data-filtering-dedup-mixing.md)
gives the *algorithm*, in one abstract statement plus five worked recipes
(≈6:20–22:39). This page is the how-to.

## The building block

> Given some **target data** T and lots of **raw data** R, find subset T′ of R
> similar to T.

*Figure: `images/raw-target-schema.png`.*

![Schematic diagram of three ellipses illustrating the raw-data / target-data / filtered-subset relationship](../raw/images/14-data-filtering-dedup-mixing/raw-target-schema.png)

*The schematic that opens the filtering section: a large "Raw data R", a "More data T′" drawn inside it, and a separate "Target data T". T and T′ are related by matching colour, not by any drawn overlap — this is a conceptual picture, not a Venn diagram.*

T is "usually a small amount of high quality data"; R is "the fresh shipment of the tokens that you transform from the previous step" (≈7:07). Almost all filtering
falls into this schema, and the point of stating it abstractly is that three jobs
which sound unrelated turn out to be **the same algorithm with a different T**:

- **Language identification** — English versus everything else.
- **Quality filtering** — "the main reason for filtering." You want encyclopedia
  content, not spam (≈7:54).
- **Toxicity filtering** — "the internet has plenty of nasty content and maybe you
  don't want to train your language model on that" (≈7:54).

### Two requirements

1. **Generalize from the target data.** You want T and T′ to be *different*. "you already have the target data — you don't want to just get the target data back"
   (≈7:54). A filter that returns a copy of T has bought you nothing.
2. **Be extremely fast.** It runs on all of R: "this could be 100 trillion tokens
   worth of data" (≈8:40). Per-document cost is the binding constraint, which is
   why the classifiers below are all cheap ones.

The output is small: "generally with filtering you end up with a very small fraction — a single-digits fraction — of your entire data" (≈8:40).

## The general framework

1. Estimate a model from R and T, and derive a scoring function.
2. Keep examples in R based on their score.

Two families of scoring function (≈9:27):

| | Model | Score | Needs |
| --- | --- | --- | --- |
| **Generative** model of T | KenLM | $\text{score}(x) = p_T(x)$ | T only |
| **Discriminative** classifier | fastText | $\text{score}(x) = p(T \mid x)$ | T and R |

The generative route fits a model of the target and asks how likely a document is
under it — and it has to be cheap, so "probably you're not training a big language model. Generally, KenLM basically says, I'm going to train a five-gram model" (≈9:27).

The discriminative route is now the common one: "a more common thing, which I think
we're mostly seeing these days, is just to train a classifier" — positives are T,
negatives are a random subset of R, balanced, and "the tool that people generally
use is **fastText**, because it's fast, and it's generally just a linear classifier bag of words" (≈9:27–10:13).

Then: **keep documents with score above a threshold, sometimes stochastically**.
The threshold is set "depending on your quality bar" and is "generally fairly
heuristic" (≈12:34).

### Why nearly everyone now filters with a model

Lecture 13 recorded a real split — C4, Gopher, RefinedWeb, FineWeb and Dolma
deliberately avoided model-based filtering, while GPT-3, LLaMA and DCLM embraced
it. Lecture 14's assessment is that the argument is over: "these days I think basically everyone does some amount of model-based filtering", and the reason is economic —
"unless you are compute-plentiful — in which case you probably don't need to do as much filtering, you can just train on everything — **most people are compute-poor**, and you have to be very smart about how you're filtering, otherwise you're just wasting flops on low-quality content" (≈11:01).

## Five instantiations

### 1. Language identification

Meta's off-the-shelf **fastText language identification** model supports **176
languages**, trained on multilingual sites: **Wikipedia**, **Tatoeba** (a
translation site) and **SETimes** (Southeast European news). **Dolma** keeps pages
with $p(\text{English}) \geq 0.5$.

The lecture's assessment is that this is the easy one: "language ID is generally a fairly easy problem, compared to many other tasks that we're dealing with. A simple classifier — if you look at a few words, you can tell that it's Spanish or Japanese." There are
subtleties from code-switching and dialects, "so I wouldn't say it's an absolutely
solved problem, but this is not really the bottleneck for training a good language
model" (≈11:47).

Note what the training-data list implies: an off-the-shelf language identifier is
itself the product of a data-selection decision, and its 176-language coverage
rests on an encyclopedia, a sentence-translation site, and a regional news corpus.

### 2. OpenMathText — all three mechanisms at once

[arXiv 2310.06786](https://arxiv.org/pdf/2310.06786). The goal is a large corpus of
mathematical text out of Common Crawl, and it is the fullest worked example in the
lecture because it uses a rule, a generative model and a classifier together
(≈12:34–14:06):

1. **Rules** — does the document contain LaTeX commands?
2. **KenLM trained on ProofPile**, keeping documents with **perplexity < 15,000**.
3. **A fastText classifier** for mathematical writing, with **two thresholds:
   0.17 if the document already has math markup, 0.8 if it does not.**

The two thresholds are the stochastic-keep idea made concrete: a document already
carrying LaTeX clears a low bar, one without it must clear a high one.

The result is the strongest quantitative claim for filtering anywhere in the
lecture: **14.7B tokens**, used to train 1.4B models that "are better at math than models that were trained on 20 times as much data, which was not filtered in this way" (≈13:20). (The lecture rounds this to "15 billion tokens" when speaking; the
source program's figure is 14.7B.)

The general lesson is stated immediately after: **quality is whatever you define it
to be.** "quality filtering, again, think about it as a tool — you can define quality however you want, there's no universal notion of quality. If you define quality to be math, then you can go and get math, and you get better at it"
(≈14:06).

### 3. GPT-3 — classify against the known-good, and keep stochastically

[arXiv 2005.14165](https://arxiv.org/pdf/2005.14165), Appendix A.

- **Positives:** samples from {Wikipedia, WebText2, Books1, Books2} — the four
  sources GPT-3 already trusted.
- **Negatives:** samples from Common Crawl.
- **Model:** a linear classifier on word features.
- **Keep rule:** stochastic, and the program gives it as code:

```python
def keep_document(score: float) -> bool:
    return np.random.pareto(9) > 1 - score
```

A document with score $s$ is kept when a draw from a Pareto distribution with shape
9 exceeds $1 - s$: high-scoring documents almost always, low-scoring ones
occasionally. That preserves some of the raw distribution instead of cutting it
off. (The function is defined and never called in the lecture — it is there to show
the rule.)

### 4. LLaMA / RedPajama — one word different

[arXiv 2302.13971](https://arxiv.org/pdf/2302.13971). Same shape as GPT-3 with one
change, emphasized in the source: the positives are not Wikipedia but **pages
*referenced* by Wikipedia**, "not Wikipedia articles themselves" (≈14:52). That
gets you web pages a human editor thought worth citing — a better proxy for "good
web page" than encyclopedia prose. The keep rule is a hard classification rather
than a stochastic draw.

### 5. phi-1 — an expensive labeller, then a cheap classifier

[arXiv 2306.11644](https://arxiv.org/pdf/2306.11644). The philosophy is "really
high quality data (textbooks) to train a small model", and the filtering half runs
(≈14:52–16:24):

- **R** = the Python subset of The Stack.
- **The prompt** = "determine its educational value for a student whose goal is to
  learn basic coding concepts".
- **T** = use **GPT-4** with that prompt to classify a **100K subset of R**, and
  keep the positives.
- **The classifier** = a **random forest** trained on T, using output embeddings
  from a pretrained codegen model. (The lecture notes fastText would probably have
  worked too.)
- **Select** everything in R the classifier calls positive.

The structural point, and the reason this recipe recurs: **the target is itself the
output of an expensive classifier, and you then train a cheap classifier to imitate
it** (≈15:37). GPT-4 cannot be run over all of R; a random forest can.

The result on [HumanEval](https://huggingface.co/datasets/openai_humaneval) is
better on both axes at once: a 1.3B model trained on the raw Python subset of The
Stack reaches **12.19% after 96K steps**; trained on the filtered subset it reaches
**17.68% after 36K steps**.

### 6. Toxicity in Dolma

Same framework, with T supplied by an annotated dataset rather than inferred. The
**Jigsaw Toxic Comments dataset (2018)** annotates **Wikipedia talk page comments**
with `{toxic, severe_toxic, obscene, threat, insult, identity_hate}`, and the
project's stated goal was "to help people have better discussions online" (≈16:24)
— not pre-training. A filter built on it inherits that definition of toxicity, and
the provenance is worth keeping in mind before treating its output as neutral.

## There is no optimal threshold

This is the subtlety the lecture stops to make, and it undercuts any attempt to
quote a filtering setting out of context: **the right threshold depends on how long
you are going to train** (≈17:12).

> Intuitively, if you are going to train for a longer period of time then you can
> tolerate lower quality data. If you're training for shorter then you want higher
> quality data.

*Figure: `images/data-filtering-scale.png`.*

![Line chart, loss vs. log-scale tokens trained, comparing 5 text-extraction and quality-filtering methods](../raw/images/14-data-filtering-dedup-mixing/data-filtering-scale.png)

*A preliminary experiment (the lecture attributes it to Michael Ryan): 157M-parameter models, 100 WARCs — a tiny slice of Common Crawl — trained for increasing token counts. Loss on the y-axis, tokens trained on a log x-axis.*

The chart carries the argument in two curves (≈18:47–19:35):

- **dclm (blue)** — heavily filtered, and therefore a small pool. Loss falls, then
  **turns back up**: with so little data you must epoch over it, and eventually you
  overfit. It ends as the worst series on the plot.
- **resiliparse (purple)** — essentially no filtering, so a much larger pool. It
  starts **much worse** and decreases monotonically across the whole range,
  overtaking dclm and never turning up.

The lecture's reading: "high quality data is better in this regime where you're not
epoching, but once you get to lots and lots of tokens, high quality data is no
longer that great." And the sharper version: you would stop the high-quality run
before it overfits, "but even at this point, this is worse than if you had trained
for longer using low quality data" (≈19:35).

**Two cautions before quoting this figure.** Its legend lists nine methods but only
**five are plotted** — `llm_curated`, `llm_curated_dclm_filtered`, `nemotron_full`
and `nemotron_qhigh` appear in the legend with no visible curve, so no value may be
read for them. And it is described as a preliminary experiment, not a published
result.

Two questions from the floor are worth recording. Asked whether each point — one
training run — needs a confidence interval, the answer was that it would be good
practice, but runs are expensive and "in reality, when we have done these experiments, it generally tends to be stable, I would say, at least for pre-training" (≈20:21).
Asked whether *more* high-quality data would also show diminishing returns, the
answer was yes — "every dataset is going to have diminishing returns eventually,
that's finite" — but that its curve "would just be down here and just keep on going
down" (≈21:07).

This is the same effect that reappears in
[data mixture selection](data-mixture-selection.md) as the epoching problem, and it
is why an optimal mixture found at small scale can be catastrophic at large scale.

## The recipe, stated

> Filtering is pretty critical for building a good model, especially when you're
> compute-limited like most of us. Technically if you have infinite compute you
> don't need to filter — but realistically everyone has to filter. (≈21:53)

And the two ways to obtain T (≈22:39):

1. **Find a dataset you already like** and ask for more of it.
2. **Craft a prompt to a language model**, use it to pre-filter a large pool, then
   train a smaller classifier on the result and extrapolate to everything else.

The second is phi-1's recipe, and it is now the standard one.

## See also

- [Data filtering](data-filtering.md) — the history, and the rules-versus-models
  debate from [lecture 13](13-data-sources-datasets.md)
- [Lecture 14](14-data-filtering-dedup-mixing.md) — the lecture this comes from
- [HTML-to-text extraction](html-to-text-extraction.md) — the stage before
- [MinHash and LSH](minhash-and-lsh.md) — the stage after
- [Data mixture selection](data-mixture-selection.md) — where the scale-dependence
  argument returns
- [Synthetic data](synthetic-data.md) — using a model to *make* data rather than
  select it
- [Course material for lecture 14](../raw/slides/14-data-filtering-dedup-mixing.md#filtering)
