# How this knowledge base is organized

This repo is the knowledge base for **CS336 — Language Modeling from Scratch |
Stanford, Spring 2026 — Percy Liang and Tatsunori Hashimoto**. It is read by
Cairn's in-extension AI chat, which fetches files over `raw.githubusercontent.com`
and follows relative markdown links.

**Coverage: Lectures 1, 2, 3 and 4 of 18.** See [`kb.json`](kb.json) and the banner
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
| `raw/slides/` | The written course material, transcribed. See the note below — CS336 has no slide numbers for half its lectures, and the one deck transcribed so far prints none either. |
| `raw/transcripts/` | Copy-edited lecture transcripts with `[MM:SS]` paragraph marks. **These are the ones to read.** |
| `raw/transcripts/original/` | Verbatim auto-captions, kept as the record of what was said, plus the raw caption segment JSON they were generated from. |
| `raw/pdfs/` | Gitignored and empty. No binaries are committed; `sources.md` carries canonical URLs. |
| `sources.md` | Every lecture, deck, assignment and linked document, with URLs. |
| `kb.json` | Machine-readable coverage and provenance. Read this to know what this KB does and does not cover, and how far to trust a citation. |
| `SEE_ALSO.md` | Sibling KBs worth reading for this course, by repo URL. |
| `TODO.md` | Build tracker. Unchecked boxes are outstanding work. |

## What is special about CS336

**Half the lectures are Python programs, not slide decks.** Percy Liang's lectures
(1, 2, 6, 7, 10, 12, 13, 14, 17) are *executable lectures* — programs whose
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
3 and 4 are the ones done so far**, and they set four precedents worth following:

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

`kb.json` reports `method: mixed`, because Lectures 1–2 are `source-text` while
Lectures 3–4 are `page-images`; the per-lecture breakdown is in
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
- **Figures are described for Lectures 3 and 4, and not for Lectures 1–2.** Those
  two decks were read as page images, so `raw/slides/03-architectures.md` and
  `raw/slides/04-attention-alternatives.md` describe every figure in prose and wiki
  pages may cite them. Lectures 1 and 2 were transcribed from source text, so their
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

## Rebuilding

Built and updated by the `cairn-kb` skill. To add newly released lectures or
extend coverage, append entries to `TODO.md` and re-run the skill — it only does
unchecked work. Update `kb.json` whenever coverage changes; a stale `kb.json` is
worse than none.
