# How this knowledge base is organized

This repo is the knowledge base for **CS336 — Language Modeling from Scratch |
Stanford, Spring 2026 — Percy Liang and Tatsunori Hashimoto**. It is read by
Cairn's in-extension AI chat, which fetches files over `raw.githubusercontent.com`
and follows relative markdown links.

**Coverage: Lectures 1–7 of 18.** See [`kb.json`](kb.json) and the banner
at the top of [`INDEX.md`](INDEX.md). Any page describing later material is
repeating Lecture 1's syllabus preview and says so in a blockquote at the top.

One numbering quirk to know: Lecture 2 teaches **resource accounting**, which
belongs to the Systems unit (Lectures 5–8, 10) even though the lecture itself sits
in the Basics block. So [`wiki/course-map.md`](wiki/course-map.md) marks Unit 2 as
partially covered while the rest of that unit is still a preview.

## Layout

| Path | Contents |
| --- | --- |
| `INDEX.md` | Entry point. Course summary + annotated table of contents. |
| `wiki/` | Durable pages: one per lecture, plus cross-lecture topics. |
| `raw/slides/` | The written course material, transcribed. See the note below — CS336 has no slide numbers for half its lectures, and none of the five decks transcribed so far prints one either. |
| `raw/transcripts/` | Copy-edited lecture transcripts with `[MM:SS]` paragraph marks. **These are the ones to read.** |
| `raw/transcripts/original/` | Verbatim auto-captions, kept as the record of what was said, plus the raw caption segment JSON they were generated from. |
| `raw/images/NN-<slug>/` | Rendered slide images and the course's own figures. See **Images** below for what is and is not here. |
| `raw/pdfs/` | Gitignored and empty. No binaries are committed; `sources.md` carries canonical URLs. |
| `sources.md` | Every lecture, deck, assignment and linked document, with URLs. |
| `kb.json` | Machine-readable coverage and provenance. Read this to know what this KB does and does not cover, and how far to trust a citation. |
| `SEE_ALSO.md` | Sibling KBs worth reading for this course, by repo URL. |
| `TODO.md` | Build tracker. Unchecked boxes are outstanding work. |

## What is special about CS336

**Half the lectures are Python programs, not slide decks.** Percy Liang's lectures
(1, 2, 6, 7, 10, 12, 13, 14, 17) — four of which, 1, 2, 6 and 7, are transcribed here —
are *executable lectures* — programs whose
execution delivers the content, rendered by a trace viewer. Tatsunori Hashimoto's
(3, 4, 5, 8, 9, 11, 15, 16) are conventional PDFs.

Three consequences for anyone extending this KB:

- **Do not cite slide numbers for a Percy lecture.** There are none. Cite the
  function name and source line range; `raw/slides/` carries a section-to-line
  table for that purpose.
- **The worked values are not in the source.** Numbers appear at runtime through
  `# @inspect` annotations, so reading the `.py` gives you the code but not the
  compression ratios or token ids. Where this KB reports such a value, it was
  obtained by **executing the lecture's own code**, and the file says so at the
  head of the section. If you add material, do the same — do not recall a number
  that the program computes.
- **A website crawl misses the executable lectures entirely**, since the `.py`
  files are not linked as documents. `sources.md` lists them by hand.

For decks (lectures 3, 4, 5, 8, 9, 11, 15, 16 — of which 3, 4, 5 and 8 are done), the normal rules apply: read the
pages visually, transcribe per slide, and audit the figure descriptions. **Lectures
3, 4 and 5 are the ones done so far**, and they set four precedents worth
following:

- **Derive the slide numbering before anyone reads a page, and hand over the
  conclusion.** `lecture_03.pdf` prints no page number on any of its 67 pages;
  `slide_number_map.py` nonetheless reported one, having read the numerator of a
  fraction in the corner of page 61 as a folio. Because the deck prints nothing,
  that script's `--verify` mode cannot check the file, and the fallback is a plain
  `1..N` heading-sequence check. Slide labels in `raw/slides/03-architectures.md`
  are therefore **PDF page numbers**, and the front matter says so.
