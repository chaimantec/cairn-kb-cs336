# KB build — CS336 (Language Modeling from Scratch, Stanford, Spring 2026)

Coverage after run 7: **Lectures 1–7.** Run 8 (lecture 8) in progress. The
course has 18 recorded lectures. See `kb.json` for machine-readable coverage.

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
- [x] Figure audit, pass 2 — pages 9, 14, 20, 21, 33, 48, 55. Five clean; five
      corrections applied to slides 9 and 20. Slide 9 had four errors: the die is
      eight GPCs in a 2x4 grid, not four quadrants; the HBM2/Memory Controller bars
      run down the left and right edges (3 HBM2 and 6 controllers per side), not
      top and bottom; the red highlight is in figure 2 only, not figure 1; and the
      NVLink row is twelve boxes, not the hedged "8-10". Slide 20's "Tiling!"
      annotation feeds three stacked arrows, not two — the same error pass 1 found
      on slide 46, which shows the same chart. Also recorded slide 14's H100 column
      printing 32MB for both SMEM and Registers as a deck oddity.
      Cross-check worth keeping: slide 9's corrected eight GPCs now agrees with
      slide 10's own die panel, which prints "x8 GPC" — two independently read
      views of one die agreeing after correction.
      Both passes found ONLY structural errors. Every data series, table cell,
      bit-field and numeric spot-check across all 14 audited pages was exact.

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
- [x] Commit and push — pushed to chaimantec/cairn-kb-cs336 at dec2a4a
- n/a  kbUrl already set on the catalog entry from run 1; re-fetched and confirmed

## Run 6 — Lecture 6: Kernels, Triton

Video xnDHaNUvHBg (87 min). Percy Liang. **Executable lecture** — `lecture_06.py`,
744 lines, so `source-text`, no deck and no page numbers. Cite function names and
source line ranges, not slide numbers.

Title note: the catalog entry calls this "Lecture 6: Kernels, Triton, XLA", but
**neither the source nor the captions mention XLA or JAX anywhere** (0 hits in
both). The XLA material is not in this offering's lecture 6; the KB calls it
"Kernels, Triton" after the course site's own lecture table.

### Course material
- [x] raw/slides/06-kernels-triton.md — transcribe lecture_06.py (744 source
      lines -> 1118 lines; all 189 text() literals accounted for, 22 section
      anchors resolve). Computed values: occupancy block (20480 / 3 / 12 /
      0.1875), GeLU(1.0), the 8-block GeLU grid, softmax and row-sum outputs,
      the stride example, the 16x16 matmul grid. Benchmark timings and all four
      profiler tables are machine-dependent and NOT reproduced — this lecture is
      unusually heavy in that class, since benchmarking is its subject.
      Source discrepancy recorded: the occupancy bullet says "thread block has 64
      threads" while the code below it sets num_threads_per_block = 128.

### Transcript
- [x] 06 — verbatim captions fetched (113 paragraphs, ~13.3k words, runs to 1:26:25)
- [x] 06 — copy-edited transcript (drafted by Sonnet, adjudicated here)
- [x] 06 — verify: all three checks pass. Timestamps: 113 markers, identical
      sequence, [MM:SS] and [H:MM:SS] alike. Numbers: nothing lost; the single
      addition is "run operation" -> run_operation2 at 26:57, where the captions
      omit the digit entirely — supplied from the source program, where the
      sentence's own "creates two random matrices" distinguishes
      run_operation2 from run_operation1, and 23:07 speaks the name in full.
      Word ratios: retention 82.9% (10,928/13,181); two paragraphs a hair under
      the band at 0.72 (25:24) and 0.71 (50:55), both read and confirmed as pure
      filler — ten "you know"s in the first alone.
      Restorations: all API names, hardware names and code identifiers appear
      verbatim in the source program (A100, Blackwell, nsight, tl.load/store/
      arange, %ctaid.x, triton_gelu_kernel, run_operation2, cutlass3x_sm100).
      Two rest on context alone (Triton at 22:22, matmul at 1:04:52) and one —
      "cute" -> CuTe at 1:24:54 — rests on outside knowledge and is labelled an
      editorial judgement in the header: CuTe appears nowhere in the source
      program (the case-insensitive grep hits were all "execute").
      9 [Ed:] notes mark genuine ambiguity; 19 student questions marked.
      CHECKER BUG, caught and fixed here: the first splitter treated a paragraph
      as the block its marker starts, but this transcript lifts each student
      question onto its own line — that manufactured 15 false outliers including
      a 0.00. A paragraph is everything from one marker to the NEXT marker.

