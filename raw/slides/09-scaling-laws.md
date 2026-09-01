---
title: Lecture 9 — Scaling Laws (Basics) (course material)
lecture: 9
instructor: Tatsunori Hashimoto
source_format: slide-deck-pdf
source_file: lecture_09.pdf
source_repo: https://github.com/stanford-cs336/lectures
source_url: https://github.com/stanford-cs336/lectures/blob/main/lecture_09.pdf
pages: 57
method: page-images
numbering: >
  This deck prints NO page number on any page. The slide labels below are
  therefore PDF page numbers, not printed slide numbers: "Slide N" means "PDF
  page N", for N = 1..57, one heading per page, in order. Cite them as page
  numbers of lecture_09.pdf. The mapping was settled before any page was read.
  slide_number_map.py found no printed number anywhere; a whole-page edge scan of
  the text layer (both bottom corners plus the full margin region) returned only
  TWO bare numeric tokens across all 57 pages, both on page 33, at (435,368) and
  (455,366) on a 720x405-point page — inside the plot area, not a corner. The
  reader assigned to that range independently identified them as the "10^7"
  x-axis tick of slide 33's own chart, which closes the question. This is the
  second deck in the build (after lecture 8) to print no folio at all.
  Because the script's map is a 1..57 fallback rather than something read off the
  pages, its --verify mode degenerates into a heading-sequence check; that check
  passes (57 headings, 1..57, no gaps, no merges, no duplicates). Independently,
  56 of the 57 headings were matched verbatim against the PDF text layer at the
  top of their own page. The single exception is the title page, whose text layer
  letter-spaces "S C A LIN G LAW S - BA SIC S" — the same artifact lecture 8's
  title page showed.
figures: >
  93 raster images across 57 pages against 2,063 words of native text — about 36
  words per page, the LOWEST text density of any deck in this build (lecture 8
  ran 38). The pasted figures are not an illustration of this lecture, they are
  its content: nearly every substantive claim is a log-log plot. Only six pages
  carry no raster image at all (2, 5, 12, 21, 29, 54), and those are the section
  dividers and the two prose slides. Note that "raster image" counts pasted
  equations and pasted tables as well as charts, so a page whose entry below says
  "No chart or table on this page" may still hold a raster — slides 17, 19, 27
  and 28 are the cases here, and their rasters are pasted equations and paper
  headers rather than figures.
  Every figure below was described from the rendered page image, re-rendered and
  cropped at 600-4800 dpi wherever labels were small. Two readers went further
  than eye-reading where colour identity was load-bearing: slide 20's six dataset
  categories and slide 4's five layer-count series were resolved by classifying
  pixels against the legend swatch RGB, with axis pixel-to-value calibration off
  the detected tick marks. That method caught two errors that eye-reading had
  made — a CIFAR-10 point misassigned to CIFAR-100 on slide 20, and four spurious
  data points on slide 4 that were the legend swatches themselves sitting inside
  the plot's coordinate space. Where something could not be resolved at any
  magnification the entry says so rather than guessing; those cases are collected
  under "Known limits" below.
audit: >
  NOT YET RUN. The figure audit for this deck has not been performed. Until it
  has, treat the chart values below as a single-pass transcription: on the two
  decks in this build where an audit was run against a Sonnet read, it found real
  errors both times, including transposed series colours and a described figure
  that was not on the page. Do not cite a number from this file as settled until
  this field records an audit.
math: >
  Equations were transcribed from the rendered page, never from the text layer,
  because extraction flattens fractions onto a single line and silently produces
  plausible-looking wrong formulas. The Chinchilla parametric loss form and its
  fitted exponents (slides 46-52) are the most-quoted equations in this lecture
  and were transcribed character by character at up to 4800 dpi.
---

# Lecture 9 — Scaling Laws (Basics) — full slide text

Tatsunori Hashimoto. 57 slides. This is the first of two scaling-laws lectures;
the advanced one comes later in the course, and lecture 10 (Inference) is
delivered in between.

**Slide numbers are PDF page numbers** — this deck prints no folios. See the
`numbering` note in the front matter.

## Sections

| Slides | Section |
|---|---|
| 1–4 | Title and framing — why scaling has to be taken seriously, and what a predictive "law" buys you |
| 5 | *Part 1 divider* — scaling laws, history and background |
| 6–11 | Prehistory: statistical sample-complexity rates, the 1993 data-scaling paper, Banko & Brill, Kolachina, and Hestness et al 2017 |
| 12 | *Part 2 divider* — neural (LLM) scaling behaviors |
| 13–15 | Data scaling laws: the power-law relationship and its form for language models |
| 16–20 | Why data scaling laws hold — mean-estimation toy example, the exponent mystery, nonparametric rates, intrinsic dimensionality |
| 21–26 | Advanced data scaling: distribution shift, data-mixture selection, repetition, compute-unbounded settings, data selection under finiteness |
| 27 | Recap of data scaling laws |
| 28–29 | Scaling laws for model engineering — the hyperparameter questions |
| 30–31 | 1. Architecture: transformers vs LSTMs, and cross-architecture scaling |
| 32 | 2. Optimizer choice (Adam vs SGD) |
| 33–36 | 3. Depth and width, other transformer hyperparameters, and the "value of a parameter" under MoE |
| 37–39 | 4. Batch size and critical batch size |
| 40 | 5. Learning rates — muP and scale-aware choices |
| 41–42 | Caution: scaling behavior can differ downstream; surprising takeaways |
| 43–45 | The headline use of scaling laws — more data or a bigger model? Joint model-data scaling |
| 46–49 | Chinchilla in depth — the three methods (minimum over runs, IsoFLOPs, joint fits) |
| 50–53 | Why Kaplan and Chinchilla disagree — two explanations, and errors in Chinchilla's method 3 |
| 54 | Important caveat — train-optimal is likely not what you want |
| 55–56 | IsoFLOPs everywhere; scaling laws for models and compute |
| 57 | Recap |

## Known limits

Collected from the four readers. Each is a place where the source PDF, not the
reading pass, sets the ceiling.

- **Slide 31** (Tay et al 2022, 11-panel small-multiples grid plus a ~100-point
  aggregate scatter): dozens of overlapping point labels in the central FLOPS
  band, roughly 4×10^12 to 3×10^13, could not be individually resolved at any
  magnification. The entry describes the cluster as dense and unresolved rather
  than guessing at members. For the same page the reader described the shared
  structure and per-panel pattern rather than transcribing all ~110 individual
  values, calling out the two panels that visibly deviate.
- **Slide 41** (both panels): the top-scoring point's label is clipped in the
  source image itself, leaving "NL12-" with nothing legible after the dash.
  Confirmed at 3600 dpi — the pasted figure is cropped at that edge, so no
  magnification recovers it.