- **Audit the figures, and target charts with legends and dense tables.** Eight
  pages were checked against the PDF at 600–1200 dpi. Six were clean; the errors
  were two sparse table columns asserted to be uniformly empty when each had
  exceptions, and one self-contradiction about a covered annotation. Both large
  numeric tables were exact.
- **Cross-check repeated material against itself.** Lecture 3's deck shows five
  views of one model database on slides 7, 9, 29, 51 and 67, read in separate
  passes. Comparing the 296 overlapping (model, column) cells found zero
  disagreements — the cheapest strong evidence available, and it needs no PDF
  access.
- **Split a long deck across parallel readers, and reconcile in the parent.**
  Lecture 4's 60 pages were read by three agents over pages 1–20, 21–40 and 41–60,
  each appending incrementally to its own chunk file; the parent concatenated them
  and wrote the front matter and section table. This keeps any one agent's context
  small, survives a mid-task kill, and gives three independent reports on questions
  that span the deck — all three, for instance, confirmed the absence of printed
  page numbers, which is stronger evidence than one reader's word.

  Two lecture-4 findings worth carrying into the next deck. **A deck can contradict
  itself**, and the transcription should record both sides rather than reconcile
  them silently — slide 47's expert-index vector disagrees with its own stage
  labels, slide 56's heading disagrees with its own diagram. And **the figure audit
  should target pages the transcriber was *confident* about, not only the ones it
  flagged.** On lecture 4 every self-flagged uncertainty proved correct, while the
  one substantive error sat on a page reported as clean.

**What lecture 5 added to those precedents.**

- **Derive the numbering in the parent and hand it over as a conclusion.** Lecture
  5's mapping was settled before any page was read — the script, plus a
  corner-position text-layer scan that returned only mid-page body text — and the
  readers were told to label pages accordingly and merely confirm by eye. Three
  readers then confirmed the absence of folios independently over their own ranges,
  which is stronger evidence than one reader's word and costs nothing extra.
- **The audit's error class moves between decks, so do not tune the audit to the
  last one.** On lectures 3 and 4 the errors were chart *values* — numbers read
  slightly off a plot. On lecture 5 every value, table cell and data series was
  exact, and all nine errors were **structural**: miscounted overlay arrows, a
  colour band described as a different colour, edge labels on the wrong edges, a
  "all four of these are labelled" claim true of one in four, and one whole figure
  never mentioned. Be suspicious of any claim of uniformity, and check that every
  figure on the page is described *and* that every figure described is on the page.
- **A dirty audit sample means the sample was too small.** Four of seven pages came
  back with findings on the first pass, so a second pass over seven further pages
  was run rather than trusting the first.
- **Adjudicate the transcript agent's own self-report.** Lecture 5's draft
  volunteered that it had "expanded three paragraphs to fit" the word-ratio band.
  Two were legitimate restorations; the third had invented two words to complete the
  speaker's aborted false starts. Ask the drafting agent directly which paragraphs it
  changed and whether every word traces to the captions — it answered accurately and
  the fix took one edit. Related: **false starts are preserved, not completed.**
- **Re-check the verification script itself.** The parent's first ratio checker
  reproduced exactly the bug run 4 had already recorded — stripping whole question
  markers before counting words, which hid the real ratios behind twelve false
  outliers. Only the four-word label is an insertion; a quoted student question is
  transcribed speech and must still count.

**What lecture 6 added.** It is the third executable lecture and the first whose
subject is code the students themselves write, so three things are worth carrying
forward.

- **An executable lecture can contradict itself too, and the code wins.** Lecture
  6's warp-occupancy example says in prose "thread block has 64 threads" and then
  sets `num_threads_per_block = 128` in the code below it. Both are transcribed, the
  computed values follow the code, and the discrepancy is flagged in `raw/slides/`
  and on the topic page. This is the source-text analogue of lecture 4's
  self-contradicting slides.
