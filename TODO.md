# KB build — CS336 (Language Modeling from Scratch, Stanford, Spring 2026)

Coverage after this run: **Lectures 1, 2, 3 and 4.** The course has 18 recorded
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
- [x] raw/slides/03-architectures.md — all 67 pages read (Opus), 2058 lines
- [x] Figure audit — 8 pages (11, 12, 46, 50, 52, 55, 63, 67) checked against the
      PDF at 600-1200 dpi. 6 clean; 3 errors found and fixed: slide 67's MoE
      column is checked on GPT4 and Mixtral (not "unchecked on every row") and
      its Parametrization column carries "MuP" on the Phi3 row (not "empty on
      every row"); slide 55's "Norms" annotation is slide-native text layered ON
      TOP of the meme, visible, not covered by it. The 43x6 numeric table on 67
      was exact cell for cell, as was Table 3 on 63.
- [x] Cross-view table check — slides 7, 29, 51 and 67 are four views of the same
      model database, read in four separate chunks. 296 overlapping (model,
      column) cells compared: 0 disagreements. No second audit run; the sample
      was 6/8 clean and the one error class (a sparse column asserted uniformly
      empty) does not recur on the sibling views, which name their exceptions.
- [x] Heading-sequence check — PASS, exactly 67 headings 1..67 in order

### Transcript
- [x] 03 — verbatim captions fetched (117 paragraphs, ~17.1k words)
- [x] 03 — copy-edited transcript (drafted by Sonnet, adjudicated here)
- [x] 03 — verify: all three checks pass. Timestamps: 117 markers, identical
      sequence. Numbers: 5 differences, all adjudicated — a dropped duplicate "1"
      from the "T5 1.v v1.1" caption stutter, "Gemma's two, three, and four"
      written as digits, and "2"/"11" from an [Ed:] note citing Falcon 2 11B.
      Word ratios: one outlier at 1.11, which falls to 0.93 once the 26-word
      [Ed:] note inserted in that paragraph is excluded. Adjudication reverted
      one unsafe restoration ("Nemotron-4 340B" back to "Nemotron 340B" — slide
      26 prints "Nemotron 340B" and he was reading it) and corrected one wrong
      evidence citation in the header (P-RoPE). All 21 restored proper nouns
      appear verbatim in the deck; 9 [Ed:] notes mark genuine ambiguity.

### Wiki
- [x] wiki/03-architectures.md (249 lines)
- [x] Topic pages (8 new) — model-architecture-survey, pre-norm-and-post-norm,
      rmsnorm, gated-activations, rope, transformer-hyperparameters,
      training-stability, attention-variants
- [x] Update existing pages that now link a third lecture — arithmetic-intensity,
      flops-and-mfu, efficiency, memory-accounting-for-training, tokenization,
      scaling-laws, executable-lectures, course-map
- [x] INDEX.md — three-lecture coverage, new Lecture 3 section, raw-material and
      citation sections rewritten for the two material formats
- [x] Link sweep — 323 relative links and 19 anchors all resolve; every one of the
      25 wiki pages appears in INDEX.md; no LaTeX inside code fences

### Publish
- [x] Update sources.md (lecture_03.pdf now transcribed)
- [x] kb.json — coverage 3/18, 22 topic pages, materials.method now "mixed" with a
      per-lecture breakdown, figuresAudited true
- [x] AGENTS.md — deck conventions now live, not hypothetical; figures rule split
      by lecture format; timestamp check widened to [H:MM:SS]
- [x] Commit and push

## Run 4 — Lecture 4: Attention Alternatives and Mixture of Experts

Video cKSwj_qZ8Jg (86 min). Tatsunori Hashimoto. Second PDF-deck lecture —
`lecture_04.pdf`, 60 pages, `page-images`.

Numbering: as with lecture 3, the deck prints **no page numbers on any page**.
`slide_number_map.py` found no printed folio in either bottom corner on any of
the 60 pages and fell back to a bare 1..60. Page N = slide N; `--verify` must use
the Python heading-sequence fallback.

Density: 102 embedded raster images across 60 pages, and most pages carry only
10-40 words of native text — nearly the whole deck is pasted paper figures and
tables. Read at Opus rather than the Sonnet default for that reason (user's call).

### Course material
- [x] raw/slides/04-attention-alternatives.md — all 60 pages read (Opus, three
      agents over pages 1-20/21-40/41-60), 2370 lines
- [x] Numbering confirmed by eye, not just by script — all three readers checked
      every page and a text-layer scan for isolated 1-3 digit strings returned
      zero hits. No folio anywhere; the only corner numerals in the deck are
      citation brackets and equation tags belonging to pasted paper figures.
- [x] Figure audit — 8 pages (6, 11, 16, 20, 33, 35, 43, 52) checked against the
      PDF at 600-2200 dpi. 5 clean; 9 corrections applied on pages 6, 11 and 43.
      All 9 were chart values read slightly off (the worst a systematic ~0.13
      offset across one OLMoE validation-loss panel, from reading a linear x-axis
      as if it were spaced otherwise). NO structural errors: nothing fabricated,
      no series transposed, and all four audited tables — page 20's Llama 4
      screenshot, page 35's 12-row native routing table, page 52's MiniCPM
      Table 6, page 6's ablation table — were exact cell for cell. Every claim
      the transcribers had themselves flagged as uncertain proved correct.
- [x] Slide 3 — FlashAttention-2 at 1k sequence length is 153 TFLOPs/s. The
      legend box overprints the label so no render could resolve it; supplied by
      the user and recorded as externally confirmed rather than read off the page.
- [x] Heading-sequence check — PASS, exactly 60 headings 1..60 in order

### Known deck self-contradictions (transcribed as printed, flagged inline)
- Slide 47: the printed Expert-Indices vector is `1 2 0 2 1 2`, but stages 3-4
  label "brown" as Expert-2's token and "quick"/"fox" as Expert-0's — experts 0
  and 2 are swapped relative to the vector.
- Slide 56: the heading says "Shared (1)" for v3 while the pasted DeepSeekMoE
  diagram still shows two shared experts, and the bold line reads "V2 (671B - 37
  active)" under a v3 title.
- Slide 39: the code screenshot has `router_logits += mtf.random_uniform(...,
  minval=1-eps, maxval=1+eps)` — jitter drawn around 1 but applied additively.
- Slides 16 and 17 both plot Switch-Base runs but use different colour
  assignments and different series counts; they must not be cross-quoted by
  colour.
- Slide 20's Llama 4 table screenshot is cropped mid-row below GPQA Diamond in
  the source image (confirmed by the audit).

### Transcript
- [x] 04 — verbatim captions fetched (113 paragraphs, ~16.9k words)
- [ ] 04 — copy-edited transcript
- [ ] 04 — verify: timestamp sequence, number inventory, per-paragraph word ratios

### Wiki
- [ ] wiki/04-attention-alternatives.md
- [ ] Topic pages (linear attention, Mamba/SSMs, sparse attention, mixture of
      experts, MoE routing, load balancing, upcycling, MLA/MTP — final list TBD)
- [ ] Update existing pages that now link a fourth lecture
- [ ] INDEX.md — four-lecture coverage
- [ ] Link sweep

### Publish
- [ ] Update sources.md (lecture_04.pdf now transcribed)
- [ ] kb.json — coverage 4/18
- [ ] Commit and push
- n/a  kbUrl already set on the catalog entry from run 1; no re-link needed

## Not done (future runs)
- [ ] Lectures 5–18 — transcripts and wiki pages
- [ ] Transcribe the 6 remaining PDF decks (lectures 5, 8, 9, 11, 15, 16) — these need
      page-images, not source-text, and a figure audit
- [ ] Transcribe the 7 remaining executable lectures (6, 7, 10, 12, 13, 14, 17)
- [ ] Describe the figures in lectures 1 and 2 — the image() targets are recorded
      by path in raw/slides but their contents were never looked at. Lecture 3 now
      sets the precedent for how (page-images plus a targeted figure audit), but
      lectures 1-2 display images by URL rather than as PDF pages, so the mechanics
      differ.