- **Slide 52**: the legend of the small pasted inset chart ("Generate training
  curves for model sizes used in Kaplan's study", roughly 19 N_T values) is
  illegible at its native resolution. Separately, the two "Local fit to frontier"
  equations in the middle two-panel chains are legible only for their leading
  terms; their intercept digits are unresolvable in those insets. Both intercepts
  ARE legible on the large right-hand chart of the same page (−15.00 and −3.16),
  which the entry records as the resolved cross-reference.
- **Slide 51**: the pasted paper-header image crops off the first author's given
  name, leaving "…ell Wortsman†". Transcribed as printed rather than completed
  from outside knowledge.
- **Slide 49**: two open/hollow-outline data points near 10^19 FLOPs are visually
  distinct from the solid "Empirical data" dots but carry no caption explaining
  them. Flagged as visually distinct with no meaning asserted.
- **Slide 3**: the `model_dim` column header exists but every one of its 20 cells
  is genuinely blank in the source table. This is the deck's own gap, not a
  reading failure.

## Notes on the deck itself

- **Slides 8 and 9 carry the identical title** "Early history of scaling laws –
  data scaling", confirmed from the text layer of both pages rather than inferred.
  They cover different material — Banko & Brill 2001 on slide 8, Kolachina et al
  2012 on slide 9 — and are kept as two entries with duplicate titles.
- **Slides 45 and 50 embed the same image object**, verified byte-identical by
  MD5 of the extracted image stream. The deck deliberately shows the Kaplan-vs-
  Chinchilla comparison chart again when it turns to explaining the discrepancy.
- **Slide 14 reuses slide 10's schematic** (the "Small Data / Power-law region /
  Irreducible error" diagram). Both entries describe it independently.
- Two spelling slips in the deck are transcribed as printed and flagged inline
  rather than silently corrected: "Combing the two" for "Combining" (slide 21)
  and a missing apostrophe in "Lets turn that into an example" (slide 19).
- **Slide 25**'s paper byline uses "∞" as the equal-contribution superscript,
  confirmed at 3600 dpi as the deck's own choice rather than a rendering artifact.
- **Slide 43**'s chart legend lists parameter counts out of size order — "708M,
  302M, 85M, 3M, 25M, 393.2K" — transcribed in the printed order.

## Slide 1 — Lecture 9: Scaling Laws - Basics

The title slide. Centred on a white page, in black: "**Lecture 9**". Beneath it, in blue letter-spaced caps: "S C A L I N G   L A W S   -   B A S I C S" (the deck's own text layer renders this as "S C A LIN G LAW S - BA SIC S", i.e. letter-spaced with irregular internal spacing, consistent with how title-page text extracts on these decks). Below that, in grey: "CS336". A wide blue band runs across the bottom of the page. No figure.

## Slide 2 — Taking scaling seriously

Heading: "Taking scaling seriously". Body text: "**Imagine the following scenario..**". Centred and indented below that: "Your friend has given you ten thousand B200s for a month, and asked you to build a good open source LM." Below that: "What do you do?" followed by three bullets:

- Put together your infra team and distributed training framework (A2)
- Put together a great pretraining dataset (A4)
- Run a big model (but which one??) <- we are here.

No chart or table on this page.

## Slide 3 — Scaling isn't easy

Heading: "Scaling isn't easy". Body text: "Wide or deep? How many heads? Which nonlinearity?"

**Table — a 20-row model-architecture database, pasted as a dark-mode screenshot.** Columns, in order: Name, Has paper, Link, Year, Tokenizer type, Vocab count, Norm, Parallel Layer, Pre-norm, Position embedding, Activations, MoE, MLP factor, num_layers, model_dim. The `model_dim` column is present as a header but is **blank for every row** — no value is filled in anywhere in that column. Every other cell below is exactly as printed; a blank in any other column below means the source cell itself was empty.

| Name | Has paper | Link | Year | Tokenizer type | Vocab count | Norm | Parallel Layer | Pre-norm | Position embedding | Activations | MoE | MLP factor | num_layers |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Original transformer | Yes | arxiv.org/abs....03762 | 2017 | BPE | 37000 | LayerNorm | Serial | ☐ | Sine | ReLU | ☐ | 4 | 6 |
| GPT | Yes | cdn.openai.com/res...er.pdf | 2018 | BPE | 40257 | LayerNorm | Serial | ☐ | Absolute | GeLU | ☐ | 4 | 12 |
| GPT2 | Yes | cdn.openai.com/bet...rs.pdf | 2019 | BPE | 50257 | LayerNorm | Serial | ☑ | Sine | GeLU | ☐ | 4 | 48 |
| T5 (11B) | Yes | arxiv.org/abs....10683 | 2019 | SentencePiece | 32128 | RMSNorm | Serial | ☑ | Relative | ReLU | ☐ | 64 | 24 |
| GPT3 (175B) | Yes | arxiv.org/abs....14165 | 2020 | BPE | 50257 | LayerNorm | Serial | ☑ | Sine | GeLU | ☐ | 4 | 96 |
| mT5 | Yes | arxiv.org/abs....11934 | 2020 | SentencePiece | 250000 | RMSNorm | Serial | ☑ | Relative | GeGLU | ☐ | 2.5 | 24 |
| T5 (XXL 11B) v1.1 | Kind of | github.com/goo...d#t511 | 2020 | SentencePiece | 32128 | RMSNorm | Serial | ☑ | Relative | GeGLU | ☐ | 2.5 | 24 |
| Gopher (280B) | Yes | arxiv.org/abs....11446 | 2021 | SentencePiece | 32000 | RMSNorm | Serial | ☑ | Relative | ReLU | ☐ | 4 | 80 |
| Anthropic LM (not claude) | Yes | arxiv.org/abs....00861 | 2021 | BPE | 65536 | *(blank)* | *(blank)* | ☑ | *(blank)* | *(blank)* | ☐ | 4 | 64 |
| LaMDA | Yes | arxiv.org/abs....08239 | 2021 | BPE | 32000 | *(blank)* | *(blank)* | ☑ | Relative | GeGLU | ☐ | 8 | 64 |
| GPTJ | Kind of | huggingface.co/Ele...t-j-6b | 2021 | BPE | 50257 | LayerNorm | Parallel | ☑ | RoPE | GeLU | ☐ | *(blank)* | 28 |
| Chinchilla | Yes | arxiv.org/abs....15556 | 2022 | SentencePiece | 32000 | RMSNorm | Serial | ☑ | Relative | ReLU | ☐ | 4 | 80 |
| PaLM (540B) | Yes | arxiv.org/abs....02311 | 2022 | SentencePiece | 256000 | RMSNorm | Parallel | ☑ | RoPE | SwiGLU | ☐ | 4 | 118 |
| OPT (175B) | Yes | arxiv.org/abs....01068 | 2022 | BPE | 50272 | LayerNorm | Serial | ☑ | Absolute | ReLU | ☐ | 4 | 96 |
| BLOOM (175B) | Yes | arxiv.org/abs....05100 | 2022 | BPE | 250680 | LayerNorm | Serial | ☑ | AliBi | GeLU | ☐ | 4 | 70 |
| GPT-NeoX | Yes | arxiv.org/pdf...45.pdf | 2022 | BPE | 50257 | LayerNorm | Parallel | ☑ | RoPE | GeLU | ☐ | 4 | 44 |
| GPT4 | Ad (badged "OPEN") | arxiv.org/abs....08774 | 2023 | BPE | 100000 | *(blank)* | *(blank)* | ☐ | *(blank)* | *(blank)* | ☐ | *(blank)* | *(blank)* |
| LLaMA (65B) | Yes | arxiv.org/abs....13971 | 2023 | BPE | 32000 | RMSNorm | Serial | ☑ | RoPE | SwiGLU | ☐ | 2.6875 | 80 |
| LLaMA2 (70B) | Yes | arxiv.org/abs....09288 | 2023 | BPE | 32000 | RMSNorm | Serial | ☑ | RoPE | SwiGLU | ☐ | 3.5 | 80 |
| Mistral (7B) | Yes | arxiv.org/abs....06825 | 2023 | BPE | 32000 | RMSNorm | Serial | ☑ | RoPE | SwiGLU | ☐ | 3.5 | 32 |

Notes on the table as rendered: "Has paper" is a coloured pill per row — purple "Yes" for most rows, red "Kind of" for T5 (XXL 11B) v1.1 and GPTJ, and a grey "Ad" pill (with a separate dark "OPEN" badge next to the GPT4 name) for GPT4. "Pre-norm" and "MoE" are checkbox columns; every row's MoE box is unchecked. "Parallel Layer" is a coloured pill reading either "Serial" (gold) or "Parallel" (blue); it is blank for the three rows whose Norm is also blank (Anthropic LM, LaMDA, GPT4). Link cells are truncated URLs as printed (e.g. "arxiv.org/abs....03762"); the full URLs are not recoverable from the pasted image.

Below the table, body text: "We could cargo cult things from existing LMs… but how do these get optimized in the first place?"

## Slide 4 — Today: simple, predictive 'laws' for behaviors of LMs

Heading: "Today: simple, predictive 'laws' for behaviors of LMs". Body text: "The approach -" then, indented: "**scaling laws** which are simple, predictive rules for model performance". Below that: "**Old and unpleasant**: tune hyperparameters on big models" and "**New (over?) optimism**: tune on small models, extrapolate to large ones".

**Figure 1 (left) — Kaplan-style validation-loss-vs-compute plot, many curves colored by parameter count.** Y-axis "Validation Loss", linear scale, range 1.5 to 6, with gridlines/labelled ticks at 2, 3, 4, 5, 6 (and the bottom of the axis labelled 1.5). X-axis "Compute (PetaFLOP/s-days)", log scale, labelled ticks at $10^{-6}$, $10^{-4}$, $10^{-2}$, $10^{0}$, $10^{2}$, $10^{4}$. A vertical colorbar to the right, titled "Parameters", log scale, ticked $10^5$ (dark purple, bottom) through $10^{11}$ (yellow, top), viridis colormap. The body of the plot is a family of roughly 15-20 individual training curves (one per model size), each tracing that model's validation loss as compute increases: small models (dark purple/blue, ~$10^5$–$10^7$ params) descend briefly then bend and flatten out early, at loss values around 4-4.5, using very little of the compute range; progressively larger models (teal, then green, ~$10^8$–$10^{10}$ params) descend further before flattening, reaching lower losses (~2.5-3) at correspondingly larger compute; the single largest, yellow curve (~$10^{11}$ params) descends latest and furthest, from about (1, 5.7) down to about ($5\times10^3$, 1.7). A dashed grey line, labelled in a boxed legend entry "$L = 2.57 \cdot C^{-0.048}$", runs as the lower-envelope power-law fit across the whole plot, from about ($10^{-6}$, 5) down to about ($10^{4}$, 1.4) — each individual model's curve touches this envelope line at the compute value where that model size is optimal, then departs upward (flattens) beyond it.

**Figure 2 (right) — test loss vs. non-embedding parameter count, by number of layers.** Y-axis "Test Loss", linear scale, range 2 to 7, labelled ticks at 2, 3, 4, 5, 6, 7. X-axis "Parameters (non-embedding)", log scale, labelled ticks at $10^3$ through $10^9$ (plot extends slightly past $10^9$, to roughly $3\times10^9$). Five series, per the legend (top to bottom): **1 Layer** (dark indigo/purple), **2 Layers** (magenta), **3 Layers** (crimson/pink), **6 Layers** (orange), **>6 Layers** (gold/amber). Each series is a line with square markers at its actual data points; the series do **not** all span the same x-range:

- **1 Layer**: starts earliest/leftmost, at $x\approx7\times10^2$, $y\approx6.55$; descends steadily as the topmost (worst-loss) curve; passes about ($3\times10^6$, 4.8); ends at $x\approx5.5\times10^7$, $y\approx4.23$ — it stops well before the other series and does not reach the right side of the plot.
- **2 Layers**: starts at $x\approx5.9\times10^3$, $y\approx6.25$; passes about ($1.1\times10^5$, 5.3); ends at $x\approx1.1\times10^8$, $y\approx3.60$.
- **3 Layers**: starts at $x\approx8.9\times10^3$, $y\approx6.25$ (essentially tied with 2 Layers' start height, at a slightly larger x); passes about ($4\times10^4$, 5.74), ($1.7\times10^5$, 5.25), ($7\times10^5$, 4.76); ends at $x\approx1.7\times10^8$, $y\approx3.36$ — it extends slightly further right, and ends slightly lower, than 2 Layers.
- **6 Layers**: starts much later, at $x\approx1.1\times10^6$, $y\approx4.66$; passes about ($3\times10^6$, 4.38), ($9\times10^6$, 4.06), ($2.8\times10^7$, 3.73), ($8\times10^7$, 3.44); ends at $x\approx1.45\times10^9$, $y\approx2.72$.
- **>6 Layers**: starts latest of all, at $x\approx2.3\times10^7$, $y\approx3.76$; passes about ($4.4\times10^7$, 3.60), ($8.2\times10^7$, 3.39), ($1.6\times10^8$, 3.22), ($2.9\times10^8$, 3.02); ends at $x\approx1.6\times10^9$, $y\approx2.61$ — the rightmost and lowest-loss endpoint of any series.

At the far right the four deeper series (2, 3, 6, >6 Layers) converge into a tight cluster while 1 Layer has long since stopped; 6 Layers and >6 Layers extend furthest right, with >6 Layers ending marginally below (better than) 6 Layers. The point of the pair of charts, per the slide's own framing, is that loss is well predicted as a simple (power-law-like) function of compute or of parameter count, largely independent of other architectural choices like depth — hence "simple, predictive laws."

## Slide 5 — Part 1. Scaling laws, history and background.

Heading: "Part 1. Scaling laws, history and background." Two bullets, each with a black diamond (❖) marker and generous vertical spacing:

- ❖ Data scaling as empirical sample complexities
- ❖ Initial forays into understanding neural scaling with data

No chart or table on this page.

## Slide 6 — Sample complexity and rates

Heading: "Sample complexity and rates". Body text: "Theorists have thought about 'scaling' for a long time.."

Two boxed equations, each in its own grey-bordered box, are pasted as images:

Box 1:
$$\epsilon(\hat{h}) \le \left(\min_{h \in H} \epsilon(h)\right) + 2\sqrt{\frac{1}{m}\log\frac{2k}{\delta}}$$
Caption beneath, in body text: "(learning in a finite set of k hypotheses)"

Box 2 (a pasted screenshot of theorem text, with "1.5" rendered as a blue hyperlink in the source):
"Under the assumptions of Theorem 1.5, the rate of convergence of the estimator $\hat{p}_n(x_0)$ is $\psi_n = n^{-\frac{\beta}{2\beta+1}}$, which means that for a finite constant $C$ and for all $n \ge 1$ we have
$$\sup_{p \in \mathcal{P}(\beta,L)} \mathbb{E}_p\left[(\hat{p}_n(x_0) - p(x_0))^2\right] \le C\psi_n^2.$$"
Caption beneath, in body text: "(generative modeling for smooth densities)"

Body text below both boxes: "But these are upper bounds, not actual, realized loss values." Below that, a source citation (confirmed against the text layer): `https://www.cs.cmu.edu/~epxing/Class/10701/slides/lecture16-VC.pdf` and, on the next line, "Hall, 1989".

No chart or table (other than the two boxed-equation images) on this page.

## Slide 7 — Earliest (data) scaling law paper – 1993

Heading: "Earliest (data) scaling law paper – 1993". The body of the slide is a pasted screenshot of the first page of a 1993 paper, plus its Figure 2.

**Paper scan (left).** Title: "Learning Curves: Asymptotic Values and Rate of Convergence". Authors: "Corinna Cortes, L. D. Jackel, Sara A. Solla, Vladimir Vapnik, and John S. Denker", affiliation "AT&T Bell Laboratories, Holmdel NJ 07733". Abstract text (as printed): "Training classifiers on large databases is computationally demanding. It is desirable to develop efficient procedures for a reliable prediction of a classifier's suitability for implementing a given task, so that resources can be assigned to the most promising candidates or freed for exploring other classifier candidates. We propose such a practical and principled predictive method. Practical because it avoids the costly procedure of training poor classifiers on the whole training set, and principled because of its theoretical foundation. The effectiveness of the proposed procedure is demonstrated for both single- and multi-layer networks."

To the right of the title/abstract, body text continues: "A typical example of learning curves is shown in Fig. 2. The test error is always larger than the training error, but asymptotically they reach a common value, $a$. We model the errors for large sizes of the training set as power-law decays to the" followed by two displayed equations:
$$\mathcal{E}_{\text{test}} = a + \frac{b}{l^\alpha} \quad \text{and} \quad \mathcal{E}_{\text{train}} = a - \frac{c}{l^\beta}$$

**Figure (right) — "Figure 2" from the paper, error vs. training-set size.** Y-axis "error", linear scale, labelled ticks at 0, 0.05, 0.1, 0.15, 0.2, 0.25 (topmost data point extends slightly above 0.25, with a visible error bar). X-axis "training set size, $l$", labelled ticks at 2560, 7680, 15360 (evenly spaced tick marks, unlabeled intermediate ticks between them). Two series, distinguished by marker shape and by a legend inset on the chart itself ("— points used for prediction", "---- predicted learning curves"):

- **test error** (open circles): a cluster of points with visible error bars, connected by a solid line ("points used for prediction") for the first several points, falling steeply from about (2560, 0.27) down to about (2560+, 0.09); the line then continues as a dashed "predicted learning curve" for the remaining points, staying roughly flat around 0.08 out to the rightmost plotted point beyond 15360.
- **training error** (solid/filled triangles): starts near (2560, 0.01) and rises with a solid "points used for prediction" line to about 0.06; continues as a dashed "predicted learning curve," staying roughly flat and rising very slightly to about 0.07 by the rightmost point.

The two curves are converging toward each other (per the body text, both approach a common asymptotic value $a$) but have not fully met by the right edge of the plot. This chart is the slide's illustration of the 1993 paper's own power-law fit to learning curves.

## Slide 8 — Early history of scaling laws – data scaling

Heading: "Early history of scaling laws – data scaling". Bottom-of-slide caption: "Log-linear scaling with data [Banko and Brill '01]".

**Figure (left) — "Figure 1. Learning Curves for Confusion Set Disambiguation."** Y-axis "Test Accuracy", linear scale, labelled ticks at 0.70, 0.75, 0.80, 0.85, 0.90, 0.95, 1.00. X-axis "Millions of Words", log scale, labelled ticks at 0.1, 1, 10, 100, 1000. Four series, per the on-chart legend, each a distinct colour and marker:

- **Memory-Based** (dark red/maroon, open hexagon-like markers): the lowest curve throughout. Starts at $x\approx0.2$, $y\approx0.83$; at $x=1$, $y\approx0.86$; at $x=10$, $y\approx0.88$; at $x=100$, $y\approx0.92$; ends at $x=1000$, $y\approx0.94$.
- **Winnow** (blue, "X" markers): starts lowest of all series, at $x\approx0.2$, $y\approx0.75$; rises steeply to catch up by $x=1$ ($y\approx0.83$); at $x=10$, $y\approx0.95$; at $x=100$, $y\approx0.94$; ends at $x=1000$, $y\approx0.97$ — the highest final value of the four series.
- **Perceptron** (green, open triangle markers): starts at $x\approx0.2$, $y\approx0.79$; at $x=1$, $y\approx0.84$; at $x=10$, $y\approx0.91$; at $x=100$, $y\approx0.94$; ends at $x=1000$, $y\approx0.97$.
- **Naïve Bayes** (dark navy, open square markers): starts at $x\approx0.2$, $y\approx0.81$; at $x=1$, $y\approx0.85$; at $x=10$, $y\approx0.92$; at $x=100$, $y\approx0.94$; ends at $x=1000$, $y\approx0.96$.

All four series rise roughly log-linearly with training-data size; Winnow, Perceptron and Naïve Bayes overtake and pull ahead of Memory-Based by the middle of the range and finish well above it, while Memory-Based rises more slowly and steadily throughout and never catches up.

**Text box (right):** "these results suggest that we may want to reconsider the trade-off between spending time and money on algorithm development versus spending it on corpus development. At least for the problem of confusable disambiguation, none of the learners tested is close to asymptoting in performance at the training corpus size commonly employed by the field."

## Slide 9 — Early history of scaling laws – data scaling

Heading: "Early history of scaling laws – data scaling". Body text: "**Early tests of functional forms**" and "Kolachina et al 2012 – power law relation between data and downstream performance".

**Figure (left) — "Curve Fit using Recursive least squares" (pasted screenshot from Kolachina et al. 2012).** Y-axis "BLEU scores", linear scale, labelled ticks every 0.01 from 0.14 to 0.22. X-axis "Training sample size", linear scale, labelled ticks every 200000 from 0 to 1200000+ (data extends slightly past 1200000, toward roughly 1,350,000). A legend box (bottom right of the chart) lists eight series by line style: "×  Observed values" (X markers), "— Observed curve" (solid black line through the observed points), and six fitted curve families each with a distinct dotted/dashed style — "Exp3", "Exp4", "ExpP3", "Pow3", "Pow4", "ILog2" (these correspond to the six formulas in the table on the right of the slide). All series rise steeply together from about (0, 0.145) to about (100000, 0.196), continue rising more gently to about (500000, 0.206), and then diverge slightly in the flat region beyond 500000: most of the fitted curves and the observed curve track closely together around 0.207-0.209 out to the right edge, one curve (dotted, per the legend "Exp3") visibly plateaus lowest, flattening at about 0.203 from roughly 400000 onward, and one curve (dash-dot style, per the legend "ExpP3") visibly pulls away above the rest from about 600000 onward, reaching about 0.215 by the right edge. The six fitted-curve line styles are close enough in this grayscale rendering that individual mid-range values for Exp4, Pow3, Pow4 and ILog2 cannot be reliably distinguished from each other beyond noting they all sit in the tightly-clustered middle band with the observed curve.

**Table (right) — "Table 1: Curve families."**

| Model | Formula |
| --- | --- |
| Exp$_3$ | $y = c - e^{-ax+b}$ |
| Exp$_4$ | $y = c - e^{-ax^\alpha+b}$ |
| ExpP$_3$ | $y = c - e^{(x-b)^\alpha}$ |
| Pow$_3$ | $y = c - ax^{-\alpha}$ |
| Pow$_4$ | $y = c - (-ax+b)^{-\alpha}$ |
| ILog$_2$ | $y = c - (a/\log x)$ |

## Slide 10 — Hestness et al 2017

Heading: "Hestness et al 2017". Body text at the bottom: "**Earliest 'large scale neural' scaling work:** Hestness 2017" and "Predictable scaling on many tasks (MT, LM, Speech) and hypothesized scaling shape."

**Figure 1 (left) — neural machine translation learning curves, by hidden size.** Y-axis "Minimum Test Loss (Log-scale)", labelled ticks 0.41, 0.44, 0.48, 0.51, 0.54, 0.58, 0.62, 0.67. X-axis "Training Data Set Size, Number of Tokens (Log-scale)", ticks $2^{20}$ through $2^{27}$. Four series per the legend: **208 Hidden** (blue solid), **512 Hidden** (orange solid), **208 Hidden Trend** (green dashed), **512 Hidden Trend** (red dashed) — the dashed trend line for each hidden size overlays its solid counterpart almost exactly. Both series start close together at $2^{20}$ (208 Hidden $\approx0.685$, 512 Hidden $\approx0.665$) and descend together; they begin to separate past about $2^{23}$, with 512 Hidden falling faster and finishing lower: at $2^{27}$, 208 Hidden $\approx0.44$ and 512 Hidden $\approx0.395$. Two annotated fit equations sit on the plot: $\varepsilon_{208}(m) = 41.2\,m^{-0.36} + 0.39$ and $\varepsilon_{512}(m) = 21.5\,m^{-0.30} + 0.32$.

**Figure 1 continued (middle) — token error rate learning curve.** Same y-axis style, "Minimum Test Loss (Log-scale)", ticks 0.36 through 0.71; x-axis "Training Data Set Size, Number of Tokens (Log-scale)", ticks $2^{19}$ through $2^{27}$ (partially cropped in the pasted image, consistent with the left panel's range). Two series per the legend: **Token Error Rate** (blue solid) and **Token Error Rate Trend** (orange dashed), nearly coincident. The curve descends from about ($2^{19}$, 0.71) to about ($2^{27}$, 0.30), with the dashed trend line diverging slightly below the solid curve at the largest data sizes. Annotated fit: $\varepsilon(m) = 3.87\,m^{-0.13}$.

Caption beneath these two panels, as printed: "Figure 1: Neural machine translation learning curves. Left: the learning curves for separate models follow $\varepsilon(m) = \alpha m^{\beta_g} + \gamma$. Right: composite learning curve of best-fit model at each data set size."

**Figure 3 (right) — schematic "Small Data / Power-law / Irreducible Error" regions diagram.** Axes are unlabeled with numeric ticks (schematic only): y-axis "Generalization Error (Log-scale)", x-axis "Training Data Set Size (Log-scale)". Two vertical grey divider lines split the plot into three labelled regions, left to right: "Small Data Region", "Power-law Region", "Irreducible Error Region". A horizontal green dashed line, labelled "Best Guess Error", runs across the top of the plot. A horizontal red dashed line, labelled "Irreducible Error", runs across the bottom. A single dark-navy curve starts flat against the green "Best Guess Error" line through the Small Data Region, bends downward and descends roughly linearly (on these log-log-styled axes) through the Power-law Region, then flattens out just above the red "Irreducible Error" line through the Irreducible Error Region, with a very slight uptick right at the end.

## Slide 11 — Hestness II

Heading: "Hestness II". Body text: "Very ahead of its time..". Three labelled rows, each pairing a short label on the left with a boxed block of quoted paper text on the right:

- **"Emergence"** — boxed text: "Although small data set testing may be possible, it can be difficult to ensure that training data is large enough to see the power-law learning curve region. We have found that models with poor optimizer parameterization or model priors/initialization show accuracy cliffs, where accuracy is only as good as best guessing, but the model trains on enough data to be in the power-law region. Researchers must take great care when defining a "large enough" training set for small data testing. We leave the methodology for defining such a training set to future work."
- **Scaling by compute** — boxed text: "**Computational Limits:** If we have identified a desirable model to scale to larger training sets, the next potential limitation is the speed of computation. In some cases, training large models on very large data sets would take months or years of critical path compute time, making these training runs impractical for any real world problem on existing systems. However, predictable learning and model size curves may offer a way to project the compute requirements to reach a particular accuracy level. The compute requirements could inform decisions about how to scale computational capacity to unlock these compute-limited applications."
- **Speed = accuracy** — boxed text: "**The Performance-Accuracy Trade-off:** Many DL software and hardware techniques impose a trade-off between model accuracy and the speed of computation. Learning curves and model size growth can indicate whether these techniques could regain lost accuracy by improving the speed of computation. For example, low-precision computation/quantization and sparse models give up some model accuracy (e.g., up to 20%) in order to improve compute throughput. If the compute throughput improvements allow DL developers to train larger models on larger data sets, these accuracy losses might be easily recoverable."

No chart or table on this page (the three boxes are pasted text images, not figures).

## Slide 12 — Part 2. Neural (LLM) scaling behaviors

Heading: "Part 2. Neural (LLM) scaling behaviors". Three numbered items, each with a bold/dark title line and a grey quoted or descriptive line beneath it, evenly spaced down the centre of the page:

1. **Data vs performance** — "Are there simple rules that determine how data affects performance?"
2. **Data vs model size** — Do we train on more data or bigger models?
3. **Hyper-parameters vs performance** — "How should we set hyperparameters on the big model??"

No chart or table on this page.

## Slide 13 — Scaling laws – power law relationships for many factors

Heading: "Scaling laws – power law relationships for many factors". Body text: "These scaling laws hold on *many* different kind of phenomena!" Lower-left body text: "They even hold in non standard settings". Bottom-right citation: "[Kaplan+ 2020]".

**Figure 1 (top row, three panels sharing a "Test Loss" y-axis, from Kaplan et al. 2020) — Test Loss vs. Compute, Dataset Size, and Parameters.**

*Panel 1 — Compute.* Y-axis "Test Loss", linear scale, ticks 2 through 7. X-axis "Compute" (subtitle "PF-days, non-embedding"), log scale, ticks $10^{-9}$ through $10^{1}$. Body: a dense fan of light-blue individual training-run curves (same style as slide 4's left panel), a black lower-envelope line tracing the minimum loss achieved at each compute budget, and a dashed gold/orange fit line labelled "$L = (C_{min}/2.3\cdot10^{8})^{-0.050}$" that tracks the black envelope closely across the full range, from about ($10^{-9}$, 6.5) down to about ($10^{1}$, 2.4).

*Panel 2 — Dataset Size.* X-axis "Dataset Size" (subtitle "tokens"), log scale, ticks $10^7$ (implied), $10^8$, $10^9$. A single blue line with circular markers descends almost exactly along a grey fit line labelled "$L = (D/5.4\cdot10^{13})^{-0.095}$", from about ($2\times10^7$, 4.15) down to about ($1.4\times10^9$, 2.73) across 7 marked points.

*Panel 3 — Parameters.* X-axis "Parameters" (subtitle "non-embedding"), log scale, ticks $10^5$, $10^7$, $10^9$. Y-axis ticks (shared column) 2.4, 3.2, 4.0, 4.8, 5.6. A single blue line with circular markers, closely tracking a grey fit line labelled "$L = (N/8.8\cdot10^{13})^{-0.076}$", descends from about ($4\times10^4$, 5.75) down to about ($1.3\times10^9$, 2.9).

**Figure 2 (bottom left) — "Word Unscramble" sigmoid fit.** Y-axis "Normalized Exact Match", linear scale, 0.0 to just above 0.8. X-axis "Log$_{10}$(Llama-2-Equiv. FLOPs (1E21))", linear scale, ticks -1 to 4. Annotated on the chart: fit equation "$y = \text{sigmoid}(2.00x - 6.11)$", "$MSE_{train} = 1.3\text{e-}04$", "$MSE_{test} = 4.0\text{e-}03$". A blue S-shaped sigmoid curve rises from near 0 (flat for $x<0$) through a steep rise around $x=1.5$ to $2.5$, reaching about 0.82 by $x=4$. Scattered points in blue, red and green (various marker shapes — stars, X's, circles, triangles, squares, crosses, pentagons; no legend on this panel identifies what the colours/shapes denote) sit close to the fitted curve throughout, densest in blue at low x (near-zero score) and shifting to red/green at higher x as scores rise toward the curve's plateau.

**Figure 3 (bottom middle) — "Persian QA" sigmoid fit.** Y-axis "Normalized Accuracy", linear scale, 0.0 to just above 0.6. X-axis, same as Word Unscramble: "Log$_{10}$(Llama-2-Equiv. FLOPs (1E21))", ticks 1 to 4. Annotated: "$y = \text{sigmoid}(2.32x - 8.43)$", "$MSE_{train} = 1.8\text{e-}04$", "$MSE_{test} = 3.2\text{e-}03$". Same blue/red/green scattered-marker style as Word Unscramble; the fitted sigmoid rises from near 0 at low x through a steep rise around $x=2.5$ to $3.5$, reaching about 0.64 by $x=4$.

**Figure 4 (bottom right) — "Epoch Capabilities Index (ECI)" scatter, sourced from Epoch AI.** Title "Epoch Capabilities Index (ECI)" with the Epoch AI logo; annotation "136 Results". Y-axis "Score", ticks at least 90, 105, 120, 135, 150 visible. X-axis "Release date", ticks Apr. 2023, Oct. 2023, Apr. 2024, Oct. 2024, Apr. 2025, Oct. 2025. A legend "Organization" lists six colour categories: OpenAI (pink/magenta), Google (teal), Anthropic (purple), Meta AI (orange), xAI (blue), Other (brown). The scatter of roughly 136 points trends upward over time (higher ECI score for more recently released models). Selected points carry text labels with leader lines, in roughly chronological/score order: LLaMA-65B ($\approx104$, earliest), GPT-4 (Mar 2023) ($\approx122$), GPT-4 (Jun 2023) ($\approx119$), Claude 3 Opus, Llama 3-405B, Claude 3.5 Sonnet (Jun 2024), Claude 3.5 Sonnet (Oct 2024), DeepSeek-R1, o1-mini (high), o1 (high), Gemini 2.5 Pro Exp (Mar 2025), o3 (high), Grok 4, and GPT-5 (medium) ($\approx150$, the highest-scoring and most recent labelled point). Small "CC-BY" and "epoch.ai" watermark text sit at the bottom corners.

## Slide 14 — Data vs performance

Heading: "Data vs performance". Body text: "What's a data scaling law?" followed by, indented: "**Data scaling laws** : simple formula that maps dataset size (n) to error". Below that: "What do we expect out of scaling laws?" Left-side label: "Monotonic, logistic-like curves". Bottom-right citation: "[Hestness+ 2017]".

**Figure — the same "Small Data / Power-law / Irreducible Error" schematic diagram as slide 10's right-hand panel**, reused at a larger size. Y-axis "Generalization Error (Log-scale)" (no numeric ticks), x-axis "Training Data Set Size (Log-scale)" (no numeric ticks). Two grey vertical dividers separate "Small Data Region," "Power-law Region," and "Irreducible Error Region." A green dashed horizontal line labelled "Best Guess Error" runs across the top; a red dashed horizontal line labelled "Irreducible Error" runs across the bottom. The dark-navy curve is flat against the green line through the Small Data Region, descends through the Power-law Region, and flattens just above the red line (with a slight uptick at the very end) through the Irreducible Error Region — identical in shape to slide 10's version.

## Slide 15 — Data scaling laws for language models

Heading: "Data scaling laws for language models". Body text: "First, an empirical observation" and, as a sub-heading: "**Loss and dataset size is linear on a log-log plot**". Right-side text: "'Scale-free' or 'Power law'". Bottom caption: "(For language modeling, from Kaplan+ 2020)".

**Figure — Test Loss vs. Dataset Size, enlarged version of the middle panel from slide 13's Figure 1.** Y-axis "Test Loss", linear scale, labelled ticks 2.7, 3.0, 3.3, 3.6, 3.9, 4.2. X-axis "Dataset Size" (subtitle "tokens"), log scale, labelled ticks $10^8$, $10^9$. One blue line with 7 circular markers, closely tracking a grey fit line labelled "$L = (D/5.4\cdot10^{13})^{-0.095}$" (boxed legend entry). Values at the seven marked points, left to right: ($\approx2\times10^7$, 4.15), ($\approx5\times10^7$, 3.83), ($\approx9\times10^7$, 3.56), ($\approx1.8\times10^8$, 3.32), ($\approx3.5\times10^8$, 3.15), ($\approx7\times10^8$, 2.93), ($\approx1.4\times10^9$, 2.73) — a near-perfectly straight descending line on these log-x, linear-y axes, illustrating the slide's point that loss falls linearly in log(dataset size).
## Slide 16 — Conceptual foundations of data scaling laws.

Title (blue): "**Conceptual foundations of data scaling laws.**"

A pale-blue bordered box: "**Q:** Why do scaling laws show up?" Below it, indented: "We know error should be monotone" and "But why is it a power law / linear in log-log?" A dark-blue arrow points right, from this text toward a schematic chart.

Below that, a second pale-blue box: "**A (?):** Estimation error naturally decays polynomially."

Body text below both boxes: "But this answer may take a moment to understand. Let's work through an example." and "**Example:** If our task is to estimate the mean of a dataset, what's the scaling law?"

**Figure — schematic (illustrative, not measured data) line chart of the three-regime shape of generalization error.** Axes carry no numeric ticks. Y-axis label: "Generalization Error (Log-scale)". X-axis label: "Training Data Set Size (Log-scale)". Two vertical grey solid lines divide the plot into three labelled regions, left to right: "Small Data Region", "Power-law Region", "Irreducible Error Region". Two horizontal reference lines run the full width of the plot: a green dashed line near the top, labelled "Best Guess Error", and a red dashed line near the bottom, labelled "Irreducible Error". One data series — a solid navy-blue curve, no legend entry — runs left to right: it starts flat, hugging just under the green "Best Guess Error" line through the "Small Data Region"; through the "Power-law Region" it declines in a smooth, roughly straight diagonal (consistent with a power law on log-log axes); entering the "Irreducible Error Region" it curves over and flattens, with a small dip-and-recover wiggle just past the second divider before settling onto the red "Irreducible Error" line for the rest of the plot.

The chart illustrates, conceptually, the three-part shape (flat, power-law decline, flat again) that motivates the rest of the slide's question — why the middle region is a power law rather than some other functional form.

## Slide 17 — Toy example: mean estimation

Title (blue): "**Toy example: mean estimation**"

A pale-blue bordered box:
"**Input**: $x_1 \ldots x_n \sim N(\mu, \sigma^2)$"
"**Task**: estimate the average as $\hat\mu = \dfrac{\sum_i x_i}{n}$"

Body text: "**What's the error?** By standard arguments.."

Displayed equation:
$$E[(\hat\mu-\mu)^2] = \frac{\sigma^2}{n}$$

Body text: "**This is a 'scaling law'**"

Displayed equation:
$$\log(Error) = -\log n + 2\log \sigma$$

Body text: "More generally, any polynomial rate $1/n^\alpha$ is a scaling law"

No chart or table on this page.

## Slide 18 — Scaling law exponents: an intriguing mystery

Title (blue): "**Scaling law exponents: an intriguing mystery**"

Body text: "**Fact**: Similar arguments show most 'classical' models (regression, etc) have $\frac{1}{n}$ scaling"

"This means we should see $y = -x + C$"

"What do we find in neural scaling laws?"

Three log-log line charts sit side by side, each with its own caption underneath ("Machine translation", "Speech", "Language modeling").

**Chart 1 (left) — "Machine translation."** Two series: a solid blue line, "Token Error Rate", and a dashed orange line, "Token Error Rate Trend" (legend, top-right of the chart). Y-axis: "Minimum Test Loss (Log-scale)", log scale, labelled ticks (unevenly spaced, low to high): 0.36, 0.39, 0.42, 0.46, 0.50, 0.55, 0.60, 0.65, 0.71. X-axis: "Training Data Set Size, Number of Tokens (Log-scale)", log scale, labelled ticks $2^{19}$ through $2^{27}$ in steps of one power. Both series start at the same point, $(2^{19}, \approx0.71)$, and are visually indistinguishable — the dashed trend line overlays the solid measured line almost exactly — through about $2^{24}$ ($\approx0.46$). Past that point they diverge: the solid blue "Token Error Rate" line bends and flattens, ending at $(2^{27}, \approx0.393)$, while the dashed orange trend line continues on its original steeper slope, ending lower, at $(2^{27}, \approx0.36)$. A label midway up the chart gives the fitted trend: "$\varepsilon(m) = 3.87\, m^{-0.13}$".

**Chart 2 (middle) — "Speech."** Four series (legend, top-right of the chart): solid blue "DS2", solid orange "Attention", dashed green "DS2 Trend", dashed red "Attention Trend". Y-axis: "Minimum Validation Loss (Log-scale)", log scale, labelled ticks 0.11, 0.14, 0.18, 0.23, 0.29, 0.37, 0.48, 0.61, 0.78. X-axis: "Training Data Set Size, Hours of Audio (Log-scale)", log scale, labelled ticks 8, 16, 32, 64, 128, 256, 512, 1024, 2048 (powers of two), with the plotted series ending just short of the 2048 tick (around x≈1750).
- **DS2** (solid blue): runs from $(8, \approx0.505)$ down to $(\approx1750, \approx0.125)$.
- **DS2 Trend** (dashed green): begins at the same left-hand point, $(8, \approx0.505)$, and overlaps the solid blue line almost exactly through the middle of the range; past roughly $x\approx300$ it visibly separates below the solid line, ending lower, at about $(\approx1750, \approx0.105)$.
- **Attention** (solid orange): runs from $(8, \approx0.775)$ down to $(\approx1750, \approx0.16)$.
- **Attention Trend** (dashed red): begins slightly to the right of the Attention series' own start — at roughly $x\approx9$–$10$, already visibly below the orange line's starting value ($\approx0.68$ vs. $\approx0.775$) — then runs a few points below/parallel to the solid orange line through the rest of the range, ending at about $(\approx1750, \approx0.145)$.
Two fitted-trend labels are printed on the chart: "$\varepsilon(m) = 1.36\, m^{-0.30}$" (upper, by the orange/red Attention pair) and "$\varepsilon(m) = 0.95\, m^{-0.30}$" (lower, by the blue/green DS2 pair).

**Chart 3 (right) — "Language modeling."** One series: a solid blue line connecting seven circular data markers, plotted against a solid grey fitted-trend line (legend, top of chart: "$L = (D/5.4\cdot10^{13})^{-0.095}$"). Y-axis: no axis-title text printed, log-scale numeric ticks 2.7, 3.0, 3.3, 3.6, 3.9, 4.2. X-axis: "**Dataset Size**" in bold, with "tokens" in smaller grey type beneath it as a subtitle; log scale, labelled ticks $10^8$ and $10^9$. The seven data points run from about $(3\times10^7, \approx4.15)$ down to $(\approx1.5\times10^9, \approx2.72)$; the blue measured line and the grey fitted line lie almost exactly on top of each other across the whole range, with only the leftmost point sitting marginally above the grey trend line.

Below the three charts: "Very different from predictions.. Why might this be?" — the point being that language modeling's own trend line ($L=(D/5.4\cdot10^{13})^{-0.095}$) continues to fit the measured loss even at the largest data sizes, while the machine-translation and speech curves both visibly bend away from (flatten relative to) their own fitted trends at large data sizes.

## Slide 19 — Detour: scaling laws for (nonparametric) learning

Title (blue): "**Detour: scaling laws for (nonparametric) learning**"

Body text: "Neural nets can approximate arbitrary functions. Lets turn that into an example." (printed as "Lets," without an apostrophe — transcribed as printed.)

A pale-blue bordered box:
"**Input**: $x_1 \ldots x_n$ uniform in 2D unit box. $y_i = f(x_i) + N(0,1)$"
"**Task**: estimate f(x)"
"**Approach**: cut up the 2D space into boxes with length $n^{-\frac{1}{4}}$"

Body text: "**What's our estimation error?**"

"Informally, we have $\sqrt{n}$ boxes, each box gets $\sqrt{n}$ samples."

Displayed equation:
$$Error \approx \frac{1}{\sqrt{n}} + (other\ smoothness\ terms)$$

Body text: "In $d$-dimensions, this becomes $Error = n^{-1/d}$ - **This means scaling is** $y = -\dfrac{1}{d}x + C$"

"**Takeaway:** flexible 'nonparametric' learning has dimension dependent scaling laws."

No chart or table on this page.

## Slide 20 — Intrinsic dimensionality theory of data scaling laws

Title (blue): "**Intrinsic dimensionality theory of data scaling laws**"

A pale-blue bordered box:
"**Some have made the following argument (Bahri 2021)**"
1. Scaling laws arise due to polynomial rates of learning $\frac{1}{n^\alpha}$
2. Some argue the slope $\alpha$ is closely connected to the *intrinsic dimensionality* of the data.

**Figure — scatter plot, "$4/\alpha_D$" vs. "Dimension."** Y-axis: "$4/\alpha_D$", linear scale, labelled ticks 0, 5, 10, 15, 20, 25. X-axis: "Dimension", linear scale, labelled ticks 2, 4, 6, ..., 26 (steps of 2). Two dashed reference lines (legend, top-left): a black dashed line labelled "$4/\alpha_D$" running from about $(2,2)$ to $(26,25)$ — slope $\approx1$ — and a grey dashed line labelled "$2/\alpha_D$" running from about $(2,1)$ to $(26,12.5)$ — slope $\approx0.5$, i.e. about half as steep.

A separate legend beneath the chart gives six dataset categories, each with its own colour: Teacher-Student (bright magenta), CIFAR-10 (violet-magenta), CIFAR-100 (medium purple), SVHN (periwinkle/blue-violet), FashionMNIST (sky blue), MNIST (cyan/turquoise). Point identities below were confirmed by sampling the plotted marker colours against these legend swatches directly (colour-matched pixel by pixel), not by eye alone, since several of the six colours are close in hue.

- **Teacher-Student** (bright magenta, filled circles only): eight points, running from dimension 2 to 9, at approximate $4/\alpha_D$ values 3.3, 4.3, 5.4, 5.8, 6.6, 7.1, 8.3, 8.8 — i.e. one point per integer dimension, sitting almost exactly on the black "$4/\alpha_D$" reference line for the first few points and drifting slightly below it by dimension 9.
- **MNIST** (cyan/turquoise, filled only): one point, at approximately (9, 10.1).
- **FashionMNIST** (sky blue): a filled point at approximately (10.2, 19.3), sitting above the black reference line, and a separate open (hollow-ring) point at approximately (10.7, 15.6), sitting closer to the black line.
- **SVHN** (periwinkle/blue-violet): a filled point at approximately (15.1, 16.5), and an open point at approximately (10.3, 10.9), both below the black reference line.
- **CIFAR-10** (violet-magenta): a filled point at approximately (15.0, 20.2), sitting above the black reference line, and an open point at approximately (13.0, 6.9), sitting well below both reference lines.
- **CIFAR-100** (medium purple): a filled point at approximately (25.0, 24.4), landing almost exactly on the black reference line's right-hand end, and an open point at approximately (21.5, 10.0), sitting close to the grey "$2/\alpha_D$" line.

Every dataset except Teacher-Student and MNIST contributes exactly one filled and one open point (two apparent dimension estimates per dataset); Teacher-Student contributes eight filled points swept across dimensions 2–9, and MNIST contributes a single filled point. No dataset's points are connected by a line; only the two dashed references are lines.

Below the figure: "But estimators of intrinsic dimension are sketchy, and this is not airtight.."

## Slide 21 — Other data scaling laws

Title (blue): "**Other data scaling laws**"

Body text: "**Data scaling thus far**: how does dataset size relate to performance?" and "**Related question**: how does dataset *composition* affect performance"

A pale-blue bordered box with three bullets (❖), each with generous vertical spacing:
- ❖ Picking optimal data mixture using small scale models
- ❖ Deciding whether to repeat data or not
- ❖ Combing the two and balancing quality with repetition rate

(The deck's own text reads "Combing," not "Combining" — transcribed as printed; likely a typo in the source.)

No chart or table on this page.

## Slide 22 — Other advanced data scaling law: distribution shift

Title (blue): "**Other advanced data scaling law: distribution shift**"

Body text: "**Data scaling thus far**: how does dataset size relate to performance?" and "**Related question**: how does dataset *composition* affect performance"

"**A:** Data composition affects the offset, not the slope."

"These 'distribution shift' scaling laws can tell us about the importance of collecting diverse data!"

**Figure 1 (left) — "Excess error" vs. "Training data size," log-log line chart.** Y-axis: "Excess error", log scale, labelled ticks $10^0$ and $10^{-2}$ (the plotted lines continue below the $10^{-2}$ tick, unlabelled, to roughly the low-$10^{-3}$s). X-axis: "Training data size", log scale, labelled ticks $10^2$ and $10^3$ (the lines continue right of $10^3$, unlabelled). Three series, distinguished by a legend headed "q": blue "0.00", orange "0.22", green "0.56".
- **q=0.00** (blue): the topmost line throughout, from about $(10^2, 0.9)$ down to about $(5\times10^3, 0.02)$. Partway along (from roughly $x=2000$ onward) this line passes behind the legend box and renders visibly paler/whiter where it crosses it — the same single series continuing underneath, not a fourth series.
- **q=0.22** (orange): from about $(10^2, 0.06)$ down to about $(5\times10^3, 0.004)$, below the blue line throughout.
- **q=0.56** (green): from about $(10^2, 0.04)$ down to about $(5\times10^3, 0.003)$, the lowest line throughout, tracking just under the orange line for most of its length.
All three lines show a small, shared downward notch at roughly the same two x-positions (around the high-hundreds and again in the low thousands) — consistent across all three series, so likely a shared artifact in the underlying measurement rather than a per-series feature.

**Figure 2 (right) — "Expected error intercept," titled line chart.** Y-axis: "Log C(q)", linear scale, labelled ticks 2 and 4. X-axis: "Data source proportion", linear scale, labelled ticks 0.0, 0.2, 0.4, 0.6, 0.8, 1.0. One series, a solid blue line, no legend (single line, no markers): starts at about $(0.0, 4.7)$, drops steeply to about $(0.1, 2.3)$, continues down more gently to a shallow minimum of about 1.5 spanning roughly $x=0.35$–$0.55$, then rises again, gently at first and then steeply, back up to about $(0.9, 2.3)$ and $(1.0, 4.6)$ — a roughly symmetric U-shape with a flat bottom and steep shoulders at both ends.

Citation below the figures: "[Hashimoto 2021]"

Together the two figures make the slide's point: the per-source excess-error curves (left) are roughly parallel power laws (composition changes the vertical offset, not the slope), and the offset itself (right) is minimized by a roughly balanced mixture and rises sharply toward either pure-source extreme (q near 0 or 1).

## Slide 23 — In practice: data mixture selection via scaling is hard

Title (blue): "**In practice: data mixture selection via scaling is hard**"

Two pasted paper screenshots sit side by side, each followed by its own caption below.

**Figure 1 (left) — screenshot of the paper "Data Mixing Laws: Optimizing Data Mixtures by Predicting Language Modeling Performance."** Byline: "Jiasheng Ye$^{1,*}$ Peiju Liu$^{1,*}$ Tianxiang Sun$^1$ Jun Zhan$^1$ Yunhua Zhou$^{2,\dagger}$ Xipeng Qiu$^{1,\dagger}$", with contact emails "{jsye23,pjliu23}@m.fudan.edu.cn zhouyunhua@pjlab.org.cn xpqiu@fudan.edu.cn" and affiliations "$^1$Fudan University $^2$Shanghai AI Laboratory". Below the byline, a pasted figure from the paper (its own "Figure 1") in two parts:
- Left part: a vertical flow of four labelled stages connected by circled step numbers ①②③: "Small Steps, Small Models, Seen Mixture" → (①) → "Large Steps, Small Models, Seen Mixture" → (②) → "Large Steps, Large Models, Seen Mixture" → (③, in red) → "Large Steps, Large Models, Unseen Mixture" (in red). A boxed legend beneath reads "① Training Step Laws; ② Model Size Laws; ③ Data Mixing Laws (ours)".
- Right part: two connected 3D surface/line plots. The first, axes "Losses" (vertical), "Training Steps" and "Model Sizes" (horizontal), shows a set of curved lines rising from "Observed Samples" through points labelled "① Training Step Laws" and "② Model Size Laws". A dashed arrow leads to the second 3D plot, axes "Losses" (vertical), "Proportion (Domain 2)" and "Proportion (Domain 1)" (horizontal), showing a bowl-shaped surface labelled "③ Data Mixing Laws" with an "×N mixtures" annotation and a point marked "Minimum Loss" (in red) at the bottom of the bowl, plus two black "×" marks elsewhere on the surface.

The paper's own figure caption is reproduced beneath the image: "Figure 1: Illustration on our pipeline to optimize data mixture. **Left:** Our pipeline takes three steps. Starting from small-scale training results, the three steps use the scaling laws of training steps, model sizes, and data mixing laws to predict model performance on large steps, large models, and unseen mixtures, respectively. **Right:** Visualization of the three-step pipeline to predict model performance on the target model size, training step, and mixtures."

Caption beneath this whole figure, on the slide itself: "**Natural idea** – build data scaling laws"

**Figure 2 (right) — screenshot of the paper "DataDecide: How to Predict Best Pretraining Data with Small Experiments."** Title in pink/magenta with a small icon before it ("❖DataDecide"), subtitled "How to Predict Best Pretraining Data with Small Experiments". Byline: "Ian Magnusson$^{*12}$ Nguyen Tai$^{*3}$ Ben Bogin$^{*1}$ David Heineman$^1$ Jena Hwang$^1$ Luca Soldaini$^1$ Akshita Bhagia$^1$ Jiacheng Liu$^{12}$ Dirk Groeneveld$^1$ Oyvind Tafjord$^1$ Noah A. Smith$^{12}$ Pang Wei Koh$^{12}$ Jesse Dodge$^1$". Below the byline, a pasted figure from the paper, on a pale peach/cream background, laid out in three parts:
- Left: two icon-labelled boxes stacked vertically, "Targets" (a balance-scale icon) above three sub-icons "Evaluation" (a dial/gauge), "Seeds" (a die), "Large Scale" (a small network diagram) — each with its own small icon — and, beside "Targets", a box "Best Data:" listing "1. Dolma", "2. DCLM", "3. …" with small star icons.
- Middle: "Predictions" (a crystal-ball icon) above two sub-icons "(Proxy) Evaluation" (a dial) and "Smaller Scale(s)" (a small grid icon), with a matching "Best Data:" box listing "1. DCLM", "2. Dolma", "3. …". A large pink "?" and "=" sit between the Targets and Predictions columns, with a callout box reading "Pretrain 25 datasets @ 150M to predict pairs of 25 datasets @ 1B ~80% correct".
- Right: a scatter/line chart, "Decision Accuracy" (y-axis, 0.3 to 0.9) vs. "Proportion of Target Compute (%C)" (x-axis, log scale, $10^{-5}$ to $10^0$), with a secondary top x-axis "Compute (FLOPs)" labelled $7e{+}15$ through $7e{+}20$. Many overlapping coloured trend lines with shaded confidence bands run left to right, colour-coded by a "Prediction Method" legend on the right listing model sizes: 4M, 6M, 8M, 10M, 14M, 16M, 20M, 60M, 90M, 150M, 300M, 530M, 750M, 1B, and "Multi-Scale Fit" (marked with a green star). The bands generally trend upward (higher decision accuracy at higher compute proportion), from around 0.4–0.6 on the left to around 0.85–0.95 on the right.

Caption beneath this whole figure, on the slide itself: "**Empirical eval** – just take the best small dataset"

Together the two pasted papers illustrate two different practical answers to the same problem (predicting the best data mixture or dataset from small-scale experiments): building an explicit scaling law over mixtures (left paper) versus running many small-scale trials and just picking the empirically best option, without fitting a mixture-level scaling law (right paper).

## Slide 24 — Scaling laws under data repetition

Title (blue): "**Scaling laws under data repetition**"

Body text: "In practice, we have finite data – how does repeating examples affect scaling?"

**Chart 1 (left) — "Return on compute when repeating."** Y-axis: "Final test loss", linear scale, labelled ticks 2.0, 2.2, 2.4, 2.6, 2.8, 3.0, 3.2, 3.4. X-axis: "Tokens (Epochs)", log-spaced labelled ticks pairing a token count with an epoch count: 12B (1), 48B (4), 120B (10), 480B (40), 1.2T (100). Three series (legend beneath the chart, shared with chart 2 below): black-outlined orange-filled circles/stars, "Models trained" (the actual data points); an orange dotted line, "Loss assuming repeated data is worth the same as new data"; a thick orange solid line, "Loss predicted by our data-constrained scaling laws". A magnified inset box near the top of the chart zooms in on six of the trained-model points clustered between roughly 30B and 90B tokens, showing the solid and dotted lines beginning to separate there.
- **Models trained** (points): about eight points from $(12B, \approx2.85)$ down through the 30–90B range (detailed in the inset, roughly 2.69 down to 2.52) to a rightmost plotted point a little past 48B.
- **Loss assuming repeated data = new data** (dotted): tracks the solid line and the points closely from the left edge through about 48B (4 epochs), then continues declining on essentially its original trajectory, diverging below the solid line, reaching about 2.17 by the right edge of the plot (past the 1.2T/100-epoch tick).
- **Loss predicted by our data-constrained scaling laws** (thick solid): starts at about $(\text{just left of }12B, \approx3.32)$, matches the points and the dotted line through about 48B, then bends and flattens, reaching about 2.42 by 480B (40 epochs) and staying nearly flat out to and past 1.2T (100 epochs) — the line visibly fades/lightens toward the right edge.

Three annotations sit inside the plot: a red dashed vertical line at the 48B/4-epoch tick with red text "Up to ≈4 epochs repeating is almost as good as new data"; orange text (no line of its own) "Rapidly diminishing returns for more repetitions" positioned between the two vertical dashed lines; and a light-orange dashed vertical line at the 480B/40-epoch tick with matching light-orange text "At ≈40 epochs, repeating is worthless".

**Chart 2 (right) — "Allocating compute when repeating."** Y-axis: "Parameters", with two labelled values, 8.67B (upper) and 6.34B (lower). X-axis: "Tokens (Epochs)", with two labelled value-pairs, 178B (7.1) and 242B (9.7). Legend (beneath the chart): blue dashed, "Regime of same compute (IsoFLOP)"; solid black, "Efficient frontier assuming repeated data is worth the same as new data"; a black-to-red-to-orange gradient solid line, "Efficient frontier predicted by our data-constrained scaling laws". The blue dashed line, annotated "$10^{22}$ FLOPs", runs from upper-left to lower-right across the whole plot. The black solid line and the gradient line both rise from the origin (bottom-left corner) toward the upper-right, the black line at a steeper angle than the gradient line. A gold/yellow star, labelled "Loss: 2.376", sits where the black line crosses the blue dashed line, at the 178B(7.1) tokens position and a parameters value near (but at this resolution not exactly on) the 8.67B gridline. A red star, labelled "Loss: 2.359", sits where the gradient line crosses the blue dashed line, at the 242B(9.7) tokens position and a parameters value near the 6.34B gridline. The point of the chart: at the same fixed compute budget ($10^{22}$ FLOPs), the data-constrained scaling law's efficient frontier (gradient line, ending at loss 2.359) reaches a lower loss than the naive frontier that treats repeated data as equally valuable as new data (black line, loss 2.376), by choosing a smaller model trained on more token-epochs.

Bottom-left, a pasted citation block for the source paper: "**Scaling Data-Constrained Language Models**", authors "Niklas Muennighoff$^1$ Alexander M. Rush$^1$ Boaz Barak$^2$ Teven Le Scao$^1$" (first row) and "Aleksandra Piktus$^1$ Nouamane Tazi$^1$ Sampo Pyysalo$^3$ Thomas Wolf$^1$ Colin Raffel$^1$" (second row), affiliations "$^1$Hugging Face $^2$Harvard University $^3$University of Turku", and a contact email in blue monospace, "n.muennighoff@gmail.com".

Bottom-centre, a displayed equation from that paper:
$$D' = U_D + U_D R_D^*\left(1-e^{-R_D/R_D^*}\right).$$

Bottom-right, plain-text definitions for the equation's symbols (printed without matching the equation's exact subscript styling):
"D' = Effective data"
"Ud = Unique tokens"
"Rd* = Constant"
"Rd = Repetition"

## Slide 25 — Scaling laws in *compute unbounded* settings

Title (blue, with "compute unbounded" in italic): "**Scaling laws in *compute unbounded* settings**"

Top-right, a pasted citation block for a second paper: "**Pre-training under infinite compute**", authors "Konwoo Kim$^\infty$, Suhas Kotha$^\infty$, Percy Liang, Tatsunori Hashimoto" (the superscript is printed as an infinity symbol, "∞", not an asterisk — transcribed as printed), affiliation "Stanford University".

Body text: "**Important notes:**"
1. Scaling laws can 'break' if you blindly apply it
2. Scaling laws are *lower bounds* so you can always potentially do better

Three small charts, each with its own title and a data table underneath (charts 1 and 2 share a y-axis range).

**Chart 1 (left) — "Increasing epoch count."** X-axis: "Epochs", log-spaced labelled ticks 1, 2, 4, 8, 16, 32, 64, 128. Y-axis: "Loss", linear scale, ticks 3.8 to 5.0 in steps of 0.2. One series: a red line connecting red circular markers, U-shaped — $(1, 4.99)$, $(2, 4.16)$, $(4, 3.82)$, $(8, 3.78)$, $(16, 3.94)$, $(32, 4.31)$, $(64, 4.66)$, $(128, 4.96)$ — minimum loss at 8 epochs, rising back up almost to its starting value by 128 epochs. Table beneath: row "**Tuned H**" with columns 1, 8, 128; row "Learning rate" with values 1e-3, 1e-3, 3e-3.

**Chart 2 (middle) — "Increasing parameter count."** X-axis: "Parameter count", log-scale labelled ticks 150M, 300M, 600M, 1.4B. Y-axis: "Loss", same 3.8–5.0 range as chart 1. One series: a red line with circular markers, nearly flat — $(150M, 3.84)$, $(300M, 3.79)$, $(600M, 3.75)$, $(1.4B, 3.77)$ — a shallow dip rather than the pronounced U-shape of chart 1. Table beneath: row "**Tuned H**" with columns 150M, 300M, 600M, 1.4B; row "Learning rate" with values 3e-3, 1e-3, 1e-3, 3e-4; row "Epoch count" with values 8, 8, 4, 4.

**Chart 3 (right) — "Varying seed token count D."** X-axis: "Seed token count D", log-scale labelled ticks 209M, 419M, 839M, 1.67B. Y-axis: "Loss", linear scale, ticks 2.6 to 3.8 in steps of 0.2. Three dashed series (legend, top-right of the chart), each with a fitted-power-law label: red, "Standard recipe (Fit: $1.30/D^{0.23}+1.89$)"; purple, "Regularized asymptotes (Fit: $1.03/D^{0.23}+1.96$)"; gold/yellow, "Ensemble asymptotes (Fit: $0.88/D^{0.24}+1.90$)". All three series share the same four x-positions and use matching marker shapes at each: circle at 209M, triangle at 419M, square at 839M, "×" at 1.67B.
- **Standard recipe** (red, topmost): $(209M, 3.75)$, $(419M, 3.48)$, $(839M, 3.24)$, $(1.67B, 3.05)$.
- **Regularized asymptotes** (purple, middle): $(209M, 3.43)$, $(419M, 3.22)$, $(839M, 3.03)$, $(1.67B, 2.88)$.
- **Ensemble asymptotes** (gold, lowest): $(209M, 3.17)$, $(419M, 2.98)$, $(839M, 2.81)$, $(1.67B, 2.67)$.
All three lines run roughly parallel across the four points, with the red "Standard recipe" line highest throughout and the gold "Ensemble asymptotes" line lowest throughout.

## Slide 26 — Data selection scaling and accounting for finiteness

Title (blue): "**Data selection scaling and accounting for finiteness**"

Body text: "Given that repeated data is less valuable.." and "Data selection should then be adaptive to scale !"

Top-right, a pasted citation block for a third paper: "**Scaling Laws for Data Filtering— Data Curation *cannot* be Compute Agnostic**" (title, with "cannot" italicised), authors "Sachin Goyal$^{*\dagger}$ Pratyush Maini$^{*\dagger}$" (first row) and "Zachary C. Lipton$^\dagger$ Aditi Raghunathan$^\dagger$ J. Zico Kolter$^{\dagger,\ddagger}$" (second row), affiliations "Carnegie Mellon University$^\dagger$ Bosch Center for AI$^\ddagger$", and a contact line "{sachingoyal,pratyushmaini,zlipton,raditi,zkolter}@cmu.edu".

Four figures follow, left to right.

**Figure 1 — "Web Data is Non-Homogenous."** A single vertical stacked bar with six segments, top to bottom labelled E, D, C, A, B, F, coloured from dark green (E, top) fading down to pale yellow-green (F, bottom). A downward black arrow to the left of the bar, spanning its full height, is labelled "Lower Quality Data Pools" — i.e., E is the highest-quality bucket and F the lowest.

**Figure 2 — "Quality-Quantity Tradeoff (QQT) for Data Filtering."** A grid with three labelled columns, "Epoch 1", "Epoch 2", "Epoch 3" (a rightward arrow above the columns is labelled "Lower Utility of Repeated Data"), and three row-buckets, E, D, C, top to bottom (a downward arrow to the left of the rows is labelled "Lower Quality Data Pools", matching Figure 1). The E row has all three epoch cells (E, E, E), shaded dark green → medium green → pale green left to right. The D row likewise has all three (D, D, D), shaded dark green → medium green → pale yellow-green. The C row has only two cells, at Epoch 1 and Epoch 2 (C, C, shaded medium green → pale green); there is no Epoch-3 cell for bucket C. Two dashed outlines overlay the grid, crossed by faint diagonal hatch lines in matching colours: a blue outline, labelled "Pool 1", traces around the entire E and D rows across all three epochs (six cells: E1, E2, E3, D1, D2, D3); a red/pink outline, labelled "Pool 2", traces a stepped path enclosing epochs 1 and 2 of all three rows (six cells: E1, E2, D1, D2, C1, C2) while excluding epoch 3 entirely. A small legend at the bottom-right of the panel reads "**Pool 1**" (blue) / "v/s" (black) / "**Pool 2**" (red).

**Figure 3 — "Best Data Pool Changes with Total Compute."** Three stacked, dashed-outline boxes. Top, pale-green outline, labelled "Small Compute": two small dark-green boxes, "Epoch 1: E" and "Epoch 2: E". Middle, pale-blue outline, labelled "Medium": two green boxes both reading "E + D". Bottom, pale-red outline, labelled "Large": two green boxes both reading "E + D + C".

**Figure 4 — "Estimated Scaling Curves" chart.** Y-axis: "ImageNet-1k Estimated Error", linear scale, ticks 0.60 to 0.95 in steps of 0.05. X-axis: "Millions of Total Training Samples Seen", log scale, labelled ticks $10^2$ and $10^3$. Two legends: a marker-style key, top-right ("Actual" = grey square, "Estimated" = grey line), and a series key, left ("Bucket E only" = green circle, "E+D (Pool 1)" = blue star, "E+D+C (Pool 2)" = red triangle, "E+D+C+A" = yellow square). Two black vertical dotted lines, at roughly $x\approx80$ and $x\approx450$, divide the plot into three annotated regions: green text "Small Compute: Highly aggressive filtering is best" (left of the first line), blue text "Medium Compute: Mildly aggressive filtering is best" (between the two lines), red text "Large Compute: Less aggressive filtering is best" (right of the second line).
- All four **Estimated** lines start clustered near the left edge (around $x\approx30$–$40$) between about 0.90 and 0.94, ordered top to bottom: yellow highest, then red, then blue, then green lowest.
- **Bucket E only** (green): declines fastest at first but flattens from about $x\approx100$ onward, ending nearly flat at about 0.685–0.69 all the way to the right edge ($x\approx2000$) — despite leading the decline early on, it ends as the *worst* (highest-error) of the four lines.
- **E+D (Pool 1)** (blue), **E+D+C (Pool 2)** (red), and **E+D+C+A** (yellow) all keep declining well past where the green line flattens. At the right edge they end, lowest to highest: red $\approx0.575$, yellow $\approx0.585$, blue $\approx0.605$.
- **Actual** markers (matching each series' colour and shape) sit on or near their own line at a few sampled x-positions (roughly $x\approx40$, $x\approx130$, and $x\approx600$); one exception is an isolated green "Actual" circle near $x\approx600$ that sits noticeably above the flat green "Estimated" line at that point.

## Slide 27 — Recap: data scaling laws

Title (blue): "**Recap: data scaling laws**"

A pale-blue bordered box with four bullets (❖), each with generous vertical spacing:
- ❖ Remarkably linear relationship between log-data size and log-error
- ❖ Holds across domains and models
- ❖ Theory understanding: similar to generalization bounds: mean estimation example
- ❖ Applications: data collection / curation

No chart or table on this page.

## Slide 28 — Scaling laws for model engineering

Title (blue): "**Scaling laws for model engineering**"

Body text: "Now for what I promised at the start: **model scaling!**"

A pale-blue bordered box:
"**Our motivation:** how can we efficiently design huge LMs?"
- LSTMs vs Transformers
- Adam vs SGD

A second pale-blue bordered box, below the first:
"How should we allocate our limited resources?"
- Train models longer vs train bigger models?
- Collect more data vs get more GPUs?

Body text below both boxes: "Scaling laws provide a simple procedure to answer these."

No chart or table on this page.

## Slide 29 — Hyperparameter questions

Title (blue): "**Hyperparameter questions**"

Body text: "We'll consider some of these choices in the context of the classic Kaplan scaling paper"

A pale-blue bordered box with four bullets, each with generous vertical spacing:
- Architecture
- Optimizer
- Aspect ratio / depth
- Batch size

No chart or table on this page.
## Slide 30 — 1. Architecture: transformers vs LSTMs

Heading: "1. Architecture: transformers vs LSTMs". A light-blue rounded box below the heading reads: "**Q:** Are transformers better than LSTMs?" and "Brute force way: spend tens of millions to train a LSTM GPT-3". Below the box, body text: "Scaling law way:"

**Figure — log-linear line chart, test loss vs parameter count, transformers vs LSTMs [Kaplan+ 2021].** Y-axis "Test Loss", linear scale, ticked 2.4, 3.0, 3.6, 4.2, 4.8, 5.4. X-axis "Parameters (non-embedding)", log scale, ticked 10^5 through 10^9. Four series, distinguished by colour and a curved-arrow label pointing at each cluster ("Transformers" and "LSTMs"), plus per-curve text labels "1 Layer", "2 Layers", "4 Layers" for the three LSTM curves:

- Blue "Transformers": the single lowest curve on the plot. Runs the full width of the chart, from about (1.5×10^5, 5.2) down to about (1.5×10^9, 2.3).
- Light-pink "1 Layer" (LSTM): starts at about (1×10^5, 4.95), overlapping closely with "2 Layers" at the start, and ends around (7×10^7, 3.35) — the worst (highest-loss) of the three LSTM curves at the point where it stops.
- Medium-red "2 Layers" (LSTM): starts at about (1×10^5, 4.95) alongside "1 Layer", tracks just below it, and continues further right than the other two LSTM curves, ending around (2×10^8, 3.05) — the rightmost and lowest-loss endpoint of the LSTM group.
- Dark-maroon "4 Layers" (LSTM): joins the plot later than the other two, around (2.5×10^6, 3.9), and ends earliest (leftmost) of the three LSTM curves, around (8×10^7, 3.15).

Citation "[Kaplan+ 2021]" printed to the right of the chart.

The figure illustrates the "scaling law way" of answering the slide's question: rather than brute-force training a full-size LSTM to compare against GPT-3, fit small-scale curves for each architecture and compare their trends. The Transformers curve sits visibly and consistently below (better than) all three LSTM depths across the entire measured range.

## Slide 31 — 1. Many architectures

Heading: "1. Many architectures". A two-part figure occupies most of the page, followed by a one-line caption.

**Figure (left) — a large aggregate scatter/bubble plot, "Negative Log-Perplexity" vs "FLOPS."** Y-axis "Negative Log-Perplexity", linear scale, ticked -3.0, -2.8, -2.6, -2.4, -2.2, -2.0, -1.8, -1.6, -1.4 (top); less-negative values toward the top are better. X-axis "FLOPS", log scale, ticked 1.1e12, 2.2e12, 4.4e12, 8.8e12, 1.8e13, 3.5e13, 7.0e13, 1.4e14. The plot is a dense field of several dozen circular bubble markers, one per named model checkpoint, coloured by architecture family (teal, brown, red, orange, blue, pink, purple, yellow, grey — the same families as the small-multiple panels on the right) with bubble size varying (generally larger toward the top-right, smaller toward the bottom-left). Individually legible labels include, among the smaller/lower-left points, "Performer Tiny", "Funnel Tiny", "UT Tiny", "Albert NH8", "Evolved NL2", "Albert NL2/NL4/NL8/NL16/NL24/NL36", "Albert FF6K/FF9K/FF12K/NH16/NH24/NH32", "GLU Tiny", "MoS- Tiny", "DConv Tiny/Small", "LConv Tiny/Small/Base/NL2/NL4/NL16/FF9K/FF12K/Large/3B", "Switch 460M", "UT NR1–NR6", "Performer Small/Base/Large/PK Base/PK Small", "MLP Mixer Small/Large/3B"; toward the top-right, "GLU 3B", "MoS- ..." (a large bubble whose label is cut off after "MoS-" at the right edge of the plot), "Evolved 3B", "Switch 4B", "DConv Large". In the dense central band (roughly 4×10^12–3×10^13 FLOPS) dozens of overlapping labels and circles could not be individually resolved at any magnification tried. The overall pattern is a rising cloud: FLOPS increasing from bottom-left to top-right tracks with negative log-perplexity rising from about -3.0 (worst, e.g. "Performer Tiny") to about -1.4 (best, e.g. the large "MoS-..." bubble at the top right).

**Figure (right) — an 11-panel grid of small multiples, labelled (a) through (k), one per architecture: (a) ALBERT, (b) DConv, (c) Evolved, (d) Funnel, (e) Transformer-GLU, (f) LConv, (g) MLP Mixer, (h) MoS Transformer, (i) Performer, (j) Switch Transformer, (k) Universal Transformer.** Every panel shares the same axis structure: y-axis "Negative Log-Perplexity", linear, ticked -3.0 to -1.4 (top); x-axis "FLOPS", log scale, ticked from about 1.1e12 up to 7.0e13 in most panels, extending to about 1.4e14 in panels (a), (c), (j) and (k). Each panel has exactly two series:

- **Green circles, connected**, whose points are labelled with generic size names "Mini", "Small", "Base", "Large", "XL" — an apparent common reference/baseline curve that looks the same shape across all 11 panels.
- **Red/pink open circles, connected**, whose points are labelled with that panel's own architecture name and size — e.g. panel (b) DConv: "DConv Tiny", "DConv Small", "DConv Base", "DConv Large"; panel (j) Switch Transformer: "Switch 173M", "Switch 460M" (labelled "Switch 460M" near "Base"), "Switch 2B", "Switch 4B", "Switch XL3"; panel (k) Universal Transformer: "UT Tiny", "UT Small", "UT Base", "UT Large".

Reading the panels: in most of them (ALBERT, DConv, Evolved, Funnel, Transformer-GLU, LConv, MLP Mixer, MoS Transformer), the red architecture-specific curve sits at or slightly below the green reference curve through the small-to-mid sizes and then converges toward it by the largest size (e.g. DConv's "DConv Large" ends close to, but still below, the green curve's "Large"/"XL" region). Two panels stand out from that pattern: in (j) Switch Transformer the red curve's largest points ("Switch 4B", "Switch XL3") sit at or slightly above the green curve — matching or slightly beating the reference at scale; in (k) Universal Transformer the red curve starts much lower ("UT Tiny" far below "Mini") and stays clearly below green at every labelled size, the widest and most persistent gap of any panel. Panel (i) Performer is partially cropped at the left edge of the rendered page (its "Tiny"/"Small" labels are cut) but shows the same two-series, same-axis structure as the others.

Below both figures, a one-line caption: "Cross-architecture scaling in Tay et al 2022."

## Slide 32 — 2. Optimizer choice

Heading: "2. Optimizer choice". Body text: "What about ADAM vs SGD?"

**Figure — log-log line chart, "Minimum Validation Loss (Log-scale)" vs "Training Data Set Size, Number of Chars (Log-scale)" [Hestness+ 2017].** Y-axis ticked 0.86, 0.93, 1.00, 1.08, 1.17, 1.26, 1.36, 1.47, 1.59 (evenly spaced in log space). X-axis ticked 2^19, 2^21, 2^23, 2^25, 2^27. Four series, per the legend:

- Solid blue "Depth-10 RHNs, SGD": runs the full width of the plot, from about (2^18, 1.55) down to (2^27, ≈0.93–0.96).
- Solid orange "Depth-10 RHNs, Adam": starts alongside the blue curve at about (2^18, 1.5) but stops early, around x = 2^24, at about y = 1.15 — it does not reach the right edge of the plot.
- Dashed green "Depth-10 RHNs, SGD Trend": a straight-line fit spanning the full x-range, closely tracking the solid blue SGD curve; labelled "$\varepsilon(m) = 5.37\,m^{-0.094}$" where the fit crosses the middle of the plot; ends at about (2^27, 0.87).
- Dashed red "Depth-10 RHNs, Adam Trend": a straight-line fit spanning the full x-range, closely tracking the solid orange Adam curve while data exists and continuing past where the Adam data stops; labelled "$\varepsilon(m) = 5.25\,m^{-0.095}$"; ends lowest of all four series, at about (2^27, 0.80).

Citation "[Hestness+ 2017]" printed to the right of the chart.

Below the figure, body text: "(Note, this is in 2017, so pre-transformers. RHN is recurrent highway nets)"

The chart shows the fitted power-law exponents for SGD (-0.094) and Adam (-0.095) are nearly identical, with the Adam trend sitting consistently below (better than) the SGD trend — i.e. the optimizer choice shifts the curve's constant slightly but barely changes its scaling exponent.

## Slide 33 — 3. Depth/Width: Number of layers

Heading: "3. Depth/Width: Number of layers". Body text: "Does depth or width make a huge difference?"

**Figure — log-linear line chart, test loss vs parameter count, by layer count.** Y-axis "Test Loss", linear, ticked 2 through 7. X-axis "Parameters (non-embedding)", log scale, ticked 10^3 through 10^9. Five series, per the legend (colour scale runs indigo → magenta/pink → orange → gold):

- Dark purple/indigo "1 Layer": the topmost (worst) curve, from about (8×10^2, 6.4) to about (5×10^7, 3.6) — the shortest of the five curves, stopping well before the others.
- Medium purple "2 Layers": from about (8×10^2, 6.0) down to about (1.3×10^8, 3.15).
- Pink/rose "3 Layers": from about (8×10^2, 6.0), tracking closely with "2 Layers" through the low-to-mid range and then continuing further right, ending around (1.3×10^9, 2.55).
- Orange "6 Layers": joins the plot later, around (2×10^6, 4.6), tracking almost on top of "> 6 Layers" for the rest of its length, ending around (1.3×10^9, 2.5).
- Gold/yellow "> 6 Layers": also joins around (2×10^6, 4.6), ending around (1.3×10^9, 2.45) — the best (lowest-loss) curve at the largest scale shown.

Below the figure, two bullets:
- "1 vs 2 layers makes a huge difference."
- "More layers have diminishing returns below $10^7$ params"

(This chart's x-axis tick label "$10^7$" is the source of the stray "10" and "7" tokens the parent's page-numbering scan found on this page — they are the base and exponent of that axis tick, not a folio.)

## Slide 34 — 3. Depth/Width: and other Transformer hypers

Heading: "3. Depth/Width: and other Transformer hypers". Body text: "Do hyperparameters like the aspect ratio depend on scale?"

**Figure — three side-by-side line charts, all sharing a "Loss Increase" y-axis (linear, 0% to 10%, ticked every 2%).**

**Panel 1, "Feed-Forward Ratio ($d_{ff}/d_{model}$), 50M Parameters":** x-axis log scale, ticked 10^0, 10^1. Two series, nearly coincident throughout: blue x-markers "$n_{head} = 8$" and orange circles "$d_{model}/n_{head} = 64$". Values (shared between the two series except where noted): x≈0.5 → 0.7%; x=1 → ≈0%; x=2 → ≈0.05%; x=4 → ≈0.3%; x=8 → blue ≈1.5%, orange ≈1.8%; x=20 → blue ≈4.8%, orange ≈4.7%; x=40 (rightmost point) → blue ≈7.8%, orange ≈8.4%.

**Panel 2, "Aspect Ratio ($d_{model}/n_{layer}$)":** x-axis log scale, ticked 10^1, 10^2, 10^3. Three series: blue circles "50M Params" (8 points, x≈4 to x≈700), orange x-markers "274M Params" (sparser, 4 points, x≈2 to x≈1700–2000), green stars "1.5B Params" (5 points, x≈4 to x≈700). All three dip to a shared near-zero minimum around x=30–90, then rise again at both ends. Approximate values — 50M Params: x=4→2.7%, x=12→1.4%, x=32→0.4%, x=50→≈0.05%, x=90→≈0.1%, x=250→0.6%, x=350→2.1%, x=700→4.6%. 274M Params: x=2→1.1%, x=25→≈0%, x=200→0.55%, x=1700→7.9% (its last point, the highest value in the panel). 1.5B Params: x=4→2.1%, x=13→0.5%, x=27→≈0.05%, x=250→0.6%, x=700→2.8% (its last point; unlike the other two series it does not extend past x≈700). Two vertical black bracket lines mark a wide low-loss range on the x-axis (roughly x=10 to x=300), with the annotation "A wide range of architectures achieve similar performance" above them.

**Panel 3, "Attention Head Dimension ($d_{model}/n_{head}$), 25M Parameters":** x-axis log scale, ticked 10^1, 10^2. Three series, five points each, all spanning x≈16 to x≈300: blue circles "$d_{model} = 256$" (rises from ≈0.1% to ≈1.9% at the last two points), orange x-markers "$d_{model} = 512$" (rises from ≈0.2% to ≈1.7%), green triangles "$d_{model} = 1024$" (stays lowest throughout, rising only to ≈0.8% at x=300). A small vertical bracket icon is annotated "22% additional compute compensates for 1% loss increase".

Below the three panels, a native-text figure caption, reproduced in full: "**Figure 5** Performance depends very mildly on model shape when the total number of non-embedding parameters $N$ is held fixed. The loss varies only a few percent over a wide range of shapes. Small differences in parameter counts are compensated for by using the fit to $L(N)$ as a baseline. Aspect ratio in particular can vary by a factor of 40 while only slightly impacting performance; an $(n_{layer}, d_{model}) = (6, 4288)$ reaches a loss within 3% of the $(48, 1600)$ model used in [RWC+19]."

## Slide 35 — 3. Depth/Width: But not all parameters are made equal

Heading: "3. Depth/Width: But not all parameters are made equal". Body text: "We've been thinking about 'parameters' but not all parameters are equal"

**Figure — two side-by-side log-linear line charts, test loss vs parameter count, by layer count.** Both panels share a "Test Loss" y-axis, linear, ticked from 2 up to about 7.

**Left panel**, x-axis "Parameters (with embedding)", log scale, from about 10^6 to 10^9. Six series: dark-navy "0 Layer" — a near-flat curve running only from about (2×10^5, 6.9) to (5×10^7, 5.85), the shortest curve on the panel and the only one that barely improves as parameters increase — plus the same five layer-count series as slide 33: dark purple "1 Layer", magenta "2 Layers", pink "3 Layers", orange "6 Layers", gold "> 6 Layers", each following the same relative shape as on slide 33 but re-plotted against the "with embedding" parameter count.

**Right panel**, x-axis "Parameters (non-embedding)", log scale, from about 10^3 to 10^9 — the same five-series chart as slide 33 (no "0 Layer" series here): purple "1 Layer" (shortest, ending around (5×10^7, 3.6)), magenta "2 Layers", pink "3 Layers", orange "6 Layers", gold "> 6 Layers" (the latter four converging by the right edge, around (1.3×10^9, 2.4–2.55)).

Below the figure, body text: "Embedding layer parameters don't behave the same!" and a further bullet: "**Related**: recent papers on scaling laws for mixtures of experts."

The point of the two-panel comparison is that a 0-layer (embedding-only) model's loss barely falls as its embedding-parameter count grows, unlike every layered model — so counting embedding parameters on the same axis as "real" transformer parameters is misleading, which is why the deck's other depth/width charts use the "non-embedding" parameter count.

## Slide 36 — Side note – 'Value of parameters'

Heading: "Side note – 'Value of parameters'". Body text: "With MoEs, we expect parameter scaling to change"

**Pasted paper header (top right).** Title: "Parameters vs FLOPs: Scaling Laws for Optimal Sparsity for Mixture-of-Experts Language Models". Authors, in two rows: Samira Abnar* (Apple), Harshay Shah* (MIT), Dan Busbridge (Apple); Alaaeldin Mohamed Elnouby Ali (Apple), Josh Susskind (Apple), Vimal Thilak* (Apple).

**Figure — two panels from that paper, spanning most of the slide.**

**Panel (b), "Optimal Total Parameters $N^*$":** x-axis "Total Parameters $N$", log scale, ticked 178M, 485M, 1B, 4B, 10B, 26B; y-axis is pretraining loss (unlabelled tick numerals not legible in the rendered crop, but on the same general scale as panel (c)). A family of U-shaped curves, one per MoE sparsity level, coloured from orange (0% sparsity) through pink/magenta to dark purple (95% sparsity), per a colour-bar legend below the panel ("MoE Sparsity $S$", 0% to 95%). A separate green curve traces the lower envelope across all the sparsity curves, bottoming out around (10B, its minimum). White star markers ("Optimal total parameters", per the panel's own legend) trace one point on each sparsity curve, forming a diagonal band running from the upper-left down to the green envelope's minimum as sparsity increases.

**Panel (c), "Optimal Active Parameters $N_a^*$":** x-axis "Active Parameters $N_a$", log scale, ticked 178M, 294M, 485M, 800M, 1B, 2B; y-axis "Pretraining Loss $L$", linear, ticked 2.2 to 2.7. The same family of sparsity-coloured U-shaped curves (orange 0% through purple 95%) plus the same style of green envelope curve, whose minimum sits around (800M, 2.2). White star markers ("Optimal active parameters") again trace a band across the family, running from about (1B, 2.44) at low sparsity down to the green curve's minimum at high sparsity.

**A third, smaller panel — "(b) IsoFLOP surface over sparsity and active parameters" — a 3D surface plot**, positioned below/right of the two 2D panels. Axes: "Active Parameters $N_{active}$" (log scale, floor axis, ticked 10B, 6B, 4B, 2B, 1B, 800M, 485M, 294M, 178M, 108M, 66M, decreasing left to right), "MoE Sparsity $S$" (floor axis, ticked 0%, 39%, 63%, 79%, 88%, 93%, 96%, 97%, 98%, increasing toward the back-right), "Loss $\mathcal{L}$" (vertical, ticked 2.1(white) to 2.8(dark grey) via a white-to-grey surface colour scale, also shown as a separate colour-bar legend at top labelled "Loss $\mathcal{L}$", 2.1 to 2.8). A grey saddle/bowl-shaped surface is drawn across the floor grid, and overlaid on it are strings of coloured dots (coloured by sparsity, same orange-to-purple scale as the other two panels, per a "Sparsity $S$" colour-bar legend at top, 0% to 98%) connected by dashed lines — one string per sparsity level — each string forming its own shallow U-shaped valley in active parameters, with the valley's minimum shifting toward fewer active parameters and lower loss as sparsity increases from 0% (orange, valley around 2.5–2.6) to about 93–96% (dark purple, valley around 2.15–2.2).

The slide's point, per its own heading and text, is that under MoE sparsity the relationship between parameter count and useful capacity changes, motivating the paper's two separate parameter-scaling laws (total vs. active) shown in panels (b) and (c).

## Slide 37 — 4. Batch size: Critical batch size

Heading: "4. Batch size: Critical batch size".

**Pasted paper header (top right).** Title: "An Empirical Model of Large-Batch Training". Authors: Sam McCandlish* (OpenAI, sam@openai.com), Jared Kaplan (Johns Hopkins University, OpenAI, jaredk@jhu.edu), Dario Amodei (OpenAI, damodei@openai.com), "and the OpenAI Dota Team†".

**Figure 1 (left) — a contour/ellipse diagram with two overlaid arrows.** A small legend at top-left: a red square swatch labelled "Smaller batch" and a blue square swatch labelled "Larger batch". The main figure is a set of six concentric ellipses (contour lines) around a central black dot, labelled from inside out: 0.02, 0.10, 0.20, 0.40, 0.70, 1.00. A vertical axis label on the right edge reads "$\epsilon_{opt}(B)/\epsilon_{max}$". Two arrows originate from the central black dot: a longer solid blue arrow pointing right and slightly up, whose tip lands inside the 0.02–0.10 band (the "Larger batch" direction, per the legend colour); and a shorter, steeper red dash-dot arrow pointing up and slightly left, whose tip is marked with a red "X" landing on the outermost 1.00 contour (the "Smaller batch" direction).

**Figure 2 (right) — "Predicted Training Speed" log-log line chart.** Y-axis "$\epsilon_{opt}(B)/\epsilon_{max}$", log scale, ticked 10^-2, 10^-1, 10^0. X-axis "Batch Size / Noise Scale $(B/\mathcal{B})$", log scale, ticked 10^-2 through 10^2. One solid blue curve: rises linearly on the log-log plot (i.e. as a power law) from about (10^-2, 10^-2) up through a vertical grey dashed reference line at x=10^0 (where the curve sits at about y=0.5), then bends over and flattens, reaching about y=0.95 by x≈10 and staying essentially flat (y≈1.0) out to x=100. Two text annotations with leader lines point at the curve: "**Perfect scaling**" pointing at the linear rising portion (around x=0.05–0.2), and "**Ineffective scaling**" pointing at the flattened plateau portion (around x=10–30), to the right of the dashed reference line.

Below the two figures, two bold lines of body text: "**Batch size** – known to have strong diminishing returns past a certain point." and "**Critical batch** = min number of examples before diminishing returns (what does that mean?)"

## Slide 38 — 4. Critical batch size definitions..

Heading: "4. Critical batch size definitions..". Body text: "**What is critical batch size, more precisely?**" (bold), followed by two bullets:
- "Pick a target loss and train 1). Steps needed (S), 2). Examples needed."
- "Sweep over batches"

Below the bullets: "The curve should follow roughly.."

Displayed equation:
$$\frac{S}{S_{min}} - 1 = \left(\frac{E}{E_{min}} - 1\right)^{-1}$$

Body text: "Fit for $S_{min}$, $E_{min}$ on this curve and pick"

Displayed equation:
$$\mathcal{B}_{crit} = \frac{E_{min}}{S_{min}}$$

Body text: "This balances both sides of the equation, giving roughly 2x the steps / passes optimal"

A smaller line of body text below that: "(this is claimed to be close to the ratio of the trace of the gradient covariance and squared norm of the gradient)"

No chart or table on this page.

## Slide 39 — 4. Batch size: critical batch size

Heading: "4. Batch size: critical batch size".

**Figure — "Critical Batch Size vs. Performance" log-log scatter/line chart.** Y-axis "Critical Batch Size (Tokens)", log scale, ticked 10^3, 10^4, 10^5, 10^6. X-axis "WebText2 Train Loss", log scale but running in decreasing order left to right, ticked 10^1, 6×10^0, 4×10^0, 3×10^0 (i.e. loss decreases moving right). Four series, per the legend:

- Light-green dots, "Noise Scale Measurement": several hundred individual scattered points, densest in a tight vertical cluster at the far left (around x=10^1, spanning y≈800 to 4000) and then spreading out with increasing scatter as x decreases (moving right), tracking the general upward trend up to about (3×10^0, 2×10^6) at the top right.
- Blue line+markers, "Empirical $B_{crit}$, $N=3M$": a smoothed trace through the scatter, starting around (10^1, 3×10^3), dipping slightly then climbing steadily; it has a sharp upward spike to about 4×10^5 partway along before dropping back down and continuing to track the trend, ending around (3×10^0, ≈4×10^5).
- Orange line+markers, "Empirical $B_{crit}$, $N=85M$": a similar smoothed trace, starting around (10^1, 3.5×10^3), climbing steadily and peaking near (3×10^0-ish, just under 10^6) before dropping back down at its very last point to about 4×10^5.
- Grey dashed line, "$B_{crit} = 2.1\times10^8$ tokens $\cdot L^{-4.8}$": a straight power-law reference line spanning the full plot width, from about (10^1, 3×10^3) to about (3×10^0, 1.5×10^6).

Below the figure, body text: "The smaller the loss target, / The bigger the batch" (two lines), alongside the displayed equation

$$C_{min}(C) \equiv \frac{C}{1 + B/B_{crit}(L)}$$

and, to its right, the parenthetical note "(minimum compute, at $B \ll B_{crit}$)".

## Slide 40 — 5. Learning rates: muP and scale-aware LR choices

Heading: "5. Learning rates: muP and scale-aware LR choices".

**Figure (left) — two side-by-side line charts, "Standard Practice" vs "Our Work" [Yang et al 2022].** Both share a "Training Loss" y-axis, linear, ticked 3.5 to 7.0 in steps of 0.5. Both plot log2(LearningRate) on the x-axis, with a shaded uncertainty band around each line. Seven series per panel, one per model width, per the legend "Width: 128, 256, 512, 1024, 2048, 4096, 8192" (colour scale running from light pink/cream at 128 to dark navy/purple at 8192). A horizontal black dotted reference line runs across both panels near training loss ≈ 3.85.

- **"Standard Practice"** panel, x-axis ticked -16, -14, -12, -10 (extending further left, toward about -20, per the wider view): each width's curve descends to its own minimum and then rises again sharply (near-vertically), i.e. training diverges/blows up past some learning rate specific to that width. The minima are staggered: the darkest (largest-width) curve's minimum sits furthest left, around log2(LR) ≈ -15, and each lighter (smaller-width) curve's minimum sits progressively further right, out to about -10 for the lightest curve. An annotation "optimum shifts", with an upward arrow, points at the leftmost minimum.
- **"Our Work"** (muP) panel, x-axis ticked -20, -18, -16, -14, -12, -10: all seven curves descend together along nearly the same path from the upper left, separating only slightly near their shared broad minimum, which sits around log2(LR) ≈ -10 to -11 for every width. An annotation "optimum stable →" points at this shared minimum region.

Citation "Yang et al 2022" printed beneath the left figure.

**Table (right) — "$\mu$P function for a model $M'$ that is r times the widths of M" [Yao et al 2024].** Above the table, native-text caption (reproduced in full): "*Table 2.* $\mu$P function for a model $M'$ that is r times the widths of $M$. If a parameter tensor has 2 dimensions that goes infinite when the model width goes infinite, it is "matrix-like" (*e.g.*, a fully-connected hidden layer); if the number is 1 or 0, it belongs to the "others" class. Note that embedding layers are "others". "Output" means the layer that maps an infinite dimension to a finite dimension, which is the word decoding layer ($lm\_head$) in Transformers. A multiplier is a constant multiplied by a parameter tensor, which has a similar function to softmax temperature."

| Hyperparameter (weight) | $M$ | $M' \sim r$ |
| --- | --- | --- |
| AdamW learning rate (matrix-like) | $l$ | $l/r$ |
| AdamW learning rate (others) | $l$ | $l$ |
| Initialization variance (matrix-like) | $\sigma$ | $\sigma/r$ |
| Initialization variance (others) | $\sigma$ | $\sigma$ |
| Multiplier (output) | $\tau$ | $\tau/r$ |
| Multiplier (others) | $\tau$ | $\tau$ |

Citation "Yao et al 2024" printed beneath the table.

Below both figures, body text: "**If we naively scale up** – optimal learning rate depends on scale." and "We need *scaling aware* initialization and learning rate scaling"

## Slide 41 — Caution – scaling behaviors can differ downstream

Heading: "Caution – scaling behaviors can differ downstream". Two lines of body text: "**Thus far**: scaling is predictable and depends mainly on parameters" and "**Catch**: downstream scaling can often be much less predictable"

**Figure — two side-by-side scatter plots, upstream perplexity vs. downstream accuracy, for the same set of named model variants [Tay et al 2023].** Both panels share an x-axis "Params", log scale, ticked 2.7e8, 5.4e8, 1.1e9, 2.1e9, 4.3e9, 8.6e9, 1.7e10. Each of the two panels plots the same 13 named model variants, each with its own fixed marker shape and colour (consistent between the two panels): purple diamond "NL12-" (label cut off at the right edge of the pasted source image in both panels — the rest of the name is not printed/legible), green cross "NL6-XXXL", blue upward-triangle "NL12-XXL", blue open square "XL", red open square "NL8-XXL", pink left-pointing triangle "NL32-XL", grey open circle "NL6-XXL", yellow right-pointing triangle "NL32-LG", red/crimson open circle "Large", brown downward-triangle "NL8-XL", orange diamond "NL36", violet cross "NL32", and olive-green upward-triangle "NL24".

- **Left panel, y-axis "Negative Log-Perplexity"**, linear, ticked -1.70 to -1.35. Ranking from best (top, least negative) to worst (bottom, most negative): NL12- (≈-1.36, largest params, best perplexity) > NL6-XXXL (≈-1.44) > NL12-XXL (≈-1.47) > {XL, NL8-XXL, NL32-XL clustered together, ≈-1.49 to -1.50} > NL6-XXL (≈-1.52) > NL32-LG (≈-1.58) > {Large, NL8-XL clustered, ≈-1.59 to -1.60} > NL36 (≈-1.63) > NL32 (≈-1.64) > NL24 (≈-1.665, smallest params, worst perplexity).
- **Right panel, y-axis "SuperGlue Accuracy"**, linear, ticked 72 to 80. Ranking is visibly different: NL32-XL (≈79.7, the *best* score here despite being only mid-pack on perplexity) > {XL, NL12- clustered, ≈77} > NL12-XXL (≈75.6) > {Large, NL32-LG clustered, ≈75.3} > NL8-XXL (≈75.0) > NL6-XXXL (≈74.6) > {NL8-XL, NL24 clustered, ≈74.3-74.5} > NL6-XXL (≈73.6) > NL32 (≈72.9, the worst score, though not the smallest model).

Citation "Tay et al 2023" printed beneath the two panels.

The point of the pairing is exactly the slide's stated "Catch": NL32-XL, roughly mid-table on upstream perplexity, is the single best model on downstream SuperGlue accuracy, while NL12- — the best upstream model — only reaches the middle of the SuperGlue ranking; the ordering of the same 13 checkpoints is visibly reshuffled between the two panels.

## Slide 42 — Some surprising takeaways

Heading: "Some surprising takeaways". A light-blue rounded box contains: "The effect of hyperparameters on big LMs can be predicted *before* training!" followed by a centered sub-list:
- Optimizer choice
- Model depth
- Architecture choice

Below the box: "**The scaling law based design procedure.**" followed by a numbered list:
1. Train a few smaller models
2. Establish a scaling law (e.g. ADAM vs SGD scaling law)
3. Select optimal hyperparam based on the scaling law prediction.

No chart or table on this page.

## Slide 43 — One important use of scaling laws

Heading: "One important use of scaling laws". Body text: "**Q:** Do we need more data or bigger models?"

"Clearly, lots of data is wasted on small models"

"**Joint data-model scaling laws** describe how the two relate"

"From Rosenfeld+ 2020,"

Displayed equation:
$$Error = n^{-\alpha} + m^{-\beta} + C$$

"From Kaplan+ 2020"

Displayed equation:
$$Error = [m^{-\alpha} + n^{-1}]^{\beta}$$

"Provides surprisingly good fits to model-data joint error."

**Figure 1 (top right) — "Loss vs Model and Dataset Size" log-linear line chart.** Y-axis "Loss" (unlabeled in the rendered crop, ticked at approximately 2.5, 3.0, 3.5, 4.0, 4.5 per the wider page view), X-axis "Tokens in Dataset", log scale, from about 10^7 to beyond 10^10. Six dotted series with circular markers, per the legend "Params": yellow "708M", light-green "302M", teal-green "85M", dark-blue "3M", teal "25M", dark-purple "393.2K" (the legend's own ordering, not sorted by parameter count). Every curve starts near the same loss at the smallest dataset size (upper left, around 4.4–4.9) and descends as dataset size grows, then flattens into its own horizontal plateau toward the right; the plateau height is set by model size — the two largest-parameter curves (yellow 708M, light-green 302M) plateau lowest (around 2.5), while the smallest model (dark-purple 393.2K) plateaus highest (around 4.3-4.5) and barely descends at all across the same token range.

**Figure 2 (bottom right) — "(a) Wiki103 error (cross entropy) landscape" 3D surface plot [Rosenfeld+ 2020].** Axes: "log2(data fraction)" (one floor axis, ticked -5, -4, -3, -2, -1, 0), "log2(model fraction)" (the other floor axis, ticked 0, -2, -4, -6, -8, -10, -12), "log10(err)" (vertical axis, ticked 0.55 to 0.80). A saddle-shaped 3D surface is drawn, coloured on a blue (low error, large-model/less-data corner) to red/orange (high error, small-model corner) gradient, with small purple dot markers tracing sampled points across the surface's ridge. Additional 2D projections of the same data are drawn flattened against the walls of the plot's bounding box: a family of nearly-horizontal red-toned curves projected onto the upper back wall, and a family of blue- and red-toned curves projected onto the floor of the box.

The two figures together illustrate the slide's point: extra tokens help a small model much less than they help a large one (Figure 1's flat low-parameter curves), and the Rosenfeld et al. joint error surface (Figure 2) is the kind of landscape the two displayed equations are fit to.
## Slide 44 — Model-data joint scaling is accurate

Heading (blue): "**Model-data joint scaling is accurate**". Body text: "From Rosenfeld – fit scaling exponents on small data, small models. Predict rest."

**Figure — three panels, labelled (a), (b), (c) beneath each.**

**(a) — grid diagram, "Illustration."** Y-axis "model fraction: log2(m/M)", ticks 0, -2, -4, -6, -8, -10, -12 (top to bottom). X-axis "data fraction: log2(n/N)", ticks -6, -5, -4, -3, -2, -1, 0 (left to right). At every (row, column) intersection sits one circle, colored one of three ways: solid green, solid red, or an open/hollow grey outline circle. Row by row:
- Row 0: open circles at columns -6, -5, -4; solid red circles at columns -3, -2, -1, 0.
- Row -2: open circles at columns -6, -5, -4; solid red circles at columns -3, -2, -1, 0 (same pattern as row 0).
- Row -4: solid green circles at columns -6, -5, -4, -3; solid red circles at columns -2, -1, 0.
- Row -6: solid green circles at columns -6, -5, -4, -3; open circles at columns -2, -1, 0.
- Row -8: same as row -6 (green at -6..-3, open at -2..0).
- Row -10: same as row -6.
- Row -12: same as row -6.

So the green cells are the (small model fraction, small data fraction) corner — used to fit the scaling law — the red cells are the (large model fraction, large-ish data fraction) corner — held out to test extrapolation — and the open/grey cells are the remaining combinations, not used. This matches the "fit"/"extrapolated" legend used in panels (b) and (c).

**(b) — scatter, "Extrapolation on ImageNet."** X-axis "measured top1 error", linear scale 0.0 to 1.0. Y-axis "estimated top1 error", linear scale 0.0 to 1.0. A blue diagonal reference line runs corner to corner (y = x). Legend: green dot = "fit", red dot = "extrapolated". A text annotation inside the plot reads "μ:-4.5%" and "σ:4.681%" (upper left) and "model fraction 1/16" / "data fraction 1/8" (lower area). Green ("fit") points track the diagonal closely from about (0.0, 0.0) up to about (0.55, 0.55). Red ("extrapolated") points sit in the low-middle of the range, clustered roughly between (0.25, 0.3) and (0.5, 0.5), also tracking near the diagonal.

**(c) — scatter, "Extrapolation on WikiText-103."** X-axis "measured test loss", linear scale 3 to 7. Y-axis "estimated test loss", linear scale 3.0 to 7.0. Same blue y=x diagonal reference line. Same legend: green = "fit", red = "extrapolated". Annotation: "μ:0.5%", "σ:1.689%", "model fraction 1/16", "data fraction 1/8". Green points run the full diagonal from about (3.3, 3.3) up to about (6.5, 6.5). Red points cluster at the lower end, roughly (3.7-4.3, 3.7-4.5), overlapping the green cluster there.

The point of the figure: fitting the joint model-size/data-size scaling exponents on small models and small data slices (the green corner) accurately predicts loss/error at larger model and data fractions (the red corner), on both an image classification task and a language-modeling task.

Below the figure, body text: "Trading off data size and model size: optimize $n^{-\alpha}+m^{-\beta} + C$ with your costs."

## Slide 45 — 'Optimal' compute and data tradeoffs as a case study.

Heading (blue): "**'Optimal' compute and data tradeoffs as a case study.**" Body text: "Rosenfeld, Kaplan both predict relationship of data, model and perf." Bold line: "**Kaplan claims**: $N_{opt} = C^{0.73}$, $D_{opt} = C^{0.27}$ (tokens per param decreases w/ C)". Bold, centered: "**Chinchilla [Hoffman et al] argue these fits are quite off.**"

**Figure — log-log scatter with fitted lines and named-model stars.** X-axis "FLOPs", log scale, ticks $10^{17}$, $10^{19}$, $10^{21}$, $10^{23}$, $10^{25}$. Y-axis "Parameters", log scale, ticks 10M, 100M, 1.0B, 10B, 100B, 1T.

Series (legend, top group — solid/dashed trend lines):
- Blue solid line "Approach 1"
- Orange solid line "Approach 2"
- Teal/green solid line "Approach 3"
- Black dashed line "Kaplan et al (2020)"

Series (legend, bottom group — star markers for named models, no trend line):
- Teal star "Chinchilla (70B)"
- Orange star "Gopher (280B)"
- Red star "GPT-3 (175B)"
- Purple star "Megatron-Turing NLG (530B)"

Underlying the lines is a dense cloud of small blue dots (unlabelled scatter of empirical training runs) spanning roughly $10^{18}$–$2\times10^{21}$ FLOPs and 100M–10B parameters, which the three solid trend lines are fit to.

Values: the three solid trend lines (Approach 1, 2, 3) run almost on top of each other from about $10^{17}$ FLOPs / 30M parameters up to $10^{25}$ FLOPs, where they fan out slightly: Approach 1 (blue) reaches roughly 300B, Approach 2 (orange) roughly 250B, Approach 3 (teal) roughly 150B — Approach 3 sits visibly lowest of the three at the high-FLOPs end. The dashed black Kaplan line has a steeper slope: it starts below the three solid lines at low FLOPs (below 10M at $10^{17}$) and crosses above them by roughly $10^{21}$ FLOPs, continuing steeply up past the top of the plotted range near $10^{24}$–$10^{25}$ FLOPs. The four star markers sit clustered around $2\times10^{23}$–$10^{24}$ FLOPs: GPT-3 (red, 175B) is leftmost/lowest of the cluster; Gopher (orange, 280B) sits just above and to the right of GPT-3; Megatron-Turing NLG (purple, 530B) sits highest and rightmost, close to the dashed Kaplan line; Chinchilla (teal, 70B) sits lower and further right than the other three stars, close to where the three solid trend lines pass.

The chart's point: Kaplan's steeper dashed scaling line predicts far more parameters per unit of compute than the three Chinchilla-paper approaches, and real large models (GPT-3, Gopher, Megatron-Turing NLG) were trained closer to Kaplan's prediction than to the corrected fits — while Chinchilla itself, trained according to the corrected joint scaling law, sits right on the solid trend lines.

Below the figure, body text: "Why such a big difference (when both fit joint scaling laws?)"

Bottom-right corner: citation "Hoffman+ 2022".

## Slide 46 — Chinchilla in depth – 3 methods

Heading (blue): "**Chinchilla in depth – 3 methods**"

**Table.**

| Approach | Coeff. $a$ where $N_{opt} \propto C^{a}$ | Coeff. $b$ where $D_{opt} \propto C^{b}$ |
| --- | --- | --- |
| 1. Minimum over training curves | 0.50 (0.488, 0.502) | 0.50 (0.501, 0.512) |
| 2. IsoFLOP profiles | 0.49 (0.462, 0.534) | 0.51 (0.483, 0.529) |
| 3. Parametric modelling of the loss | 0.46 (0.454, 0.455) | 0.54 (0.542, 0.543) |
| Kaplan et al. (2020) | 0.73 | 0.27 |

("Kaplan et al. (2020)" is a blue hyperlinked citation; its row is separated from the three numbered approaches by a horizontal rule.)

Below the table, bold text: "**The chinchilla authors suggest 3 ways of fitting scaling laws – we'll go over each.**" Below that: "They mostly (minus method 3) suggest similar constants. More on this later.."

No other figure on this page besides the table.

## Slide 47 — Method 1 – minimum over runs.

Heading (blue): "**Method 1 – minimum over runs.**" Body text, centered, two lines: "Similar to the FLOPS figure on Kaplan –" / "the minimum over the union of all training curves is a power law."

**Figure — three panels, reproduced from the Chinchilla paper (Figure 2).**

**Left panel — "Training loss" vs "FLOPS" (log scale, ticks $10^{17}$–$10^{22}$), y-axis 2.0 to 6.0.** Many individual training-curve lines, each colored by model size via a continuous colorbar legend running from black/dark-purple (75M, smallest) through purple, red, orange, up to yellow (largest); the legend lists tick labels, bottom to top: 75M, 250M, 500M, 1B, 2.5B, 5B, 10B. Each curve traces one model's training loss falling as FLOPs increase, and the curves for different sizes overlap and cross; a grey dotted/dot-marker envelope traces the lower boundary (minimum loss at each FLOP value) across all curves, running from about (2×10^17, 6.0) down to about (10^22, 2.0).

**Center panel — "Parameters" (y-axis, log scale, 100M to 10B) vs "FLOPs" (x-axis, log scale, $10^{17}$–$10^{25}$).** Grey dot scatter (the envelope's minimum-loss points, extracted from the left panel) running from about ($10^{17}$, 100M) to about ($2\times10^{21}$, 6-8B). A red dashed line is the power-law fit through these points, extended out to $10^{25}$. A teal/green annotation — a horizontal line at y≈67B meeting a vertical line at Gopher's compute budget — is labelled "67B" in small teal text just left of the vertical line, at the point where the vertical line crosses the red dashed fit line.

**Right panel — "Tokens" (y-axis, log scale, $10^9$–$10^{12}$) vs "FLOPs" (x-axis, log scale, $10^{17}$–$10^{25}$).** Same style: grey dot scatter from about ($10^{17}$, 5×$10^8$) up to about ($2\times10^{21}$, $10^{11}$), red dashed power-law fit extended to $10^{25}$, and a teal annotation — horizontal line at y≈1.5T meeting a vertical line at Gopher's compute budget, labelled "1.5T" in small teal text just left of the vertical line.

Caption below the figure: "Figure 2 | **Training curve envelope.** On the **left** we show all of our different runs. We launched a range of model sizes going from 70M to 10B, each for four different cosine cycle lengths. From these curves, we extracted the envelope of minimal loss per FLOP, and we used these points to estimate the optimal model size (**center**) for a given compute budget and the optimal number of training tokens (**right**). In green, we show projections of optimal model size and training token count based on the number of FLOPs used to train *Gopher* ($5.76\times10^{23}$)."

## Slide 48 — Method 2 - IsoFLOPS

Heading (blue): "**Method 2 - IsoFLOPS**" Body text, two lines: "Pick a range of FLOP budgets, vary the total parameter count, take the min over these" / "convex shapes. The minima form a power law."

**Figure — three panels, reproduced from the Chinchilla paper (Figure 3).**

**Left panel — "Training Loss" (y-axis, linear, 2.0 to 3.2) vs "Parameters" (x-axis, log scale, 100M to 30B).** Nine series, one per FLOP budget, each a convex (U-shaped) curve of large dot markers (measured runs) with a thin dashed-line extension at both ends beyond the marker range. Legend, lightest to darkest (matching a light-teal-to-near-black colormap): 6e18, 1e19, 3e19, 6e19, 1e20, 3e20, 6e20, 1e21, 3e21. The lightest/topmost curve (6e18) sits highest on the loss axis with its minimum around 100-300M parameters; each successively darker/lower curve (larger FLOP budget) sits lower on the loss axis and its minimum shifts right to a larger parameter count, with the darkest/bottom curve (3e21) bottoming out somewhere in the few-billion-parameter range. Curves overlap toward their outer (dashed) ends, where a lower-FLOP-budget curve's rising right arm crosses over neighboring curves.

**Center panel — "Parameters" (y-axis, log scale, 100M–10B+) vs "FLOPs" (x-axis, log scale, $10^{17}$–$10^{25}$).** Black dot markers (one per IsoFLOP curve's minimum from the left panel) plus a red dashed power-law fit line through them, extended to higher FLOPs. A teal annotation reads "63B" at a horizontal/vertical crosshair positioned at Gopher's FLOP budget (matching the style of slide 47's center panel).

**Right panel — "Tokens" (y-axis, log scale) vs "FLOPs" (x-axis, log scale, $10^{17}$–$10^{25}$).** Same style: black dot markers at each IsoFLOP curve's minimum, red dashed fit line, and a teal annotation reading "1.4T" at the Gopher-budget crosshair.

Caption below the figure: "Figure 3 | **IsoFLOP curves.** For various model sizes, we choose the number of training tokens such that the final FLOPs is a constant. The cosine cycle length is set to match the target FLOP count. We find a clear valley in loss, meaning that for a given FLOP budget there is an optimal model to train (**left**). Using the location of these valleys, we project optimal model size and number of tokens for larger models (**center** and **right**). In green, we show the estimated number of parameters and tokens for an *optimal* model trained with the compute budget of *Gopher*."

## Slide 49 — Method 3 – Joint fits

Heading (blue): "**Method 3 – Joint fits**" Body text: "Run a bunch of models on the size-data grid. Use least squares to fit a joint scaling law"

**Figure — two panels, reproduced from the Chinchilla paper (Figure 4).**

**Left panel — "IsoLoss contours", titled above the plot.** X-axis "Training FLOPs", log scale, ticks $10^{18}$ through $10^{23}$, plus a labelled dashed vertical grey line at the right edge marked "Gopher budget". Y-axis "Model size", log scale, ticks 100M, 1B, 10B. The plot area is filled with many curved iso-loss contour lines, colored by a red-to-black "Loss" colormap (a colorbar at the right of the right panel runs from red/orange ≈4.00 down through magenta/purple to near-black ≈2.00; lower loss = darker). Overlaid are:
- A scatter of "Empirical data" points (round markers, outlined, filled with the same loss-colormap color as their measured loss), densely packed between about $6\times10^{18}$ and $3\times10^{21}$ FLOPs, 60M to several billion parameters.
- A solid blue line, "Efficient frontier" — a straight line in this log-log space, running from about ($2\times10^{18}$, 100M) up through roughly ($10^{22}$, 10B), passing through each iso-loss contour at its minimum-FLOPs point.
- A family of dashed vertical lines (legend entry "IsoFLOPs slice"), each colored to match its FLOP budget's curve in the right panel (light teal at low FLOPs through near-black at high FLOPs), positioned at $6\times10^{18}$, 1e19, 3e19, 6e19, 1e20, 3e20, 6e20, 1e21, 3e21 — matching the nine IsoFLOP budgets from slide 48.
- Two visually distinct hollow/open-outline data points near $10^{19}$ FLOPs (one near 9B parameters, one near 3B parameters) that stand out from the solid-filled empirical-data dots around them; the slide gives no caption explaining these specifically, so they are noted here as visually distinct without a stated meaning.

**Right panel — "IsoFLOPs slices", titled above the plot, with its own legend.** X-axis "Model size", log scale, ticks 100M, 1B, 10B, 40B. Y-axis "Loss", linear, ticks 2.00, 3.00, 4.00. Legend, nine dashed curves plus one additional dashed curve: "6e+18", "1e+19", "3e+19", "6e+19", "1e+20", "3e+20", "6e+20", "1e+21", "3e+21", and "Gopher" (all rendered as dashed lines in the same light-teal-to-black colormap as the left panel's vertical lines, with "Gopher" drawn as a thick black dashed line). Each series is a shallow parabola (dots = measured runs, dashed line = the parametric fit) with a visible minimum: the lightest/topmost series (6e+18) has its minimum around 200-300M parameters at loss ≈ 2.95-3.0; each successively darker series' minimum shifts right and down, with the near-black "3e+21" series bottoming out around 3-6B parameters at loss ≈ 2.15-2.2. The separate "Gopher" dashed curve sits below and to the right of all nine FLOP-budget series, its minimum around 10-20B parameters at loss just under 2.0 — it is a projection at Gopher's own (larger) compute budget rather than one of the nine measured IsoFLOP slices.

Caption below the figure: "Figure 4 | **Parametric fit.** We fit a parametric modelling of the loss $\hat{L}(N,D)$ and display contour (**left**) and isoFLOP slices (**right**). For each isoFLOP slice, we include a corresponding dashed line in the left plot. In the left plot, we show the efficient frontier in blue, which is a line in log-log space. Specifically, the curve goes through each iso-loss contour at the point with the fewest FLOPs. We project the optimal model size given the *Gopher* FLOP budget to be 40B parameters."

## Slide 50 — Why do we have this big difference?

Heading (blue): "**Why do we have this big difference?**" Bold line: "**Kaplan claims**: $N_{opt} = C^{0.73}$, $D_{opt} = C^{0.27}$ (tokens per param decreases w/ C)"

**Figure — identical to the chart on slide 45** (same log-log "Parameters" vs "FLOPs" plot, same series: solid blue/orange/teal lines "Approach 1"/"Approach 2"/"Approach 3", dashed black "Kaplan et al (2020)" line, and the four stars "Chinchilla (70B)" teal, "Gopher (280B)" orange, "GPT-3 (175B)" red, "Megatron-Turing NLG (530B)" purple, plus the same underlying blue-dot scatter). See slide 45 for the full structure-and-values description; this page repeats the same figure without the "Chinchilla … argue these fits are quite off" text box above it.

Below the figure, centered body text: "Why such a big difference (when both fit joint scaling laws?)"

## Slide 51 — Explanation 1

Heading (blue): "**Explanation 1**"

**Figure — five-panel diagram reproduced from a paper, "Resolving Discrepancies in Compute-Optimal Scaling of Language Models," plus a flow of three arrows.** In the top-right corner, a pasted image of the paper's own title block: "Resolving Discrepancies in Compute-Optimal Scaling of Language Models," with an author byline whose first name is cut off in the pasted image itself (the crop's left edge coincides with the title's own left edge) — legible as "…ell Wortsman†  Jenia Jitsev‡  Ludwig Schmidt†  Yair Carmon*".

Top row, left to right, three panels each plotting $N^*(C)$ (y-axis, log scale, roughly $10^6$–$10^9$) against "Compute $C$ [FLOPs]" (x-axis, log scale, $10^{17}$–$10^{19}$ visible), each with data points (with error bars), a colored power-law fit line, a black dash-dot "Hoffmann law" reference line, and a grey dotted "Kaplan law" reference line (per the shared legend defined below):
- **(a) "Reproducing Kaplan et al."** (purple title and fit line). Fitted values shown inside the panel: $a = 0.835\ (0.82, 0.85)$; $N^*(C_C) = 3T\ (3T, 4T)$.
- **(b) "Counting last layer FLOPs"** (blue title and fit line). $a = 0.706\ (0.69, 0.72)$; $N^*(C_C) = 787B\ (630B, 916B)$.
- **(c) "Correcting warmup"** (orange title and fit line). $a = 0.602\ (0.59, 0.62)$; $N^*(C_C) = 292B\ (249B, 355B)$.

Two thick black arrows lead down from panel (c): one to a panel labelled "**e**" (bottom-left position), one to a panel labelled "**d**" (bottom-right position) — the letter labels are not in left-to-right alphabetical order with their screen position.

Bottom row, two panels, same axis style ($N^*(C)$ vs "Compute $C$ [FLOPs]"):
- **Panel "e", titled "Optimizer tuning (no decay)"** (red title and fit line, positioned on the left). $a = 0.497\ (0.49, 0.50)$; $N^*(C_C) = 77B\ (70B, 86B)$.
- **Panel "d", titled "Cosine decay (no tuning)"** (green title and fit line, positioned on the right). $a = 0.571\ (0.56, 0.59)$; $N^*(C_C) = 183B\ (152B, 240B)$.

A shared legend (placed between the two bottom panels) lists: a grey shaded band = "95% confidence region"; a red dashed line = "Power law fit"; a black dash-dot line = "Hoffmann law"; a grey dotted line = "Kaplan law"; a dot with error bar = "Observations".

Below the figure, three bullets:
- Kaplan removed last layer param from the count
- Warmup at very small compute budgets was too high
- (Decay itself is maybe not critical if batch / LR is properly tuned)

## Slide 52 — Explanation 2

Heading (blue): "**Explanation 2**". A downward black arrow leads from the heading into a large rounded box. In the top-right corner, a pasted screenshot of a paper's title block: "**Reconciling Kaplan and Chinchilla Scaling Laws**", authors "Tim Pearce *Microsoft Research*" and "Jinyeop Song *MIT*", with a line "Reviewed on OpenReview: https://openreview.net/forum?id=NLoaLymURF".

**Left rounded box.** Text: "Start from fitted model of **Chinchilla's** training curves". Displayed equation:
$$\text{Loss}(N_T, D) = \frac{482}{N_T^{0.35}} + \frac{2085}{D^{0.37}} + 1.82$$
Below the equation, an arrow leads down to the text "Generate training curves for model sizes used in **Kaplan's** study (1k to 1.5B params)", and below that a small inset chart: "Loss" (y-axis, log scale) vs an unlabelled x-axis (D, log scale), showing roughly 19 overlapping curves, each one training-loss-vs-tokens curve for one model size, coloured on a purple-to-orange/yellow gradient (small models purple, large models yellow). A legend to the right of the curves lists each curve's $N_T$ value (e.g. "$N_T = 0.04M$" down to roughly "$N_T = 1090M$" or larger), but the legend text is small enough that it is not legible even at high magnification — this is a resolution limit of the pasted source image itself, not of the reading pass.

**Two arrows lead right from the left box**, one to a top pathway and one to a bottom pathway, each a two-panel chain:

**Top pathway.** First panel, titled "Find compute optimal frontier in terms of **total** parameters $N_T$ and compute as in **Chinchilla**": a chart of the same many-curve style (loss vs. $N_T$, coloured purple-to-orange by compute budget) with a legend entry "Compute efficient frontier" (small red squares) tracing the minimum-loss point of each curve; x-axis "Total compute, $C_T$" (log scale, roughly $10^{14}$–$10^{22}$). An arrow leads right to a second panel, titled "Find power law scaling coefficient of 0.51, close to Chinchilla's 0.50": a log-log scatter of "Total parameters, $N_T^*$" (y-axis) vs "Total compute, $C_T$" (x-axis), with the red "Compute efficient frontier" points and a pink/magenta "Local fit to frontier" line whose printed equation (small pasted-image text) is legible only as beginning "$\log(N_T^*) = 0.51\log(C_T) - ...$" — the intercept digits are not resolvable at any magnification. An annotation in the lower-right of the panel reads $N_T^* \propto C_T^{0.51}$.

**Bottom pathway.** First panel, titled "Find compute optimal frontier in **non-embedding** parameters $N_{\setminus E}$ and compute as in **Kaplan**": same many-curve style, red "Compute efficient frontier" dots. An arrow leads right to a second panel, titled "Find **local** power law scaling coefficient of 0.78, close to Kaplan's 0.73": log-log scatter of "Non-embed parameters, $N_{\setminus E}^*$" vs "Total compute, $C_T$", red frontier dots, pink fit line (equation likewise illegible at the pasted image's native resolution beyond the leading "$\log(N_{\setminus E}^*) = 0.78\log(C_{\setminus E}) - ...$"). Annotation: $N_{\setminus E}^* \propto C_{\setminus E}^{0.78}$.

**Far-right chart** (a single larger, clearly legible chart, positioned to the right of both pathways without its own connecting arrow — reproduced directly from the cited paper). X-axis $C_{\setminus E}$, log scale, ticks $10^{11}$ through $10^{23}$. Y-axis $N_{\setminus E}^*$, log scale, ticks $10^3$ through $10^{13}$. Legend:
- Red dot markers: "Compute efficient frontier"
- Black dashed line: "Analytical $N_{\setminus E}^*$ vs. $C_{\setminus E}$"
- Blue solid line: "Local fit to frontier: $\log(N_{\setminus E}^*) = 0.78\log(C_{\setminus E}) - 15.00$"
- Orange solid line: "Large $N_{\setminus E}$ regime fit: $\log(N_{\setminus E}^*) = 0.51\log(C_{\setminus E}) - 3.16$"

The red dot / black dashed frontier runs from about ($10^{11}$, $2\times10^2$) up to about ($5\times10^{20}$, $2\times10^9$), curving slightly. The blue "local fit" (slope 0.78) tracks the frontier closely at low-to-mid compute (up to about $10^{19}$) but overshoots it at high compute, reaching about $10^{12}$ by $2\times10^{24}$. The orange "large-regime fit" (slope 0.51) sits above the frontier at low compute (starting around $3\times10^2$ at $10^{11}$, already higher than the frontier there) but tracks the frontier closely at high compute, converging with the black/red frontier around $10^{20}$–$10^{21}$ and continuing to about $10^{10}$ by $2\times10^{24}$. So the true frontier's slope is close to Kaplan's steeper 0.78 exponent at smaller compute budgets and close to Chinchilla's shallower 0.51 exponent at larger compute budgets — a single curved frontier that looks like each paper's law only locally, in its own compute range.

Below the whole figure, body text: "Non-embedding vs total param choice + small nonlinearities"

## Slide 53 — Fun addendum – errors in chinchilla method 3

Heading (blue): "**Fun addendum – errors in chinchilla method 3**". Body text: "Note that this method three was likely flawed in the original paper. Some authors did data forensics, recovered the raw data, and re-did the fit and got results more consistent with methods 1 and 2"

**Figure — two panels.**

**Left panel — strip-plot-plus-half-violin of fit residuals.** Y-axis "Residuals", linear, ticks -0.10, -0.05, 0.00, 0.05, 0.10 (a few points sit above 0.10). X-axis: two categories, "Hoffmann et al." (green, left) and "Ours" (blue/navy, right). A black dashed horizontal reference line marks y = 0.00. For each category, individual translucent dots are jittered along a narrow vertical strip, with a half-violin (kernel density silhouette) drawn to its right in the matching colour.
- "Hoffmann et al." (green): dots span roughly -0.10 to +0.06, clearly clustered below zero; a short solid blue horizontal line marks the group's mean at about -0.047; the violin's widest point sits around -0.03 to -0.05.
- "Ours" (blue): dots cluster tightly between about -0.03 and +0.04 (with two individual outliers, one near +0.11 and one near -0.05); the violin is narrow and centred almost exactly on 0.

So Hoffmann et al.'s original method-3 fit has a systematic negative bias in its residuals, while the reworked ("Ours") fit is centred on zero.

**Right panel — line chart with confidence bands.** Y-axis "Optimal tokens per parameters ratio", log scale, ticks 10, 100. X-axis "Training compute (FLOP)", log scale, ticks $10^{19}$, $10^{21}$, $10^{23}$, $10^{25}$, $10^{27}$. Legend: navy/blue line "Optimal policy (ours)", green line "Optimal policy (Hoffmann et al.)", black dot "Chinchilla model". A grey dashed horizontal line, labelled "$D/N = 20$ rule of thumb", runs across the whole plot at y = 20.
- "Optimal policy (ours)" (blue): starts around 25-30 tokens/param at $10^{18}$-$10^{19}$ FLOPs, declines slowly, reaching roughly 15-17 by the right edge of the plot (beyond $10^{27}$). It is drawn with a widening shaded confidence band (dashed blue bounds): narrow (roughly ±5) at low compute, fanning out to span from under 10 to well over 100 by the highest compute shown.
- "Optimal policy (Hoffmann et al.)" (green): starts at about the same 20-25 tokens/param at low compute, but rises steadily and steeply, crossing the blue line around $10^{20}$ FLOPs and continuing up to roughly 200+ by $10^{27}$-$10^{28}$. Its confidence band (dashed green bounds) stays narrow throughout, hugging the solid green line.
- "Chinchilla model" (black dot): sits at approximately ($3$-$4\times10^{23}$, 20), right on the grey "D/N = 20" line.

The chart's point: the corrected ("ours") fit keeps the optimal tokens-per-parameter ratio roughly flat near 20 out to very large compute, consistent with the "D/N=20" rule of thumb and the Chinchilla model itself, whereas Hoffmann et al.'s original fit predicts a steadily rising ratio — though the corrected fit's own uncertainty band widens enormously at high compute, so this flatness is not tightly pinned down by the data.

Bottom-right corner: citation "[Besiroglu et al 2024]".

## Slide 54 — Important note – train-optimal is likely not what you want

Heading (blue): "**Important note – train-optimal is likely not what you want**"

Body text: "**Chinchilla** aims to tell you what gives the best model for fixed training compute.." / "But most of the compute in a real deployment is inference.. So we should 'over' train"

Bulleted list (tokens per parameter, for named models):
- **GPT3** – 2 tokens / param
- **Chinchilla** – 20 tokens / param
- **LLaMA65B** – 22 tokens / param
- **Llama 2 70B** – 29 tokens / param
- **Mistral 7B** – 110 tokens / param
- **Llama 3 70B** – 215 tokens / param
- … etc etc

Below the list, body text: "The more usage we expect, the more it becomes worth it to pay the upfront cost"

No chart or table on this page.

## Slide 55 — Isoflops everywhere

Heading (blue): "**Isoflops everywhere**". Body text: "Methods like IsoFLOPS are pretty easy to execute and usually pretty clean"

**Figure — four panels, two rows, reproduced from two different papers, plus a third small chart between the top and bottom rows.**

**Top-left panel — IsoFLOP profiles for autoregressive models (left half of a "Figure 5").** Y-axis (label cropped in the pasted image, values match "NLL(val)" as in the neighbouring panel), ticks 3.2 to 4.0. X-axis "Non-Embedding Parameters", log scale, ticks $10^7$, $10^8$. Legend, five series (each a shallow parabola of dot markers plus one larger star marking its minimum): blue "$1.0\times10^{16}$ FLOPs", green "$4.0\times10^{16}$ FLOPs", red "$1.6\times10^{17}$ FLOPs", purple "$6.4\times10^{17}$ FLOPs", gold/tan "$2.6\times10^{18}$ FLOPs". Minima, low-to-high FLOPs: blue ≈ 4M-6M params at loss ≈ 4.03; green ≈ 3×10^7 params at loss ≈ 3.79; red ≈ 6×10^7 params at loss ≈ 3.58; purple ≈ 1×10^8 params at loss ≈ 3.39; gold ≈ 2×10^8 params at loss ≈ 3.20. Each successive (larger-FLOP) curve sits lower on the loss axis and its minimum shifts right to more parameters.

**Top-middle panel — IsoFLOP profiles for diffusion models (right half of the same "Figure 5").** Y-axis "NLL (val)", ticks 3.6 to 4.6. X-axis "Non-Embedding Parameters", log scale, $10^6$ to just past $10^8$. Legend, five series in the same style: green "$4.0\times10^{16}$ FLOPs", red "$1.6\times10^{17}$ FLOPs", purple "$6.4\times10^{17}$ FLOPs", gold "$2.6\times10^{18}$ FLOPs", light-blue "$1.0\times10^{19}$ FLOPs". Minima: green ≈ 4×10^6 at loss ≈ 4.57; red ≈ 7×10^6 at loss ≈ 4.27; purple ≈ 1.5×10^7 at loss ≈ 4.01; gold ≈ 6×10^7 at loss ≈ 3.79; light-blue ≈ 1×10^8 (near the right edge of the plot) at loss ≈ 3.60.

Caption beneath these two panels: "Figure 5: IsoFLOP profiles for autoregressive models (left) and diffusion models (right)."

**Top-right panel — NLL(val) vs Non-Embedding FLOPs, scaling trend comparison.** Y-axis "NLL (val)", linear, ticks 3.0 to 5.0. X-axis "Non-Embedding FLOPs", log scale, ticks $10^{16}$, $10^{18}$, $10^{20}$. Legend: blue line "Diffusion", green line "Autoregressive". Underlying each trend line is a dense fan of thin, faint individual training curves (light blue for diffusion, light green for autoregressive) plus larger solid dots marking measured IsoFLOP-minimum points along the trend. Blue "Diffusion" dots run from about ($2\times10^{16}$, 4.57) through ($4\times10^{16}$, 4.27), ($8\times10^{17}$, 4.02), ($1.5\times10^{18}$, 3.79), to ($3\times10^{18}$, 3.58), and the solid blue trend line continues straight out to about ($10^{20}$, 3.22). Green "Autoregressive" dots run from about ($1.3\times10^{16}$, 4.02) through ($2.5\times10^{16}$, 3.79), ($5\times10^{16}$, 3.58), ($1\times10^{17}$, 3.39), to ($2\times10^{17}$, 3.21), with its trend line ending around ($3\times10^{17}$, 3.0) at the bottom of the plotted range. At every matched loss value, the autoregressive trend line sits at lower FLOPs than the diffusion trend line — i.e., autoregressive models reach the same loss with less compute than diffusion models do, over this range. Citation to the lower right of this panel: "Diffusion - Gulrajani+ 2023."

**Bottom row — two 3D surface plots, panels (a) and (b), reproduced from a second paper.**

**(a) "IsoFLOP surface over sparsity and total parameters."** Axes: x-axis (receding left) "Total Parameters $N$", ticks 26B, 16B, 10B, 6B, 4B, 2B, 1B, 800M, 485M, 294M; depth-axis (receding right) "MoE Sparsity $S$", ticks 0%, 39%, 63%, 78%, 86%, 91%, 95%, 98%; vertical z-axis "Loss $\mathcal{L}$", ticks 2.2 to 2.9. A grey 3D surface mesh forms a warped, saddle-like bowl with a visible ridge. Two small colour scales sit above the panel: one labelled "Sparsity $S$" (0% to 98%, dark-purple-to-orange gradient) and one labelled "Loss $\mathcal{L}$" (2.2 to 2.9). Several dashed line-and-dot series are traced across the surface, dot colour following the purple-to-orange sparsity/loss gradient, running from the low-parameter/high-sparsity corner up toward the high-parameter/low-sparsity corner, each tracing a shallow valley shape consistent with an IsoFLOP-style minimum extended into the added sparsity dimension.

**(b) "IsoFLOP surface over sparsity and active parameters."** Same style: x-axis "Active Parameters $N_{active}$", ticks 10B, 6B, 4B, 2B, 1B, 800M, 485M, 294M, 178M, 108M, 66M; depth-axis "MoE Sparsity $S$", 0% to 98%; z-axis "Loss $\mathcal{L}$", ticks 2.1 to 2.8; same two colour scales. Here the grey surface rises to a visibly sharper peak at the back-left corner (near 0% sparsity, low active parameters, loss ≈ 2.8) than panel (a)'s surface does. The same style of coloured dashed dot-series trace valleys across the surface.

Caption beneath the two 3D panels: "(a) IsoFLOP surface over sparsity and total parameters" / "(b) IsoFLOP surface over sparsity and active parameters." Citation to the lower right: "MoEs – Abnar+ 2025."

## Slide 56 — Scaling laws for models and compute

Heading (blue): "**Scaling laws for models and compute**"

Body text, centered: "Log-linearity extends to model parameters and compute!"

Two pale-blue bordered boxes, stacked:
- "**Lets us set the following based on small models**" / "- Pick optimizer" / "- Pick architecture and model sizes"
- "**Also lets us make smart resource tradeoffs**" / "- Big models vs more data?"

No chart or table on this page.

## Slide 57 — Recap: scaling laws – surprising and useful!

Heading (blue): "**Recap: scaling laws – surprising and useful!**"

A single pale-blue bordered box containing three bullets, each marked with a short dash, with generous vertical spacing:
- **Data scaling**: understand how data affects models, clean theory
- **Model scaling**: dramatically reduce costs for training
- **Scaling as prediction:** understand what problems can be 'brute forced'

No chart or table on this page.
