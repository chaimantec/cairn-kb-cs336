# KB build — CS336 (Language Modeling from Scratch, Stanford, Spring 2026)

Coverage after this run: **Lectures 1 and 2.** The course has 18 recorded
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
- [x] 02 — verbatim captions fetched (78 paragraphs, ~12.2k words)
- [ ] 02 — copy-edited transcript
- [ ] 02 — verify: timestamp diff, number inventory, per-paragraph word ratios

### Wiki
- [ ] wiki/02-pytorch-resource-accounting.md
- [ ] Topic pages — resource accounting, arithmetic intensity / roofline,
      precision and data types, einops, memory accounting for training,
      activation checkpointing + gradient accumulation
- [ ] Update existing topic pages that now have a second lecture to link
      (efficiency, course-map, executable-lectures)
- [ ] INDEX.md — add lecture 2 and its topic pages

### Publish
- [ ] Update sources.md (lecture_02.py now transcribed)
- [ ] kb.json — coverage 2/18
- [ ] Commit and push

## Not done (future runs)
- [ ] Lectures 3–18 — transcripts and wiki pages
- [ ] Transcribe the 8 PDF decks (lectures 3, 4, 5, 8, 9, 11, 15, 16) — these need
      page-images, not source-text, and a figure audit
- [ ] Transcribe the 7 remaining executable lectures (6, 7, 10, 12, 13, 14, 17)
- [ ] Describe the figures in lectures 1 and 2 — the image() targets are recorded
      by path in raw/slides but their contents were never looked at
