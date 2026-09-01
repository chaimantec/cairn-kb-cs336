---
title: Lecture 8 — Parallelism (Part 2) (course material)
lecture: 8
instructor: Tatsunori Hashimoto
source_format: slide-deck-pdf
source_file: lecture_08.pdf
source_repo: https://github.com/stanford-cs336/lectures
source_url: https://github.com/stanford-cs336/lectures/blob/main/lecture_08.pdf
pages: 73
method: page-images
numbering: >
  This deck prints NO page number on any page. The slide labels below are
  therefore PDF page numbers, not printed slide numbers: "Slide N" means "PDF
  page N", for N = 1..73, one heading per page, in order. Cite them as page
  numbers of lecture_08.pdf. The mapping was settled before any page was read.
  slide_number_map.py found no printed number anywhere, and unlike lectures 3, 4
  and 5 — each of which had at least one mid-page digit fall inside the scan
  region — a corner-position text-layer scan of this deck returned ZERO bare
  digit tokens across all 73 pages. All five readers additionally checked every
  page in their range by eye, at magnifications up to 4800 dpi, and none found a
  folio in any corner. One near-miss is worth naming so a later reader does not
  rediscover it as a finding: an isolated "10" sits just right of slide 56's
  table, and it is the 10^-2 y-tick of the adjacent chart, not a page number.
  Because the script's map is a 1..73 fallback rather than something read off the
  pages, its --verify mode degenerates into a heading-sequence check; that check
  passes (73 headings, 1..73, no gaps, no merges, no duplicates). Independently,
  72 of the 73 headings were matched verbatim against the PDF text layer at the
  top of their own page, the single exception being the title page, whose text
  layer letter-spaces "PA R A LLE LIS M BA SIC S".
figures: >
  86 raster images across 73 pages against roughly 2,749 words of native text —
  about 38 words per page — so, as in lectures 3, 4 and 5, the pasted figures ARE
  the content rather than an illustration of it. Only seven pages carry no image
  at all (3, 13, 14, 21, 28, 55, 72), and several of those are the deck's own
  dense native tables. Every figure below was described from the rendered page
  image, re-rendered and cropped at 600-4800 dpi wherever labels were small.
  Chart values on slides 56, 60, 61 and 62 were obtained by colour-masked pixel
  sampling against the detected axis ticks rather than by eye. Where something
  could not be resolved at any magnification the entry says so rather than
  guessing; there are two such places, both recorded under "Known limits" below.
audit: >
  Figure audit pass 1 covered eight pages — 12, 21, 30, 37, 55, 60, 67 and 72 —
  at least one from each of the five readers' ranges, chosen from the pages the
  readers themselves nominated as most chart- or table-dense. Six came back
  clean: page 12's two Huawei-vs-Nvidia sub-tables (15 rows, plus the Chinese
  photo annotations), page 21's DDP-vs-ZeRO-1 comparison table, page 55's 42-cell
  recap table including every red-highlighted cell, page 60's chart (all 12 data
  points within 1-3 teraFLOP/s), page 67's 72-cell Llama 3 failure table, and
  page 72's 40-cell overview table. Four errors were found and corrected, two
  each on pages 30 and 37; see the entries themselves. Three further errors were
  found in cross-slide COMMENTARY on pages 67 and 72 rather than in the pages'
  own content, and are also corrected. Every value, table cell and structural
  claim on the six clean pages was exact.
math: >
  Equations were transcribed from the rendered page, never from the text layer,
  which flattens fractions onto one line. This matters most for the
  activation-memory algebra on slides 46-49, where the whole argument for
  sequence parallelism turns on exactly which terms carry a 1/t divisor and which
  do not, and for the ZeRO per-rank memory expressions on slides 18-28. Both sets
  were checked for internal consistency: the ZeRO formulas reproduce their own
  printed GB values, and the activation-memory rows compose as the deck says they
  do (34 = 10 + 24).
---

# Lecture 8 — Parallelism (Part 2) (course material)