- **Check a catalog title against the material.** The catalog calls this lecture
  "Kernels, Triton, XLA"; XLA and JAX appear zero times in both the program and the
  captions, so the KB uses the course site's name. One `grep` settled it.
- **Machine-dependent values can be most of a lecture.** Benchmarking and profiling
  *are* lecture 6's subject, so its timings and all four profiler tables are
  measurements of one GPU. None is reproduced, and the GeLU comparison is stated
  qualitatively. Resist the urge to supply a plausible number for a comparison the
  lecture makes in words.

**What lecture 7 added.** It is the fourth executable lecture, and it breaks the
format's two standing assumptions — worth knowing before you transcribe another one.

- **Check whether the course published the lecture's runtime output.** Every prior
  executable lecture had its `@inspect` measurements withheld as machine-dependent.
  Lecture 7's are not: the course committed the program's own stdout from a real
  four-GPU run to `var/traces/lecture_07_stdout.txt` in the lectures repo, and the
  program itself links it. That single file supplied the measured bandwidths, the
  per-rank losses and the printed tensors. **Look for it before writing "not
  reproduced".** It is quoted under a distinct marker, `(recorded run)`, kept
  separate from `(computed)`, and every derived figure was re-checked against the
  lecture's own formula — all eight bandwidths reproduced the printed values to
  within one unit in the last place.
- **An executable lecture may not be traceable.** Lecture 7 really launches
  processes, so its `spawn()` helper detects a tracer and runs single-process with
  every `torch.distributed` call replaced by a no-op and rank pinned to 0. The trace
  view therefore shows no communication and rank-0-only `@inspect` values. Read the
  source, not the trace, and say so in the slide file.
- **Check a catalog title against the material — again, and in the other
  direction.** Lecture 6's lesson was that a catalog title can name material the
  lecture does not contain. Lecture 7's is that two *different* lectures can share a
  title: lectures 7 and 8 are both "Parallelism", and they are by different
  instructors in different formats. `INDEX.md`, `kb.json`, `sources.md` and the
  lecture page all say which one is covered, because a reader asking about FSDP will
  otherwise assume this KB failed rather than that it stops at lecture 7.
- **Quote-check the wiki against the transcript, and expect a real yield.** Run 6
  found nine quoting slips this way; run 7 found **thirty-four** across 146 quotes,
  because this lecture's prose quotes heavily. Nearly all were the same fault:
  silently smoothing the speaker's false starts *inside* quotation marks ("about
  four — about four x — slower" quoted as "about four x slower"). Two were worse —
  a quote run straight across a passage the transcript marks `[Ed:]` as garbled, and
  a paraphrase presented inside quotation marks. Two mechanical notes: pair quote
  characters **sequentially** rather than regexing for a minimum length, or short
  quotes desynchronize the pairing and every later "failure" is an artifact; and
  strip `[MM:SS]` markers from the haystack, or any quote spanning a paragraph
  boundary reports as missing.

`kb.json` reports `method: mixed`, because Lectures 1, 2, 6 and 7 are `source-text`
while Lectures 3–5 are `page-images`; the per-lecture breakdown is in
`materials.byLecture`.

## Conventions

- **INDEX.md is the front door.** The chat reads it first on every conversation.
  Every wiki page must appear there with a one-line description of what it holds.
  An unindexed page is effectively invisible.
- **Relative links only.** Absolute GitHub URLs break when the repo is renamed or
  forked. Links to *other* KBs are the exception and live in `SEE_ALSO.md`.
- **Cite everything.** Claims trace back to a transcript timestamp or a named
  section of the lecture source.
- **Never invent course content.** If the transcript is unclear, say so on the
  page. Do not fill the gap from outside knowledge — the chat presents these pages
  as authoritative material from this course.
