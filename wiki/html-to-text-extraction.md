# HTML-to-text extraction (and PDFs)

The first stage of the data pipeline, and the one most easily assumed away.
"Raw data does not come as text. It is HTML, PDF (arXiv), or directories (code
repositories)" — so before any filtering or deduplication can happen, something has
to turn bytes into a token sequence. [Lecture 14](14-data-filtering-dedup-mixing.md)
opens with this (≈1:40–6:20); [web crawling](web-crawling.md) covers how the bytes
were obtained in the first place.

Most of the effort goes to HTML, "because most of the web is in HTML" (≈1:40).

## What the extractor has to do

Two jobs: **remove boilerplate** — navigation, ads, headers, footers, menus — and
**extract content**, the main body of the page.

Neither is well defined. "What is content and what is not content is not always
clear" (≈2:26), and the lecture gives a reason not to be glib about it: you
normally strip navigation elements, "but you could imagine cases where some of the
navigation elements might be helpful, to learn what web pages look like" (≈1:40).
The boilerplate is not noise in an absolute sense; it is noise relative to what you
want the model to learn.

Then there is everything that is not prose. Images and tables both sit inside the
page, and only one of them has any chance of surviving.

## Why it is inherently lossy

This is a property of the data, not a failure of the tool. HTML is **hierarchical**
— and, rendered, **visual** — while a training example is a flat sequence of
tokens, so linearizing it necessarily throws structure away (≈2:26).

Tables are the sharp case. "Simple tables you can render using Markdown, but if you
have nested tables then that becomes quite challenging — you have to give up at
some point, or approximate at some point" (≈2:26). A nested table has no faithful
linear form, so every pipeline is choosing how to fail.

## Why the tools are rule-based

The named tools — **trafilatura**, **resiliparse**, **jusText**, **lynx** — are all
rule-based, and the lecture gives two reasons (≈3:12):

1. **Speed.** These run over the entire crawl, so per-document cost is the binding
   constraint.
2. **You do not need much intelligence for the task.** "You're not trying to do too
   much here... so rule-based generally works."

The lecture does leave the door open — "I think there could be a case for
model-based interventions at this point. They have to be very fast, and they have
to do something more intelligent" — but immediately notes the cost of the current
approach: "if you ever look at data, you'll notice there are imperfections in the
data, just because any rule-based processing is going to have some failure rate"
(≈3:12).

## Accuracy matters, and the differences are measurable

DCLM ([arXiv 2406.11794](https://arxiv.org/abs/2406.11794)) compared extractors by
training on their output and evaluating downstream:

*Figure: `images/dclm-wet.png`.*

![Small table comparing three HTML-to-text extraction tools (resiliparse, trafilatura, WET files) on CORE and EXTENDED benchmark scores](../raw/images/14-data-filtering-dedup-mixing/dclm-wet.png)

| Text extraction | CORE | EXTENDED |
| --- | --- | --- |
| resiliparse | 24.1 | **13.4** |
| trafilatura | **24.5** | 12.5 |
| WET files | 20.7 | 12.2 |

Bolded values are the column-wise best, as printed in the source table.

Two readings. First, the two dedicated extractors are close — trafilatura wins CORE
by 0.4, resiliparse wins EXTENDED by 0.9 — so **the choice between them is a real
but modest effect**, and the lecture's spoken summary is appropriately hedged:
"resiliparse, on these extended DCLM evals, works better than the others" (≈3:59).

Second, and much larger: **Common Crawl's own WET files are worst on both
columns**, by 3.4 and 1.2 points. WET is the pre-extracted plain text Common Crawl
ships alongside the raw WARC, and taking it instead of re-extracting from the HTML
costs more than any choice between extractors does. That is the concrete form of
the argument in [web crawling](web-crawling.md#warc-vs-wet-and-why-html-to-text-is-a-modelling-decision):
WET is somebody else's extraction decision, made once, for everybody.

## The PDF path

PDFs are the same problem, harder at every step. The reference work is
**FinePDFs** ([HuggingFace blog](https://huggingface.co/spaces/HuggingFaceFW/FinePDFsBlog)),
built from Common Crawl (≈4:44–6:20).

- **You may not know a document is a PDF before you fetch it.** Common Crawl focuses
  on text, but "if you're just given a URL, you might not even know before you fetch
  it whether it's a PDF or not, if it doesn't have the extension."
- **Many crawled PDFs are truncated**, because PDFs are big — which means the
  pipeline has to **re-crawl** them to get complete files.
- **Extraction means OCR**, not parsing. Some PDFs "might be just scans as well, right, so they're essentially images." The tools named are **RolmOCR** (a vision-language model) and
  **Docling**. This "obviously can be much more expensive than what we were doing
  before with text", and the source's own parenthetical instruction is "(make these
  run fast)".
- **Layout information is lost.** This is worse than for HTML and for a structural
  reason: "in HTML you have various tags, like H1 and P, that give you some semantic information. PDFs are by design all about layout, so they don't necessarily preserve the kind of semantic structure" (≈6:20).

### Why bother

Because of a quality argument the program does not state and the lecture does
(≈5:32). PDFs are "a very small fraction of the whole internet, but they are very
valuable, because generally if you bother to make a PDF that means you probably
have something interesting to say, as opposed to a web page. So the quality of a PDF — an average PDF — is generally higher than for an HTML file."

That is a **selection effect used as a quality signal** — the same move that
[quality classifiers](quality-classifiers.md) make deliberately, arriving here for
free from the format itself.

## Where this sits

Transformation is upstream of everything else in the pipeline: filtering,
deduplication and mixing all operate on its output, and none of them can recover
what it discarded. It is also the step with the least glamorous failure mode —
a table flattened wrongly or a navigation bar left in does not announce itself, it
just becomes training data.

## See also

- [Lecture 14](14-data-filtering-dedup-mixing.md) — the lecture this comes from
- [Web crawling](web-crawling.md) — WARC vs WET, and how the bytes were obtained
- [Quality classifiers](quality-classifiers.md) — the next stage
- [Data filtering](data-filtering.md) — the history and the rules-vs-models debate
- [Pre-training datasets](pretraining-datasets.md) — which corpus used which
  extractor
- [Course material for lecture 14](../raw/slides/14-data-filtering-dedup-mixing.md#transformation-raw-bytes-to-text)
