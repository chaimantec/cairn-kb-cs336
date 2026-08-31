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
| `raw/slides/` | The written course material, transcribed. See the note below — CS336 has no slide numbers for half its lectures, and none of the three decks transcribed so far prints one either. |
| `raw/transcripts/` | Copy-edited lecture transcripts with `[MM:SS]` paragraph marks. **These are the ones to read.** |
| `raw/transcripts/original/` | Verbatim auto-captions, kept as the record of what was said, plus the raw caption segment JSON they were generated from. |
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

For decks (lectures 3, 4, 5, 8, 9, 11, 15, 16), the normal rules apply: read the
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