- **Figures are described for Lectures 3, 4 and 5, and not for Lectures 1, 2, 6 or 7.**
  Those three decks were read as page images, so `raw/slides/03-architectures.md`,
  `raw/slides/04-attention-alternatives.md` and `raw/slides/05-gpus-tpus.md`
  describe every figure in prose and wiki pages may cite them. Lectures 1, 2, 6 and 7 were transcribed from source text, so their
  `image()` targets are recorded **by path only, with no description**, and no wiki
  claim may rest on one. Check which kind of lecture you are citing before quoting a
  figure.

  One exception is recorded explicitly: the FlashAttention-2 value at 1k on lecture
  4's slide 3 is overprinted by the chart's own legend and could not be read at any
  resolution. It is given as 153 TFLOPs/s on external confirmation, and the entry
  says so rather than implying it was read off the page.
- **Mark preview material.** Lecture 1 previews four units it does not teach. Pages
  covering them open with a coverage blockquote, so a reader never mistakes the
  framing for the treatment.
- **Distinguish computed from measured values.** Executable lectures produce their
  numbers at runtime. Where the expression is deterministic arithmetic, this build
  recomputes it and marks the result "(computed)". Where the value is a
  *measurement* — a wall-clock timing, a measured FLOP/s, an MFU, a peak-memory
  reading — it is a fact about the machine the lecture ran on, so `raw/slides/`
  marks it "machine-dependent, not reproduced" and gives no number. Do not quote a
  number for those, and do not substitute a plausible one.
- **Prose over fragments.** The chat quotes these pages to learners; bullet
  fragments quote badly.
- **Math in LaTeX**: `$...$` inline, `$$...$$` displayed. Renders both in Cairn's
  chat and on github.com. Never inside a code fence — that shows the source instead
  of the formula. Define each symbol on first use, and follow the course's own
  notation rather than a tidier one, since the learner is comparing the page
  against the lecturer's screen.
