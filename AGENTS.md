# How this knowledge base is organized

This repo is the knowledge base for **CS336 — Language Modeling from Scratch |
Stanford, Spring 2026 — Percy Liang and Tatsunori Hashimoto**. It is read by
Cairn's in-extension AI chat, which fetches files over `raw.githubusercontent.com`
and follows relative markdown links.

**Coverage: all 18 recorded lectures.** See [`kb.json`](kb.json) and the banner
at the top of [`INDEX.md`](INDEX.md). The course is complete here: every lecture
has a copy-edited transcript and a wiki page. **Lecture 18 is the exception worth
knowing about** — it is the guest lecture, no course material exists for it
anywhere, so its pages rest on the transcript alone, cite timestamps rather than
slides, and carry no images.

One numbering quirk to know: Lecture 2 teaches **resource accounting**, which
belongs to the Systems unit (Lectures 5–8, 10) even though the lecture itself sits
in the Basics block. So [`wiki/course-map.md`](wiki/course-map.md) records
resource accounting under Systems even though the lecture sits earlier; the whole
of that unit (Lectures 5–8 and 10) is covered here.

## Layout

| Path | Contents |
| --- | --- |
| `INDEX.md` | Entry point. Course summary + annotated table of contents. |
| `wiki/` | Durable pages: one per lecture, plus cross-lecture topics, plus one page per covered assignment (`assignment-1-basics.md` to `assignment-4-data.md`). |
| `raw/slides/` | The written course material, transcribed. See the note below — CS336 has no slide numbers for half its lectures, and none of the eight decks prints one either. |
| `raw/handouts/` | The Assignment 1, 2, 3 and 4 handouts, transcribed in full (prose and code verbatim from the PDF text layer, math and figures from page images; Assignments 3 and 4 from span data, with math checked against crops). MIT-licensed, unlike the rest of `raw/`. |
| `raw/transcripts/` | Copy-edited lecture transcripts with `[MM:SS]` paragraph marks. **These are the ones to read.** |
| `raw/transcripts/original/` | **Gitignored, not in the repo.** The verbatim auto-captions are a complete reproduction of the lecture with none of this KB's own work in them, so they are kept locally and not published. Regenerate with `fetch_transcript.py <video_id>` piped through `transcript_to_md.py`; the video id is in each transcript's front matter. |
| `raw/images/NN-<slug>/` | Rendered slide images and the course's own figures. See **Images** below for what is and is not here. |
| `raw/pdfs/` | Gitignored and empty. No binaries are committed; `sources.md` carries canonical URLs. |
| `sources.md` | Every lecture, deck, assignment and linked document, with URLs. |
| `kb.json` | Machine-readable coverage and provenance. Read this to know what this KB does and does not cover, and how far to trust a citation. |
| `SEE_ALSO.md` | Sibling KBs worth reading for this course, by repo URL. |
| `LICENSE.md` | Three layers of copyright, and which one covers what. Read it before reusing anything. |
| `TODO.md` | Open work only, read in full by every run. Unchecked boxes are outstanding work; a run's section moves to `BUILD_LOG.md` once it is pushed. |
| `BUILD_LOG.md` | The build record: precedents by topic, then the run log, one section per run, with what each found, decided and got wrong. Precedent for builders; never read by the chat. |

## What is special about CS336

**Half the lectures are Python programs, not slide decks.** Percy Liang's lectures
(1, 2, 6, 7, 10, 12, 13, 14, 17) — **all nine of which are transcribed here** — are *executable lectures* — programs whose
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

