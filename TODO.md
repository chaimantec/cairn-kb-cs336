# KB build — CS336 (Language Modeling from Scratch, Stanford, Spring 2026)

Coverage after this run: **Lectures 1, 2, 3 and 4.** Lecture 5's raw material
(deck and transcript) is committed and verified, but it has NO wiki page yet, so
kb.json still reports 4 of 18 — deliberately, not by oversight. Run 5 was halted
after the transcript step. The course has 18 recorded
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
- [x] 04 — copy-edited transcript (drafted by Sonnet, adjudicated here)
- [x] 04 — verify: all three checks pass. Timestamps: 113 markers, identical
      sequence, both [MM:SS] and [H:MM:SS] forms. Numbers: 6 differences, all
      adjudicated — "Open as 03"->"OpenAI o3", "AIQ"->"AI2", "llama four"->
      "Llama 4" (x2), "one to N"->"1 to N", "V1 and twos"->"V1's and V2's", plus
      an [Ed:] note citing slide 52's printed 13.6B against the 13.4B he says.
      Word ratios: 3 outliers at 0.68-0.72, all read against the original and
      confirmed filler-only; this speaker's "you know"/"sort of"/"kind of" tics
      put whole-transcript retention at ~84% vs lecture 3's ~90%.
      All 13 restored proper nouns appear verbatim in the deck EXCEPT "AI2",
      which the draft itself flagged as contextual rather than a deck string —
      the deck prints "Allen Institute for AI" on slide 14, so the abbreviation
      is well-founded. 5 [Ed:] notes mark genuine ambiguity. One non-verbal
      caption artifact ("[clears throat and snorts]" at 43:12) was dropped as
      caption machinery and the header says so.
      Note: the verification script's first draft stripped whole
      "[Question from the floor: ...]" markers before counting words, which hid
      5 paragraphs' real ratios behind false outliers. Only the LABEL is an
      insertion; a quoted question is transcribed speech and must still count.

### Wiki
- [x] wiki/04-attention-alternatives.md (330 lines)
- [x] Topic pages (10 new) — linear-attention, state-space-models, sparse-attention,
      mixture-of-experts, moe-routing, load-balancing-losses, expert-parallelism,
      upcycling, multi-head-latent-attention, multi-token-prediction
- [x] Update existing pages that now link a fourth lecture — attention-variants
      (new "Where lecture 4 takes this" section), training-stability (MoE router
      softmax, fp32 router, router z-loss), efficiency, executable-lectures (two
      decks now, both unnumbered), model-architecture-survey (points at slide 35's
      MoE table as the sparse-model companion), course-map (Unit 1 architecture and
      Training bullets now resolve to lecture 4 pages)
- [x] INDEX.md — four-lecture coverage, new Lecture 4 section with 10 annotated
      entries, raw-material and citation sections updated for two decks
- [x] Link sweep — 513 relative links and 73 anchors all resolve; all 36 wiki pages
      appear in INDEX.md; no LaTeX inside code fences.
      Note: the first anchor checker reported 35 false failures because it collapsed
      whitespace runs when slugging a heading. GitHub does not — it strips the em
      dash in "Unit 2 — Systems" and turns each remaining space into its own hyphen,
      giving `unit-2--systems`. Replace each space individually; do not use `\s+`.

### Publish
- [x] Update sources.md (lecture_04.pdf now transcribed)
- [x] kb.json — coverage 4/18, 32 topic pages, byLecture."4" = page-images,
      slideDecks transcribed 2 of 8, 13 caveats
- [x] AGENTS.md — four deck precedents including parallel readers and the
      audit-the-confident-pages lesson; transcript checker gotchas recorded
- [x] Commit and push
- n/a  kbUrl already set on the catalog entry from run 1; no re-link needed
- n/a  kbUrl already set on the catalog entry from run 1; no re-link needed

## Run 5 — Lecture 5: GPUs, TPUs

Video izZba4UA7iY (79 min). Tatsunori Hashimoto. PDF deck — `lecture_05.pdf`,
55 pages, `page-images`.

Numbering: derived in the parent BEFORE any page was read. `slide_number_map.py`
reports no printed number on any of the 55 pages. A corner-position text-layer
scan returns four bare digit tokens — `32` (p11), `8` (p35), `5` (p40), `2` (p44)
— all mid-page content, none in a folio position. Mapping is a plain 1..55,
page N = slide N; `--verify` must use the Python heading-sequence fallback.

Density: 83 embedded images across 55 pages against only ~1,811 words of native
text (~33 words/page). As in lecture 4, the figures ARE the content.

### Course material
- [x] raw/slides/05-gpus-tpus.md — three parallel Sonnet readers over pages
      1–19 / 20–37 / 38–55, appending incrementally; parent concatenates and
      writes front matter and the section table
- [x] Figure audit, pass 1 — pages 6, 10, 17, 27, 29, 46, 47 at 600–2400 dpi.
      Clean: 17, 29, 47. Nine corrections applied across 6, 10, 27 and 46, all in
      diagram structure and hand-drawn overlays (a five-region colour wash called
      four, an annotation with no arrow described as having one, die-edge bars
      said to run on all four edges when they run on two, a uniformity claim true
      of one block in four, a 4x8 grid in eight colour groups called four, a
      legend in the wrong corner, two overlay arrows where there are three) plus
      one whole figure on slide 27 that had gone unmentioned. Every data series,
      bit-field table and numeric spot-check matched the source exactly.
      Confirmed as printed, not transcription error: slide 17's "M80" and slide
      29's "E4M3" against slide 27's "E8M0".