- **Transcripts stay spelled-out.** Captions render notation as speech ("n
  squared"); reconstructing it as $n^2$ belongs in `wiki/`, not `raw/transcripts/`.

## Images

Every lecture 1-9 has images. They are committed, not hotlinked, and they are the only
part of this KB that redistributes course material rather than pointing at it.

| Lecture | Files | Where they came from |
| --- | --- | --- |
| 1 Overview, Tokenization | 9 | the course's own `images/*.png`, fetched from the lectures repo |
| 2 PyTorch, Resource Accounting | 6 | same |
| 3 Architectures | 46 of 67 pages | rendered from `lecture_03.pdf` |
| 4 Attention Alternatives, MoE | 48 of 60 pages | rendered from `lecture_04.pdf` |
| 5 GPUs, TPUs | 51 of 55 pages | rendered from `lecture_05.pdf` |
| 6 Kernels, Triton | 4 | the course's own `images/*.png` |
| 7 Parallelism | 4 | same |
| 8 Parallelism 2 | 51 of 73 pages | rendered from `lecture_08.pdf` |
| 9 Scaling Laws | 40 of 57 pages | rendered from `lecture_09.pdf` |

Lectures 10-18 are not in this KB at all, so they have no images.

### Using them

**Use an image path you have actually read in a file. Never construct one from the
pattern, and never assume a slide has an image because a neighbouring one does.** Roughly
a third of each deck's pages were deliberately not rendered (see the next section), so
`slide-31.jpg` existing tells you nothing about `slide-32.jpg`. Reading a path that is not
in the repo returns an error rather than a URL, which costs a turn; a guessed path is
never worth it.

Links are **relative**, like every other link here — `../raw/images/09-scaling-laws/slide-44.jpg`
from `wiki/`, `../images/09-scaling-laws/slide-44.jpg` from `raw/slides/`. To show one,
read the path and use the URL that comes back; do not write an absolute
`raw.githubusercontent.com` URL into a file, because that bakes the owner, repo and branch
into the page and a fork or rename breaks every figure.

To list a lecture's images without reading the whole page:
`grep -o 'raw/images/[^)]*' wiki/09-scaling-laws.md`.

Three further conventions:

- **Prefer the transcription for numbers.** `raw/slides/` reproduces every table cell and
  equation as text, checked against the page. Quote that, and use the image to *show* the
  reader what you are quoting. Reading a value off a 1400px JPEG is strictly worse than
  reading it out of the file that was written from the page at 600-4800 dpi.
- **Show one image, not a gallery.** Each is a whole slide at 1400px; two in an answer is
  already a lot.
- **Keep the citation.** The image sits next to the slide citation it belongs to. An answer
  that shows a figure should still say which slide it is, so the reader can find the rest
  of that slide's content in `raw/slides/`.

### What was rendered, and what was not

A deck page was rendered when the PDF carries a raster covering more than 4% of the page
**and** the slide file's own prose describes an actual figure there. Neither signal is
sufficient alone, and both were overruled by hand where they disagreed: lecture 3's slide
32 (the hand-drawn RoPE rotation diagrams) and lecture 5's slide 2 are vector art with no
pasted raster, so the PDF test missed them; lecture 3's slides 38, 43, 44 and 47, lecture
4's 21 and 22, and lecture 8's 59, 66, 67, 69 and 71 carry pasted rasters that are *tables*
the slide file already reproduces cell by cell.

That last case is the rule worth remembering: **a transcribed table beats a picture of
one**, because it can be quoted, searched and cited by cell. The same goes for the pasted
equation blocks on lecture 4's slides 38 and 41 and lecture 8's 46 and 49. Title cards,
outline slides, section dividers and pure-text bullets are not rendered either — their
content is already fully in the slide file, so an image is bytes for nothing.

Two of the course's own figures were deliberately not fetched: `course-staff.png`
(a photo grid of the teaching staff, no course content) and `ranks.png` (four boxes
labelled Rank 0-3, which the prose states completely).

`raw/slides/` holds an image for **every** page that was rendered. `wiki/` holds one only
where a page cites that slide, which is the KB's convention anyway — so most images are
slide-file-only, and that is the intended outcome rather than a shortfall. Do not add a
citation merely to give an image a home.

### Provenance and attribution

Every image here comes from Stanford CS336 (Spring 2026), by Percy Liang and Tatsunori
Hashimoto, from the course's own lecture repository at
<https://github.com/stanford-cs336/lectures>:

- **Slide renders** are whole pages of `lecture_03.pdf`, `lecture_04.pdf`, `lecture_05.pdf`,
  `lecture_08.pdf` and `lecture_09.pdf`, at 1400px wide, JPEG q85 or PNG whichever came out
  smaller. Each is named by its PDF page number, which is what this KB cites as a slide
  number — none of the five decks prints a folio.
- **The course's own figures** for lectures 1, 2, 6 and 7 are the files those executable
  lectures pass to `image()`, fetched unmodified from `images/` in the same repository.
  Figures those lectures display by *external* URL — NVIDIA documentation, arXiv, Wikimedia,
  Springer, the JAX scaling book — were **not** copied; they stay as links in `raw/slides/`.

**The source repository carries no LICENSE file.** The material is published for public
reading but is not explicitly licensed for redistribution, and the decks reproduce figures
from third-party papers — among them Hoffmann et al.'s Chinchilla IsoFLOP figure, Wei et
al.'s emergent-abilities panels, Kaplan et al.'s scaling curves, Rosenfeld et al.'s joint
scaling grid, Shazeer et al.'s GLU conclusions, the Switch Transformer and OLMoE ablations,
the DeepSeek and Megatron-LM tables and diagrams, and a screenshot of a public tweet by
Stephen Roller. These are reproduced here for study, at the resolution the deck itself used,
with the deck linked at its canonical URL beside every image.

If the course, or an author of a reproduced figure, asks for something to come down: delete
the file and every Markdown image line that points at it, and note it in `kb.json`.

## Verifying an edit to the transcripts

The edited transcript is checked mechanically against the verbatim original. If
you re-edit it, re-run all three:

1. **Timestamp sequence** — the `[MM:SS]` markers must be identical and in order.
   **Match `[H:MM:SS]` too.** Lectures past an hour — Lecture 3 runs 1:29 — emit
   markers like `**[1:00:43]**`, and a regex of `\d+:\d+` silently skips every one
   of them. That happened during this build: 38 of Lecture 3's 117 markers went
   unchecked until the pattern was widened to `(?:\d+:)?\d+:\d+`, and checks 1
   and 3 were both quietly covering only the first hour.
2. **Number inventory** — every number in the original must still be present.
   Differences are legitimate only if deliberate; the header records the numeral
   conventions that account for the current ones.
3. **Per-paragraph word-count ratio** — each edited paragraph against its original.
   Filler removal puts this around 0.72–1.10. Anything outside that band means
   content moved *across* an `[MM:SS]` boundary, which breaks exactly the citation
   the markers exist for. Checks 1 and 2 both pass when that happens, so this one
   is not optional.

   **Two things the checker itself must get right**, both learned by getting them
   wrong on Lecture 4. Subtract `[Ed: …]` notes before counting, since they are
   inserted text — but subtract only the *label* of a
   `[Question from the floor: …]` marker, never the question's own words, which are
   transcribed speech. Stripping whole question markers manufactured five false
   outliers at ratios of 0.49–0.61 that would have sent a reader hunting for content
   drift that did not exist. And a genuinely low ratio is not automatically a fault:
   Lecture 4's three real outliers sat at 0.68–0.72 and were all pure filler, because
   that speaker uses "you know" / "sort of" / "kind of" far more than Lecture 3's.
   The band flags where to look; it does not classify.

   **A paragraph is everything from one marker to the NEXT marker**, not the single
   block the marker starts. Lecture 6's transcript lifts each student question onto
   its own line, and a splitter that stopped at the first blank line reported fifteen
   false outliers including a 0.00 — every one of them an artifact. This is the third
   distinct bug found in this same checker across three runs; write it fresh at your
   peril, and sanity-check any outlier by reading the paragraph before believing it.

## Rebuilding

Built and updated by the `cairn-kb` skill. To add newly released lectures or
extend coverage, append entries to `TODO.md` and re-run the skill — it only does
unchecked work. Update `kb.json` whenever coverage changes; a stale `kb.json` is
worse than none.

## Run 8 precedents (lecture 8, a 73-page deck)

- **Cross-check heading titles against the PDF text layer.** After assembling a
  deck transcription, slug each `## Slide N — Title` and look for it near the top
  of page N's extracted text. On lecture 8, 72 of 73 matched **verbatim**, the
  single exception being a title page whose text layer letter-spaces its words.
  This confirms page-to-heading attribution — that no reader skipped or shifted a
  page — **without opening a single page image in the parent**, which is the
  expensive thing the cost rules forbid. It costs one script and it is now the
  cheapest strong check available at this step. Run it before the figure audit.

- **The front matter is the least-checked layer of a slide file.** The figure
  audit targets pages; nothing targets the aggregating prose the parent writes
  *after* every reader has finished. On lecture 8, a sweep of all 48 cross-slide
  assertions found four errors, and **two were in that front matter** rather than
  in any reader's page — a legend list naming a slide that has no such legend, and
  an audit note whose "within 1-3 teraFLOP/s" read as a data range when it meant an
  agreement tolerance. Sweep the cross-slide claims explicitly, and include the
  front matter in the sweep.

- **A cross-slide claim is a distinct error class from a figure error.** All three
  of the audit's remaining findings were sentences that described a *neighbouring*
  slide, written by readers whose own pages were perfect. Add "check every
  assertion about another slide" to the audit prompt; it needs no page images and
  is therefore nearly free.

- **Verify an auditor's findings before applying them.** Lecture 8's cross-slide
  auditor reported four errors; one was its own. It read the readers' "no figure on
  this page" remarks as contradicting a raster count, but every one of those pages
  does carry a raster — pasted equations and tables rather than figures. Checking
  it against the PDF took one command and saved a correct sentence from being
  "fixed" into a wrong one.

- **A dirty audit sample does not always mean the sample was too small.** Run 12 of
  the CS224N build set the rule that a dirty sample warrants a second pass. Here
  pass 1 came back 6 of 8 clean with the two failures on one page each, and pass 2
  found **nothing** — because it could be aimed at a specific hypothesis rather than
  sampling more pages. The slide-30 error was a series' x-positions inherited from a
  neighbouring series; pass 2 checked the four other paired-series charts and
  established that each has a genuine *single* categorical axis, so the failure mode
  had no opportunity to occur. Prefer a targeted second pass over a larger one.

- **Mis-hearings get fixed in the transcript body; speaker slips get an `[Ed:]`
  note.** The drafting agent rewrote a spoken "ZeRO stage two" to "stage 3" because
  the slide he was reading makes stage 3 unambiguous. The substance was right and
  the change was still wrong: "two" and "three" are not acoustically confusable, so
  the captions are reliable and it is the *speaker* who slipped. Contrast the same
  transcript's "computation hungry" → "communication-hungry", which was kept — those
  two words are a plausible ASR confusion and the same paragraph ends "very
  communication hungry". **The test is whether the caption could plausibly have
  mis-heard it.** If not, the body records what was said.

- **LaTeX must never enter a transcript.** A drafting agent rendered a spoken "W
  naught" as `$W_0$`. `raw/transcripts/` is a verbatim record and stays spelled
  out; the wiki is where notation gets reconstructed. Same agent also interpolated
  the word "layer" into a sentence the speaker did not put it in. Both reverted.

- **A fourth checker bug, in the transcript ratio script.** Stripping the
  floor-question label with a bare `]*` replace *also* matches inside the
  `**[0:05]**` markers and destroys every one of them, so the ratio check reports
  "105 paragraphs vs 0". Strip labels **per paragraph, after splitting**, and use
  `\]\*(?!\*)` so a marker's `]**` is never touched.

- **Two quote-checker bugs, both new.** Stripping `$...$` LaTeX from the haystack
  desynchronizes exactly like the old `"([^"]{12,})"` quote regex did — an unpaired
  or display-math `$` eats large spans and produces a wave of false failures.
  **Do not strip math from either side.** And tolerate a quote that ends early and
  adds a full stop: that is ordinary truncation, not misquotation, and it is
  otherwise indistinguishable from a real slip in the output.

- **Quote-check yield, lecture 8: 6 real slips in 115 fragments.** A comma where the
  speaker has an em dash; "which involves" for "This then involves"; a quote begun a
  word early; LaTeX reformatted inside quotation marks (twice); and — the one to
  watch for — **the parent's own editorial aside set as a blockquote**, which reads
  as a quotation of the lecturer and is not one.


## Run 9 precedents (lecture 9, a 57-page deck and the dirtiest transcription yet)

Lecture 9 is the most figure-dependent deck in this KB — 93 pasted images against
2,063 words of native text, 36 words per page — and it produced more transcription
errors than the previous four decks combined. What follows is what that cost and
what it taught.

- **Audit yield was 11 dirty pages out of 18, 36 errors.** Runs 8 and 9 of the
  CS224N build set the expectation at roughly two small corrections per deck. That
  expectation does not hold for a deck where the charts *are* the content. Budget
  two audit passes for such a deck from the start rather than discovering the need
  after pass 1.

- **The error that matters is the one that inverts the slide's argument.** On slide
  41 a set of SuperGlue values read systematically low led the file to state that
  the best upstream model falls to mid-pack downstream. It places 2nd-3rd of 13. The
  lecture's actual point is that a *mid-table* model rises. Nothing but an audit
  catches this: the numbers were individually plausible, the prose was fluent, and
  the conclusion was stated with confidence. **When a slide entry ends with "the
  point of this chart is X", audit X against the data, not just the data against the
  page.** That check is now in the audit prompt.

- **Two axes were described as linear that are logarithmic** (slides 49 and 13, the
  latter shared across all three Kaplan Figure-1 panels). Both were settled by
  measuring inter-tick pixel spacing. An axis-scale error silently corrupts every
  value in the entry, so it is worth more than any individual value check. Tell
  auditors to *measure* tick spacing rather than judge scale by eye.

- **"The source image is cropped" is the new "not legible at this resolution".**
  Three readers, on three different pages, explained something they could not read
  by asserting a crop in the source. Two were pure fabrication — the pages are
  complete. The third (slide 41's "NL12-") is a real truncation with a wrong cause
  attached: both instances sit in open white space. **Verify the explanation, not
  just the observation.**

- **False illegibility is now 0-for-every-instance ever tested in this build.**
  Three more on slide 52 — two equation intercepts and a ~19-entry legend — all read
  cleanly at zoom. Treat the phrase in a slide file as a flag, never a fact.

- **Pixel-classification beats eye-reading on colour, and the legend can poison it.**
  Two readers resolved series identity by sampling legend-swatch RGB and calibrating
  axes off detected ticks, and it caught two errors eye-reading had already made. But
  on slide 4 a naive colour trace invented four data points that were **the legend
  swatches themselves, sitting inside the plot's coordinate space**. Mask the
  legend's bounding box before classifying pixels. This is the known "a label is not
  a series" failure wearing a different hat.

- **Audit agents must append per page, exactly like transcription agents.** Run 9's
  third audit pass was killed by a session rate limit and returned *nothing*, because
  it had been told to report findings as text at the end. The four transcription
  agents in the same run all survived interruption-free precisely because they
  appended each chunk. **Instruct audit agents to write findings to a file as they
  finish each page.** A kill should cost one page, not the whole pass.

- **When an audit cannot run, cross-check against the transcript instead — it is
  free.** Every figure the wiki quoted from the seven unaudited pages was checked
  against the copy-edited transcript, which is independently verified. All six
  Chinchilla exponents, Kaplan's 0.73/0.27, the 67B and 63B projections, the 20:1
  ratio and GPT-3's 3:1 are spoken aloud and matched. Only two quoted values were not
  transcript-corroborated, and both are flagged as provisional at the point of use.
  **A number that appears in both the deck and the speech is far better attested than
  one that appears in only the deck** — prefer those when writing prose over an
  unaudited deck.

- **State the reliability boundary in the slide file's own front matter.** Every error
  in both passes was in a chart description; none was in slide text or a native-text
  caption. That asymmetry is worth recording explicitly, because it tells a later
  reader exactly what an unaudited page can still be trusted for.

- **Checker bug, the fifth in this build: strip markers before comparing quotes.**
  A quote that spans an `[MM:SS]` boundary is legitimate — the captions break
  mid-sentence constantly — but a checker that compares against the raw file reports
  every one as a misquotation. Strip `**[MM:SS]**` from the *source* before matching,
  and split the *wiki* quote on ellipses so an elided quotation is checked fragment by
  fragment. Scope the check to all transcripts and slide files, not just the current
  lecture's, or every quote a cross-lecture page inherited is reported as false.

- **Checker bug, the sixth: a question convention can deflate a word-ratio check.**
  Lecture 9's transcript puts each floor question's full text inside an
  `*[Question from the floor: …]*` block, where lecture 8 used a bare label. A
  `clean()` that strips those blocks wholesale deletes real words and pushed two
  paragraphs to 0.66 and 0.68. Retaining the question text put every paragraph inside
  the band at 84.6% retention. **Strip only the label, never the content.**

- **Quote-check yield, lecture 9: 8 real slips in 127 fragments.** Two em dashes
  silently became a comma and a semicolon; one dropped a pair of em dashes inside a
  quotation; two dropped inner quotation marks; one dropped the speaker's "kind of";
  and — the repeat offender from run 8 — **two of the parent's own paraphrases were
  set in quotation marks**, reading as quotations of the lecturer when they were not.
  That failure has now appeared in two consecutive runs. Check every quoted string
  you wrote yourself, not only the ones you copied.