### Wiki
- [x] wiki/06-kernels-triton.md (200 lines) — five hardware details, measure-first,
      the GeLU race, then the four kernels as a difficulty ladder
- [x] Topic pages (8 new) — triton (the block-level model and the kernel skeleton),
      ptx (what it compiles to; thread coarsening visible), benchmarking (warm up,
      synchronize, CUDA events, the constant-time floor), profiling (reading a CUDA
      kernel name), torch-compile (the three-horse race; it emits Triton),
      warp-occupancy (the 128x160 -> 18% example), bank-conflicts (32 banks, the
      32-way column conflict, swizzling), fused-softmax (5MN+M vs MN, -inf padding)
- [x] Extend rather than duplicate: tiling.md gains the lecture-6 matmul kernel
      (the naive/idealized/tiled ladder, strides, pointer matrices, the fused ReLU,
      and the tiles-are-not-blocks warning) instead of a new page;
      arithmetic-intensity.md gains the O(1)/O(N)/O(tile) table
- [x] Update existing pages that now link a sixth lecture — operator-fusion,
      gpu-execution-model, memory-coalescing, wave-quantization, gpu-architecture,
      flash-attention (lecture 6 is the ingredients list for the assignment),
      efficiency, course-map, executable-lectures
- [x] INDEX.md — six-lecture coverage: banner, Start here entry, a Lecture 6 section
      with 8 annotated entries, raw-material section now three executable lectures
- [x] Link sweep — 819 relative links and 97 anchors all resolve; all 56 wiki pages
      appear in INDEX.md; no LaTeX inside code fences