This is the written content of CS336 Lecture 8, transcribed page by page from
[`lecture_08.pdf`](https://github.com/stanford-cs336/lectures/blob/main/lecture_08.pdf)
(Tatsunori Hashimoto, Stanford CS336, Spring 2026). The deck's own title page
reads "PARALLELISM BASICS".

**Two lectures in this course are called "Parallelism".** Lecture 7 is Percy
Liang's executable lecture, which builds the collectives and the three cuts from
`torch.distributed` upwards. This one is the sequel: it takes those primitives as
given and asks what you actually do with them on a real cluster — how ZeRO and
FSDP shard optimizer state, gradients and parameters; why pipeline parallelism
has a bubble and how to shrink it; how tensor and sequence parallelism divide a
transformer block; where expert parallelism fits; and how real training runs
combine four or more of these at once. It is the half of the subject lecture 7
repeatedly defers to.

Headings are **PDF page numbers** — the deck prints no slide numbers. See the
`numbering` note in the front matter.

## Sections → slide ranges

| Section | Slides |
| --- | --- |
| Title, outline and organisation | 1–3 |
| Part 1 — why multi-GPU: the limits of one chip | 4–6 |
| Part 1 — collective communication refresher | 7–8 |
| Part 1 — network topologies: TPU mesh vs GPU tree, domain sizes | 9–12 |
| Part 1 recap | 13 |
| Part 2 — naïve data parallelism and its memory problem | 14–17 |
| Part 2 — ZeRO stages 1, 2 and 3 (FSDP) | 18–28 |
| Part 2 — where data parallelism runs out | 29–30 |
| Model parallelism — layer-wise and pipeline parallel | 31–38 |
| Model parallelism — tensor parallel | 39–43 |
| Activation memory and sequence parallelism | 44–49 |
| Expert parallelism and its composition with the rest | 50–53 |
| Other strategies, and the two comparison tables | 54–56 |
| 3D/4D parallelism: rules of thumb and the evidence for them | 57–62 |
| Part 3 — what recent language models actually do | 63–72 |
| Whole-lecture recap | 73 |

## Known limits of this transcription

Two things on these pages could not be fully resolved, and are recorded here
rather than guessed at:

- **Slide 44's activation-memory timeline** prints an eight-category legend
  (PARAMETER, OPTIMIZER_STATE, INPUT, TEMPORARY, ACTIVATION, GRADIENT,
  AUTOGRAD_DETAIL, Unknown), but only five of the eight are visually separable as
  distinct bands even at 2400 dpi. INPUT, TEMPORARY and AUTOGRAD_DETAIL could not
  be located as separate coloured regions.
- **Slide 26's FSDP timeline** (from arXiv:2304.11277) has adjacent boxes in the
  "GPU Comp. Stream" row that touch with no gap, so their printed labels run
  together visually at any magnification. This is a layout feature of the source
  image, not a resolution limit; the entry separates the labels as far as the
  image allows and flags the residual ambiguity.

One further item is a crop in the source rather than a reading failure: on
**slide 56** the last cell of the pasted TPU-book table is cut off at the right
edge of the pasted image, so the closing parenthesis of `(8BD/X + 8DF/Y)` is not
printed. The entry supplies it from symmetry and says that it did.

## The deck's own typos and self-contradictions

These are transcribed **as printed** and flagged inline where they occur. They
are collected here so a reader does not mistake a faithful transcription for a
transcription error.

- **"FDSP" for FSDP, three times** — slides 26, 35 and 63. Three independent
  readers found it in three separate ranges, so it is the deck's consistent
  misspelling rather than a one-off slip.
- **Slide 4**: "The word's fastest supercomputers" — "word's" for "world's".
- **Slide 22**: the bullet text calls the gradient shards "pink slices", but the
  diagram — and every legend on slides 18, 19, 23 and 24 — renders gradients
  orange.
- **Slide 37**: the interleaved pipeline schedule visibly uses four cell colours
  (two shades of blue, two of green) for two model chunks per device, while the
  figure's printed legend labels only two of them.
- **Slide 50**: "MoE Transfomer Encoder", twice, in the pasted GShard figure.
- **Slide 53**: "we cant".
- **Slide 58**: the title reads "Metagron's" for Megatron's.
- **Slide 59**: a note reads "Dara parallel" for Data parallel. Separately, the
  slide's table carries a DP-size column that is **not** present in the pasted
  source image — the deck added it.
- **Slide 60**: the deck's caption "More GPUS, same, flat utilization!" is true
  only of the two orange PTD-P series; the two blue ZeRO-3 series fall by roughly
  two-thirds across the same range. The pasted caption also calls the 175B lines
  "dotted"; they are drawn dashed.
- **Slide 61**: the deck's line says "across 64 machines" directly under a pasted
  caption reading "64 A100 GPUs"; the x-axis pairs multiply to 64 devices.
- **Slide 63**: the caption spells "FDSP" (see above).
- **Slide 67**: the Llama 3 GPU-failure table's 18 printed percentages sum to
  about 94.9%, not 100%, although the interruption counts do sum to 419. The gap
  is in the source table as pasted.
- **Slide 71**: the sub-heading reads "225B-A22B" while its own table names the
  model "Qwen3-235B-A22B" four times.
- **Slides 70 and 72 disagree with each other** about the same model: slide 70
  gives Nemotron 3 Super's pipeline-parallel degree as 0 in "TP / PP / CP / EP
  (2/0/64/64)", while slide 72's overview table records its PP as "??". The
  distinction between "not used" and "unknown" is exactly what that table is read
  for, so both are left as printed.

## Slides

## Slide 1 — Lecture 8: Parallelism Basics

The title slide. Centred on a white page, in black: "**Lecture 8**". Beneath it, in blue letter-spaced caps: "P A R A L L E L I S M   B A S I C S". Below that, in grey: "CS336" and "Tatsu H". A wide blue band runs across the bottom of the page. No figure. No printed page number visible anywhere on the page.

## Slide 2 — Outline and goals

A blue banner reads "Outline and goals". Below it, a pale-blue bordered box lists three bullets, each with generous vertical spacing between them:

- Understand the systems complexities of training huge models
- Different parallelization paradigms and why people use multiple at once
- What large scale training runs often look like

No figure on this page.

## Slide 3 — Organization today:

Heading: "Organization today:". Below it, three bullets marked with small blue diamond bullets, each with a bold "Part N" label followed by its description:

- ❖ **Part 1**: Basics of networking for LLMs
- ❖ **Part 2**: Different forms of parallel LLM training
- ❖ **Part 3**: Scaling and training big LMs with parallelism

No figure on this page.

## Slide 4 — Limits to GPU-based scaling – compute

Heading: "Limits to GPU-based scaling – compute". Two figures side by side, followed by two lines of body text below them.

**Figure 1 (left) — NVIDIA "Single-Chip Inference Performance – 1000X in 10 years" slide capture.** This is a pasted screenshot of an NVIDIA presentation slide (itself containing an inset photo of a presenter, apparently a smaller superimposed chart thumbnail near the middle of the frame). On the left side of the pasted slide is a bulleted list headed "Gains from," attributing the 1000x, ten-year improvement in single-chip inference performance to five sources, each with a bullet of contributing factors and an approximate multiplier:
- Number Representation: FP32, FP16, Int8, (TF32, BF16) — ~16x
- Complex Instructions: DP4, HMMA, IMMA — ~12.5x
- Process: 28nm, 16nm (green), 7nm (cyan), 5nm (purple/magenta) — ~2.5x
- Sparsity: ~2x
- Model efficiency has also improved – overall gain > 1000x

To the right of that bulleted list is a line chart titled "Single-Chip Inference Performance - 1000X in 10 years," y-axis "Int 8 TOPS" ticked 0.00 to 4500.00 in steps of 500, x-axis a series of dates from 4/1/12 to 3/15/23 (4/1/12, 8/14/13, 12/27/14, 5/10/16, 9/22/17, 2/4/19, 6/18/20, 10/31/21, 3/15/23). There is one dark-red line/series connecting labelled points, each annotated with a chip/technology name, a short feature label, and its Int8 TOPS value:
- Scalar FP32, K20X: 3.94
- M40: 6.84
- FP16 DP4A, P100: 21.20
- HMMA Tensor Cores, V100: 125.00
- IMMA Int8 Tensor Cores, Q8000: 261.00
- A100, Structured Sparsity: 1248.00
- H100, FP8 Transformer Eng: 4000.00

A small superimposed inset near the middle of the pasted slide shows a photo of a person presenting in front of a projected screen, with a small duplicate/zoomed thumbnail chart (showing gridlines at 500.00, 1000.00, 1500.00 and a point "Scalar FP32 K20X 3.94") layered behind/around the photo — this appears to be an artifact of the original presentation slide capture (a presenter photo overlapping a partial redraw of the same chart) rather than a distinct data series.

**Figure 2 (right) — "Projected Performance Development" log-scale scatter/line chart.** Y-axis is flop/s on a log scale, ticked 100 MFlop/s, 1 GFlop/s, 10 GFlop/s, 100 GFlop/s, 1 TFlop/s, 10 TFlop/s, 100 TFlop/s, 1 PFlop/s, 10 PFlop/s, 100 PFlop/s, 1 EFlop/s, 10 EFlop/s. X-axis is year, ticked 1990 to 2025 in steps of 5, with gridlines extending to roughly 2028. There are three data series (points plus a fitted trend line each), with no legend printed anywhere on this slide identifying them:
- Green circles (with a green trend line): the topmost series, rising from about 1 TFlop/s around 1994 to about 10 EFlop/s by roughly 2023-2024.
- Orange/tan triangles (with an orange trend line): the middle series, rising from about 100 GFlop/s around 1994 to about 1 EFlop/s by roughly 2022-2023.
- Dark-blue squares (with a blue trend line): the bottom series, rising from about 400-500 MFlop/s around 1993 to roughly 1-2 PFlop/s by around 2020, then visibly flattening/plateauing through 2024 rather than continuing to track its own trend line.

The chart carries no axis title distinguishing the three series (e.g., no "Sum/#1/#500" legend is printed on this page), so the entry above describes them purely by marker shape, colour, and position.

Below the two figures, two lines of body text: "There are limits to single-GPU scaling." and, indented, "The word's fastest supercomputers have *exaflops* of compute" (the deck's own text reads "word's," not "world's" — transcribed as printed; this is very likely a typo in the source).

## Slide 5 — Limits to GPU-based scaling - memory

Heading: "Limits to GPU-based scaling - memory". Body text below the heading: "Models are getting really big..".

**Figure — log-scale line chart of model size over time.** Y-axis "Model Size (in billions of parameters)," log-scaled, ticked 0.01, 0.1, 1, 10, 100, 1000. X-axis "year," ticked 2018, 2019, 2020, 2021, 2022 (unlabeled axis title, but the values are years). There is one blue-line series (dots connected by a solid blue line) tracing named models, each labelled with a leader line to its point, plus one dashed red reference/trend line with no series markers of its own. The blue series' labelled points, in order of increasing year:
- ELMo (94M) — near 2018, y ≈ 0.094
- BERT-Large (340M) — a bit later in 2018/2019, y ≈ 0.34
- GPT-2 (1.5B) — just before 2019, y ≈ 1.5
- Megatron-LM (8.3B) — mid-2019, y ≈ 8.3
- T5 (11B) — mid/late-2019, y ≈ 11
- Turing-NLG (17.2B) — early 2020, y ≈ 17.2
- GPT-3 (175B) — mid-2020, y ≈ 175
- Megatron-Turing NLG (530B), labelled in bold — late 2021/2022, y ≈ 530

The dashed red line is a straight trend/reference line on the log-scale plot, running from about (2018, 0.1) up to about (2022, >1000); it is not itself a labelled data series and carries no points of its own — it tracks below the blue series' final (Megatron-Turing) point, i.e., the actual model-size growth outpaces this reference line by the end of the plotted range.

Below the figure, body text: "A single GPU can't fit most of these large models!"

## Slide 6 — What do we do? Multi-GPU, multi-machine parallelism

Heading: "What do we do? Multi-GPU, multi-machine parallelism". Two text labels sit above the figure: "Intra-node parallelism via high-speed interconnects" (left) and "High-speed inter-node parallelism" (right).

**Figure — a labelled node/interconnect topology diagram (2-CPU, 8-GPU server with InfiniBand fabric).** At the very top, four interconnect-type legends with double-headed arrows giving a per-lane rate, left to right:
- "HDR InfiniBand" (purple) — 50 GT/s per lane
- "PCI Express 4.0" (black) — 16 GT/s per lane
- "xGMI-2" (dark blue) — 16 GT/s per lane
- "NVLink 3.0" (green, drawn as a plain line rather than a double arrow) — 400 GT/s per lane

Below that, two large purple boxes labelled "CPU₀" and "CPU₁" sit side by side, connected by a thick dark-blue double-headed arrow bundle labelled "16x" running between them (representing 16 lanes of the xGMI-2/inter-CPU link).

From each CPU, black double-headed arrows labelled "16x" drop down to two grey boxes labelled "PLX" (PCIe switches) — CPU₀ connects to two PLX boxes, CPU₁ connects to two PLX boxes (four PLX boxes total). Each CPU also connects directly (with a purple double-headed arrow labelled "4x", passing through a point labelled "Switch₀" or "Switch₁") down to an orange "HCA" box — CPU₀'s HCA is "HCA₀" (via Switch₀), and there is "HCA₁" reached from the PLX box next to CPU₀ (also linked to Switch₁ at 4x), "HCA₂" next to CPU₁ (via Switch₀ at 4x), and "HCA₃" at CPU₁'s far side (via Switch₁ at 4x). Each PLX box also has a 16x black double-headed arrow directly to its adjacent HCA box.

Each PLX box connects downward with black double-headed arrows labelled "16x" to two green "GPU" boxes below it. Reading left to right the eight GPU boxes are GPU₀, GPU₁, GPU₂, GPU₃, GPU₄, GPU₅, GPU₆, GPU₇ — four are under the CPU₀ side (fed by the two left PLX boxes) and four under the CPU₁ side (fed by the two right PLX boxes).

Below the row of eight GPUs is a dense mesh of thin green lines connecting every GPU to six light-green boxes labelled "NVSwitch₀" through "NVSwitch₅", each GPU–NVSwitch link annotated "2x" (labelled once, on the leftmost GPU₀–NVSwitch₀ link). This dense crossing pattern represents each GPU connecting to every NVSwitch (a full all-to-all NVSwitch fabric across all 8 GPUs).

Below the figure, body text: "Split up memory and compute requirements across GPUs and machines".

## Slide 7 — But first.. Some basics about collective communication

Heading: "But first.. Some basics about collective communication". The body of the slide is four labelled collective-communication diagrams (All reduce, Broadcast on the left column; Reduce, All Gather, Reduce Scatter on the right column), each showing per-rank input/output boxes before and after an arrow, plus the resulting formula beneath.

**Diagram 1 — "All reduce" (left, top).** Four input columns labelled "rank 0," "rank 1," "rank 2," "rank 3," each holding a colored box: rank 0 = blue "in0", rank 1 = red "in1", rank 2 = green "in2", rank 3 = yellow "in3". A grey arrow points right to four output columns (rank 0–3), each holding a white box labelled "out". Formula beneath: $out[i] = sum(inX[i])$.

**Diagram 2 — "Broadcast" (left, bottom).** Four rank columns; only "rank 2 (root)" holds a box, labelled "in" (white/unfilled). A grey arrow points right to four output columns (rank 0–3), each now holding a white box labelled "out". Formula beneath: $out[i] = in[i]$.

**Diagram 3 — "Reduce" (right, top).** Four input columns (rank 0–3) with colored boxes: rank 0 = blue "in0", rank 1 = red "in1", rank 2 = green "in2", rank 3 = yellow "in3". A grey arrow points right to four rank columns, but only "rank 2 (root)" receives an output box, labelled "out" (white). Formula beneath: $out[i] = sum(inX[i])$.

**Diagram 4 — "All Gather" (right, middle).** Four input columns (rank 0–3), each with a small colored box only partly filling its column: rank 0 = blue "in0" (small, top of column), rank 1 = red "in1" (small), rank 2 = green "in2" (small), rank 3 = yellow "in3" (small). A grey arrow points right to four output columns (rank 0–3), each now a full box stacked in four colored bands, top to bottom: blue "out" (=in0), red "out" (=in1), green "out" (=in2), yellow "out" (=in3) — i.e., every rank's output column contains all four inputs stacked in the same order. Formula beneath: $out[Y*count+i] = inY[i]$.

**Diagram 5 — "Reduce Scatter" (right, bottom).** Four input columns (rank 0–3), each a box divided into four equal horizontal segments outlined with dashed lines: rank 0 = blue box with segments "in0" (repeated/subdivided), rank 1 = red box "in1", rank 2 = green box "in2", rank 3 = yellow box "in3" (each column's box appears divided into four sub-pieces, one per destination rank). A grey arrow points right to four small output boxes, one per rank: rank 0 = "out0", rank 1 = "out1", rank 2 = "out2", rank 3 = "out3" (each progressively lower/offset in the layout). Formula beneath: $outY[i] = sum(inX[Y*count+i])$.

## Slide 8 — Important detail – all reduce vs reduce-scatter-gather.

Heading: "Important detail – all reduce vs reduce-scatter-gather." Body text below heading: "Reduce can be implemented as two steps: reduce-scatter and all-gather". Below that, three labelled panels under a shared "GPUs" bracket label, connected by a grey "=" sign between panels 1 and 2, and a grey "+" sign between panels 2 and 3.

**Panel 1 — "All Reduce."** Four columns of GPUs, each a single tall box shaded a progressively darker blue left to right, labelled A, B, C, D respectively (four separate GPU boxes, top row). A downward grey arrow leads to a second row of four dark-navy boxes, each labelled "A+B+C+D" — i.e., after all-reduce every GPU holds the full sum of A, B, C, and D.

**Panel 2 — "Reduce-Scatter."** Four GPU columns, each holding a stack of four smaller sub-boxes labelled with a letter+index: column 1 (pink shades) has A0, A1, A2, A3; column 2 (rose) has B0, B1, B2, B3; column 3 (magenta) has C0, C1, C2, C3; column 4 (dark maroon) has D0, D1, D2, D3. A downward grey arrow leads to four output columns, each with only one populated dark-maroon cell (the rest empty/white): column 1's populated cell (top) reads "A0+B0+C0+D0"; column 2's populated cell (second row) reads "A1+B1+C1+D1"; column 3's populated cell (third row) reads "A2+B2+C2+D2"; column 4's populated cell (fourth row) reads "A3+B3+C3+D3" — each GPU ends up holding the reduced sum for one shard/index only, and which row is populated shifts diagonally down by one per column.

**Panel 3 — "All-gather."** Four GPU columns, each initially holding just one populated cell out of four rows (light purple shades): column 1 has "A" populated in row 1; column 2 has "B" populated in row 2; column 3 has "C" populated in row 3; column 4 has "D" populated in row 4 (a diagonal pattern, one letter per column, each in a different row). A downward grey arrow leads to four output columns that are now identical: each has four stacked rows, "A" (top), "B", "C", "D" (bottom), all four columns matching — i.e., every GPU now holds the full concatenation A, B, C, D.

Below the three panels, body text: "Importantly, in the bandwidth-limited regime, this is the best you can do".

## Slide 9 — TPUs vs GPUs – design differences at the comm level

Heading: "TPUs vs GPUs – design differences at the comm level". Two labelled text callouts on the left: "**TPU networking** — toroidal mesh" (upper) and "**GPU networking** — All-to-all (up to 256)" (lower). Three figures occupy the right/main area of the slide.

**Figure 1 (top) — 4×4 toroidal mesh diagram.** A 4-row by 4-column grid of boxes labelled "Chip," with black double-headed horizontal and vertical arrows connecting orthogonally adjacent chips into a regular mesh. Large red curved arrows wrap around each row and each column at the edges (e.g., from the rightmost chip in a row curving back to the leftmost chip in the same row, and similarly top-to-bottom for each column), representing the wraparound/toroidal connections that close each row and column into a ring. The two leftmost chips in the top two rows are shaded orange while the rest of the grid is shaded grey — no caption explains the orange highlighting, so it is transcribed as printed without a stated meaning.

**Figure 2 (middle) — two side-by-side NVIDIA DGX SuperPOD network diagrams.** Left diagram, boxed in light blue, titled "DGX A100 256 SuperPOD": four light-blue boxes labelled "IB HDR spine switches" at top, connected by thin grey lines down to three light-blue boxes labelled "... IB HDR leaf switches ...", which connect down to three clusters of green server icons labelled "... 32 nodes (256 GPUs) ...". The connections from spine to leaf switches are sparse/partial lines (not fully all-to-all looking), consistent with a fat-tree-style hierarchical fabric. Right diagram, boxed in green, titled "DGX H100 256 SuperPOD": a row of dark-green boxes labelled "NVS" (with "..." indicating more, and a callout arrow labelled "New NVLink Switch" pointing at the rightmost NVS box), connected by thick green lines in a dense crossing (all-to-all-looking) pattern down to the same style of "... 32 nodes (256 GPUs) ..." server-icon clusters. Beneath both diagrams, centered text reads "Fully NVLink-connected / Massive bisection bandwidth" (this caption sits under the right, H100 diagram).

**Table (bottom) — A100 vs H100 SuperPod comparison.**

| | A100 SuperPod: Dense PFLOP/s | A100 SuperPod: Bisection [GB/s] | A100 SuperPod: Reduce [GB/s] | H100 SuperPod: Dense PFLOP/s | H100 SuperPod: Bisection [GB/s] | H100 SuperPod: Reduce [GB/s] | Speedup: Bisection | Speedup: Reduce |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 1 DGX / 8 GPUs | 2.5 | 2,400 | 150 | 16 | 3,600 | 450 | 1.5x | 3x |
| 32 DGXs / 256 GPUs | 80 | 6,400 | 100 | 512 | 57,600 | 450 | 9x | 4.5x |

## Slide 10 — Mesh vs tree vs other

Heading: "Mesh vs tree vs other". Sub-heading: "Why mesh? Why tree?" followed by two lines of body text: "Pro mesh – Cheaper to operate, can be made fast (and just do tensor parallel)" and "Pro tree/A2A – Better for less structured communications (expert parallel)".

**Figure — pasted screenshot of a Substack article excerpt.** A browser-style URL bar reads "taekim.substack.com/p/nvidias-bill-dally-and-googles-jeff". Below it, the article text (quoted verbatim as printed on the slide):

"Dally and Dean on Google TPUs versus Nvidia GPUs. GPUs are good for .."

"Mixture of Experts models and TPUs are good for local workloads"

"Dally: \"You can't say one is better than the other because it really has to do with what is the workload and what is the traffic pattern. If you have a very local workload, a direct connected network with relatively low radix like a 3D torus, is ideal because you can map your problem. So part of the problem is on this TPU and part of the problem is on this TPU and you have to go one hop to get your data over there. But then there are other cases where, for example, you have an MOE [mixture of experts] model and you've got a lot of experts, and they're scattered all over the place and now you're having to take many hops to get to a given expert. So it would be more efficient to do one hop up to a switch and one hop down to the expert that you want to be at\" \"Given a traffic pattern, you'd come up with an optimum network, but there's no network that is good for all traffic patterns.\""

"Dean: \"I totally agree. It depends on exactly what your workload is.\""

## Slide 11 — But then things change.. TPU8i/t

Heading: "But then things change.. TPU8i/t". Three figures span the slide, followed by two bolded lines of body text.

**Figure 1 (left) — two hierarchical fully-connected TPU diagrams.** The first (smaller, left) shows a graph of 8 chip icons arranged in a circle (each icon a small square TPU board with a chip pictogram), with every pair connected by a thin black line, forming a complete graph (all-to-all mesh) among the 8 nodes. Caption beneath: "Fully-connected 4 TPUs on each board (machine)" and, below that, "Fully connected 8 boards in each group (rack)" — i.e., the 8 nodes drawn each represent one board of 4 TPUs, and the 8 boards are fully connected within a rack/group. The second, larger diagram to its right shows a much bigger grid (roughly 6 rows × 6 columns) of the same small circular fully-connected-clusters icon repeated, tiling the whole grid — representing many groups tiled together. Caption beneath: "Fully-connected 36 groups in each pod (total 1152 chips)".

**Figure 2 (right) — "Virgo Network" / "Jupiter Network" data-center network diagram.** A large grey-bordered panel is divided into two labelled sub-sections, "Virgo Network" (left) and "Jupiter Network" (right), with a "Distributed Global WAN" grey box at the top right connected by an arrow up from the Jupiter Network section. Three numbered blue callout tags (1, 2, 3) point to parts of the diagram, each explained by a numbered legend entry on the far right of the figure:
- Tag "1" points at the Virgo Network's "2-layer switching" block, which shows three groups of blue square nodes each drawn with crossing (bowtie/X) connector lines between an upper and lower row within the group, plus dotted lines connecting the three groups horizontally. Legend 1: "2 layer switching, fully non-blocking."
- Tag "2" points at a set of blue lines running from the 2-layer switching block down to grey rectangular blocks labelled "Accelerator racks" (two groups of grey bars, connected to each other by a dotted line, i.e., two accelerator-rack rows are shown). Legend 2: "Resilient fabric with independent planes."
- Tag "3" is placed at the bottom-left corner of the whole panel, which itself is drawn as a stack of three overlapping/offset grey rectangles (a "stack of pages" effect) to represent repetition. Legend 3: "Expandable to multi-datacenter sites."

The Jupiter Network sub-section shows a white box labelled "Optical Circuit Switch (Apollo)" above three grey blocks labelled "Aggregation blocks," which connect down to a "Data center building" labelled row of grey bars (similar accelerator-rack-style bars), and up via an arrow into the "Distributed Global WAN" box at top right.

Below the two large figures, two bold lines of body text: "**TPU8i networking** – closer to tree-style topologies (maybe for MoEs?)" and "**TPU8t scale-out networking** – 'Virgo' with switched networks".

## Slide 12 — Why not connect everything?

Heading: "Why not connect everything?" Sub-heading text: "**Domain sizes** – why not connect everything?"

**Table (left) — pasted screenshot, "Key Capabilities - Huawei Ascend 910C Cloud Matrix 384 vs Nvidia GB200 NVL72."** The pasted image is itself a table with an orange title bar, then two stacked sub-tables under dark-blue section headers "Chip and Package Level" and "System Level," each with grey column-header rows.

Chip and Package Level sub-table:

| | Unit | GB200 | Ascend 910C | Huawei vs Nvidia |
| --- | --- | --- | --- | --- |
| BF16 dense TFLOPS | TFLOPS | 2,500 | 780 | 0.3x |
| HBM capacity | GB | 192 | 128 | 0.7x |
| HBM bandwidth | TB/s | 8.0 | 3.2 | 0.4x |
| Scale Up Bandwidth | Gb/s uni-di | 7,200 | 2,800 | 0.4x |
| Scale Out Bandwidth | Gb/s uni-di | 400 | 400 | 1.0x |

System Level sub-table:

| | Unit | Nvidia GB200 NVL72 | Cloud Matrix CM384 | Huawei vs Nvidia |
| --- | --- | --- | --- | --- |
| BF16 dense PFLOPS | PFLOPS | 180 | 300 | 1.7x |
| HBM capacity | TB | 13.8 | 49.2 | 3.6x |
| HBM bandwidth | TB/s | 576 | 1,229 | 2.1x |
| Scale Up Bandwidth | Gb/s uni-di | 518,400 | 1,075,200 | 2.1x |
| Scale Up Domain Size | GPUs | 72 | 384 | 5.3x |
| Scale Out Bandwidth | Gb/s uni-di | 28,800 | 153,600 | 5.3x |
| All-In System Power¹ | W | 145,000 | 599,821 | 4.1x |
| All-in Power per BF16 dense FLOP | W/TFLOP | 0.81 | 2.00 | 2.5x |
| All-in Power per memory bandwidth | W per TB/s | 251.7 | 488.1 | 1.9x |
| All-in Power per memory capacity | kW/TB | 10.5 | 12.2 | 1.2x |

Footnote printed in a black bar at the bottom of the pasted table image: "1. All-in System Power is total cluster power including scale-out networking, storage, etc." A translucent orange/blue "semianalysis" watermark logo (a stylized circuit-tree icon above the wordmark "semianalysis") is overlaid across the middle of the table image.

**Figure (right) — photograph of a data-center rack row, annotated with Chinese labels.** The photo shows an overhead/angled view of a long server-rack aisle with an overhead cable tray carrying many red/dark cable bundles down into a bright cyan-highlighted horizontal band, above racks of dark server chassis. The same orange "semianalysis" watermark logo appears overlaid on the left-center of the photo. Superimposed Chinese-language annotations (transcribed as printed, with an approximate English gloss in brackets) read:
- "3168根光纤" ["3168 optical fibers"] — labelling the cyan cable-tray band at the top.
- "超节点网络交换机" ["Supernode network switch"], with "6912个 400G光模块" ["6912 400G optical modules"] beneath it — labelling the central rack section.
- "昇腾服务器" ["Ascend server"] — appears twice, labelling the racks to the left and to the right of the central switch section.
- Along the bottom: "2.8Tbps卡间互联带宽" ["2.8 Tbps inter-card interconnect bandwidth"], "内存池化，统一编址" ["Memory pooling, unified addressing"], and "UB统一通信协议" ["UB unified communication protocol"].

Below both figures, a source citation: `https://newsletter.semianalysis.com/p/huawei-ai-cloudmatrix-384-chinas-answer-to-nvidia-gb200-nvl72` (confirmed against the text layer).

## Slide 13 — Part 1 recap

Heading: "Part 1 recap". Three bullets, each marked with a black diamond bullet (❖), with generous vertical spacing:

- ❖ New unit of compute – the datacenter
- ❖ What we want from multi-machine scaling:
  - ❖ Linear memory scaling (max model params scales with num gpus) [sub-bullet, blue diamond, grey text]
  - ❖ Linear compute scaling (model flops scale linearly with num gpus) [sub-bullet, blue diamond, grey text]
- ❖ Simple collective comms primitives

No figure on this page.

## Slide 14 — Part 2 – Standard LLM parallelization primitives

Heading: "Part 2 – Standard LLM parallelization primitives". Body text: "How do we parallelize LLMs? 3 important ideas". Below that, a bulleted outline with bold top-level items and grey/lighter sub-bullets:

- **Data parallelism**
  - Naïve data parallel
  - ZeRO levels 1-3
- **Model parallelism**
  - Pipeline parallel
  - Tensor parallel
- **Activation parallelism**
  - Sequence parallel

No figure on this page.

## Slide 15 — Naïve data parallelism

Heading: "Naïve data parallelism". Body text: "**Starting point** – imagine we are doing naïve SGD".

Displayed equation:
$$\theta_{t+1} = \theta_t - \eta \sum_{i=1}^{B} \nabla f(x_i)$$

Body text below the equation: "**Naive parallelism:** split the elements of B sized batch across M machines. Exchange gradients to synchronize".

Below that, a pale-blue bordered box headed "How does this do?" with three lines:
- "Compute scaling – each GPU gets B/M examples."
- "Communication overhead – transmits 2x # params every batch. OK if batches are big"
- "Memory scaling – none. Every GPU needs # params at least"

No other figure on this page.
## Slide 16 — What's wrong with naïve data parallel?

No printed page number seen (top-left, top-right, bottom-left, bottom-right all checked at 200 dpi).

A wide light-blue band runs across the top of the page. Below it, the title in blue: "**What's wrong with naïve data parallel?**"

Figure: four identical rounded-rectangle boxes in a row, each split into two stacked halves. The top half of each box is labelled "**Model Copy**" and contains a small neural-network icon: a column of small teal/blue dots on the left, a column of small orange dots in the middle, connected by lines, separated from a column of small grey/white dots on the right by a thin vertical maroon/red bar. The bottom half of each box is labelled "**GPU 0**", "**GPU 1**", "**GPU 2**", "**GPU 3**" respectively (left to right) and contains a small graphics-card icon. Below the four boxes, an icon of a stack of photos (captioned "**dataset**") sits centered beneath them, with four arrows fanning out and up from the dataset icon to each of the four GPU boxes — i.e., the one dataset is split and distributed out to all four GPUs, each of which holds a full copy of the model.

Text below the figure, centered:
"Memory seems like it'd be a problem – we copy the model parameters to each GPU."
"Let's take a closer look.."

## Slide 17 — What's wrong with naïve data parallelism? - Memory

Title (blue): "**What's wrong with naïve data parallelism? - Memory**"

Text: "**Our memory situation is actually *terrible*.**"

"Depending on our precision.."

A light-blue highlighted box, centered: "We need 5 copies of weights and 16 bytes per param!"

Bulleted list below:
- 2 bytes for FP/BF 16 model parameters
- 2 bytes for FP/BF 16 gradients
- 4 bytes for FP32 master weights (the thing you accumulate into in SGD)
- 4 (or 2) bytes for FP32/BF16 Adam first moment estimates
- 4 (or 2) bytes for FP32/BF16 Adam second moment estimates

A vertical bracket line to the right of the last three bullets (the FP32 master-weights line, the first-moment line, and the second-moment line) groups them under the label "**"Optimizer state"**" — the bracket does not extend up to cover the parameters or gradients bullets. Arithmetic check: 2 (params) + 2 (gradients) + 4 (master weights) + 4 (first moment) + 4 (second moment) = 16 bytes/param at full precision, consistent with the boxed claim; this also matches the "K=12" constant (4+4+4) used for optimizer-state bytes-per-param on the following slides.

## Slide 18 — ZeRO – solving the memory overhead issue of DP

Title (blue): "**ZeRO – solving the memory overhead issue of DP**"

Text: "**Core idea:** split up the expensive parts (state) and use the reduce-scatter equivalence."

Figure: a table-like diagram with four rows (Baseline, $P_{os}$, $P_{os+g}$, $P_{os+g+p}$) and, for each row, three example GPU columns — $gpu_0$, $gpu_i$, $gpu_{N-1}$ — with "…" ellipses between them standing in for the omitted intermediate GPUs. Each GPU column is drawn as a small stack of colored bars: **blue = Parameters, orange = Gradients, green = Optimizer States** (per the legend at the bottom). To the right of the diagram is a two-column mini-table: "Memory Consumed" (a symbolic formula) and a numeric value, computed using the constants shown in the corner: **K=12, Ψ=7.5B, N_d=64**.

Row by row:
- **Baseline**: every GPU column ($gpu_0$, $gpu_i$, $gpu_{N-1}$) shows a full-width thin blue bar (Parameters), a full-width thin orange bar (Gradients) just below it, and a full-width thick green block (Optimizer States) below that — i.e., every GPU holds a complete, unsharded copy of parameters, gradients, and optimizer state. Memory consumed: $(2+2+K)*\Psi$ = **120GB**. (Check: $(2+2+12)\times7.5=120$.)
- **$P_{os}$**: every GPU column still shows full-width thin blue (Parameters) and orange (Gradients) bars, unchanged from baseline, but the green Optimizer-States block has shrunk to a short, narrow sliver hanging beneath the orange bar at the left edge of the column — i.e., only optimizer state is sharded across the $N_d$ GPUs; parameters and gradients are still fully replicated. Memory consumed: $2\Psi + 2\Psi + \frac{K*\Psi}{N_d}$ = **31.4GB**. (Check: $15+15+12\times7.5/64=15+15+1.4=31.4$.)
- **$P_{os+g}$**: every GPU column still shows a full-width thin blue bar (Parameters), unchanged, but now both the orange (Gradients) and green (Optimizer States) bars have shrunk to narrow slivers stacked together beneath it at the left edge — i.e., gradients are now also sharded, on top of the already-sharded optimizer state; only parameters remain fully replicated. Memory consumed: $2\Psi + \frac{(2+K)*\Psi}{N_d}$ = **16.6GB**. (Check: $15 + 14\times7.5/64 = 15+1.64=16.64$.)
- **$P_{os+g+p}$**: every GPU column now shows all three — blue, orange, and green — reduced to narrow slivers stacked vertically in a small column (blue on top, orange in the middle, green at bottom), with no full-width bar remaining — i.e., parameters, gradients, and optimizer state are all sharded across the $N_d$ GPUs. Memory consumed: $\frac{(2+2+K)*\Psi}{N_d}$ = **1.9GB**. (Check: $16\times7.5/64=120/64=1.875\approx1.9$.)

Legend at bottom: blue swatch = "Parameters", orange swatch = "Gradients", green swatch = "Optimizer States".

## Slide 19 — ZeRO stage 1. optimizer state sharding

Title (blue): "**ZeRO stage 1. optimizer state sharding**"

Figure: the same diagram style as slide 18, but showing only the top two rows (Baseline and $P_{os}$), with the same three example columns ($gpu_0$, $gpu_i$, $gpu_{N-1}$), the same legend colors (blue = Parameters, orange = Gradients, green = Optimizer States), and the same memory-consumed formulas/values as slide 18: Baseline $(2+2+K)*\Psi$ = 120GB; $P_{os}$: $2\Psi + 2\Psi + \frac{K*\Psi}{N_d}$ = 31.4GB, using the same constants K=12, Ψ=7.5B, N_d=64. Visually: Baseline shows full-width blue, orange, and thick green bars on every column; $P_{os}$ shows the same full-width blue and orange bars but the green Optimizer-States block reduced to a short sliver at the left edge of each column.

Text below the figure:

"**High level idea:**"
- Split up the optimizer state (first + second moments) across GPUs
- Everyone has the parameters + gradients

"Each worker is responsible for updating a subset of params (corresponding to their slice)"

## Slide 20 — ZeRO stage 1. how it works

Title (blue): "**ZeRO stage 1. how it works**"

Four numbered steps, with two illustrative diagrams:

"**Step 1.** Everyone computes a full gradient on their subset of the batch"

"**Step 2.** ReduceScatter the gradients – incur #params communication cost"

Diagram for Step 2 (ReduceScatter): On the left, four labelled columns "rank 0", "rank 1", "rank 2", "rank 3", each containing one tall rectangle divided by dotted horizontal lines into three equal segments: rank 0's rectangle is solid blue and labelled "in0" in its middle segment; rank 1's is solid red/crimson and labelled "in1"; rank 2's is solid green and labelled "in2"; rank 3's is solid yellow/gold and labelled "in3". A grey arrow points right to four output columns, again "rank 0"–"rank 3". On the right side, each rank now holds only a single small white box, positioned at a different vertical height in a descending staircase: rank 0's box "out0" is highest, rank 1's "out1" is a bit lower, rank 2's "out2" lower still, and rank 3's "out3" is the lowest. Caption beneath the diagram: "outY[i] = sum(inX[Y*count+i])" — i.e., each rank's output segment is the elementwise sum, across all four ranks' input rectangles, of the segment corresponding to that rank's position.

"**Step 3.** Each machine updates their param using their gradient + state."

"**Step 4.** All Gather the parameters – incur #params communication cost"

Diagram for Step 4 (AllGather): On the left, four columns "rank 0"–"rank 3", each holding a single small colored box at a different height in the same descending staircase as Step 2's output: rank 0 holds a blue box "in0" (highest), rank 1 a red box "in1" (lower), rank 2 a green box "in2" (lower still), rank 3 a yellow box "in3" (lowest). A grey arrow points right to four output columns "rank 0"–"rank 3", each of which now holds an identical tall rectangle stacked, top to bottom, blue / red / green / yellow, each segment labelled "out" — i.e., every rank ends up with the full concatenation of all four ranks' pieces. Caption beneath the diagram: "out[Y*count+i] = inY[i]".

The four-color scheme (blue = rank 0, red = rank 1, green = rank 2, yellow = rank 3) is used consistently across both the ReduceScatter and AllGather diagrams on this page.

## Slide 21 — Comparing ZeRO stage 1 and naïve data parallel

No printed page number seen.

Title (blue): "**Comparing ZeRO stage 1 and naïve data parallel**"

Table (three columns, four rows including header):

| | Naïve DDP | ZeRO stage 1 |
| --- | --- | --- |
| Communication primitive | One all-reduce (gradients) | One reduce scatter (send gradients) + all gather (collect params) |
| Communication cost | 2* # params | 2* # params |
| Memory | (4+K) * #params | (4+K/Ngpu) * #params |

The header row ("Naïve DDP", "ZeRO stage 1") is in white text on a dark-blue band; the body rows alternate light-grey/light-blue-grey shading. The "Communication cost" row was checked carefully at 900 dpi: both cells read identically, "**2\* # params**" — i.e., the deck's claim is that ZeRO stage 1's reduce-scatter-plus-all-gather costs the same total communication volume as naïve DDP's single all-reduce (each is 2x the parameter count, matching the standard result that ring all-reduce ≈ reduce-scatter + all-gather in cost). The "Memory" row shows naïve DDP's per-GPU memory as $(4+K)\cdot\#\text{params}$ against ZeRO stage 1's $(4+K/N_{gpu})\cdot\#\text{params}$ — only the optimizer-state term $K$ is divided by the GPU count $N_{gpu}$; the $4$ (2 bytes params + 2 bytes gradients) is not divided, i.e. still fully replicated on every GPU.

Text below the table: "ZeRO stage 1 is *free* (in the bandwidth limited regime) memory wins"

## Slide 22 — ZeRO stage 2. the simple extension to gradient sharding

Title (blue): "**ZeRO stage 2. the simple extension to gradient sharding**"

Figure: a single diagram row, labelled "$P_{os+g}$" on the left, showing three example GPU columns ($gpu_0$/$gpu_i$/$gpu_{N-1}$-style, with "…" ellipses between them, matching the visual style of slides 18–19). Each column shows a full-width thin blue bar (Parameters) on top, with a short two-color sliver — a thin orange (Gradients) segment directly above a green (Optimizer States) segment — hanging beneath it at the left edge of the column; the green portion of the sliver is taller than the orange portion. To the right, the "Memory Consumed" formula $2\Psi + \frac{(2+K)*\Psi}{N_d}$ = **16.6GB** is given (identical formula/value to $P_{os+g}$ on slide 18).

Text: "Emboldened by our success, let's shard even more stuff"

"**High level idea**"
- Also keep the gradients (pink slices) sharded across the machines.
- Use the same (rough) tricks as stage 1.

"**Complexity** – we can never instantiate a full gradient vector, but each worker must compute a full gradient (since we're data parallel)"

Note: the bullet calls the gradient slices "pink," but the diagram (and the legend on slides 18/19/23) render the gradient color as orange. Transcribed as printed — this is the deck's own inconsistency, not a resolution artifact.

## Slide 23 — ZeRO stage 2. how it works

Title (blue): "**ZeRO stage 2. how it works**"

Four steps with two diagrams, differing from slide 20's stage-1 version in Step 1:

"**Step 1.** Everyone incrementally goes backward on the computation graph"
"    Step 1a. After computing a layer's gradients, immediately reduce to send this to the right worker"

Diagram for Step 1a (a plain **Reduce**, not a reduce-scatter): four labelled columns "rank 0"–"rank 3", each holding a full solid-colored rectangle: rank 0 blue "in0", rank 1 red "in1", rank 2 green "in2", rank 3 yellow "in3" (same layout as slide 20's Step 2 "before" side). A grey arrow points right to four output columns "rank 0", "rank 1", "rank 2 (root)", "rank 3" — only rank 2, labelled "(root)", holds an output box (a white rectangle labelled "out"); ranks 0, 1, and 3 show no output box at all. Caption beneath: "out[i] = sum(inX[i])" — i.e. the full elementwise sum of all four ranks' gradients is reduced onto the single "root" rank that owns that layer's parameter shard, unlike stage 1's reduce-scatter which spreads the summed pieces across all ranks.

"    Step 1b. Once gradients are not needed in the backward graph, immediately free it."
"**Step 2.** Each machine updates their param using their gradient + state."
"**Step 3.** All Gather the parameters."

Diagram for Step 3 (AllGather): identical in layout to slide 20's Step 4 diagram. Left side: four columns "rank 0"–"rank 3" each holding one small colored box at a descending staircase height — rank 0 blue "in0" (highest), rank 1 red "in1", rank 2 green "in2", rank 3 yellow "in3" (lowest). Grey arrow to four output columns "rank 0"–"rank 3", each now holding an identical full stack of four segments, top to bottom blue/red/green/yellow, each labelled "out". Caption: "out[Y*count+i] = inY[i]".

## Slide 24 — ZeRO stage 3 (aka FSDP) shard everything

Title (blue): "**ZeRO stage 3 (aka FSDP) shard *everything***"

Figure: a single diagram row labelled "$P_{os+g+p}$", with three example GPU columns ($gpu_0$/$gpu_i$/$gpu_{N-1}$-style, "…" between them). Each column now shows only a small stack of thin slivers — a thin blue (Parameters) segment on top, a thin orange (Gradients) segment below it, and a taller green (Optimizer States) segment at the bottom — with no full-width bar remaining at all (unlike every previous row on slides 18/19/22, where at least the parameters bar spanned the full column width). To the right: "Memory Consumed" formula $\frac{(2+2+K)*\Psi}{N_d}$ = **1.9GB** (same formula/value as $P_{os+g+p}$ on slide 18).

Legend below the figure: blue swatch = "Parameters", orange swatch = "Gradients", green swatch = "Optimizer States".

Text: "We've gotten almost everything for free so far.. lets try to solve *all* our memory issues"

"**High level idea**"
- Shard everything – incl parameters!
- Use the same 'incremental communication / computation' ideas
- Send and request parameters on demand while stepping through the compute graph.

"Is it possible to do this with low overhead?"

## Slide 25 — ZeRO stage 3 (aka FSDP) how it works (baby version)

Title (blue): "**ZeRO stage 3 (aka FSDP) how it works (baby version)**"

Figure: a two-row flowchart (this is the standard diagram from the PyTorch FSDP tutorial, cited by URL at the bottom of the page). Each row lays out, left to right, a forward-pass pipeline, a backward-pass pipeline, a collapsed placeholder for further layers, and a final update step; the two rows are visually identical in structure and are read as two GPU ranks (or two time-steps of the same rank) running the same sequence in parallel.

**Top row, forward side** (grey rounded panel labelled beneath it "FSDP instance 1: N layers"): four teal boxes chained left to right by dark-blue arrows — "LOAD-MODEL SHARD" → "ALL-GATHER" → "FORWARD (LOCAL)" → "FREE FULL WEIGHTS". A dotted arrow drops down into "LOAD-MODEL SHARD" from a caption above the panel: "Load shard From CPU if CPU Offloaded."

**Top row, backward side** (a second grey panel, also labelled "FSDP instance 1: N layers", to the right of the forward panel): "ALL-GATHER" → "BACKWARD (LOCAL)" → "REDUCE-SCATTER" → "FREE FULL WEIGHTS". A dotted arrow rises from "FREE FULL WEIGHTS" up to a caption above: "Offload grads to CPU if CPU offload is enabled." A dash-dot arrow then leads right into a pair of small, unlabeled teal squares connected by a dash-dot arrow (a collapsed stand-in for the remaining layers), captioned beneath "FSDP instance N: M layers," and then a solid arrow into a final box "UPDATE WEIGHTS (LOCAL)".

**Bottom row**: an exact mirror of the top row — same four-box forward panel ("LOAD-MODEL SHARD" → "ALL-GATHER" → "FORWARD (LOCAL)" → "FREE FULL WEIGHTS", labelled "FSDP instance 1: N layers", with its own "Load shard From CPU if CPU Offloaded" dotted-arrow caption below it this time), the same four-box backward panel ("ALL-GATHER" → "BACKWARD (LOCAL)" → "REDUCE-SCATTER" → "FREE FULL WEIGHTS", labelled "FSDP instance 1: N layers", with its own "Offload grads to CPU if CPU offload is enabled" caption below it), and the same collapsed "FSDP instance N: M layers" placeholder leading to "UPDATE WEIGHTS (LOCAL)".

**Cross-row connections.** Between the top and bottom rows sit three text captions on vertical dotted double-headed arrows that link each row's matching boxes: "GATHER WEIGHTS (ALL_GATHER)" sits below the forward-side "ALL-GATHER" boxes (dotted arrow running from the top row's ALL-GATHER down to the bottom row's ALL-GATHER), a second "GATHER WEIGHTS (ALL_GATHER)" sits below the backward-side "ALL-GATHER" boxes, and "SYNC GRADS (REDUCE_SCATTER)" sits below the "REDUCE-SCATTER" boxes — i.e. these captions name the collective communication (all-gather of weights, reduce-scatter of gradients) that keeps the two rows' shards synchronized.

**Data flow.** A dark grey/black rounded box labelled "Data" sits at the left, between the two rows. A solid teal line leaves "Data" horizontally and forks: one branch curves up into the top row's "FORWARD (LOCAL)" box, the other curves down into the bottom row's "FORWARD (LOCAL)" box — the same data feeds both rows' local forward computation. Separately, a long solid teal arc runs from near the top-left forward panel up and over the top of the diagram, arriving with a downward arrowhead into the top-right panel's "BACKWARD (LOCAL)" box; a mirror-image arc runs from near the bottom-left forward panel up into the bottom-right panel's "BACKWARD (LOCAL)" box — depicting the forward pass's local computation feeding into the corresponding backward computation.

Caption beneath the whole figure: "Communication cost – 2 all gather (#param), 1 reduce-scatter (#param)."

Source URL printed at the very bottom of the page: `https://pytorch.org/tutorials/intermediate/FSDP_tutorial.html`

## Slide 26 — Actual picture of how FDSP / ZeRO stage 3 works

Title (blue): "**Actual picture of how FDSP / ZeRO stage 3 works**" — note the title itself misspells FSDP as "**FDSP**"; the body text one line below spells it correctly ("Let's walk through a FSDP example…"). Transcribed as printed; this looks like a typo in the title only.

Text: "Let's walk through a FSDP example to see some important ideas"

"**Incremental computation / communication**"
- Parameters / gradients are requested / sent and then immediately freed

"**Overlapping communication and computation** $(W_1W_0 + W_2W_0)x = y$"
- The all-gathers happen all at once while forward happens, masking the comm cost.

Figure: a timeline diagram (reproduced from the FSDP paper, arXiv:2304.11277 — URL printed at bottom-right of the page: `https://arxiv.org/pdf/2304.11277.pdf`), with a color legend at top right: pink/salmon square = "All-Gather (AG)", light purple square = "Reduce-Scatter (RS)", light blue square = "Forward Comp. (FWD)", light green square = "Backward Comp. (BWD)", pale yellow square = "Parameter Free", and italic "$i$" = "FSDP Unit $i$" (i.e., the small numerals printed inside boxes are unit indices, not quantities).

The figure has three stacked horizontal rows, aligned so that later rows execute what an earlier row scheduled:

- **CPU** (top row): a strip of small adjoining boxes, split into a **Forward** region on the left and a **Backward** region on the right by a vertical dotted divider. In the Forward region, narrow pink (All-Gather) boxes alternate with wider blue (Forward Comp.) boxes; in the Backward region, narrow pink (All-Gather) boxes and narrow purple (Reduce-Scatter) boxes alternate with wider green (Backward Comp.) boxes. Each box carries a small unit-index numeral. Reading the printed numerals left to right: Forward region reads **0, 0, 1, 1, 0, 2, 2, 2**; Backward region (after the divider) reads **2, 2, 0, 1, 1, 1, 0, 0**. (The boxes abut with no visible gap, so which exact box border bounds which numeral could not be individuated with full confidence even at 2000+ dpi; the sequence of printed numerals itself is exact.) Dashed arrows in four colors (blue, dark red, green, purple) run down and to the right from boxes on this row to boxes on the two rows below, indicating that the CPU issues these operations ahead of when they actually execute on the GPU.
- **GPU Comp. Stream** (middle row): a sequence of boxes reading, left to right: a blue "**FWD0**" box, then (after a gap) a blue "**FWD1**" box immediately abutting a narrow yellow "**Parameter Free**" bar labelled "1", immediately abutting a blue "**FWD0**" box, immediately abutting a blue "**FWD2**" box, then a yellow bar labelled "2", then a green "**BWD2**" box, then a yellow bar labelled "2", immediately abutting a green "**BWD0**" box; then (after a gap) a green "**BWD1**" box abutting a yellow bar labelled "1" abutting a green "**BWD0**" box abutting a yellow bar labelled "0". The boxes touch with no gap between several of them, so the printed labels visually run together (e.g. "FWD1" against "1" against "FWD0" against "FWD2" reads as one continuous string on the page); the sequence above is the best-effort separation of that string at 1800–2400 dpi. Note the compute-stream sequence appears to repeat "FWD0" and "BWD0" a second time each, as printed — this is transcribed as printed rather than corrected.
- **GPU Comm. Stream** (bottom row): a sequence of wider boxes reading, left to right: pink "**AG0**", pink "**AG1**", pink "**AG2**", pink "**AG2**" (printed twice in a row), then a gap, then purple "**RS2**", pink "**AG1**", purple "**RS1**", purple "**RS0**".

Overall the figure illustrates the slide's two bullet points: the CPU issues all-gathers and frees ahead of time (top row), the actual forward/backward compute happens on a GPU compute stream (middle row), and the communication (all-gather / reduce-scatter) happens concurrently on a separate GPU communication stream (bottom row), so that, e.g., unit 1 and 2's all-gathers can be issued and executed while unit 0's forward is still computing — masking the communication latency behind compute.

Source URL at bottom right: `https://arxiv.org/pdf/2304.11277.pdf`

## Slide 27 — What's the point?

Title (blue): "**What's the point?**"

Text: "Distributed data parallel costs 2*# param communication"

"**What about ZeRO?**"

A light-blue highlighted box containing three bullets:
- **Zero stage 1** is 2*# param – it's free! – you might as well always do it
- **Zero stage 2** is 2*# param – it's (almost) free (ignoring overhead)
- **Zero stage 3** is 3*# param – 1.5x comm cost, but that's not bad! (ignoring latency..)

Text below the box: "This is also conceptually very simple – write a FSDP block wrapper."

## Slide 28 — ZeRO in practice: will it fit?

Title (blue): "**ZeRO in practice: will it fit?**"

Text: "Pure BF16 training (with Kahan summation), is viable and optimizer states are less beefy. Let's say BF16 for everything but the master weights – 12 bytes per param"

"On a 8X A100 80G.."

Table (three columns, five rows including header):

| | Max size (params) | Formula for B/param |
| --- | --- | --- |
| Baseline | 6.66.. B | 12 |
| Zero stage 1 | 16 B | 5 |
| Zero stage 2 | 24.62 B | 2 (param) + (10 (grad+state))/8) |
| Zero stage 3 | 53.33 B | 12/8 |

The header row is white text on a dark-blue band; body rows alternate light-grey/light-blue-grey shading, same visual style as the table on slide 21. Arithmetic checks out against an 80GB-per-GPU budget on 8 GPUs: baseline 80GB/12B/param = 6.67B params; stage 1's 5 bytes/param (2 replicated params + 2 replicated grads, both BF16, plus 8 bytes of BF16-heavy optimizer state ÷ 8 GPUs = 1) gives 80/5 = 16B; stage 2's "2 (param) + (10 (grad+state))/8)" = 2 + 1.25 = 3.25 bytes/param gives 80/3.25 ≈ 24.6B; stage 3's fully-sharded 12/8 = 1.5 bytes/param gives 80/1.5 ≈ 53.3B. All four printed "Max size" values match this arithmetic exactly.

## Slide 29 — Issues remain with data parallel – compute scaling

Title (blue): "**Issues remain with data parallel – compute scaling**"

Text: "With data parallel**, #machines < batch size** (and near this, comm overhead is high)"

"And there's diminishing returns to batch sizes"

Figure: a single line chart titled "Predicted Training Speed" (this is a reproduction of the standard OpenAI-style critical-batch-size / gradient-noise-scale plot). Axes are both log-scaled. Y-axis: "$\epsilon_{opt}(B)/\epsilon_{max}$", ranging from $10^{-2}$ to $10^0$. X-axis: "Batch Size / Noise Scale $(B/\mathcal{B})$", ranging from $10^{-2}$ to $10^2$.

There is **one data series**: a solid blue curve that rises from the bottom-left corner $(10^{-2}, 10^{-2})$, climbing steeply and roughly linearly (on the log-log axes) through about $(10^{-1}, 7\times10^{-2})$ and $(10^0, 5\times10^{-1})$, then bending over and flattening as it approaches $(10^1, 0.9)$ and continuing nearly flat out to $(10^2, \approx1.0)$.

A vertical grey dashed line marks $x=10^0$ (i.e. $B=\mathcal{B}$) — this is a reference line, not a second data series. Two text labels with leader lines point at the curve: "**Perfect scaling**" pointing at the steep, roughly-linear rising portion of the curve (left of the dashed line), and "**Ineffective scaling**" pointing at the flattened portion of the curve (right of the dashed line, around $x\approx10^{1}$–$10^{2}$) — both are annotations of the single curve, not additional series.

## Slide 30 — Issues remain with data parallel – models don't fit

Title (blue): "**Issues remain with data parallel – models don't fit**"

Text: "**Zero stages 1 and 2** don't let you scale memory"

"**Zero stage 3** is nice in principle, but *does not reduce activation memory*"

Figure: a line chart (reproduced from an external paper on large-scale training throughput). Y-axis: "Achieved teraFLOP/s per GPU", linear scale from 0 to 200 (gridlines at 0, 50, 100, 150, 200). X-axis: "Number of GPUs", linear scale with labelled ticks at 768, 1152, 1536, 1920 (the two solid-line series continue to a further, unlabeled data point past 1920).

There are **four data series**, distinguished by color (blue vs. orange) and by line style/marker (dashed-circle vs. solid-diamond for blue; dashed-triangle vs. solid-square for orange):

- **ZeRO-3, 175B** (dashed blue line, circle markers): three points, approximately (just left of 768, ≈143), (≈960, ≈88), (1536, ≈44) — a steep decline as GPU count grows.
- **ZeRO-3, 530B** (solid blue line, diamond markers): three points, approximately (just right of 768, ≈140), (≈1250, ≈97), (rightmost/unlabeled x beyond 1920, ≈48) — also declining, but staying above the 175B dashed-blue curve until they nearly meet by the last point. Note that the two 530B series (this one and PTD-P 530B) share their own set of x-positions, shifted right of the two 175B series: the 530B points sit at roughly 815 and 1250 where the 175B points sit just left of 768 and at roughly 960. Do not assume the four series share x-positions.
- **PTD-P, 175B** (dashed orange line, triangle markers): three points, approximately (just left of 768, ≈152), (≈960, ≈147), (1536, ≈140) — nearly flat, gently declining.
- **PTD-P, 530B** (solid orange line, square markers): three points, approximately (just right of 768, ≈172), (≈1250, ≈166), (rightmost/unlabeled x beyond 1920, ≈158) — nearly flat, the highest curve on the chart throughout.

Both orange (PTD-P) series stay well above both blue (ZeRO-3) series across the whole chart, and the blue (ZeRO-3) series decline much more steeply as GPU count increases — the chart's visual point being that ZeRO-3's per-GPU efficiency degrades faster with scale than PTD-P's (a pipeline/tensor-parallel scheme).

Text below the chart: "Better ways to split up the model is needed…"
## Slide 31 — Beyond data parallel – model parallelism

"Scaling up in memory (without changing batch size) with model parallelism"

Blue callout box, "What model parallelism is..":
- It splits up the parameters across GPUs (like zero3)..
- But communicate activations (while zero3 sends params).

"We cover three different types of parallelism"

1. Pipeline parallel
2. Tensor parallel (+ Sequence parallel)
3. Expert parallel

"These correspond to three different ways of cutting up the model."

No chart or table on this page.

## Slide 32 — Layer-wise parallel

**Figure — four-layer pipeline across four GPUs.** Four vertical rounded-rectangle boxes are drawn left to right, labelled "Layer 0" (pale blue), "Layer 1" (pale yellow), "Layer 2" (pale red/pink), "Layer 3" (pale orange). A red arrow labelled "forward" runs left-to-right above the chain, connecting Layer 0 → Layer 1 → Layer 2 → Layer 3. A blue arrow labelled "backward" runs right-to-left below the chain, connecting Layer 3 → Layer 2 → Layer 1 → Layer 0. Beneath each layer box sits a small GPU icon, labelled respectively "GPU 0", "GPU 1", "GPU 2", "GPU 3" — i.e., each layer (or subset of layers) is pinned to one GPU in sequence.

Below the figure: "**Layer-wise parallel** cuts up layers, assigns some subset to GPUS." / "**Activations and partial gradients** are passed back and forth"

## Slide 33 — What's wrong with layer-wise parallel

"Utilization of layer-wise parallelism is *terrible*.."

"With n gpus, each gpu is active $\frac{1}{n}$ of the time."

**Figure — single-microbatch pipeline schedule (staircase, one forward + one backward per GPU).** A schedule diagram with four rows of coloured cells, each row belonging to one GPU/stage, using only a single subscript ($F_0$, $B_0$) — i.e. this shows one micro-batch/no micro-batching, illustrating the naive layer-wise pipeline from the previous slide. Reading left to right (time axis, labelled "Time" with a rightward arrow beneath the diagram):

- Bottom row (gray): one cell, "$F_0$", starting at the leftmost time step.
- Next row up (tan/orange): one cell, "$F_0$", shifted one step to the right of the gray row's cell (starts after the gray row's cell ends).
- Next row up (blue): one cell, "$F_0$", shifted one more step right.
- Top row (mauve/pink): two adjacent cells, "$F_0$" then "$B_0$" — this is the last stage, so its backward starts immediately after its own forward finishes, with no gap.
- The schedule then steps back down: the blue row has a "$B_0$" cell (positioned to the right of the mauve row's cells, i.e. later in time), then the tan/orange row has a "$B_0$" cell (later still), then the gray row has a "$B_0$" cell (latest of all, at the far right before the update column).
- Far right: a vertical stack of four small "Update" boxes, one per row, coloured to match their row — mauve/pink (top), blue, tan/orange, gray (bottom) — drawn as one aligned column at the end of the timeline.

The overall shape is a symmetric staircase: ascending steps of solitary forward cells culminating in the top row's forward+backward pair, then descending steps of solitary backward cells, so each of the four GPUs is busy for only 2 of the roughly 8 time-slots spanned by the diagram (idle the rest of the time) — illustrating the "$1/n$ active" claim above.

"Each GPU is idling most of the time, waiting for the backward pass to propagate back"

## Slide 34 — A solution: pipeline parallel

**Figure — four-stage, four-microbatch pipeline schedule (the classic GPipe-style bubble diagram).** Four rows of coloured cells, one row per pipeline stage/GPU, each cell double-subscripted $\text{stage},\text{microbatch}$. Colours match the previous slide's per-row scheme: bottom row gray (stage 0), next tan/orange (stage 1), next blue (stage 2), top mauve/pink (stage 3). There are 4 micro-batches (indices 0–3) per stage. Reading left to right:

- **Row 0 (gray, bottom):** $F_{0,0}\ F_{0,1}\ F_{0,2}\ F_{0,3}$ contiguous, starting at time 0. Then a gap (idle/bubble) before $B_{0,3}\ B_{0,2}\ B_{0,1}\ B_{0,0}$, then an "Update" box (gray).
- **Row 1 (tan/orange):** starts one time-step later than row 0 (staircased right by one column): $F_{1,0}\ F_{1,1}\ F_{1,2}\ F_{1,3}$, then a shorter gap, then $B_{1,3}\ B_{1,2}\ B_{1,1}\ B_{1,0}$, then "Update" (tan/orange).
- **Row 2 (blue):** starts two time-steps later than row 0: $F_{2,0}\ F_{2,1}\ F_{2,2}\ F_{2,3}$, then a small gap, then $B_{2,3}\ B_{2,2}\ B_{2,1}\ B_{2,0}$, then "Update" (blue).
- **Row 3 (mauve/pink, top):** starts three time-steps later than row 0: $F_{3,0}\ F_{3,1}\ F_{3,2}\ F_{3,3}$ immediately followed by $B_{3,3}\ B_{3,2}\ B_{3,1}\ B_{3,0}$ with **no gap** (last stage's backward for the last microbatch starts right after its own forward finishes), then "Update" (mauve/pink).
- The forward cells across the four rows form an ascending staircase (row 0 leftmost/earliest start, row 3 rightmost/latest start), mirroring slide 33's shape but now each row carries 4 microbatches back-to-back instead of 1.
- A rounded-rectangle label box reading "**Bubble**" is placed in the middle of the diagram, in the empty gap region between the forward and backward blocks of the two bottom rows (gray and tan/orange), which have the largest idle gaps since they must wait longest for the backward pass to propagate down from stage 3.
- The backward cells across the four rows form a mirrored, descending staircase, and all four rows' "Update" boxes are drawn as a single aligned column at the far right of the figure (top-to-bottom: mauve/pink, blue, tan/orange, gray).

Blue callout box: "**Solution: Pipeline-parallel.**" / "Process 'micro-batches' (in this case, 4)." / "Send off the first microbatch and start computing the second."

"The ratio of bubble time to useful compute is .. $\dfrac{n_{stages}-1}{n_{micro}}$ so we need a big batch size!"

## Slide 35 — Why pipeline parallel?

"Pipelines seem terrible. Why do we do it?"

Blue callout box:
1. Pipelines save memory (compared to DDP)
2. Pipelines can have good communication properties (compared to FDSP) – it depends only on activations ($b \times s \times h$) and is *point to point*

(As printed: "FDSP" — almost certainly a typo for "FSDP", transcribed as printed rather than silently corrected.)

"Generally, we will use pipelines on slower network links (i.e. inter-node) as a way to get better memory-wise scaling."

No chart or table on this page.

## Slide 36 — Pipeline performance is highly dependent on batch size

**Chart — line plot, two series.** Y-axis: "Achieved teraFLOP/s per GPU", ticked 0, 50, 100, 150, 200. X-axis: "Pipeline-parallel size", ticked 1, 2, 4, 8 (categorical/log-like spacing, not linear). Two data series, both with marker-connected lines:

- **Batch size = 8** (blue, circle markers): ~163 at x=1, ~142 at x=2, ~121 at x=4, ~87 at x=8 — a steady, steep decline.
- **Batch size = 128** (orange, diamond markers): ~176 at x=1, ~168 at x=2, ~165 at x=4, ~160 at x=8 — nearly flat, only a slight decline.

"Batch sizes are key to hiding the bubble – otherwise pipeline rapidly degrades perf"

## Slide 37 — Trading communication bandwidth for utilization

**Figure — two stacked Gantt-style pipeline schedules (4 devices each), before/after "assign multiple stages to each device."** Both schedules share a legend at the bottom of the figure: dark blue = "Forward Pass", medium green = "Backward Pass" (gray cells = idle). A large downward arrow between the two schedules is labelled "Assign multiple stages to each device." A thin vertical black line partway across each schedule marks a step/iteration boundary; numbers resume past it rather than resetting to 1.

**Top schedule (before — one stage per device, i.e. standard 1F1B/GPipe-style layout).** Four rows, "Device 1"–"Device 4". Reading left to right: each device does 4 forward passes (blue, numbered 1–4) staggered so Device 1 starts first, Device 2 one step later, Device 3 two steps later, Device 4 three steps later (ascending staircase), separated by idle gray cells that grow with device depth. Backward cells (green) then appear staggered in the reverse order once the top-most stage's backward begins propagating back, interleaved with more forward passes of later microbatches (e.g. Device 1's sequence continues "... 1(green), 5(blue), 2(green), 6(blue), 3(green), 7(blue), 4(green), 8(blue), 5(green) ..."), i.e. steady-state 1-forward-1-backward alternation, with occasional single idle gray cells still visible between later backward cells for the earlier devices. After the vertical step-boundary line, the pattern repeats: Device 1 shows forward passes 9,10,11,12 (blue) immediately, then (after a short gray gap) backward passes 9, 10 begin appearing (green); Device 3 similarly shows forwards 9–12 then backward 9, then a forward labelled **13**, then backward 10, gray gap, backward 11 — i.e. the microbatch counter keeps incrementing past 12 across the boundary rather than resetting, consistent with a continuously-overlapping training loop rather than a hard per-iteration reset (the deck does not explain this explicitly).

**Bottom schedule (after — multiple, non-contiguous stages per device, the "interleaved" schedule).** Four rows, "Device 1"–"Device 4", but now **four** distinct cell colors appear rather than two: dark blue, a lighter/pale blue, a pale green, and a darker olive green — evidently forward and backward passes for two different model chunks assigned to the same device (e.g., Device 1's opening cells read 1,2,3,4 in dark blue, then 1,2,3,4 in pale blue — two equal-length forward sequences of four back to back — before returning to dark blue for 5,6). **This is a contradiction/gap in the deck**: the legend printed under the figure labels only two colors ("Forward Pass" = dark blue, "Backward Pass" = dark/olive green), but the bottom schedule itself clearly uses two shades of blue and two shades of green (four colors total) with no separate legend entries for the paler shades. Structurally, the bottom schedule has visibly fewer and shorter gray idle gaps than the top schedule, especially in the steady-state region — consistent with the slide's point that interleaving stages improves utilization — while the numbering pattern (verified at high zoom for Devices 1–2 immediately after the step-boundary line) shows each device cycling through its two assigned chunks, e.g. Device 1 reads dark-blue 9,10,11,12, then pale-blue 9,10,11,12, then a three-cell gray gap, then a single dark-blue 13, then alternating dark-blue/pale-green pairs numbered 13,9,14,10,15,11,16,12, and then a *separate* block of alternating pale-blue/olive-green pairs numbered 13,9,14,10,15,11 — i.e. forward and backward passes for the device's two chunks interleaved at fine grain, but as two sequential two-colour blocks rather than all four colours interleaving in one run.

"Some more crazy pipeline patterns can improve utilization, but at the cost of bandwidth"

## Slide 38 — 'Zero bubble' pipelining

"Split up backwards into two parts"
1. Backpropagating activations (z,x)
2. Computing weight gradients (W)

"The second part can be done whenever"

**Figure 1 — "Computation Graph for MLP" (left of page).** A boxed diagram split into a "Forward" column and a "Backward" region (further divided into "B" and "W" sub-columns), with dotted horizontal lines marking the layer boundary and dotted vertical lines separating Forward from Backward, and "$N\times$" / "$\times N$" annotations at left/right margins indicating the block repeats for $N$ layers. **Forward (column "F"):** input $x$ (arrow down) into an orange box "$Wx$" (with $W$ entering from the left), producing $z$ (arrow down) into a white box "$\sigma(z)$", producing $y$ (arrow down, exits at bottom). **Backward, sub-column "B":** entering from below is $\nabla_y L$ (arrow up) into a white box "$\frac{d\sigma(z)}{dz}\nabla_y L$", producing $\nabla_z L$; this splits — one branch (arrow up) feeds an orange box "$W^T \nabla_z L$" which outputs $\nabla_x L$ (exits at top) — this is the "B" sub-column (backpropagating activations). **Backward, sub-column "W":** the other branch of $\nabla_z L$ (arrow right) feeds an orange box "$\nabla_z L\, x^T$", which outputs $\nabla_W L$ (arrow right, exits at right) — this is the "W" sub-column (computing the weight gradient). Column labels along the bottom of the figure: "F", "B", "W". Caption: "Figure 1: Computation Graph for MLP."

**Figure 2 — "1F1B pipeline schedule" (right of page, upper).** Four rows, "Device 1"–"Device 4", 8 microbatches, three-color legend: blue = "Forward", orange/tan = "Backward", pale yellow = "Optimizer step". This is the standard staggered 1-forward-1-backward steady-state schedule: Device 1 does forward 1–4 (blue) immediately, then idles 3 steps, then alternates backward/forward (orange 1, blue 5, orange 2, blue 6, orange 3, blue 7, orange 4, blue 8), then finishes backward 5–8 (orange, with a single idle gray gap between each of the later ones), then a pale-yellow "Optimizer step" block, then the next iteration's forward 1–4 begins. Device 2–4 show the same shape shifted later by one, two, and three steps respectively (Device 4's forward and backward for a given microbatch are adjacent with no gap, since it is the last stage). Caption: "Figure 2: 1F1B pipeline schedule."

**Figure 3 — "Handcrafted pipeline schedules, top: ZB-H1; bottom: ZB-H2" (right of page, lower).** Four rows each, 8 microbatches, four-color legend this time matching the diagram exactly: blue = "F", teal/cyan = "B", green = "W", pale yellow = "Optimizer step".
- **Top (ZB-H1).** Device 1: forward 1–4 (blue), idle x3, then interleaved (teal $B_1$, green $W_1$, blue $F_5$, teal $B_2$, green $W_2$, blue $F_6$, teal $B_3$, green $W_3$, blue $F_7$, teal $B_4$, green $W_4$, blue $F_8$, teal $B_5$, green $W_5$, teal $B_6$, green $W_6$, teal $B_7$, green $W_7$, teal $B_8$, green $W_8$), then Optimizer step (pale yellow), then the next iteration's forward 1–4 begins. Device 4 (the last stage) instead does immediate forward/backward pairs with no gap ($F_1,B_1,F_2,B_2,F_3,B_3,F_4,B_4$) and only starts inserting weight-gradient ("W") cells afterward, filling in $W_1,F_5,B_5,W_2,F_6,B_6,W_3,F_7,B_7,W_4,F_8,B_8,W_5,W_6,W_7,W_8$ — i.e. it defers all its $W$ steps to where they would otherwise be idle bubble time. Devices 2–3 show the same shape staggered by one and two steps.
- **Bottom (ZB-H2).** Device 1 does a longer run of 7 forward passes (blue, $F_1$–$F_7$) before its first backward, then $B_1, W_1, F_8$ (its 8th and last forward), then $B_2,W_2,B_3,W_3,B_4,W_4,B_5,W_5,B_6,W_6,B_7,W_7,B_8,W_8$, then Optimizer step, then the next iteration repeats the same shape (7 forwards again). Device 4 (last stage) does immediate forward/backward pairs for all 7 early microbatches ($F_1,B_1,\dots,F_7,B_7$), then $W_1, F_8, B_8$, then all remaining weight-gradient cells $W_2$–$W_8$ back to back with essentially no idle gray cells visible in the steady state — a visibly smaller bubble than both Figure 2 and ZB-H1, consistent with the "zero bubble" framing, achieved by carrying more microbatches "in flight" at once (7 vs. ZB-H1/1F1B's 4) and deferring weight-gradient computation into the gaps.

## Slide 39 — Model parallel along the width axes

"Are there model parallel schemes with better utilization?"
"We can think of pipeline parallel as cutting up along depth. What about width?"

**Figure — worked $2\times4$ by $4\times2$ matrix-multiply, decomposed into submatrices.** Top row: matrix $X$ (pink/red cells, $2\times4$): row 0 = [0, 1, 2, 3], row 1 = [4, 5, 6, 7]; times matrix $A$ (purple cells, $4\times2$): rows top-to-bottom = [10, 14], [11, 15], [12, 16], [13, 17]; equals matrix $Y$ (green cells, $2\times2$): row 0 = [74, 98], row 1 = [258, 346]. (Verified: $X A = Y$ exactly, e.g. $0\cdot10+1\cdot11+2\cdot12+3\cdot13=74$.)

A downward arrow leads to the decomposed version below: $X$ is split into two $2\times2$ column-blocks $X_1$ = [[0,1],[4,5]] and $X_2$ = [[2,3],[6,7]]; $A$ is split into two $2\times2$ row-blocks $A_1$ = [[10,14],[11,15]] and $A_2$ = [[12,16],[13,17]]. $Y_1 = X_1 A_1$ = [[11,15],[95,131]] and $Y_2 = X_2 A_2$ = [[63,83],[163,215]] (both green). These are added ("+") to give $Y$ = [[74,98],[258,346]] (green), matching the top figure's $Y$ exactly. (Verified: $Y_1+Y_2=Y$ cell by cell.)

"Simple matrix multiplication observation: decompose into submatrices, add partial sums"

## Slide 40 — Tensor parallel – GPUs have submatrices

**Figure — Megatron-style tensor-parallel MLP block diagram, two dashed panels side by side.** Left panel, headed "$Y = \text{GeLU}(XA)$": input $X$ (tan box) → $f$ (dark green box) → splits into two parallel rows (one per GPU): top row $X$ (tan) → $XA_1$ (cyan) → GeLU (pink) → $Y_1$ (bright green); bottom row $X$ (tan) → $XA_2$ (cyan) → GeLU (pink) → $Y_2$ (bright green). Caption at the bottom of the panel: "$A = [A_1, A_2]$" (i.e., $A$ split by columns). Right panel, headed "$Z = \text{Dropout}(YB)$": top row $Y_1B_1$ (cyan) → $Z_1$ (bright green); bottom row $Y_2B_2$ (cyan) → $Z_2$ (bright green); both rows feed into $g$ (dark green box) → Dropout (pink) → $Z$ (bright green). Caption at the bottom: "$B = \begin{bmatrix}B_1\\B_2\end{bmatrix}$" (i.e., $B$ split by rows). Arrows throughout point left to right (forward-pass direction).

"Assign columns ($A_1$, $A_2$) and rows ($B_1$, $B_2$) to separate GPUs."
- In the forward pass, $f$ is the identity, and $g$ is an all-reduce.
- In the backward pass, $f$ is an all-reduce, $g$ is the identity.

## Slide 41 — Row vs Column tensor parallel

**Figure (a) — "MLP" (left half of page).** Identical to slide 40's diagram: left dashed panel "$Y = \text{GeLU}(XA)$" with $X \to f \to$ two parallel rows ($XA_1\to$GeLU$\to Y_1$; $XA_2\to$GeLU$\to Y_2$), caption "$A=[A_1,A_2]$"; right dashed panel "$Z=\text{Dropout}(YB)$" with two rows ($Y_1B_1\to Z_1$; $Y_2B_2\to Z_2$) feeding $g\to$Dropout$\to Z$, caption "$B=\begin{bmatrix}B_1\\B_2\end{bmatrix}$". Captioned beneath: "(a) MLP".

**Figure (b) — "Self-Attention" (right half of page).** Left dashed panel headed "$Y=\text{Self-Attention}(X)$": input $X$ (tan) $\to f$ (dark green) splits into two head-rows. Row 1 (head 1): $X$ (tan) branches to three cyan boxes $V_1$, $Q_1$, $K_1$ (top to bottom); $Q_1$ and $K_1$ feed a circled-× (matmul) node $\to$ Softmax (pink) $\to$ Dropout (pink) $\to$ a second circled-× node that also takes $V_1$ (routed in via a curved arrow from above) $\to$ output $Y_1$ (bright green). Row 2 (head 2): $X$ (tan) branches to $K_2$, $Q_2$, $V_2$ (top to bottom — note the order is mirrored relative to row 1) $\to$ same circled-×/Softmax/Dropout/circled-× chain, with $V_2$ routed in from below $\to$ output $Y_2$ (bright green). Below the panel: "split attention heads $\to$" followed by a brace giving $Q=[Q_1,Q_2]$, $K=[K_1,K_2]$, $V=[V_1,V_2]$. Right dashed panel headed "$Z=\text{Dropout}(YB)$": row 1 $Y_1B_1$ (cyan) $\to Z_1$ (green); row 2 $Y_2B_2$ (cyan) $\to Z_2$ (green); both feed $g$ (dark green) $\to$ Dropout (pink) $\to Z$ (bright green), caption "$B=\begin{bmatrix}B_1\\B_2\end{bmatrix}$". Captioned beneath: "(b) Self-Attention".

"How do we split up in tensor parallel across a transformer block?"
- Columnwise – QKV, up-projection
- Rowwise – Attention output, down-projection
- Replicated – norms, routers, etc

## Slide 42 — When do we tensor parallel?

"On GPUs, tensor parallel within a node (up to 8 GPUs) due to high speed interconnects."

**Chart — bar chart, one data series plus delta annotations.** Title printed on the chart itself: "Throughput Scaling with TP (3B Model)". Y-axis: "Tokens/sec/GPU", ticked 0, 5k, 10k (bars exceed the 10k gridline). X-axis: "Tensor Parallelism (TP)", with categorical positions 2, 4, 8, 16, 32. One bar series (teal/blue-green):

- TP=2: ≈13.7k tokens/sec/GPU (tallest bar, no annotation).
- TP=4: ≈12.3k, with a pink annotation "-10.8%" — this is the percentage drop from the TP=2 bar, not a second series (a vertical pink bracket/line connects the two bar tops, so it reads as a delta callout).
- TP=8: ≈10.8k, annotated "-12.2%" (drop from the TP=4 bar).
- TP=16: ≈6.2k, annotated "-42.7%" (drop from the TP=8 bar).
- TP=32: ≈2.1k, annotated "-65.6%" (drop from the TP=16 bar).

Each percentage is consistent with a bar-over-bar (not cumulative-from-baseline) decrease: e.g. $13.7\text{k}\times(1-0.108)\approx12.2\text{k}$, matching the TP=4 bar.

## Slide 43 — Tensor parallel – pros and cons vs pipeline parallel

"How do things compare to pipeline parallel?"

Blue callout box:

**Pros**
- no bubble. If your network is fast enough, there's no waiting for others.
- low complexity – simple to 'wrap' models without major infra changes
- doesn't need large batch sizes to work well

**Cons** – *much* larger communication than pipeline parallel.
- Pipeline: $bsh$ point-to-point communication per microbatch
- Tensor: $8bsh\left(\dfrac{n_{devices}-1}{n_{devices}}\right)$ *per layer* and *all-reduce* communication.

"Use tensor parallel whenever we have low-latency, high-bandwidth interconnects"

No chart or table on this page (aside from the boxed text above).

## Slide 44 — A final complexity – memory is dynamic!

"Memory isn't just the static bits, but also activations! This can be big"

**Chart — stacked memory-timeline / allocator snapshot, one series per memory category (8 legend entries).** Text printed above the plot: "Max memory allocated: 0.53 GB" / "Max memory reserved: 0.59 GB". Y-axis: "Memory (GB)", ticked 0.0, 0.1, 0.2, 0.3, 0.4 (the plotted curves reach just under 0.5 at their peaks, above the last labelled tick). X-axis: "Time (ms)", ticked 0, 50, 100, 150, 200, 250. Legend, top to bottom: PARAMETER (dark green), OPTIMIZER_STATE (tan/gold), INPUT (dark gray/near-black), TEMPORARY (light purple), ACTIVATION (red), GRADIENT (blue), AUTOGRAD_DETAIL (light blue), Unknown (gray, rendered as dense vertical hatching rather than a smooth fill).

This is a stacked-area "memory over time" trace (a GPU memory-allocator snapshot, not a conventional multi-line chart), showing roughly 6 repeating sawtooth cycles (training steps) across the ~270 ms window:

- **PARAMETER** (dark green): a flat, constant band from $y=0$ up to about $0.095$ GB for the entire time range — parameter memory never changes.
- **OPTIMIZER_STATE** (tan/gold): absent (zero width) from $t=0$ to about $t=50$ ms, then jumps in and forms a flat plateau from about $0.095$ to $0.19$ GB for the rest of the trace ($t\approx55$–$270$ ms) — consistent with optimizer state being allocated only after the first optimizer step.
- **ACTIVATION** (red): a series of rounded arch/hump shapes sitting on top of the green/tan baseline, rising and falling once per training step — a smaller hump from about $t=0$–$50$ ms peaking around $0.18$ GB, then five larger, roughly evenly-spaced humps between $t\approx55$ and $t\approx270$ ms peaking around $0.28$ GB each, falling back to baseline at the end of each hump (forward accumulates activations, backward frees them).
- **GRADIENT** (blue): sits above the red humps in each cycle, also arch-shaped, peaking around $0.34$ GB near the end of each hump (as activations are freed during backward, gradients accumulate), then dropping sharply back down to the OPTIMIZER_STATE/PARAMETER baseline at the start of the next cycle (consistent with gradients being cleared after each optimizer step).
- **Unknown** (gray hatching): sits above the blue/red region in every cycle, following the same up-down cyclical shape but rendered as dense, jagged, thin vertical bars rather than a smooth curve, peaking around $0.47$ GB at each cycle's high point.
- **INPUT** (dark gray/black), **TEMPORARY** (light purple), and **AUTOGRAD_DETAIL** (light blue) are listed in the legend but **not visibly distinguishable as separate bands in the plot at this resolution** — re-rendered at 2400 dpi, no separately-colored region matching these three swatches could be identified; they may be too thin to render as visible bands, or are present only as a sliver blended into an adjacent band. Flagged rather than guessed.

## Slide 45 — A final complexity – activation memory

"Thus far, we have only really discussed parameter memory."

"Tensor and pipeline parallel can linearly reduce those.. **but what about activations?**"

**Chart — grouped/stacked bar chart, two data series plus one reference line.** Y-axis: "Memory (GB)", ticked 0, 40, 80, 120, 160. X-axis: four model-size groups — "22B", "175B", "530B", "1T" — each with two bars, "baseline" and "present work". A red dashed horizontal reference line is drawn at $y=80$ GB (not a data series). Two stacked series, per the legend: "parameters and optimizer state memory" (blue, bottom segment of each bar) and "activation memory" (green, top segment):

- **22B / baseline:** blue ≈46 GB, total bar height ≈105 GB (activation segment ≈59 GB).
- **22B / present work:** blue ≈46 GB, total ≈56 GB (activation ≈10 GB).
- **175B / baseline:** blue ≈45 GB, total ≈112 GB (activation ≈67 GB).
- **175B / present work:** blue ≈45 GB, total ≈58 GB (activation ≈13 GB).
- **530B / baseline:** blue ≈32 GB, total ≈145 GB (activation ≈113 GB).
- **530B / present work:** blue ≈32 GB, total ≈54 GB (activation ≈22 GB).
- **1T / baseline:** blue ≈34 GB, total ≈163 GB (activation ≈129 GB).
- **1T / present work:** blue ≈34 GB, total ≈59 GB (activation ≈25 GB).

All values are approximate (read off bar heights against the gridlines). The pattern: parameter+optimizer memory (blue) is roughly constant within a model size regardless of "baseline" vs "present work," while activation memory (green) is dramatically larger under "baseline" and shrinks by roughly 5–10× under "present work," and grows with model size in the baseline case but stays roughly flat under "present work."

Citation, bottom right of the slide: "[Korthikanti et al 2022]"
## Slide 46 — What's the activation memory per layer?

Sub-heading: "**Starting point:** activation memory needed if storing everything".

The formula, printed alone inside a thin grey box across the middle of the page:

$$\text{Activations memory per layer} = sbh\left(34 + 5\frac{as}{h}\right).$$

Bullets below the box:

- The $5\frac{as}{h}$ terms come from the quadratic attention terms incl dropout
- As with flash attention, we can drop this term via recomputation

A two-column symbol legend is set in small type at the bottom right of the page.
It recurs on the following slides in this stretch, and is the notation the whole
activation-memory argument uses:

| Symbol | Meaning | Symbol | Meaning |
| --- | --- | --- | --- |
| $a$ | number of attention heads | $p$ | pipeline parallel size |
| $b$ | microbatch size | $s$ | sequence length |
| $h$ | hidden dimension size | $t$ | tensor parallel size |
| $L$ | number of transformer layers | $v$ | vocabulary size |

No figure on this page beyond the boxed equation and the legend.

## Slide 47 — Activation under tensor parallel

The displayed formula, printed at the top of the page (not boxed; a short
horizontal rule is drawn under the words "Activations memory" only, as an
underline-style emphasis):

$$\text{Activations memory per layer} = sbh\left(10 + \frac{24}{t} + 5\frac{as}{ht}\right)$$

Text below:

- "**Tensor parallel** splits out the matrix multiplies in attention + MLP"
- "The remaining **10** term is for the LayerNorm (4sbh), Dropout (2sbh), and inputs to the attention and MLP (4sbh). These terms alone will continue to grow with size"

So of the $34$ of slide 46, $24$ becomes $24/t$ and the remaining $10$ does
**not** divide by $t$; the attention term goes from $5as/h$ to $5as/(ht)$. The
$10$ is the term the rest of the argument is about — it is set in bold in the
deck's own prose, and it is the only term that survives tensor parallelism
undivided.

The same $a$/$b$/$h$/$L$/$p$/$s$/$t$/$v$ legend from slide 46 is repeated at the
bottom right. No other figure.

## Slide 48 — Making memory truly linear – sequence parallel

**Figure — the Megatron sequence-parallel transformer-layer diagram**, a wide
left-to-right pipeline running the full width of the page. Reading left to right:

- A bright-green vertical box labelled "Transformer Layer Input".
- A dashed rounded region labelled "**Sequence Parallel**" containing a cream/tan
  box "LayerNorm".
- A dark-green narrow bar labelled "$g$".
- A dashed rounded region with a grey gradient fill labelled "**Tensor Parallel**"
  containing a blue-grey box "Self Attention" and a cyan box "Linear".
- A dark-green narrow bar labelled "$\bar{g}$".
- A dashed rounded region labelled "**Sequence Parallel**" containing a pink box
  "Dropout", then a yellow circled-plus ($\oplus$) residual-add node.
- A second dashed "**Sequence Parallel**" region containing a cream/tan box
  "LayerNorm".
- A dark-green "$g$" bar.
- A second grey "**Tensor Parallel**" region containing cyan "Linear", pink
  "GeLU", cyan "Linear".
- A dark-green "$\bar{g}$" bar.
- A final dashed "**Sequence Parallel**" region containing pink "Dropout" and a
  second yellow $\oplus$ node.
- A bright-green box "Transformer Layer Output".

Two long white residual-skip arrows arch over the top of the diagram: the first
from the transformer-layer input (before the first LayerNorm) down into the first
$\oplus$; the second from that $\oplus$ (before the second LayerNorm) down into
the second $\oplus$. Small white block arrows connect each box to the next along
the main path.

Text below the figure:

- "**Observation:** all the 10sbh terms are pointwise ops over the sequence"
- (indented continuation) "… so split up the layer norm/dropout layers along the sequence axis."
- "In the forward pass, '$g$' is an all gather, '$\bar{g}$' is reduce-scatter"
- "In the backward pass, the two are reversed."

## Slide 49 — Making activation memory fully scale with more machines

Sub-heading: "Putting it together to get full linear scaling for memory."

The page is a single five-row table (a pasted LaTeX booktabs table, no colour):

| Configuration | Activations Memory Per Transformer Layer |
| --- | --- |
| no parallelism | $sbh\left(34 + 5\frac{as}{h}\right)$ |
| tensor parallel (baseline) | $sbh\left(10 + \frac{24}{t} + 5\frac{as}{ht}\right)$ |
| tensor + sequence parallel | $sbh\left(\frac{34}{t} + 5\frac{as}{ht}\right)$ |
| tensor parallel +<br>selective activation recomputation | $sbh\left(10 + \frac{24}{t}\right)$ |
| tensor parallel + sequence parallel +<br>selective activation recomputation | $sbh\left(\frac{34}{t}\right)$ |

(The last two Configuration cells are set on two printed lines each, as shown by
the line break above.) The bottom row is the punchline of the whole stretch: with
tensor + sequence parallel + selective activation recomputation, every term
divides by $t$ and activation memory per layer is $sbh \cdot 34/t$ — fully linear
in the number of machines. Note the pattern across rows: the $10$ (LayerNorm,
Dropout, and attention/MLP inputs) is undivided in the two "tensor parallel"
rows and is folded into $34/t$ in the two "sequence parallel" rows; the $5as/h$
attention term becomes $5as/(ht)$ under tensor parallel and disappears entirely
under selective activation recomputation.

No figure on this page other than the table.

## Slide 50 — Expert parallelism

Sub-heading: "Instead of splitting up the matmul, split up the experts and route activations"

**Figure — the GShard/MoE device-placement diagram**, two panels separated by a
vertical dotted line, with a grey block arrow at the top left and a second grey
block arrow pointing right across the divider.

*Left panel — titled "MoE Transfomer Encoder"* (printed exactly so, with
"Transfomer" missing its second `r`; this is a typo in the source figure,
transcribed as printed). A single grey rounded column, read bottom to top:

- "Input embeddings + Positional embeddings" (white box, below the column)
- Orange "Multi-Head Attention" → yellow "Add & Norm" (with a black residual arrow curving around it)
- A **red-outlined** box labelled "MoE" containing two pink boxes "FFN$_1$" … "FFN$_E$" (with an ellipsis between them) and a green box "Gating" beneath them
- Yellow "Add & Norm"
- Orange "Multi-Head Attention" → yellow "Add & Norm"
- Blue "Feed Forward FFN" → yellow "Add & Norm"
- "Encoder output" (white box, above the column)

A marginal annotation "(N/2)x" sits to the right of the column, marking that the
two-block group repeats $N/2$ times.

*Right panel — titled "MoE Transfomer Encoder with device placement"* (same typo).
Two grey rounded columns, "Device 1" on the left and "Device E" on the right, with
a horizontal ellipsis "• • •" and the caption "Devices 1…E" between them. Each
column has the same stack as the left panel, but:

- The bottom box reads "Input embeddings + Positional embeddings (shard 1)" and "(shard E)" respectively; the top boxes read "Encoder output (shard 1)" and "Encoder output (shard E)".
- The MoE block is now a single **red-outlined rectangle spanning both devices**, labelled in its centre "Model-parallel MoE". Device 1 holds one pink "FFN$_1$"; device E holds one pink "FFN$_E$". Each device has its own green "Gating" box.
- Two white ellipse-shaped nodes sit between the devices inside the red region. The lower one, "**All-to-All Dispatch**", takes crossing arrows from each device's Gating box up into the *other* device's FFN as well as its own — i.e. tokens are routed across devices. The upper one, "**All-to-All Combine**", takes crossing arrows from each device's FFN up into each device's "Add & Norm" — i.e. expert outputs are routed back.
- "(N/2)x" is again annotated at the right of each device column.

No text on this page other than the title, the sub-heading, and the figure's own
labels. No citation line is printed on the page.

## Slide 51 — Why EP?

Line under the title: "EP is *roughly* like TP in behavior for MLPs – high bandwidth, reduces activation" ("roughly" is italicised in the deck).

**Figure — a pasted screenshot of an NVIDIA-style dark-themed documentation panel**, black background with white text and lime-green accents. Its own heading reads "**Guideline 4: Prefer EP over TP for Expert Layers**". Beneath it a two-column table, with a lime-green rule under the header row:

| EP Advantages | Details |
| --- | --- |
| Better GEMM efficiency | Larger local matrix sizes improve GPU utilization |
| Lower communication | EP has less communication overhead than TP for MoE layers |
| Simpler computation graph | Easier to overlap communication with computation |
| Token permutation | When `EP = num_experts`, local token permutation is eliminated |

Below the table, still inside the black panel: "**Example:** For Mixtral 8x7B, `EP8×TP1` outperforms `EP4×TP2`." The code-styled fragments (`EP = num_experts`, `EP8×TP1`, `EP4×TP2`) are rendered as green monospace text in boxed outlines.

Below the pasted panel, the deck's own line: "But, splitting matmuls can reduce efficiency vs routing activations". No citation URL is printed on this page.

## Slide 52 — Complexity – combining EP and others

Text above the figure:

- "Naively – we can compose EP and all other strategies.."
- "**But there are important notes** – DP usually shares replicas with EP splits (so EP<DP)"
- " DP and TP can interact badly to lower utilization"

**Figure — a four-panel schematic pasted from a paper, captioned "Fig. 8".** The four subfigures are laid out with (a) filling the left third of the image, and (b) top-right, (c) bottom-middle, (d) bottom-right.

- **(a) Data + Expert Parallelism.** Two grey rounded device boxes, "GPU1" (left) and "GPU2" (right). Inputs $X_1$ (green) and $X_2$ (orange) enter at the bottom. Each device runs, bottom to top: a yellow "Self-Attention" box, a red "Add + Normalize" box, then a green "Gate", a grey "Encode". A pale-blue rounded region spans both devices and contains a dark-blue full-width bar "All-to-All Dispatch", above it the light-blue "FFN1" (on GPU1) and "FFN2" (on GPU2), and above those a dark-blue full-width bar "All-to-All Combine". Above the blue region each device has a grey "Decode" box and a red "Add + Normalize", emitting $Y_1$ (green) and $Y_2$ (orange) at the top. Green and orange residual lines run up the outside of each device, colour-matched to $X_1$ and $X_2$.
- **(b) Data + Expert + Tensor Parallelism.** Four devices, GPU1–GPU4, in two pairs. $X_1$ (green) feeds the GPU1/GPU2 pair, $X_2$ (orange) the GPU3/GPU4 pair. In each pair a single yellow "Self-Attention" box is shared across the two GPUs (tensor-parallel), feeding two green "Gate" boxes (one per GPU). Above them, inside a pale-blue expert region, two stacked full-width blue bars: "FFN1" over "FFN2" for the first pair, "FFN3" over "FFN4" for the second — i.e. each expert is itself sharded across the pair's two GPUs.
- **(c) Data + Expert + Pipeline Parallelism.** Four devices in a 2×2 arrangement: GPU1 and GPU2 form the lower pipeline stage, GPU3 and GPU4 the upper. Lower stage: $X_1$ (green) into GPU1, $X_2$ (orange) into GPU2, each through a yellow "SA1" box then a blue "FFN11" (GPU1) / "FFN12" (GPU2) inside a pale-blue expert region, with crossing blue arrows between the two devices. Those outputs feed upward into the upper stage's yellow "SA2" boxes on GPU3/GPU4, and then into blue "FFN21" (GPU3) / "FFN22" (GPU4), again inside a pale-blue region with crossing arrows.
- **(d) Expert + Tensor Parallelism.** Two devices, GPU1 and GPU2, with a single $X_1$ (green) input. One yellow "Self-Attention" box spans both devices. Each device has a green "Gate", and above them a pale-blue expert region holding two blue expert boxes per device: "FFN1", "FFN2" on GPU1 and "FFN3", "FFN4" on GPU2.

Figure caption, printed in small type at the foot of the image and transcribed verbatim:

"Fig. 8. Schematic depiction of diverse parallel strategies for MoE. For clarity and conciseness, this illustration omits some All-to-All, All-Reduce, Point-to-Point communication within parallelism, and Normalization, Encode, Decode, Gate in subfigures (b), (c), and (d)."

No source citation beyond the figure's own "Fig. 8" label is printed on the page.

## Slide 53 — Complexity: Decoupling attention / expert parallelism

Text:

- "**Recall**: MoEs apply to the MLPs, not the attention (with some exotic exceptions)"
- "This can lead to some imbalance between attention and MLPs for parallelism"
- "High TP – useful for attention parallelization (since we cant use EP)" (printed without the apostrophe in "can't")
- "Low TP – useful for MLP parallelization (we'd rather EP than TP)"
- "What to do? More flexible parallelism strategies (i.e. in Megatron)"

**Figure — a pasted light-themed documentation panel** with a white/grey background. Its lead line reads: "**MoE Parallel Folding** is Megatron Core's solution that **decouples attention and MoE parallelism**:". Below it, a small table with a dark-green rule under the header row:

| Parallelism Group | Attention Layers | MoE Layers |
| --- | --- | --- |
| Dimensions | TP × CP × DP × PP | ETP × EP × EDP × PP |

Below the panel, the deck's own line: "Separates out TP/CP/DP (for attention) and (ETP/EP/EDP) for MLPs".

Citation at the bottom right of the page: `https://docs.nvidia.com/megatron-core/developer-guide/latest/user-guide/features/moe.html`

Note that both dimension products end in `PP` — pipeline parallelism is the one axis the folding does *not* decouple — and that the deck's summary line lists the two triples in different orders from the table cells (table: "TP × CP × DP × PP" and "ETP × EP × EDP × PP"; prose: "TP/CP/DP" and "ETP/EP/EDP"), which agree.

## Slide 54 — Other parallelism strategies

**Figure — the Ring Attention / Blockwise Parallel Transformer schematic**, labelled "(a)" at its top left. Two dashed-outlined vertical columns, labelled "Device 1" (left) and "Device 2" (right) beneath them. Each column contains a grey square panel with, bottom to top:

- an arrow up from a "query block" label printed below the dashed column,
- an orange box "Blockwise Attention",
- a circled-plus $\oplus$ node, fed also by a residual line that branches off below the attention box and curves up the left side,
- a blue box "Blockwise FeedForward",
- a second $\oplus$ node, fed also by a residual branching off below the feedforward box,
- an arrow continuing up out of the top of the column.

Three grey left-pointing block arrows, each labelled "**Key Value Block**", sit at the same vertical height as the attention boxes: one to the left of Device 1, one between Device 1 and Device 2, and one to the right of Device 2 — the ring passing key/value blocks from device to device.

Caption line printed below the figure, in the deck's own text: "**Context parallel / Ring attention** (split activations across GPUs in a long sequence)".

That caption is the entire body text of the page; there are no bullets and no citation line.

## Slide 55 — Recap: LLM parallelism table

A single full-width table with a dark-blue header row (white text) and alternating light-grey/white body rows. Several cells are set in **red** — the deck uses red for the point it wants remembered in that cell. Red cells are marked below.

| Method | Comm/Sync | Param memory per rank | Activation / KV memory per rank | Main bandwidth cost | Scales global batch? | Easy to use? |
| --- | --- | --- | --- | --- | --- | --- |
| DDP / ZeRO-1 | gradient all-reduce per step | **No param scaling** *(red)* (ZeRO-1 only opt state) | **None** *(red)* | **Gradient traffic ~ O(params)** | **Yes, linear in DP** *(red)* | **Very** |
| FSDP / ZeRO-3 | Gradient all reduce – can be overlapped | **~1/DP** for params/grads/optimizer state | **None** *(red)* | **Param traffic ~ O(params)**, higher than DDP, overlap | **Yes, linear in DP** *(red)* | **Moderate** |
| Pipeline parallel | Activation between layers, **pipeline bubbles** *(red)* | **~1/PP** | Depends on pipeline buffers | **Activation traffic between stages** | **No**, but needs microbatches | **Hard** *(red)* |
| Tensor parallel | **Blocking activation communication** *(red)* | **~1/TP** for TP-sharded weights | **~1/TP** for relevant matmul activations w/ SP | **Activation-sized collectives every block** *(red)* | **No** | **Hard** |
| Sequence / Context parallel | per-layer sequence-shard exchange | **None** *(red)* | **~1/SP or 1/CP** on sequence-side activations / KV | **Activation / KV communication** | **No** | **Hard** |
| Expert parallel (MoE) | **token dispatch - all-to-all per MoE** *(red)* | **~1/EP** for expert weights only | **None** *(red)* | **Token-routing all-to-all** *(red)* | **No**, but needs enough tokens per expert | **Hard** |

Six rows, seven columns; no total or average row. Note the two distinct senses of "None" in this table: in the "Param memory per rank" column it means the method does not shard parameters (Sequence/Context parallel), while in the "Activation / KV memory per rank" column it means the method does not reduce activation memory (DDP/ZeRO-1, FSDP/ZeRO-3, Expert parallel).

No figure or citation on this page beyond the table.

## Slide 56 — Model vs Tensor parallel (TPU book)

Left half of the page carries a centred call-out: "**Key quantity**" over "Global batch size (divided by GPU)".

Below it, a small pasted LaTeX table (booktabs style, no colour). Its two right-hand headers each run over two lines:

| Strategy | Compute per layer<br>(ignoring gating einsum) | Comms per layer<br>(bytes, forward + backward pass) |
| --- | --- | --- |
| DP | $4BDF/X + 8BDF/X$ | $0 + 8DF$ |
| FSDP | $4BDF/X + 8BDF/X$ | $4DF + 8DF$ |
| MP | $4BDF/Y + 8BDF/Y$ | $4BD + 4BD$ |
| FSDP + MP | $4BDF/(XY) + 8BDF/(XY)$ | $(4BD/X + 4DF/Y) + (8BD/X + 8DF/Y)$ |

Note on the last cell: the pasted table image is cropped at its right edge, so the **closing parenthesis of the final term is cut off** on the page — it reads "$(4BD/X + 4DF/Y) + (8BD/X + 8DF/Y$" as printed. The closing bracket is supplied above from the obvious symmetry of the expression; this is stated so a reader is not surprised by the page. Confirmed at 900 dpi: it is a crop of the pasted image, not an occlusion by the chart.

The deck does not define $B$, $D$, $F$, $X$, $Y$ on this page. From the surrounding context (this is the TPU book's notation, not the $sbh$ notation of slides 46–49) $B$ is the batch, $D$ the model dimension, $F$ the MLP dimension, $X$ the data-parallel axis and $Y$ the model-parallel axis.

**Figure (right half) — line chart, "Batch-size scaling behavior of parallelization strategies on a 4x4x4 mesh".**

- y-axis: "FLOPS time / Comms time", log scale, ticked $10^{-2}$, $10^{-1}$, $10^{0}$; the plot area runs from $10^{-2}$ up to about $2.6$.
- x-axis: "*B/N*, batch size divided by total number of chips", linear, ticked 0, 250, 500, 750, 1000, 1250, 1500, 1750, 2000.

**Three data series** (legend, bottom right, in this order):

- **Blue — "FSDP Only".** Rises steeply from below $10^{-2}$ near the origin, then flattens on the log axis: ≈0.30 at B/N = 250, ≈0.59 at 500, ≈0.89 at 750, ≈1.18 at 1000, ≈1.47 at 1250, ≈1.77 at 1500, ≈2.06 at 1750, ≈2.3 at the right end. It crosses the ratio-1 line at B/N ≈ 850.
- **Orange — "MP Only".** A perfectly flat horizontal line at ≈0.55 across the whole x range. It never reaches 1, so pure MP is communication-bound at every batch size shown.
- **Green — "FSDP + MP".** Rises fastest of the three: ≈0.06 at B/N ≈ 0, ≈0.80 at 250, ≈1.12 at 500, ≈1.37 at 750, ≈1.59 at 1000, ≈1.77 at 1250, ≈1.94 at 1500, ≈2.10 at 1750, ≈2.24 at the right end. It crosses the ratio-1 line at B/N ≈ 400. Green is above blue for most of the range; the two curves cross at roughly B/N ≈ 1800–1850, after which blue is slightly higher.

**Annotations, not data series:**

- A black **dotted horizontal line at $10^{0}$** (ratio 1), labelled "Computation bound" above it and "Communication bound" below it.
- Two grey **dashed vertical lines**, at B/N ≈ 400 and B/N ≈ 850.
- Three in-plot text blocks, one per region: "No scheme works when $B < 400$" (left of the first dashed line), "Only mixed FSDP + MP works when $B < 850$" (between the two), "Both mixed FSDP + MP and pure FSDP work when $B > 850$" (right of the second).

No citation line is printed on the page beyond "(TPU book)" in the title.

## Slide 57 — '3D (4D) parallelism' – putting it all together

Text (left half):

"**Simple rules of thumb from the literature.**"

1. Until your model fits in memory..
   - Tensor / expert parallel up to GPUs / machine
   - Pipeline parallel across machines
   - (Or use Zero-3, depending on BW) — this third item is set with a different, chevron-style bullet (›) rather than the round blue bullet of the two above it

2. Then until you run out of GPUs
   - Scale the rest of the way with data parallel

Below, in smaller type: "If your batch size is small.. gradient accumulate to trade higher batch sizes for better communication efficiency."

**Figure (right half) — the DeepSpeed 3D-parallelism diagram.** Two large light-grey panels stacked vertically, labelled "**Data Parallel Rank 0**" (top) and "**Data Parallel Rank 1**" (bottom). Each panel contains three yellow rounded boxes side by side: "Pipeline Stage 0", "Pipeline Stage 1", "Pipeline Stage 2", captioned beneath with "Network Layers 0-7", "Network Layers 8-15" and "Network Layers 16-23" respectively.

Inside each pipeline stage sits a stack of card-like slabs drawn in perspective, each slab split into four horizontal colour bands — purple, green, orange, blue — with vertical labels down the left edge reading "MP-0", "MP-1", "MP-2", "MP-3" (top to bottom). Between adjacent pipeline stages, four double-headed block arrows, colour-matched to MP-0…MP-3, carry the pipeline activations.

Between the two data-parallel panels, a row of vertical double-headed block arrows each labelled "**ZeRO**" — four arrows per pipeline stage, again colour-matched purple/green/orange/blue — showing the ZeRO/data-parallel communication that connects the same model-parallel rank across the two data-parallel ranks.

No citation is printed on the page.

## Slide 58 — Example from Metagron's current recommendations

The title is printed as "Metagron's" — a typo for "Megatron's"; transcribed as printed.

Two column headings: "The "**basics**"" (left) and "Plus more advanced "**extras**"" (right). The body of the page is five pasted screenshots of NVIDIA's Megatron Core documentation, three on the left and two on the right, each a "Guideline N" heading with an Aspect/Recommendation table.

**Left column.**

*Guideline 1: Minimize Model Parallelism, Maximize Data Parallelism*

| Aspect | Recommendation |
| --- | --- |
| Goal | Keep TP/EP/PP as small as possible while avoiding OOM |
| Why | Model parallelism introduces communication overhead that hurts performance |
| How | Use distributed optimizer (`--use-distributed-optimizer`) to shard optimizer states across DP ranks, freeing memory for larger DP size |

*Guideline 2: Keep EP and TP Communication Within NVLink Domain*

| Aspect | Recommendation |
| --- | --- |
| Goal | Ensure EP×TP fits within a single node (typically 8 GPUs) |
| Why | EP and TP are communication-intensive; NVLink provides much higher bandwidth than cross-node interconnects |
| Scaling | When scaling beyond one node, prefer PP over expanding TP/EP across nodes |

Under that table: "**Note:** For very large MoE models like DeepSeek-V3, the EP communication may exceed the NVLink bandwidth. In this case, consider using 1F1B A2A Overlap to overlap the EP communication."

*Guideline 3: Use Pipeline Parallelism (PP) for Multi-Node Scaling*

| Aspect | Recommendation |
| --- | --- |
| Goal | Use PP to distribute layers across nodes while keeping EP×TP within NVLink |
| VPP | Enable Virtual Pipeline Parallelism to reduce pipeline bubbles when `PP ≥ 2` |
| Config | Set `--num-layers-per-virtual-pipeline-stage` to control VPP size |

**Right column.**

*Guideline 4: Prefer EP over TP for Expert Layers* — this is the same panel that appears on slide 51, here in the light theme rather than the dark one.

| EP Advantages | Details |
| --- | --- |
| Better GEMM efficiency | Larger local matrix sizes improve GPU utilization |
| Lower communication | EP has less communication overhead than TP for MoE layers |
| Simpler computation graph | Easier to overlap communication with computation |
| Token permutation | When `EP = num_experts`, local token permutation is eliminated |

Under it: "**Example:** For Mixtral 8x7B, `EP8×TP1` outperforms `EP4×TP2`."

*Guideline 5: Enable Context Parallelism (CP) for Long Sequences*

| Aspect | Recommendation |
| --- | --- |
| When to use | Sequence length ≥ 8K tokens |
| Key factor | CP efficiency depends on overlapping communication with computation |
| Config | Set `--context-parallel-size` to partition sequences across GPUs |

Citation at the foot of the page: `https://docs.nvidia.com/megatron-core/developer-guide/latest/user-guide/features/moe.html`

## Slide 59 — Scaling strategies from Narayanan 2021

At the top of the page, a pasted title block from the paper:

"**Efficient Large-Scale Language Model Training on GPU Clusters Using Megatron-LM**"

Author line, transcribed verbatim: "Deepak Narayanan‡\*, Mohammad Shoeybi†, Jared Casper†, Patrick LeGresley†, Mostofa Patwary†, Vijay Korthikanti†, Dmitri Vainbrand†, Prethvi Kashinkunti†, Julie Bernauer†, Bryan Catanzaro†, Amar Phanishayee\*, Matei Zaharia‡" with the affiliation line "†*NVIDIA* ‡*Stanford University* \**Microsoft Research*".

**Table — the paper's scaling configurations**, pasted as an image with cyan/blue header cells and alternating white / pale-blue body rows. Ten data rows, nine columns; the ninth column, "DP size", is **not part of the pasted image** — it is added by the deck in its own sans-serif type to the right of the table, with no cell borders.

| Number of parameters (billion) | Attention heads | Hidden size | Number of layers | Tensor model-parallel size | Pipeline model-parallel size | Number of GPUs | Batch size | Achieved teraFLOP/s per GPU | Percentage of theoretical peak FLOP/s | Achieved aggregate petaFLOP/s | DP size |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 1.7 | 24 | 2304 | 24 | 1 | 1 | 32 | 512 | 137 | 44% | 4.4 | 32 |
| 3.6 | 32 | 3072 | 30 | 2 | 1 | 64 | 512 | 138 | 44% | 8.8 | 32 |
| 7.5 | 32 | 4096 | 36 | 4 | 1 | 128 | 512 | 142 | 46% | 18.2 | 32 |
| 18.4 | 48 | 6144 | 40 | 8 | 1 | 256 | 1024 | 135 | 43% | 34.6 | 32 |
| 39.1 | 64 | 8192 | 48 | 8 | 2 | 512 | 1536 | 138 | 44% | 70.8 | 32 |
| 76.1 | 80 | 10240 | 60 | 8 | 4 | 1024 | 1792 | 140 | 45% | 143.8 | 32 |
| 145.6 | 96 | 12288 | 80 | 8 | 8 | 1536 | 2304 | 148 | 47% | 227.1 | 24 |
| 310.1 | 128 | 16384 | 96 | 8 | 16 | 1920 | 2160 | 155 | 50% | 297.4 | 15 |
| 529.6 | 128 | 20480 | 105 | 8 | 35 | 2520 | 2520 | 163 | 52% | 410.2 | 9 |
| 1008.0 | 160 | 25600 | 128 | 8 | 64 | 3072 | 3072 | 163 | 52% | 502.0 | 6 |

(There is no total or average row. The DP-size column is consistent with the rest of the table: DP = GPUs / (TP × PP) in every row.)

Below the table:

"**Notes**"

- Tensor parallel first up to 8, then caps out at 8.
- Pipeline parallel goes up to make the model fit.
- Dara parallel gradually decreases with scale, with the largest model having DP=6

("Dara" is printed for "Data" in the third note — transcribed as printed.)

## Slide 60 — Careful '3D' parallelism gives linear gains

**Figure — a pasted line chart from the Narayanan 2021 paper.**

- y-axis: "Achieved teraFLOP/s per GPU", linear, ticked 0, 50, 100, 150, 200.
- x-axis: "Number of GPUs", linear, ticked 768, 1152, 1536, 1920 (the plotted range extends a little past 1920 to about 2150).

**Four data series.** The legend sits above the plot in two rows of two, ordered: "ZeRO-3, 175B" (blue, circle marker, dashed) and "PTD-P, 175B" (orange, triangle marker, dashed) on the top row; "ZeRO-3, 530B" (blue, diamond marker, solid) and "PTD-P, 530B" (orange, square marker, solid) on the bottom row. Blue = ZeRO-3, orange = PTD-P; dashed = 175B, solid = 530B, matching the pasted caption. Each series has three markers. Read left to right along each line:

- **ZeRO-3, 175B — blue dashed, filled circles.** ≈143 at ~680 GPUs, ≈88 at ~975 GPUs, ≈44 at ~1560 GPUs. The steepest fall on the chart: throughput per GPU drops to roughly a third.
- **ZeRO-3, 530B — blue solid, filled diamonds.** ≈138 at ~815 GPUs, ≈98 at ~1240 GPUs, ≈48 at ~2105 GPUs. Also falls steeply, but less so than the 175B ZeRO-3 line at comparable GPU counts.
- **PTD-P, 175B — orange dashed, filled triangles.** ≈152 at ~680 GPUs, ≈148 at ~975 GPUs, ≈140 at ~1560 GPUs. Nearly flat, sagging by about 12 teraFLOP/s over the range.
- **PTD-P, 530B — orange solid, filled squares.** ≈171 at ~815 GPUs, ≈167 at ~1240 GPUs, ≈159 at ~2105 GPUs. The highest series throughout, and nearly flat.

There is no ideal-scaling reference line and no gridline annotation on this chart.

Pasted caption below the figure, transcribed verbatim: "**Figure 10: Throughput per GPU of PTD-P and ZeRO-3 for two different GPT models (the 175B GPT-3 model is shown with dotted lines, and the 530B model is shown with solid lines). Global batch sizes are fixed and ZeRO-3 is used without any model parallelism.**"

Below the caption, in the deck's own type: "More GPUS, same, flat utilization!"

Note the caption says "dotted lines" for the 175B series; the lines are drawn dashed rather than dotted. The deck's own line "More GPUS, same, flat utilization!" is true only of the two orange PTD-P series — the two blue ZeRO-3 series fall sharply — so the slide's summary applies to the strategy it is arguing for, not to the whole chart.

## Slide 61 — Tensor parallel = 8 is often optimal

**Figure — a pasted line chart from the same Narayanan 2021 paper ("Figure 13").**

- y-axis: "Achieved teraFLOP/s per GPU", linear, ticked 0, 50, 100, 150, 200.
- x-axis: "(Pipeline-parallel size, Tensor-parallel size)", categorical, with five tick labels: (2, 32), (4, 16), (8, 8), (16, 4), (32, 2). Each pair multiplies to 64.

**Two data series** (legend inside the plot, lower left, in this order):

- **Blue, filled circles — "Batch size = 32".** ≈102 at (2, 32), ≈132 at (4, 16), ≈140 at (8, 8), ≈111 at (16, 4), ≈91 at (32, 2). Peaks at (8, 8) and falls off steeply on the right. At (2, 32) the blue marker is largely hidden behind the orange one.
- **Orange, filled diamonds — "Batch size = 128".** ≈107 at (2, 32), ≈146 at (4, 16), ≈165 at (8, 8), ≈162 at (16, 4), ≈148 at (32, 2). Above the blue series at every point; also peaks at (8, 8), but its fall-off to the right is much gentler.

No reference or ideal-scaling line is drawn.

Pasted caption below the chart, transcribed verbatim: "**Figure 13: Throughput per GPU of various parallel configurations that combine pipeline and tensor model parallelism using a GPT model with 162.2 billion parameters and 64 A100 GPUs.**"

Below the caption, in the deck's own type: "When parallelizing across 64 machines – it's best to use a 8 x 8 configuration."

**Flagged contradiction:** the deck's line says "64 **machines**" while the pasted caption it sits under says "64 A100 **GPUs**", and the x-axis pairs multiply to 64 devices, not 64 machines. Both are transcribed as printed; the figure's own caption is the one that agrees with the axis.

## Slide 62 — Activation recomputation can pay for itself (via memory)

**Figure — a two-series line chart** (no pasted paper caption on this page).

- y-axis: "Throughput (sequences/second)", linear, ticked 0.0, 2.5, 5.0, 7.5, 10.0.
- x-axis: "Batch size", ticked 1, 2, 4, 8, 16, 32, 64, 128, 256 — evenly spaced, so the axis is logarithmic in batch size.

**Two data series** (legend at the top left of the plot, in this order):

- **Blue, filled circles — "Act. recomputation".** Runs across the whole x range: ≈0.6 at batch 1, ≈0.95 at 2, ≈1.75 at 4, ≈3.0 at 8, ≈4.3 at 16, ≈5.7 at 32, ≈6.8 at 64, ≈7.4 at 128, ≈7.8 at 256. Rising and flattening out towards the right.
- **Orange, filled diamonds — "W/o act. recomputation".** Only four points, and the series **stops at batch 8**: ≈0.75 at batch 1, ≈1.35 at 2, ≈2.35 at 4, ≈3.9 at 8. Over the range where both exist the orange series is slightly *above* the blue one — recomputation costs throughput at a fixed batch size — but it has no points beyond batch 8, which is the argument of the slide: without recomputation you cannot fit a larger batch at all.

Below the figure, in the deck's own type: "Activation recomputation enables larger batches, improving throughput (t=8, p=16)" — i.e. the measurement is at tensor-parallel size 8 and pipeline-parallel size 16, in the notation of slides 46–49.

No citation URL is printed on the page.
## Slide 63 — Recent LMs – what do they do?

Heading, in blue: "Recent LMs – what do they do?" This opens the case-study section: what real recent language models actually do for parallelism.

The page pastes a boxed screenshot from a paper, headed "**3.1 Distributed Training Framework**". The pasted paragraph reads (verbatim):

> We train our models using the *ZeRO* optimizer strategy (Rajbhandari et al., 2019) via PyTorch's FSDP framework (Zhao et al., 2023), which reduces memory consumption by sharding the model weights and their corresponding optimizer state across GPUs. At the 7B scale, this enables training with a micro-batch size of 4096 tokens per GPU on our hardware (see Section 3.4). For OLMo-1B and -7B models, we use a constant global batch size of approximately 4M tokens (2048 instances, each with a sequence length of 2048 tokens). For OLMo-65B model (currently training), we use a batch size warmup that starts at approximately 2M tokens (1024 instances), then doubles every 100B tokens until reaching approximately 16M tokens (8192 instances).
>
> To improve throughput, we employ mixed-precision training (Micikevicius et al., 2017) through FSDP's built-in settings and PyTorch's `amp` module. The latter ensures that certain operations like the softmax always run in full precision to improve stability, while all other operations run in half-precision with the `bfloat16` format. Under our specific settings, the sharded model weights and optimizer state local to each GPU are kept in full precision. The weights within each transformer block are only cast to `bfloat16` when the full-sized parameters are materialized on each GPU during the forward and backward passes. Gradients are reduced across GPUs in full precision.

The citations "Rajbhandari et al., 2019" and "Zhao et al., 2023" and "Micikevicius et al., 2017" are rendered as blue hyperlinked text in the source screenshot. This is the Dolma/OLMo paper's distributed-training section (unnamed on the slide itself beyond the caption below).

Below the box, in the lecturer's own words (bold model name, plain caption):

"**Dolma** – 7B model, FDSP (probably fits intra-node)"

Note: the slide's own caption spells it "FDSP" (not "FSDP") — transcribed as printed. This appears to be a typo for FSDP given the pasted text above says "FSDP framework."

## Slide 64 — DeepSeek

Heading, in blue: "DeepSeek".

The page pastes a boxed screenshot from a paper, headed "**2.4. Infrastructures**". The pasted paragraph reads (verbatim):

> We use an efficient and light-weight training framework named HAI-LLM (High-flyer, 2023) to train and evaluate large language models. Data parallelism, tensor parallelism, sequence parallelism, and 1F1B pipeline parallelism are integrated into this framework as done in Megatron (Korthikanti et al., 2023; Narayanan et al., 2021; Shoeybi et al., 2019). We also leverage the flash attention (Dao, 2023; Dao et al., 2022) technique to improve hardware utilization. ZeRO-1 (Rajbhandari et al., 2020) is exploited to partition optimizer states over data parallel ranks. Efforts are also made to overlap computation and communication to minimize additional waiting overhead, including the backward procedure of the last micro-batch and reduce-scatter operation in ZeRO-1, and GEMM computation and all-gather/reduce-scatter in sequence parallel. Some layers/operators are fused to speed up training, including LayerNorm, GEMM whenever possible, and Adam updates. To improve model training stability, we train the model in bf16 precision but accumulate gradients in fp32 precision. In-place cross-entropy is performed to reduce GPU memory consumption, i.e.: we convert bf16 logits to fp32 precision on the fly in the cross-entropy CUDA kernel (instead of converting it beforehand in HBM), calculate the corresponding bf16 gradient, and overwrite logits with its gradient.

This is (by its own text) the DeepSeek-V2 paper's infrastructure section.

Below the box, in the lecturer's own words:

"**DeepSeek** – ZeRO stage 1 with Tensor, Sequence, and Pipeline parallel"
"**V3** – PP (16), EP (64-way, 8 nodes), ZeRO stage 1"
"      EP uses 1F1B A2A Overlap"

(The third line is indented under the "V3" line, describing DeepSeek-V3's expert-parallel implementation detail: that its expert-parallel all-to-all communication is overlapped using a 1F1B — one-forward-one-backward — schedule.)

## Slide 65 — Yi

Heading, in blue: "Yi".

The page pastes a boxed screenshot from a paper, headed "**Performance and Cost Efficiency**". The pasted paragraph reads (verbatim, including its own bracketed citation numbers):

> *Memory* and *communication* restrictions are the two major technical challenges of large scale model training requiring integrated solutions beyond adding more GPUs. We use and improve upon the following techniques to tackle the memory and communication restrictions: (1) ZeRO-1 [60] to remove the memory consumption by partitioning optimizer states cross data-parallel processes; (2) tensor parallel combined with pipeline parallel [70] within each compute node to avoid inter-node communication bottleneck, and the 3D parallel strategy is well designed and optimized to avoid using activation checkpointing and minimize the pipeline bubbles; (3) kernel fusion techniques like flash attention [15] [14] and JIT kernels to reduce redundant global memory access and consumption; (4) topology-aware resource allocation (ranking strategy) to minimize the communication across different layers of switches, which is the limitation of a typical fat-tree-topology.

The reference numbers "[60]", "[70]", "[15]", "[14]" are rendered as blue hyperlinked text in the source screenshot.

Below the box, in the lecturer's own words:

"**Yi** - ZeRO stage 1 + Tensor + Pipeline parallel"
"Yi-lightning (2025) – Tensor replaced by Expert parallelism"

## Slide 66 — Llama3 405B

Heading, in blue: "Llama3 405B".

A pasted table, transcribed cell by cell (columns: GPUs, TP, CP, PP, DP, Seq. Len., Batch size/DP, Tokens/Batch, then a vertical rule, then TFLOPs/GPU, BF16 MFU):

| GPUs | TP | CP | PP | DP | Seq. Len. | Batch size/DP | Tokens/Batch | TFLOPs/GPU | BF16 MFU |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 8,192 | 8 | 1 | 16 | 64 | 8,192 | 32 | 16M | 430 | 43% |
| 16,384 | 8 | 1 | 16 | 128 | 8,192 | 16 | 16M | 400 | 41% |
| 16,384 | 8 | 16 | 16 | 8 | 131,072 | 16 | 16M | 380 | 38% |

Beneath the table, in the lecturer's own words: "(Stage 1, small bsz training, Stage 2 pretraining, Stage 3 long-context)" — glossing the three rows above as three successive training stages, in order.

Below that, a pasted block of paper prose, headed "**Network-aware parallelism configuration.**" (bold run-in header, then body text, verbatim):

> **Network-aware parallelism configuration.** The order of parallelism dimensions, [TP, CP, PP, DP], is optimized for network communication. The innermost parallelism requires the highest network bandwidth and lowest latency, and hence is usually constrained to within the same server. The outermost parallelism may spread across a multi-hop network and should tolerate higher network latency. Therefore, based on the requirements for network bandwidth and latency, we place parallelism dimensions in the order of [TP, CP, PP, DP]. DP (*i.e.*, FSDP) is the outermost parallelism because it can tolerate longer network latency by asynchronously prefetching sharded model weights and reducing gradients. Identifying the optimal parallelism configuration with minimal communication overhead while avoiding GPU memory overflow is challenging. We develop a memory consumption estimator and a performance-projection tool which helped us explore various parallelism configurations and project overall training performance and identify memory gaps effectively.

This is the Llama 3 paper's parallelism-configuration table and accompanying prose (paper not named by title on the slide itself, but identified by the slide's own heading "Llama3 405B").

No printed page number visible on slides 63–66.

## Slide 67 — Llama 3 405B

Heading, in blue: "Llama 3 405B". Note this heading is spelled "Llama 3" with a space, distinct from slide 66's "Llama3" (no space) — transcribed as printed on each page.

Sub-heading in bold black: "**Side note** – Lots of GPU failures at this scale!"

A pasted table, transcribed cell by cell (columns: Component, Category, Interruption Count, % of Interruptions):

| Component | Category | Interruption Count | % of Interruptions |
| --- | --- | --- | --- |
| Faulty GPU | GPU | 148 | 30.1% |
| GPU HBM3 Memory | GPU | 72 | 17.2% |
| Software Bug | Dependency | 54 | 12.9% |
| Network Switch/Cable | Network | 35 | 8.4% |
| Host Maintenance | Unplanned Maintenance | 32 | 7.6% |
| GPU SRAM Memory | GPU | 19 | 4.5% |
| GPU System Processor | GPU | 17 | 4.1% |
| NIC | Host | 7 | 1.7% |
| NCCL Watchdog Timeouts | Unknown | 7 | 1.7% |
| Silent Data Corruption | GPU | 6 | 1.4% |
| GPU Thermal Interface + Sensor | GPU | 6 | 1.4% |
| SSD | Host | 3 | 0.7% |
| Power Supply | Host | 3 | 0.7% |
| Server Chassis | Host | 2 | 0.5% |
| IO Expansion Board | Host | 2 | 0.5% |
| Dependency | Dependency | 2 | 0.5% |
| CPU | Host | 2 | 0.5% |
| System Memory | Host | 2 | 0.5% |

This is every row in the table; there is no total/summary row printed beneath it. Caption below the table, in bold run-in + plain text: "**Table 5**  **Root-cause categorization of unexpected interruptions during a 54-day period of Llama 3 405B pre-training.**  About 78% of unexpected interruptions were attributed to confirmed or suspected hardware issues."

Arithmetic check (not printed on the slide, noted here for the reader): the 18 Interruption Count values sum to 419, but the 18 printed percentages sum to exactly 94.9%, not 100%. This gap is in the source table as pasted (confirmed at 600 dpi against the original crop) — it is not a transcription error here, and the slide does not reconcile or explain it. Note also that the printed percentages are not percentages *of* 419 — 148/419 is 35.3%, against a printed 30.1% — so the pasted table appears to list only the leading categories out of a larger interruption total. Quote the two sums as they stand; do not compute a percentage from the counts in this table.

## Slide 68 — Gemma 2

Heading, in blue: "Gemma 2".

Left side, in the lecturer's own words: "**For 2, 9, 27B models**" then, below it: "ZeRO-3, MP (=TP+SP), DP"

Right side, a pasted screenshot from a paper, headed "**3.3. Compute Infrastructure**". The pasted paragraph reads (verbatim, with citations rendered as blue hyperlinked text in the source):

> We train our models with TPUv4, TPUv5e, and TPUv5p as outlined in Table 3. For the 2B model, we train on a 2x16x16 configuration of TPUv5e, totaling 512 chips, with 512-way data replication and 1-way model sharding. For the 9B model, we train on an 8x16x32 configuration of TPUv4, totaling 4096 chips, with 1024-way data replication and 4-way model sharding. For the 27B model, we train on an 8x24x32 configuration of TPUv5p, totaling 6144 chips, with 768-way data replication and 8-way model sharding.
>
> The optimizer state is further sharded using techniques similar to ZeRO-3 (Ren et al., 2021). For scales beyond a single pod, we perform a data-replica reduction over the data center network, using the Pathways approach of Barham et al. (2022). We also use the 'single controller' programming paradigm of Jax (Roberts et al., 2023) and Pathways (Barham et al., 2022). As in Gemma 1, we use the GSPMD partitioner (Xu et al., 2021) for training step computation and the MegaScale XLA compiler (XLA, 2019).

This is the Gemma 2 paper's compute-infrastructure section (paper not named by title on the slide itself, but identified by the slide's own heading "Gemma 2").

## Slide 69 — Mixtral 8x22B

Heading, in blue: "Mixtral 8x22B".

Text, in bold + plain: "**From megatron docs** –"

Below it, in the lecturer's own words: "TP / PP / CP / EP (4/4/1/8) – DP likely 2 (to add up to 256GPUs)"

A pasted table on a dark (black) background with white text, titled "**Mixture-of-Experts Models**". Columns: Model, Size, GPUs, TP, PP, CP, EP, Configuration Notes.

| Model | Size | GPUs | TP | PP | CP | EP | Configuration Notes |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Mixtral | 8x7B | 64 | 1 | 4 | 1 | 8 | EP=8 for 8 experts |
| Mixtral | 8x22B | 256 | 4 | 4 | 1 | 8 | TP+PP+EP for large MoE |
| DeepSeek-V3 | 671B | 1024 | 2 | 16 | 1 | 64 | Massive MoE with 256 experts |

The lecturer's note above the table ("TP / PP / CP / EP (4/4/1/8), DP likely 2") matches the Mixtral 8x22B row exactly (4×4×1×8 = 128, ×2 DP = 256 GPUs, matching the table's GPUs column for that row).

## Slide 70 — Nemotron 3 Super 120B-A12B

Heading, in blue: "Nemotron 3 Super 120B-A12B".

Sub-heading, bold black: "**From the long-context extension section**"

A pasted boxed screenshot of paper prose (no section number or header visible above it in the box itself). The pasted paragraph reads (verbatim, with the learning-rate value in the original's math formatting):

> Similar to Nemotron 3 Nano, we added a long-context phase (LC-Phase) at the end of pretraining. In the LC-Phase, we performed continuous pretraining (CPT) to equip the base model with long-context ability. We used a constant learning rate of $4.5 * 10^{-6}$ and global batch size of 16. We used 64-way context parallelism, 2-way tensor parallelism, and 64-way expert parallelism to train on GB200 GPUs. We reused the long-context document QA dataset from Nemotron 2 & 3 Nano. We allocated the document QA data to 20% in the Phase LC data blend, with the remaining 80% being downscaled Phase 2 data. We initially performed CPT on 1,048,576 (1m) context length. Such stage lasted for 34 billion tokens. Following that we added another stage to alternatingly train on both 1m and 4k sequences in order to mitigate the minor impact we observed on the math-related benchmarks. The second stage lasted for 17 billion tokens.

Below the box, in the lecturer's own words: "TP / PP / CP / EP (2/0/64/64)"

This summary line gives PP = 0, while the pasted paragraph above does not mention pipeline parallelism at all (only context, tensor, and expert parallelism are named in the prose). The TP=2, CP=64, EP=64 values in the lecturer's summary do match the "2-way tensor parallelism," "64-way context parallelism," and "64-way expert parallelism" stated in the pasted paragraph.

No printed page number visible on slides 67–70.

## Slide 71 — Qwen 3

Heading, in blue: "Qwen 3".

Sub-heading, plain black text: "225B-A22B and 30B-A3B". Confirmed at 1200 dpi — the text reads "225B" and not "235B". This is very likely a typo for "235B", since the pasted table directly below names the model "Qwen3-235B-A22B" four times; transcribed here exactly as printed, and flagged as a likely slide error rather than silently corrected.

A pasted table on a dark (black) background with white text, titled "**Parallelism Configuration**". Columns: Model, Mode, TP, PP, EP, Total GPUs, Use Case.

| Model | Mode | TP | PP | EP | Total GPUs | Use Case |
| --- | --- | --- | --- | --- | --- | --- |
| Qwen3-30B-A3B | Pretrain | 1 | 1 | 8 | 8 | Pre-training (single node) |
| Qwen3-30B-A3B | Full SFT | 1 | 1 | 8 | 8 | Full supervised finetuning |
| Qwen3-30B-A3B | LoRA/DoRA | 1 | 1 | 8 | 8 | PEFT finetuning (single node) |
| Qwen3-235B-A22B | Pretrain | 2 | 8 | 32 | 512 | Pre-training (64 nodes) |
| Qwen3-235B-A22B | Full SFT | 2 | 8 | 32 | 512 | Full supervised finetuning (64 nodes) |
| Qwen3-235B-A22B | LoRA/DoRA | 2 | 8 | 32 | 512 | PEFT finetuning (64 nodes) |

To its right, a second pasted table (also dark background, white text, no title visible above its header row), giving measured throughput bands by workload/hardware combination. Columns: Workload family, Hardware, Typical band, Representative shape.

| Workload family | Hardware | Typical band | Representative shape |
| --- | --- | --- | --- |
| DSV3, large-scale | H100 | low-to-mid hundreds TFLOPS/GPU, high-teens MFU | TP2, EP64, PP8, DeepEP |
| DSV3, large-scale | B200 | high-hundreds TFLOPS/GPU, mid-teens MFU | TP1, EP32, PP8, DeepEP |
| DSV3, large-scale | GB200 | around 1K TFLOPS/GPU, low-20s MFU | TP1, EP64, PP4, HybridEP |
| DSV3, large-scale | GB300 | above the GB200 band, often mid-20s MFU | TP1, EP64, PP4, HybridEP |
| Qwen3 235B | H100 | low-300s TFLOPS/GPU, around 30% MFU | TP2, EP32, PP8, DeepEP |
| Qwen3 235B | GB200 | high-hundreds TFLOPS/GPU in tuned runs | TP1 or TP2, EP32-64, PP4, HybridEP |
| Qwen3 30B | H100 | low-200s TFLOPS/GPU | TP1, EP8, PP1, DeepEP |
| Qwen3-Next 80B | GB200 | low-300s TFLOPS/GPU in BF16-class runs | TP1, EP32, PP2, HybridEP |

"DSV3" abbreviates DeepSeek-V3 (consistent with that model's use elsewhere in this deck). This second table's rows are not all Qwen 3 — four of its eight rows are DeepSeek-V3 ("DSV3") rows, included for hardware comparison; the slide does not caption why DeepSeek rows appear on a "Qwen 3" slide.

Below both tables, in the lecturer's own words: "Primarily expert parallel up to 8 GPUs, 2/8/32 parallelism for larger models"

## Slide 72 — Overview table of model parallelism

Heading, in blue: "Overview table of model parallelism".

This is the deck's summary table of every case-study model's parallelism configuration. Columns (left to right): an unlabeled row-label column, DP, TP/SP, EP, PP, CP. Transcribed cell by cell, re-verified at 500-700 dpi against the header row:

| | DP | TP/SP | EP | PP | CP |
| --- | --- | --- | --- | --- | --- |
| Deepseek | ?? (Zero1) | 1 | 8 | 16 | ?? |
| Deepseek v3 | ?? (Zero1) | 1 | 64 | 16 | ?? |
| Yi | ?? (Zero1) | >0 | 1 | >0 | ?? |
| Llama3 405B | 128 | 8 | 0 | 16 | 1 |
| Gemma 2 | 768 | 8 | 0 | 0 | 0 |
| Mixtral 8x22 (megatron) | 2 | 4 | 8 | 4 | 1 |
| Nemotron 3 120B (long context) | ?? | 2 | 64 | ?? | 64 |
| Qwen 3 (megatron) | ?? | 2 | 32 | 8 | 1 |

Every "??" cell above is printed exactly that way in the table (an unfilled/unknown value), not a transcription placeholder. The "(Zero1)" annotation appears only in the DP column, only for the Deepseek, Deepseek v3, and Yi rows, alongside their "??" — i.e. those three rows mark DP degree as unknown but note that ZeRO stage 1 is the mechanism used, consistent with the paper excerpts on slides 64 and 65.

Below the table, in bold + plain text: "**Patterns** – TP generally <= 8. EP can be big (but hard!). Long context phases use large CP"

Cross-check against other slides in this range: this table's Llama3 405B row (DP 128, TP/SP 8, EP 0, PP 16, CP 1) matches the **second** row of slide 66's Llama 3 GPU-count table (16,384 GPUs, TP8 / PP16 / DP128), not the first, which gives DP=64. This table does not specify which of Llama 3's three training stages its Llama3 405B row is drawn from. TP=8 and PP=16 are common to all three of slide 66's rows; CP is **not** — slide 66 reads CP1, CP1, CP16 down its three rows, so this table's CP=1 matches the first two rows only. The third row is the long-context stage, and it is also the row whose DP is 8 rather than 128. This table's Mixtral 8x22 row (DP 2, TP/SP 4, EP 8, PP 4, CP 1) matches slide 69's Mixtral 8x22B row exactly, including the "DP likely 2" inference given on that slide. This table's Qwen 3 row (TP/SP 2, EP 32, PP 8, CP 1) matches slide 71's Qwen3-235B-A22B pretrain row (TP 2, PP 8, EP 32) exactly.

One cross-slide discrepancy, flagged rather than reconciled: slide 70 states outright, in the lecturer's own words, "TP / PP / CP / EP (2/0/64/64)" for Nemotron 3 Super 120B-A12B — i.e. PP = 0. This table's Nemotron 3 row instead gives PP as "??" (unknown) rather than 0, for the same model. The TP (2), EP (64), and CP (64) values agree between the two slides; only PP disagrees between "stated as 0" (slide 70) and "marked unknown" (slide 72).

## Slide 73 — Recap for the whole lecture

Heading, in blue: "Recap for the whole lecture".

A light-blue bordered box containing three bullets (diamond bullet markers), each on its own line with vertical space between them:

- "Scaling beyond a certain point requires multi-gpu, multi-node parallelism"
- "No single solution to the parallelism problem (probably want all 3 approaches)"
- "Simple, interpretable rules of thumb for combining different forms of parallelism"

No figure, table, or citation on this page. This is the final slide of the deck (page 73 of 73).

No printed page number visible on slides 71–73.