- [~] Figure audit, pass 2 — IN PROGRESS over pages 9, 14, 20, 21, 33, 48, 55.
      First attempt was killed by an API error after page 9 and its findings were
      lost with it; the agent was resumed and now appends each page's verdict to
      scratchpad/slides05/audit2_findings.md as it finishes, so a further
      interruption costs one page rather than the pass. Steered at pass 1's error
      class: structure, counts, positions, uniformity claims, and figures present
      on the page but missing from the file.
- [x] Heading-sequence check — PASS, exactly 55 headings 1..55 in order

### Transcript
- [x] 05 — verbatim captions fetched (103 paragraphs, ~16.2k words)
- [x] 05 — copy-edited transcript (drafted by Sonnet, adjudicated here)
- [x] 05 — verify: all three checks pass. Timestamps: 103 markers, identical
      sequence. Numbers: 9 differences, all adjudicated — +9 "8"s from restoring
      FP8 (x7) and MXFP8 (x2) from the captions' "FPA"/"MXFPA"; "5257"->"50257"
      and "5304"->"50304", confirmed verbatim against slide 45's pasted Karpathy
      tweet; "m0"->"M00" at 59:08, matching the M00/N00 tile naming in the same
      sentence; one "32" dropped at 38:30 as a caption stutter ("a scaling factor
      in 32, FP32" -> "in FP32"); one "108" that now sits inside an [Ed:] note
      and so is not counted in the body. Word ratios: retention 82.3%, ZERO
      paragraphs outside the 0.72-1.10 band.
- [x] 05 — adjudication: the draft reported expanding three low-ratio paragraphs.
      Two were legitimate restorations of material the first draft had cut. The
      third had invented two words to complete the speaker's aborted false starts
      ("divisible by two" for "not even divi-", "diminishing returns" for
      "diminishing or sorry, no penalties"); both reverted, and the header now
      states that false starts are preserved rather than completed. Restored
      proper nouns: matmul/FP8/MXFP8/SMEM all appear verbatim in the deck; Groq
      and Chris Ré do not, and the header now labels both as context plus outside
      knowledge rather than implying deck support. "systolic array" was never a
      restoration — it is verbatim in the captions.

### Wiki
- [x] wiki/05-gpus-tpus.md — the lecture page (174 lines). Three parts: GPU hardware and
      execution model, six tricks for making workloads fast, then FlashAttention
      as the victory lap that combines tiling and recomputation.
- [x] Topic pages (10 new) — gpu-architecture (SMs, the memory hierarchy, the
      A100 latency table on slide 10, why a chip is not all SRAM),
      gpu-execution-model (threads/blocks/warps, SIMT, control divergence),
      tpus (convergent evolution, MXU, the tensor-core naming collision, systolic
      arrays), tensor-cores (matmul as the privileged operation since V100),
      microscaling-formats (MXFP8/MXFP4, E8M0 block scale factors, the transpose
      problem and the two quantized copies), operator-fusion, memory-coalescing
      (DRAM bursts, row-major traversal), tiling (the N/T math on slide 42, tile
      sizing, max-autotune, alignment and padding), wave-quantization (98 vs 120
      tiles against 108 SMs), flash-attention (online softmax, tiling plus
      recomputation)
- [x] Extend rather than duplicate: arithmetic-intensity.md already has a
      roofline section — add lecture 5's four-ceiling version (slide 21) there
      instead of a new page; precision-and-data-types.md already covers fp8/fp4 —
      link it to the new microscaling page; activation-checkpointing.md already
      covers recomputation — add slide 35/36's 8-vs-5 memory-access example
- [x] Update existing pages that now link a fifth lecture — efficiency,
      attention-variants (FlashAttention now has a mechanism page), course-map
      (Unit 2 Systems becomes properly covered, not a preview),
      executable-lectures (three decks now), memory-accounting-for-training
- [x] INDEX.md — five-lecture coverage: banner, Start here entry, a new Lecture 5
      section with 10 annotated entries, raw-material section now describing three
      decks
- [x] Link sweep — 688 relative links and 73 anchors all resolve; all 47 wiki
      pages appear in INDEX.md; no LaTeX inside code fences. (GitHub's anchor
      slugging replaces each space individually; do not collapse runs with \s+.)

### Publish
- [x] Update sources.md (lecture_05.pdf transcribed; deck-numbering and
      figure-density paragraphs now cover three decks)
- [x] kb.json — coverage 5/18, 42 topic pages, slideDecks 3 of 8, byLecture."5" =
      page-images, 17 caveats
- [x] AGENTS.md — deck precedent. Two things worth recording from this run: the
      three-reader split gave three independent confirmations that the deck
      prints no folio, and the figure audit's error class shifted — on lectures 3
      and 4 the errors were chart values, here every value was right and every
      error was diagram structure or a hand-drawn overlay.
- [ ] Commit and push

## Not done (future runs)
- [ ] Lectures 6–18 — transcripts and wiki pages
- [ ] Transcribe the 5 remaining PDF decks (lectures 8, 9, 11, 15, 16) — these need
      page-images, not source-text, and a figure audit
- [ ] Transcribe the 7 remaining executable lectures (6, 7, 10, 12, 13, 14, 17)
- [ ] Describe the figures in lectures 1 and 2 — the image() targets are recorded
      by path in raw/slides but their contents were never looked at. Lecture 3 now
      sets the precedent for how (page-images plus a targeted figure audit), but
      lectures 1-2 display images by URL rather than as PDF pages, so the mechanics
      differ.
