# KB build — CS336 (Language Modeling from Scratch, Stanford, Spring 2026)

Coverage after this run: **Lectures 1, 2 and 3.** The course has 18 recorded
lectures. See `kb.json` for machine-readable coverage.

## Run 1 — Lecture 1 (complete)

- [x] 01 Overview, Tokenization — video JuoVZkPBiKk (verbatim captions fetched)
- [x] 01 Overview, Tokenization — copy-edited transcript (3 checks pass)
- [x] raw/slides/01-overview-tokenization.md — transcribe lecture_01.py
- [x] Fetch course site index: https://cs336.stanford.edu (25 pages, 41 external docs)
- [x] Write sources.md
- [x] wiki/01-overview-tokenization.md
- [x] Topic pages — tokenization, byte-pair-encoding, efficiency, scaling-laws,
      course-map, executable-lectures (6)
- [x] INDEX.md table of contents
- [x] kb.json, SEE_ALSO.md, push, PATCH kbUrl (catalog 94d9c003-2193-43e7-96e3-c1cb4ed0aba8)

## Run 2 — Lecture 2: PyTorch, Resource Accounting

Video kuYAsz7zspQ (77 min). Executable lecture (`lecture_02.py`, 856 lines) —
source-text, no deck.

### Course material
- [x] raw/slides/02-pytorch-resource-accounting.md — transcribe lecture_02.py
      (1187 lines; runtime @inspect values recomputed, GPU-dependent ones marked
      machine-dependent and not reproduced)

### Transcript
- [x] 02 — verbatim captions fetched (101 paragraphs, ~12.2k words)
- [x] 02 — copy-edited transcript (drafted by Sonnet, adjudicated here)
- [x] 02 — verify: all three checks pass. Timestamps: 101 markers, identical
      sequence. Numbers: every difference accounted for (exponent joins, spoken
      dimension names -> seq1/hidden2, one caption stutter, one 1989->1979 fix
      confirmed by source). Word ratios: 4 outliers at 0.66-0.72 read and
      confirmed as filler removal, 1 at 1.22 is an inserted [Ed:] note.
      Adjudication restored 2 dropped passages, reverted 1 unsafe number
      restoration, and added 8 [Ed:] notes. All 24 restored proper nouns appear
      verbatim in the lecture source.

### Wiki
- [x] wiki/02-pytorch-resource-accounting.md (408 lines)
- [x] Topic pages (8 new) — resource-accounting, flops-and-mfu,
      arithmetic-intensity, training-flops, memory-accounting-for-training,
      precision-and-data-types, einops, activation-checkpointing
- [x] Update existing pages that now link a second lecture — efficiency,
      course-map, executable-lectures, AGENTS.md
- [x] INDEX.md — rewritten for two-lecture coverage
- [x] Link sweep — 222 relative links and 71 anchors all resolve

### Publish
- [x] Update sources.md (lecture_02.py now transcribed)
- [x] kb.json — coverage 2/18, 14 topic pages
- [x] Commit and push
- n/a  kbUrl already set on the catalog entry from run 1; no re-link needed

## Run 3 — Lecture 3: Architectures

Video lVynu4bo1rY (89 min). Tatsunori Hashimoto. **First PDF-deck lecture in this
KB** — `lecture_03.pdf`, 67 pages, so `page-images` rather than `source-text`.

Numbering: the deck prints **no page numbers on any page**. `slide_number_map.py`
reported "printed numbers run 1-1" from a stray `1` on page 61 that is the
numerator of the MQA arithmetic-intensity fraction, not a folio. Mapping is a
plain 1..67, page N = slide N, and `--verify` must use the Python fallback.

### Course material
- [ ] raw/slides/03-architectures.md — read all 67 pages (Opus), append per chunk
- [ ] Figure audit — 6 figure/table-heavy pages checked against the PDF (Sonnet)
- [ ] Heading-sequence check (1..67, fallback for --verify)

### Transcript
- [x] 03 — verbatim captions fetched (117 paragraphs, ~17.1k words)
- [ ] 03 — copy-edited transcript (Sonnet drafts, adjudicated here)
- [ ] 03 — verify: timestamps, number inventory, per-paragraph word ratios

### Wiki
- [ ] wiki/03-architectures.md
- [ ] Topic pages (new)
- [ ] Update existing pages that now link a third lecture
- [ ] INDEX.md — three-lecture coverage
- [ ] Link sweep

### Publish
- [ ] Update sources.md (lecture_03.pdf now transcribed)
- [ ] kb.json — coverage 3/18, materials.method now mixed
- [ ] AGENTS.md — deck conventions now live, not hypothetical
- [ ] Commit and push

## Not done (future runs)
- [ ] Lectures 4–18 — transcripts and wiki pages
- [ ] Transcribe the 7 remaining PDF decks (lectures 4, 5, 8, 9, 11, 15, 16) — these need
      page-images, not source-text, and a figure audit
- [ ] Transcribe the 7 remaining executable lectures (6, 7, 10, 12, 13, 14, 17)
- [ ] Describe the figures in lectures 1 and 2 — the image() targets are recorded
      by path in raw/slides but their contents were never looked at