- [x] Citation checks — every one of the ~110 [MM:SS] citations in the new and
      extended pages matches a real marker in the lecture 6 transcript; all 142
      quoted fragments checked against the transcript AND the lecture source, which
      found 9 real quoting slips (a quote begun one word early, a transcript/source
      hybrid, a dropped "now", quotation marks around my own paraphrase, a word-order
      slip, a silently "corrected" TID.x, two unmarked elisions, and a quote that
      removed the speaker's own self-correction). All 9 fixed.

### Publish
- [x] Update sources.md (lecture_06.py now transcribed; the XLA title note recorded)
- [x] kb.json — coverage 6/18, 50 topic pages, executableLectures 3 of 9,
      byLecture."6" = source-text, 23 caveats
- [x] AGENTS.md — run 6 precedents: an executable lecture can contradict itself and
      the code wins; check a catalog title against the material; machine-dependent
      values can be most of a lecture. Plus the third distinct bug found in the
      transcript ratio checker.
- [x] Commit and push — pushed to chaimantec/cairn-kb-cs336 at 8df9244
- n/a  kbUrl already set on the catalog entry from run 1; re-fetched and confirmed

## Run 7 — Lecture 7: Parallelism

Video SzpOcwdIL0Y (81 min). Percy Liang. **Executable lecture** — `lecture_07.py`,
619 lines, so `source-text`, no deck and no page numbers. Cite function names and
source line ranges, not slide numbers.

Note: lectures 7 AND 8 are both titled "Parallelism". Lecture 7 is Percy's
executable one (building blocks + DP/TP/PP); lecture 8 is Hashimoto's
`lecture_08.pdf` deck and is NOT part of this run.

**New this run: the course publishes the lecture's own recorded stdout** at
`var/traces/lecture_07_stdout.txt` in the lectures repo. That is a real 4-GPU
run (Modal, CUDA 13.2) and it supplies the printed collective outputs, the
measured bandwidths, and the per-rank losses that the source alone cannot give.
Saved to `raw/pdfs/lecture_07_stdout.txt`. Treat its timings as measurements of
THAT machine, not as facts about GPUs in general.

### Course material
- [x] raw/slides/07-parallelism.md — transcribe lecture_07.py (619 source lines
      -> 1077 lines; all 104 text() literals accounted for, 32 section anchors
      resolve). Computed values: local_batch_size 32 and rank slices 0-32/96-128,
      local_num_dim 256, local_num_layers 2, micro_batch_size 32, num_elements
      104,857,600 = 400 MiB fp32, and both sent_bytes figures. The recorded-run
      bandwidths were re-derived from the printed durations and reproduce the
      printed 366/390/426/425 and 450/475/490/490 GB/s to within one unit in the
      last place (rounding in the displayed milliseconds).
      Transcription note: one source typo ("another GPUs memory") is transcribed
      as printed, since the block is labelled as the source's own summary.

### Transcript
- [x] 07 — verbatim captions fetched (105 paragraphs, ~11.3k words, runs to 1:20:57)
- [x] 07 — copy-edited transcript (drafted by Sonnet, adjudicated here)
- [x] 07 — verify: all three checks pass. Timestamps: 105 markers, identical
      sequence, [MM:SS] and [H:MM:SS] alike. Numbers: 4 differences, all
      adjudicated — "1 trillion" -> "one-trillion", "NVLink five" -> "NVLink 5"
      (slide prints "NVLink 5.0"), and a caption stutter at 7:50 where he
      restarts the list he is reading ("has some tensor 0 1 0 1 2 3" ->
      "0, 1, 2, 3"), which matches the source's own broadcast example
      tensor([0., 1, 2, 3]). Word ratios: retention 85.3%; two paragraphs a hair
      under the band at 0.70 (45:37) and 0.71 (46:23), both read and confirmed
      pure filler — the same signature lecture 6 showed for this speaker.
      Restorations: 20-odd terms, and all but four appear verbatim in the source
      program (HBM, NVSwitch, NCCL, gloo, FSDP, ZeRO, MoE, PCIe, RDMA, RoCE,
      NVL72, reduce_scatter_tensor, all_gather_into_tensor, sharding strategy,
      interconnects, elementwise, sum). 10 [Ed:] notes, 17 student questions.
- [x] 07 — adjudication: the drafting agent reported TWO restorations as resting
      on "outside knowledge only" — "Grace" and "critical batch size". Both
      claims were wrong in the reader's favour: the captions say "G stands for a
      grace" and "the critical batch fact uh size", so the words are present and
      only the capitalization and a stutter collapse were editorial. Both header
      rows rewritten to state the real evidence. Its isend/irecv reading is
      correctly confined to an [Ed:] note and left there. Every other stated
      evidence claim was spot-checked (HBM x3, NCCL x2, Tatsu x4 spelled
      correctly elsewhere in the captions; the "gather, gather" duplication at
      7:04 is real) and held. Also removed a duplicated "NCCL" in the header.

### Wiki
- [x] wiki/07-parallelism.md (256 lines) — the hierarchy extended past the chip,
      part 1 (collectives, hardware, torch.distributed, benchmarking), part 2 (the
      three cuts as a table), what the lecture deliberately omits, and the
      recompute/store/communicate pattern
- [x] Topic pages (7 new) — collective-operations (all eight primitives with the
      lecture's own four-rank worked examples), gpu-interconnect (the bandwidth
      tiers, RDMA, NVL72, RoCE), torch-distributed (incl. NCCL, and the two kinds of
      asynchrony that make barrier ORDER matter), data-parallelism,
      tensor-parallelism, pipeline-parallelism, sharding-vs-replication
- [x] Extend rather than duplicate: benchmarking.md gains a "Measuring a collective"
      section (the effective-bandwidth formula, independence of world size and
      topology) instead of a new page; expert-parallelism.md gains the all-to-all
      primitive it had been promising since run 4
- [x] Update existing pages that now link a seventh lecture — expert-parallelism,
      mixture-of-experts, benchmarking, efficiency, course-map (coverage banner was
      STALE at "lectures 1-4" and is now correct), executable-lectures,
      gpu-architecture, memory-accounting-for-training, activation-checkpointing,
      arithmetic-intensity, tpus
- [x] INDEX.md — seven-lecture coverage: banner (now warns that lectures 7 AND 8 are
      both "Parallelism" and only 7 is covered), Start here entry, a Lecture 7
      section with 7 annotated entries, raw-material section noting the published
      stdout
- [x] Link sweep — 993 relative links and 153 anchors all resolve; all 64 wiki pages
      appear in INDEX.md; no LaTeX inside code fences.
      SWEEPER BUG, caught and fixed here: the slug function stripped `_` as a
      markdown emphasis marker, but GitHub keeps it (it is a word character), so
      `#async_op-and-overlapping` was reported unresolved. Strip backticks and
      asterisks only.
- [x] Citation checks — all 197 [MM:SS] citations across the 8 lecture-7 pages match
      a real marker. Sweeping the whole wiki also turned up a PRE-EXISTING defect
      from run 5: three pages cited [32:19], which is not a marker; the real one is
      [32:18]. Fixed in 05-gpus-tpus, arithmetic-intensity and gpu-execution-model.
      The wiki now has zero citations to non-existent markers.
- [x] Quote checks — 146 quoted fragments checked against the transcript, the slide
      file and the raw source. **34 real slips found and fixed**, far above run 6's
      9, because this lecture's pages quote heavily. Nearly all were the same fault:
      silently smoothing a false start INSIDE quotation marks ("about four - about
      four x - slower" quoted as "about four x slower"; "your nodes - your GPUs - are
      actually across, halfway across the world" quoted as "your GPUs are actually
      halfway across the world"). Two were worse and are the ones to watch for: a
      quote run straight across a passage the transcript marks [Ed:] as garbled
      (the "collective commission" gap at 7:04), and a paraphrase presented inside
      quotation marks (the garbled cables/switches clause at 32:33). Both rewritten
      to quote only what is verbatim and to say the captions are garbled there.
      3 residual flags are nested-quote-style conversions (source "..." rendered as
      '...' inside an outer quote), verified verbatim by direct substring test.
      TWO CHECKER BUGS, both found here: a `"([^"]{12,})"` regex DESYNCHRONIZES the
      quote pairing whenever a short quote is skipped, so every later "failure" is an
      artifact (80 false positives before the fix) — pair quote characters
      sequentially instead. And the haystack must have [MM:SS] markers stripped, or
      every quote spanning a paragraph boundary reports as missing.

### Publish
- [x] Update sources.md (lecture_07.py transcribed; the two-lectures-named-
      Parallelism note; the published-stdout note)
- [x] kb.json — coverage 7/18, 57 topic pages, executableLectures 4 of 9,
      byLecture."7" = source-text, 27 caveats. Also corrected three caveats that had
      gone stale: the PARTIAL one still said "lectures 1, 2, 3, 4 and 5" after run 6,
      and the method/figures caveats still framed source-text as "lectures 1 and 2".
- [x] AGENTS.md — run 7 precedents: look for published runtime output before writing
      "not reproduced"; an executable lecture may not be traceable; two lectures can
      share a title; and quote-check the wiki, with the two checker bugs recorded
- [x] Commit and push
- n/a  kbUrl already set on the catalog entry from run 1; re-fetched and confirmed

## Run 8 — Lecture 8: Parallelism (Part 2)

Video 6-cXp-aOmdg (80 min). Tatsunori Hashimoto. PDF deck — `lecture_08.pdf`,
73 pages, `page-images`. This is the FSDP/ZeRO half of parallelism that lecture 7
deferred to repeatedly, plus pipeline/tensor/sequence/expert parallel, the
combined-strategy rules of thumb, and ten model case studies.

Title note: the catalog calls both lectures 7 and 8 "Parallelism". The KB calls
this one "Parallelism (Part 2)" to distinguish it from Percy's executable
lecture 7; the deck's own title page reads "PARALLELISM BASICS".

Numbering: derived in the parent BEFORE any page was read. `slide_number_map.py`
reports no printed number on any of the 73 pages, and a corner-position
text-layer scan for isolated 1-3 digit tokens returns ZERO hits anywhere in the
deck — cleaner than lectures 3, 4 or 5, each of which had at least one mid-page
digit land in the scan region. Mapping is a plain 1..73, page N = slide N;
`--verify` must use the Python heading-sequence fallback.

Density: 86 embedded images across 73 pages against only ~2,749 words of native
text (~38 words/page). Same profile as lectures 3-5 — the figures are the content.
Only 7 pages carry no image at all (3, 13, 14, 21, 28, 55, 72), and several of
those are the deck's dense native tables.

Model split (user's call, cost-aware): pages 46-62 read at Opus because that band
carries the activation-memory algebra and the two comparison tables; the rest at
Sonnet. Page 72's overview table is read at Sonnet and audited at Opus.

### Course material
- [x] Download lecture_08.pdf (73 pages, 6.5 MB) to raw/pdfs/
- [x] Numbering derived in the parent and handed to the readers as a conclusion
- [x] raw/slides/08-parallelism-2.md — five parallel readers over pages
      1-15 / 16-30 / 31-45 / 46-62 / 63-73, appending incrementally; parent
      concatenates and writes front matter and the section table. 1,544 lines,
      ~23.3k words. All five readers independently confirmed no folio in their
      range, at magnifications up to 4800 dpi.
- [x] Heading-sequence check — PASS, exactly 73 headings 1..73 in order, no gaps,
      no merges, no duplicates. Independently, 72 of 73 heading titles were
      matched VERBATIM against the PDF text layer at the top of their own page;
      the single exception is the title page, whose text layer letter-spaces
      "PA R A LLE LIS M BA SIC S". That cross-check is new this run and is worth
      keeping: it confirms page-to-heading attribution without opening a page
      image in the parent, which is the expensive thing the skill forbids.
- [x] Internal consistency of the two formula families, checked in the parent
      without opening a page: the activation-memory rows on slides 46-49 compose
      exactly as the deck says (34 = 10 + 24; TP gives 10 + 24/t, TP+SP gives
      34/t, and selective recomputation drops the 5as/(ht) term from each), and
      the reader for 16-30 reported the ZeRO formulas on 18/19/22/24 reproducing
      their own printed GB values. No LaTeX inside code fences.
- [x] Figure audit, pass 1 — eight pages (12, 21, 30, 37, 55, 60, 67, 72), at
      least one from each of the five readers' ranges, chosen from the pages the
      readers themselves nominated. Three auditors: Sonnet on 12/21/30 and
      37/55/60, Opus on 67/72 because those two tables are the deck's most-quoted
      and had been read at Sonnet. SIX CLEAN, two dirty, 4 errors + 3 more in
      cross-slide commentary, all 7 applied.
      Clean: 12 (both sub-tables, 15 rows, plus the Chinese photo annotations),
      21 (the load-bearing DDP-vs-ZeRO-1 table), 55 (42 cells incl. every
      red-highlighted cell, and the two distinct senses of "None" confirmed),
      60 (all 12 data points within 1-3 teraFLOP/s), 67 (72 cells), 72 (40 cells,
      column alignment PROVEN by identical text-layer span x-origins down each
      column).
      Errors on 30: the ZeRO-3 530B diamond series' first two x-positions were
      assumed to mirror the 175B circle series (768, 1152) when both 530B series
      sit at their own rightward-shifted positions (~815, ~1250). A
      series-conflation error, not a scattered value error.
      Errors on 37: the interleaved schedule's Device 1 readout — the pale-blue
      run is 4 cells not 6 and returns to dark blue for 5,6; and the gray gap
      sits BEFORE the single "13" cell, with the four colours forming two
      sequential two-colour blocks rather than one interleaved run. Every
      STRUCTURAL claim on 37 was correct (4 device rows, 2-entry legend against 4
      visible colours, numbering continuing past the step boundary).
      Two contested editorial claims were put to the auditors deliberately and
      BOTH were confirmed: slide 67's counts sum to exactly 419 while its
      percentages sum to exactly 94.9% (gap is in the pasted source), and slide
      60's own "flat utilization!" caption is contradicted by its own chart —
      the orange PTD-P series sag ~12 teraFLOP/s while both blue ZeRO-3 series
      fall 69% and 65%. A caution was added to slide 67: the percentages are not
      percentages OF 419 either (148/419 = 35.3% vs a printed 30.1%), so the
      pasted table lists only leading categories.
      NEW ERROR CLASS this run: all 3 remaining errors were in cross-slide
      COMMENTARY, not in the audited page's own content — a CP value called
      common to all three of slide 66's rows when it is 1, 1, 16; a sentence
      asserting the Llama3 DP=128 matches slide 66's first row and then
      contradicting itself in its own parenthetical; and a reference to "slide
      68's later heading" when slide 68 is Gemma 2. Cross-references were not
      something the audit prompt asked for; they should be next time.
- [x] Figure audit, pass 2 — both leads run; NOTHING wrong on any page.
      (a) Charts 36, 45, 61, 62, re-checked for the slide-30 error class by
      extracting each native raster, calibrating against located axis ticks and
      colour-matching marker blobs. ALL FOUR CLEAN, with a structural reason
      rather than a lucky sample: each of these charts has a genuine SINGLE
      categorical x-axis, so the failure mode had no opportunity to occur. Slide
      30 was the deck's only chart with two paired continuous-x series, which is
      why it was the only one to show the error. On 61 and 62 the occluded blue
      markers were recovered by least-squares circle-fitting the visible arc, and
      the recovered values are DISTINCT from the orange values covering them —
      that positively rules out a copied value rather than merely failing to find
      one. Page 45's stacked bars were confirmed to record segment heights, not
      cumulative totals, and its red dashed line sits exactly on the calibrated
      y=80 gridline, confirming it is a threshold and not a series.
      (b) Cross-slide assertions — all 48 in the file swept, text against text,
      no page images. Four wrong, three real:
        * slide 23 was listed among the legends rendering gradients orange; its
          diagrams are colour-coded by RANK (blue/red/green/yellow) and carry no
          such legend. Dropped from both lists.
        * slide 71's table was said to name "Qwen3-235B-A22B" four times; it
          prints that exact string three times, the fourth occurrence being this
          file quoting it.
        * an audit note reading "within 1-3 teraFLOP/s" was ambiguous enough to
          be read as the data range; it meant the agreement tolerance, and now
          says so.
      The fourth was the AUDITOR's error, not the file's, and is worth recording
      because it nearly cost a correct sentence: it read the readers' "no figure
      on this page" remarks as contradicting the front matter's raster count, but
      every one of those pages does carry a raster — pasted equations and tables
      rather than figures. Verified in the parent against the PDF before
      applying; the wording now states the distinction.
      LESSON: two of the four errors were in FRONT MATTER I wrote, not in any
      reader's page. The aggregating prose is the least-checked layer of a slide
      file, because it is written after every reader has finished and no audit
      targets it. Sweep it explicitly next deck.
### Transcript
- [x] 08 — verbatim captions fetched (105 paragraphs, ~15.8k words, runs to
      1:20:02), saved to raw/transcripts/original/08-parallelism-2.md
- [ ] 08 — copy-edited transcript (draft at Sonnet, adjudicate here). Must wait
      for the slide file: the edit is cross-checked against the deck.
- [ ] 08 — verify: timestamps, number inventory, per-paragraph word ratios

### Wiki
- [ ] wiki/08-parallelism-2.md
- [ ] Topic pages — zero-and-fsdp is the big gap lecture 7 left open; also
      sequence-parallelism, activation-memory, 3d-parallelism, and a case-study
      page for what real models do
- [ ] Extend rather than duplicate: data-parallelism, tensor-parallelism,
      pipeline-parallelism, expert-parallelism, sharding-vs-replication,
      collective-operations, gpu-interconnect and activation-checkpointing all
      already exist from run 7 and must be extended, not re-created
- [ ] INDEX.md — eight-lecture coverage; the banner's "lectures 7 AND 8 are both
      Parallelism, only 7 is covered" warning is now STALE and must be rewritten
- [ ] Link sweep
- [ ] Citation checks
- [ ] Quote checks

### Publish
- [ ] Update sources.md (lecture_08.pdf transcribed; the two-Parallelism note)
- [ ] kb.json — coverage 8/18, slideDecks 4 of 8, byLecture."8" = page-images
- [ ] AGENTS.md — run 8 precedents
- [ ] Commit and push
- n/a  kbUrl already set on the catalog entry from run 1

## Not done (future runs)
- [ ] Lectures 8–18 — transcripts and wiki pages. **Lecture 8 is the priority**: it
      is the other half of parallelism (FSDP/ZeRO), which lecture 7 defers to
      repeatedly, so the KB currently names those techniques without explaining them.
- [ ] Transcribe the 5 remaining PDF decks (lectures 8, 9, 11, 15, 16) — these need
      page-images, not source-text, and a figure audit
- [ ] Transcribe the 5 remaining executable lectures (10, 12, 13, 14, 17). Check each
      for a published `var/traces/lecture_NN_stdout.txt` in the lectures repo before
      writing off its runtime values as machine-dependent — lecture 7 had one.
- [ ] Describe the figures in lectures 1, 2, 6 and 7 — the image() targets are recorded
      by path in raw/slides but their contents were never looked at. Lecture 3 now
      sets the precedent for how (page-images plus a targeted figure audit), but
      these lectures display images by URL or repo path rather than as PDF pages, so
      the mechanics differ. Lecture 6 has four such figures, two of them the
      lecture's own diagrams of the softmax and row-sum kernels; lecture 7 has five
  (the node overview, ranks, and one schematic per parallelism strategy).