For the decks (lectures 3, 4, 5, 8, 9, 11, 15 and 16), the normal rules apply: read the
pages visually, transcribe per slide, and audit the figure descriptions. What earlier runs
learned doing that is under [Lessons from earlier runs](#lessons-from-earlier-runs), and
the per-lecture record behind it is in [`BUILD_LOG.md`](BUILD_LOG.md).

`kb.json` reports `method: mixed`; the per-lecture method is in `materials.byLecture`.

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
- **Figures are described for the eight deck lectures — 3, 4, 5, 8, 9, 11, 15
  and 16 — and not for the executable ones.** Those eight decks were read as page
  images, so their `raw/slides/*.md` files describe every figure in prose and wiki
  pages may cite them. Lectures 1, 2, 6, 7, 10, 12, 13, 14 and 17 were
  transcribed from source text, so their `image()` targets are recorded **by path
  only, with no description**, and no wiki claim may rest on one — except lectures
  10, 12, 13, 14 and 17, whose course-repo images *were* opened and described, and
  whose slide files say so. Check which kind of lecture you are citing before quoting a
  figure.

  Any value given on outside confirmation rather than read off the page is marked so in
  the entry itself; [`BUILD_LOG.md`](BUILD_LOG.md) records the one case.
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
- **Assignment pages describe, never solve.** `wiki/assignment-N-*.md` pages are
  built from the handout and starter repo, not a recording. It cites handout
  **section numbers and problem names**, pins the handout version and repo commit
  (handouts change mid-quarter), and gives no answer to any problem: the course's
  AI policy and the repo's `AGENTS.md` forbid AI from implementing any part, and
  the chat is an AI tool. Never compute a problem's answer, even one that looks
  like plain arithmetic (a parameter count, a FLOP total). Read the repo's tests,
  not only `adapters.py`: Assignment 2's handout misnames one adapter and never
  mentions two that its FSDP test calls, and a student finds that out only by failing.
  Where the repo has no tests for student code, read its validators instead: Assignment
  3's handout allows grouped-query attention that its config schema rejects. To learn
  which changelog entries touched the handout, diff the text of every PDF version in the
  repo's history; a code change can leave the handout describing the old behaviour.
  Check what the handout tells students to fetch, and what it says a fixed config is,
  against the repo: Assignment 4's two download addresses return 404 while its download
  script fetches a different file, and its "GPT-2 small-shaped" model has the dimensions
  Assignment 1's handout gives for GPT-2 medium.
- **Transcribing a handout: two sources, two jobs.** For a text-native PDF (the
  Assignment 1 handout is Typst output), copy prose and code **verbatim from the
  text layer** and read only math, figures and layout from page images. A reader
  retyping 19k words from images would introduce typos; the text layer has none.
  The payoff is a near-free fidelity check: align the transcription's non-math
  words against the text layer and read every difference of three or more words.
  On Assignment 1 every one was a relocation, never a rewording. Two failure
  modes it caught or that rendering caught: a page marker dropped inside a box,
  splitting a code fence; and GitHub refusing math inside footnote definitions,
  so footnotes are superscript digits plus a *Footnote N:* line, not `[^N]`.
  Hand readers two more facts from the PDF rather than letting them judge: which spans
  are code (every span in the monospace font, by font name), and which headings are
  numbered — the PDF outline numbers run-in headings (4.2.1.1) that the page prints
  without a number, and the transcription follows the page.
  A handout with no figures needs no reader: font names give code, bold and
  italic; span size and baseline give sub- and superscripts; the gap between lines gives
  paragraphs, and blank lines inside code; the PDF's drawn rectangles give problem boxes
  and code blocks. At 16 pages (Assignment 4) a script turned that into the draft, and
  only math and structure were fixed by hand. Still check each formula against a crop of
  its line. On Assignment 3 that caught a subscript's extent misread from span data that
  had recorded it correctly. Compare each printed URL with its link target too: Typst
  sets `~` as a non-breaking space, so Assignment 4 prints `edu/ ullman` for `~ullman`.
- **Math in LaTeX**: `$...$` inline, `$$...$$` displayed. Renders both in Cairn's
  chat and on github.com. Never inside a code fence — that shows the source instead
  of the formula. Define each symbol on first use, and follow the course's own
  notation rather than a tidier one, since the learner is comparing the page
  against the lecturer's screen.
- **Transcripts stay spelled-out.** Captions render notation as speech ("n
  squared"); reconstructing it as $n^2$ belongs in `wiki/`, not `raw/transcripts/`.
- **No rankings between lectures.** Do not write that a lecture has the most
  figures, is the richest, densest or thinnest, or is "next with 22". A comparison
  like that answers no question a reader has, duplicates the per-lecture table in
  this file and `byLecture` in `kb.json`, and is false the moment coverage changes —
  which is how this KB ended up asserting that two different decks were each "the
  most figure-dependent deck in this KB". State the measurement for the lecture in
  front of you and let the table hold the comparison. Superlatives that are
  *actionable* and settled are fine — "the only lecture without images", "the only
  two decks with no figure audit" — because they change what a reader does.

## Images

Every lecture 1-17 has images — **lecture 18 has none.** They are committed, not
hotlinked, and they are the only part of this KB that redistributes course
material rather than pointing at it.

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
| 10 Inference | 22 | the course's own `images/*.png` |
| 11 Scaling Laws in the Wild | 32 of 58 pages | rendered from `lecture_11.pdf` |
| 12 Evaluation | 33 | the course's own `images/*.png` |
| 13 Data I: Sources, Datasets | 14 | the course's own `images/*.png` |
| 14 Data II: Filtering, Dedup, Mixing | 13 | the course's own `images/*.png` |
| 15 Mid/Post-Training | 38 of 65 pages | rendered from `lecture_15.pdf` |
| 16 Post-Training: RLVR | 47 of 61 pages | rendered from `lecture_16.pdf` |
| 17 Multimodality | 32 | the course's own `images/*.png` |

**Lecture 18 has no images, and never will.** No course material exists for it, so never
construct an image URL for it from the pattern of the other seventeen.

**Third-party images are not redistributed.** Where a lecture hot-links an image from
another site, its slide file records the bare URL at the point it appears, with no
description, because nobody here has looked at it. Give the URL; never describe one.

**Leaderboard screenshots are dated.** Every ranking in one is a snapshot from when the
lecture was prepared; cite it as "as of the lecture", never as a current standing.

**Lectures 15 and 16 are the only page-image decks with no figure audit**, skipped at the
user's instruction to save cost. Treat their chart values as less certain; both front
matters and `kb.json` say so.

The notes behind each of these, lecture by lecture, are in [`BUILD_LOG.md`](BUILD_LOG.md).

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

**`wiki/` carries a subset; `raw/slides/` carries all 490.** Every image sits under its
`## Slide N` heading in the slide file, and a wiki page embeds one only where it cites
that slide — so a lecture's wiki page showing few images, or none, is never evidence that
the images are missing. To list every image a lecture has, read the slide file:
`grep -o '\.\./images/[^)]*' raw/slides/10-inference.md`.

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

A deck page is rendered when the PDF carries a raster covering more than 4% of the page
**and** the slide file's own prose describes an actual figure there. Neither signal is
sufficient alone, and both are overruled by hand where they disagree: vector diagrams
with no pasted raster are missed by the PDF test, and pasted rasters are often tables,
equation blocks or a paper's title block that the slide file already reproduces. A
pasted image is not a figure; ask what is *in* it.

The rule worth remembering: **a transcribed table beats a picture of one**, because it can
be quoted, searched and cited by cell. Title cards, outline slides, section dividers and
pure-text bullets are not rendered either. The one deliberate exception is a figure whose
*shape* is the argument, such as a heatmap's loss basin, which is rendered even though its
cells are transcribed.

An executable lecture has no deck to sample from: take every image the program passes to
`image()` from the course's own repository, since a program only calls it where it wants a
figure shown, and filter on *ownership*, not content. Images with no course content (a
staff photo grid) may be skipped.

Which slides each lecture rendered, and why each skipped page was skipped, is in
[`BUILD_LOG.md`](BUILD_LOG.md).

`raw/slides/` holds an image for **every** page that was rendered. `wiki/` holds one only
where a page cites that slide, which is the KB's convention anyway — so most images are
slide-file-only, and that is the intended outcome rather than a shortfall. Do not add a
citation merely to give an image a home.

### Provenance and attribution

Every image comes from Stanford CS336 (Spring 2026) and its lecture repository,
<https://github.com/stanford-cs336/lectures>, which carries no licence file. The copyright
position, the third-party figures the decks reproduce, and the attribution are in
[`LICENSE.md`](LICENSE.md); how each lecture's images were produced is in
[`BUILD_LOG.md`](BUILD_LOG.md). If the course, or an author of a reproduced figure, asks
for something to come down: delete the file and every Markdown image line that points at
it, and note it in `kb.json`.

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

When a run teaches something new, write the lesson here in general form and the run's
own record — what it found, decided and got wrong, page by page — in
[`BUILD_LOG.md`](BUILD_LOG.md). This file is loaded into every step of every build agent;
`BUILD_LOG.md` is read only when someone wants precedent.

## Lessons from earlier runs

Each lesson here is stated in general form. The run that taught it, with its numbers and
the pages involved, is in [`BUILD_LOG.md`](BUILD_LOG.md); read that when you want the
evidence behind a rule, or precedent for a deck like one already done.

### Reading a deck

- **Settle the slide numbering in the parent, before any page is read, and hand the
  readers the conclusion.** `slide_number_map.py` can be fooled by a deck that prints no
  folios — it once read a fraction's numerator as a page number — and then `--verify`
  cannot check the file. When a deck prints none, label slides by PDF page, say so in the
  front matter, and check a plain `1..N` heading sequence instead.
- **Split a long deck across parallel readers, each appending per page to its own file,
  and reconcile in the parent.** It keeps every context small, a kill costs one page, and
  several readers independently confirming a deck-wide fact is stronger evidence than one.
- **Confirm page attribution from the text layer, not page images.** Match each
  `## Slide N — Title` against page N's title, found as the topmost band's largest spans by
  position and font size, never as the first string in the content stream.
- **A source can contradict itself; record both sides as printed.** Decks contradict their
  own labels and headings. When an executable lecture's prose disagrees with its code,
  computed values follow the code. Flag it in `raw/slides/` and on the topic page.
- **A repeated figure or table is a free second opinion.** Read separate write-ups of the
  same material against each other, cell by cell where possible.

### Auditing figures

- **Audit the pages the wiki quotes, and the pages the transcriber was confident about, not
  only the ones it flagged.** Errors cluster in charts with legends and in dense tables, but
  substantive errors have sat on pages reported clean.
- **The error class moves between decks, so do not tune the audit to the last one.** It
  has been misread values; structural slips (miscounted arrows, wrong colours, a figure
  never mentioned, a false claim of uniformity); and, most damaging, the interpretive
  sentence laid over correct data. Audit "the point of this chart is X" against the data.
  Reading an entry against its own table a few lines above is the cheapest check there is.
- **A stronger reader model moves the errors; it does not remove them.** Budget the audit
  the same whatever model read the deck.
- **Check every assertion about another slide, including in the front matter the parent
  writes.** Cross-slide claims are a distinct error class, and the aggregating front matter
  is the least-checked layer of a slide file.
- **Measure, do not judge.** Axis scale by tick spacing. Series identity by sampling
  legend-swatch and bar colours, with the legend's bounding box masked so the swatches are
  not read as data. Check that every figure described is on the page and every figure on
  the page is described.
- **"Not legible at this resolution" and "the source image is cropped" are flags, not
  facts.** Every instance tested in this build was false, or true with the wrong cause.
  Verify the explanation, not only the observation.
- **Verify an auditor's findings, and any subagent's summary of its own work, before
  applying them.** Before recording a defect in the *source*, ask which part of the claim
  your evidence actually covers.
- **Audit agents append per page, like readers.** One that reports only at the end returns
  nothing when it is killed.
- **When an audit cannot run, cross-check quoted values against the transcript.** A number
  both printed and spoken is far better attested than one only printed. State the
  reliability boundary in the slide file's front matter and in `kb.json`.
- **Prefer a targeted second pass to a larger one**, aimed at a hypothesis the first pass
  raised. When to stop is under [Run order](#run-order-set-after-run-12).

### Executable lectures

- **Look for published runtime output before writing "not reproduced".** A course may
  commit a program's stdout from a real run. Quote it as `(recorded run)`, separate from
  `(computed)`, and re-derive whatever can be derived from it.
- **An executable lecture may not be traceable.** One that launches processes runs
  single-process under the tracer; read the source, not the trace, and say so.
- **Machine-dependent values can be most of a lecture.** Never supply a plausible number for
  a measurement the lecture makes in words.
- **Check a catalog title against the material, in both directions.** A title can name
  content the lecture never covers, and two lectures can share a title.

### Transcripts and quotations

- **Mis-hearings are fixed in the body; speaker slips get an `[Ed:]` note.** The test is
  whether the caption could plausibly have misheard it. False starts are preserved, not
  completed, and LaTeX never enters a transcript. Ask the drafting agent which paragraphs
  it changed and whether every word traces to the captions.
- **Quote-check every quoted string in a new page, including the ones you wrote.** Expect
  real slips every run: false starts smoothed inside quotation marks, punctuation changed,
  and, repeatedly, the parent's own paraphrase or editorial aside set as a quotation. Pair
  quote characters sequentially; strip `[MM:SS]` markers from the source; split quotes on
  ellipses; tolerate a quote that ends early with a full stop; never strip math from either
  side; and search every transcript and slide file, not only the current lecture's.

## Run order, set after run 12

**Transcript and wiki come immediately after the slide transcription — before images, and
before any figure auditing.**

Run 12 did it the other way round and the result is the lesson. It transcribed lecture 11's
58-page deck, rendered 32 figures, and ran two audit passes over 15 pages — and left the
lecture with no transcript and no wiki page. That is the least useful shape a lecture can be
left in. The deck is supporting material; the wiki page is what a learner reads and what the
chat quotes. The run bought the cheap half first and ran out of budget before the valuable
half.

The order for a deck-based lecture:

1. **Slide transcription** first — the transcript edit is cross-checked against the deck, so
   the deck has to exist.
2. **Transcript, then wiki, immediately.**
3. **Images** after that.
4. **One figure audit pass, then stop.** Two passes over 15 of lecture 11's pages cost more
   than transcribing all 58, and the second pass's yield was mostly refinement — tenths of a
   percent — alongside a handful of real faults. Prefer auditing **the pages the wiki
   actually quotes** over the pages that merely look chart-heavy: a wrong value nobody cites
   is cheaper to leave flagged than to hunt down.

The "a dirty sample means the sample was too small" rule still stands as a description of
risk. It is not a budget authority. When it and the budget disagree, state the boundary in
the slide file's front matter and in `kb.json`, and stop — an acknowledged gap beats an audit
spent past the point of diminishing return.
