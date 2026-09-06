# KB build — CS336 (Language Modeling from Scratch, Stanford, Spring 2026)

Coverage after run 15: **Lectures 1–14.** The
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
- [x] 08 — copy-edited transcript (drafted by Sonnet, adjudicated here)
- [x] 08 — verify: all three checks pass. Timestamps: 105 markers, identical
      sequence, [MM:SS] and [H:MM:SS] alike. Numbers: one loss and 21 gains, all
      adjudicated — the loss is "8x24 22B" -> "8x22B" (slide 69 lists a Mixtral
      8x22B row and the same paragraph names it a sentence earlier), and every
      gain is either a spelled-out "zero stage one/two/three" written as a digit
      to match the deck's own "ZeRO stage 1", or the TPU8i/TPU8t restoration.
      Word ratios: retention 84.0%; three paragraphs a hair under the band at
      0.66-0.72 (41:20, 42:05, 57:23), all read in full against the original and
      confirmed pure filler — this speaker's "you know"/"sort of" density is the
      same signature lectures 6 and 7 showed.
      Restored proper nouns: all 16 deck-supported terms verified present in
      raw/slides/08-parallelism-2.md by grep (TPU8i x3, TPU8t, ZeRO x83, GeLU x7,
      OLMo x3, DeepSeek x14, Nemotron x8, DeepEP x4, HybridEP x4, 8x22B x5,
      A100 x10, Kahan, GShard x2, NCCL, Megatron x17, Qwen x19). The only two
      restorations absent from BOTH deck and captions are "convergent evolution"
      (captions: "conversion evolution") and "scaling laws" (captions: "scaling
      loss") — exactly the two the draft itself labelled outside-knowledge, so
      its evidence claims held up, unlike run 7's.
- [x] 08 — adjudication: FOUR changes made here.
      (1) "$W_0$" reverted to "that W naught". LaTeX must not enter a transcript;
      raw/transcripts/ is a verbatim record and the wiki is where notation gets
      reconstructed. This is the first time a drafting agent has broken that rule
      in this build.
      (2) "all-gathering LAYER one/two" reverted to "all-gathering one/two" —
      "layer" is an interpolation, and the preceding sentence already supplies
      "the next layer's parameters".
      (3) "a quarter of the parameters" reverted to "1/4", the figure the
      captions carry.
      (4) THE IMPORTANT ONE. The draft rewrote a spoken "zero stage two" to
      "ZeRO stage 3" at 29:53 because slide 30 shows stage 3 is what cuts
      parameter memory. The substance is right — the slide is unambiguous — but
      the body has been reverted to what he said, with the [Ed:] note rewritten
      to explain it. The distinguishing test, worth reusing: "two" and "three"
      are not acoustically confusable, so the captions are reliable there and the
      SPEAKER slipped while reading his own slide. Contrast 42:05, where
      "computation hungry" was restored to "communication-hungry" and KEPT: those
      two words are a plausible ASR confusion, and the same paragraph ends "very
      communication hungry". Mis-hearings get fixed in the body; speaker slips get
      an [Ed:] note and stay as spoken.
      Left correctly as heard by the draft, with notes: "300 chips" against the
      deck's 384 (10:02), an unidentifiable "P200" accelerator, and "DeepSeek V1"
      at 1:13:31 where slide 64's excerpt is from the V2 paper — he said V1, so V1
      it stays. 7 [Ed:] notes, 12 student questions marked.
      CHECKER BUG, the fourth distinct one in this build: stripping the floor-
      question label with a bare `]*` replace ALSO matched inside the `**[0:05]**`
      markers and destroyed every one of them, so the ratio check reported
      "105 paragraphs vs 0". Strip labels per-paragraph AFTER splitting, and use
      `\]\*(?!\*)` so the marker's `]**` is never touched.

### Wiki
- [x] wiki/08-parallelism-2.md (239 lines) — three parts mirroring the deck:
      the network, the primitives, and putting it together, with the through-line
      that one idea (store sharded, materialise on demand) recurs in four disguises
- [x] Topic pages (9 new) — zero-and-fsdp (the gap lecture 7 left open: three
      stages, why two are free, the overlap argument, and why it is not pipelining),
      activation-memory (the 34sbh accounting and the five-row table),
      sequence-parallelism, context-parallelism, zero-bubble-pipelining (B vs W),
      critical-batch-size (batch size as a budget), network-topology (mesh vs tree,
      Huawei Ascend, the TPU8i news), 3d-parallelism (the prescription and the
      Narayanan evidence), parallelism-case-studies (slide 72's table plus the ten
      runs)
- [x] Extend rather than duplicate (13 pages) — data-parallelism,
      pipeline-parallelism, tensor-parallelism, expert-parallelism,
      sharding-vs-replication, collective-operations (the all-reduce identity gets
      its own section, since it is what makes ZeRO free), gpu-interconnect,
      activation-checkpointing, memory-accounting-for-training, mixture-of-experts,
      tpus, flash-attention, scaling-laws. Plus efficiency, executable-lectures,
      course-map and 07-parallelism for cross-lecture continuity.
- [x] INDEX.md — eight-lecture coverage. The banner's "Lecture 8 is NOT covered, so
      this KB has no treatment of FSDP or ZeRO" warning was the most misleading
      stale text in the KB and is rewritten; new Start-here entry, a Lecture 8 wiki
      section with 9 annotated entries, raw-material updated for a fourth deck.
- [x] Link sweep — 1,325 relative links and 162 anchors all resolve; all 74 wiki
      pages appear in INDEX.md; no LaTeX inside code fences. (The 5 "unresolved"
      hits are false positives: the regex matches Python call syntax like
      `kernel[grid](x, y, BLOCK_SIZE=...)` inside lecture 6's code fences.)
- [x] Citation checks — 968 [MM:SS] citations across the WHOLE wiki checked against
      all 8 transcripts; zero point at a non-existent marker. Four were fixed, all
      the same fault: a dropped hour prefix, [11:57] for [1:11:57] and [12:44] for
      [1:12:44]. Worth knowing that this lecture runs past an hour, so any citation
      after 1:00:00 is a candidate for it.
- [x] Quote checks — 115 quoted fragments across the 10 new pages checked against
      the transcript, the slide file and the raw captions. 6 real slips found and
      fixed: a comma where the speaker has an em dash, "which involves" for "This
      then involves", a quote begun one word early, LaTeX reformatted inside
      quotation marks (twice), and the parent's own editorial aside set as a
      blockquote — which reads as a quotation of the lecturer and was not one.
      2 residual flags verified verbatim by direct substring test; they are the
      checker failing to strip a citation that sits inside the blockquote.

### Publish
- [x] Update sources.md — lecture_08.pdf transcribed; the two-Parallelism note
      rewritten now that both halves are covered; the numbering paragraph now
      covers four decks and records that lecture 8's corner scan returned ZERO
      digit tokens, the cleanest of the four
- [x] kb.json — coverage 8/18, 66 topic pages, slideDecks 4 of 8, byLecture."8" =
      page-images, 34 caveats. Also removed a duplicate PARTIAL caveat and rewrote
      the stale one that still listed lecture 8 among the uncovered.
- [x] AGENTS.md — run 8 precedents: the heading-vs-text-layer cross-check (the new
      cheap strong check), the front matter being the least-audited layer, the
      mis-hearing vs speaker-slip test for transcripts, no LaTeX in transcripts,
      and three more checker bugs
- [x] Commit and push — pushed to chaimantec/cairn-kb-cs336 at 3e8a495
- n/a  kbUrl already set on the catalog entry from run 1

## Run 9 — Lecture 9: Scaling Laws (Basics)

Video Q15rhEWZPQ4 (78 min, 15,924 caption words). Tatsunori Hashimoto.
PDF deck `lecture_09.pdf`, 57 pages — the most figure-dense deck in the build:
93 raster images against 2,063 words of native text, 36 words/page.

### Course material
- [x] Download lecture_09.pdf into raw/pdfs/ (gitignored)
- [x] Numbering settled BEFORE any page was read. slide_number_map.py found no
      printed number on any page, and a whole-page edge scan of the text layer
      returned only two numeric tokens across all 57 pages — both on page 33, at
      (435,368) and (455,366) on a 720x405 page, i.e. inside the plot area, not a
      corner folio. Same case as lecture 8: `## Slide N` == PDF page N, 1..57.
- [x] raw/slides/09-scaling-laws.md (1,131 lines, 57 sections) — four Sonnet
      readers, pages 1-15 / 16-29 / 30-43 / 44-57, appending incrementally. Model
      choice was put to the user because the deck is unusually chart-dense; the
      user chose the Sonnet default. All four finished; none was interrupted.
      Two readers resolved colour identity by pixel-classification against the
      legend swatch RGB rather than by eye, and that method caught two errors eye-
      reading had already made: a CIFAR-10 point misassigned to CIFAR-100 on slide
      20, and four spurious "data points" on slide 4 that were the legend swatches
      themselves sitting inside the plot's own coordinate space. That second one is
      a new instance of the known "a label is not a series" failure — worth adding
      to AGENTS.md as: the legend can contaminate a colour trace, so mask its
      bounding box before classifying pixels.
- [x] Heading-sequence check — OK, 57 headings, sequence matches the deck exactly
      (--verify degenerates to this: the map is a 1..57 fallback, not read off the
      pages)
- [x] Heading-vs-text-layer cross-check — 56/57 headings matched verbatim at the
      top of their own page. The one exception is the title page, whose text layer
      letter-spaces "S C A LIN G LAW S - BA SIC S", exactly as lecture 8's did.
- [x] Page 33's stray "10"/"7" edge tokens identified by the reader as that
      chart's 10^7 x-axis tick, closing the numbering question independently
- [x] Slides 45 and 50 confirmed byte-identical images (MD5 of the extracted
      stream), so the repeat is the deck's own, not a transcription error
- [x] Figure audit pass 1 — NINE pages fully audited (3, 4, 20, 31 / 45, 49, 50,
      52, 55) plus existence checks on 17, 19, 27, 28. Two delegated Sonnet
      auditors; no page was opened in the parent. RESULT: DIRTY — 5 of 9 pages
      carried errors, 20 discrete errors, all now corrected in the file.
      Clean: 3 (300 table cells exact), 20 (all 17 points, pixel-reclassified),
      45 and 50 (the two independent write-ups of one byte-identical image agree).
      Two findings worth carrying into AGENTS.md as build-wide precedent:
      (a) FABRICATED CROPPING, twice, by two different readers — slide 31's panel
      (i) and slide 55's top-left y-axis each described as clipped in the source
      when neither is. The reader described its own crop's edge as a property of
      the slide. This is a new failure mode for this build, distinct from the
      known "invented a series from a label", and it is invisible downstream: a
      reader of the KB has no way to tell a real clipped label from an invented
      one. Audit prompts should ask explicitly whether a claimed crop is real.
      (b) FALSE ILLEGIBILITY is now 100% false across every instance ever tested
      here — three more on slide 52 (two equation intercepts and a 19-entry
      legend), all readable at zoom. The skill already says treat the phrase as a
      flag not a fact; this deck is the strongest evidence yet.
      Also surfaced a real deck inconsistency, not a transcription error: slide 52
      prints slope 0.79 inside its equation and 0.78 in the same panel's title.
- [x] Figure audit pass 2 — DONE (pages 13, 18, 24, 26 / 34, 40, 41, 48, 56).
      16 more errors on six of nine pages; all applied. Page 48 — the twin of
      page 49 and the highest-risk page in the deck — came back EXACT under
      programmatic extraction of all nine IsoFLOP minima. Page 41 was the most
      consequential find in either pass: misread SuperGlue values had produced a
      conclusion that inverted the slide's own argument about downstream
      reshuffling. Running total: 18 of 57 pages audited, 11 dirty, 36 errors.
      A third fabricated source-image defect appeared (slide 41's "NL12-"), by a
      third reader — but this one is a real truncation with a wrong cause
      attached, so the lesson is narrower: readers reach for "the source image is
      cropped" to explain anything they cannot read. Check the explanation, not
      just the observation.
- [x] Superseded note — the earlier pass-2 requirement said: 5-of-9 dirty is far worse
      than this build's usual ~2 small corrections per deck, and by the skill's own
      rule a dirty sample means the sample was too small. 48 pages remain
      unaudited. Priority targets: 48 (nine IsoFLOP parabolas — same figure family
      as 49, the dirtiest page found), then the multi-series log-log pages
      13, 18, 24, 26, 34, 40, 41, and 56. All were done.
- [x] Figure audit pass 3 — ATTEMPTED, KILLED, NOT DONE. The agent was terminated
      by a session rate limit partway through and returned no report, so pages 37,
      38, 39, 43, 44, 46 and 47 remain unaudited. Note for future runs: this agent
      was told to report findings as text at the end, which is exactly the pattern
      the skill warns about — an audit agent should be told to APPEND findings to a
      file per page, like the transcription agents are, so a kill costs one page
      rather than everything. The transcription agents in this run all survived
      because they appended; this one lost 100% of its work.
      Mitigation applied instead, at no cost: every value the wiki quotes from those
      seven slides was cross-checked against the copy-edited transcript, which is
      independently verified. All of them matched except slide 47's "1.5T tokens"
      and the exact algebra of slide 43's equations, which are not spoken aloud;
      both are flagged provisional at the point of use.
- [ ] Superseded plan (kept for the record) — pass 3 was to cover: 37, 38, 39 (critical batch size)
      and 43, 44, 46, 47 (joint scaling and Chinchilla methods 1-2). These are the
      two clusters whose CHART VALUES the wiki will quote directly, so they are
      audited before the prose is written rather than after. The other 32
      unaudited pages are left provisional-for-charts, reliable-for-text, which
      the front matter now states explicitly.

### Transcript
- [x] 09 Scaling Laws — verbatim captions at raw/transcripts/original/09-scaling-laws.md
- [x] 09 Scaling Laws — copy-edited transcript, delegated to Sonnet, adjudicated
      here. All three checks pass: 102 markers identical and in order; per-paragraph
      word ratios ALL inside the 0.72-1.10 band at 84.6% retention (matching lecture
      8's 84.0% for the same speaker); number inventory clean but for one adjudicated
      difference — at 1:13:00 the captions read "that sort of 2D surface or sorry, 3D
      surface", a self-correction, and the false start is removed. Slide 49 confirms
      Chinchilla's method 3 fits a 3D surface.
      All 20 restored proper nouns verified present in the deck by grep — the first
      lecture in this build where every single restoration is deck-supported (run 12
      of CS224N managed 51 of 60). The agent under-claimed on one: it recorded
      "DeepSeek" as outside knowledge, but slide 13's ECI scatter labels a point
      "DeepSeek-R1". Header corrected.
      SIXTH CHECKER BUG, recorded in AGENTS.md: this transcript puts each floor
      question's full text inside the *[Question from the floor: ...]* block, where
      lecture 8 used a bare label, so a clean() that strips those blocks wholesale
      deletes real words. It reported two paragraphs at 0.66/0.68 that are fine.
      Strip the label, keep the content.

### Wiki
- [x] wiki/09-scaling-laws.md (400 lines) — four parts mirroring the lecture:
      prehistory, data scaling, model engineering, compute-optimal scaling, with the
      through-line that interventions move intercepts and not slopes
- [x] Topic pages (8 new) — data-scaling-laws, compute-optimal-scaling,
      isoflop-method, upstream-vs-downstream, data-repetition, data-mixture-selection,
      learning-rate-scaling-and-mup, scaling-law-methodology
- [x] Extend rather than duplicate (6 pages) — scaling-laws (rewritten from a
      preview-only page into the hub, with a section reading lecture 1's framing
      against lecture 9's delivery), critical-batch-size (now carries both lectures,
      with an explicit note on whose timestamps are whose, since both discuss it
      around the same point in their runtime), transformer-hyperparameters,
      mixture-of-experts, model-architecture-survey, course-map
- [x] Fixed SIX stale coverage claims predating this run: four pages still said
      lecture 8 was uncovered, course-map contradicted itself about it in two places,
      and scaling-laws said lecture 2 was uncovered. This is the rot the skill warns
      about — the index is trusted and never re-read.
- [x] INDEX.md — nine-lecture coverage, new banner section on the two scaling-laws
      lectures, a Start-here entry, a Lecture 9 wiki section with 9 annotated
      entries, and the stale "preview only" entry for scaling-laws rewritten
- [x] Link sweep — 1,368 relative links, zero unresolved
- [x] Citation checks — 1,615 [MM:SS] citations across the whole wiki checked
      against all 9 transcripts. One bad, now fixed, and it was the SAME fault run 8
      found four of: a dropped hour prefix, [11:57] for [1:11:57]. It was in
      critical-batch-size.md, a page run 8 itself wrote — so run 8's sweep missed one.
- [x] Quote checks — 127 fragments; 8 real slips fixed. Two em dashes became a comma
      and a semicolon, one dropped a pair of em dashes, two dropped inner quotation
      marks, one dropped the speaker's "kind of", and TWO were the parent's own
      paraphrases set in quotation marks — the same failure run 8 caught, now twice
      in a row. Also caught myself over-correcting one quote mid-fix and reverted it.
      FIFTH CHECKER BUG: strip [MM:SS] markers from the SOURCE before matching, or
      every quote spanning a paragraph break reports as a misquotation; split the
      wiki quote on ellipses; and scope to all transcripts, not just this lecture's.

### Publish
- [x] sources.md — lecture_09.pdf was already recorded as transcribed. Fixed the header,
      which still said "covers Lectures 1-6" in one sentence and "1-7" in the next, and
      the now-false "nothing is committed as a binary"
- [x] kb.json — coverage 9/18, slideDecks 5, byLecture."9" (superseded by run 11's
      update to 10/18; boxes ticked retrospectively in run 11 after confirming both
      artifacts are present)
- [x] AGENTS.md — run 9 precedents (the "Run 9 precedents" section exists)
- [x] Commit and push — pushed to chaimantec/cairn-kb-cs336 at 4a8b703
- n/a  kbUrl already set on the catalog entry from run 1

## Not done (future runs)
- [x] Lecture 10 (Inference) — DONE in run 11.
- [ ] Lectures 16–18 — transcripts and wiki pages. (Lecture 14 done in run 15,
      lecture 15 in run 16.) **Lecture 16 (Post-Training — RLVR) is the natural
      next one**: lecture 15 defers RLVR, GRPO and reasoning models to it by name,
      and rlhf.md, ppo.md, dpo.md, reward-overoptimization.md and
      mode-collapse-and-calibration.md were all written to be extended by it —
      each already says which part lecture 16 still holds. It is a PDF deck
      (`lecture_16.pdf`, 6.8 MB), so page-images.
      *(Superseded note, kept for the record.)* Lecture 11 (advanced scaling
      laws) was the natural next one at the time: it closes the scaling-laws pair that lecture 9
      opened and lecture 10 interrupts, and the wiki already has eight topic pages
      waiting to be extended by it. It is a Hashimoto PDF deck, so page-images plus
      figure audits.
### Figure audit pass 3 (run 9 continued) — 19 chart pages, 3 at a time, sequential
Pages 7, 8, 9, 10, 16, 22, 23, 25, 30, 32, 33, 35, 36, 37, 39, 43, 44, 47, 53.
Chosen by a triage that scored each unaudited section for chart signal and for how
many of its numbers also appear in the transcript. The answer for most was "almost
none" — page 25 had 3 of 40 corroborated, page 30 3 of 18 — so unlike the Chinchilla
numbers, these values have NO second source and can only be checked by looking.
Agents now append findings per page, per the run-9 lesson.
- [x] Batch 1 — pages 7, 8, 9. 2 dirty, 5 errors, applied. Page 7: BOTH series' marker
      fills were wrong — training error is open triangles throughout (the dense
      cluster only looks solid because outlines overlap), and test error is open for
      only its first five or six points, solid black thereafter. Page 8: Winnow's 0.95
      plateau was placed a full data point too early, at x=10 where it is actually
      ≈0.91 inside a three-way bundle, plus an invented dip to 0.94 at x=100. Page 9's
      six-formula table was exact character for character.
- [x] Batch 2 — pages 10, 16, 22. 2 dirty, 3 errors, applied. Page 10: a middle-panel
      endpoint read at 0.30 when the solid curve ends at 0.39 and its trend line at
      0.36 — the 0.30 matched neither series — plus a FOURTH fabricated "cropped"
      claim, this time about a panel that is fully rendered with every tick visible.
      Page 22: the orange and green endpoints were overstated about 4x. That
      correction is self-verifying — the measured values put all three series'
      slopes at -0.94/-1.00/-0.98, closely parallel, which is precisely what the
      slide's own text claims ("data composition affects the offset, not the
      slope"), while the overstated ones implied -0.69/-0.66 and contradicted it.
      The auditor also CONFIRMED the earlier reader's hardest call on page 22: the
      pale line crossing the legend box really is the q=0.00 series continuing
      behind a semi-transparent box, not a fourth series.
- [x] Batch 3 — pages 23, 25, 30. 2 dirty, 6 errors, applied. **Page 25 came back
      CLEAN**, which is the most reassuring result of this pass: it was the highest-risk
      page in the triage — ~40 numbers, only 3 spoken aloud — and every point across
      three charts measured within ~0.01, with the three fitted exponents
      (0.23/0.23/0.24) genuinely supporting the slide's "slopes stay similar" claim.
      Page 23: three black x-marks on the data-mixing bowl, not two.
      Page 30: four errors, including the THIRD linear-vs-log axis mislabel in this
      deck (settled by fitting tick rows: R^2 0.99998 log vs 0.986 linear). Also a
      claim that "4 Layers" stops leftmost of the three LSTM curves — which the
      file's OWN recorded coordinates already contradicted, so that one was
      detectable without opening the PDF. Worth remembering: internal consistency of
      an entry is a free check.
- [x] Batch 4 — pages 32, 33, 35. ALL THREE dirty, 11 errors, applied. The worst
      batch of the pass. Pages 33 and 35 BOTH had their shared "Test Loss" y-axis
      recorded as linear when it is logarithmic — that is the fourth and fifth such
      axis in this deck, and on 33 it means the chart is log-log rather than
      log-linear, changing how every curve's position reads. Page 33 also had three
      series wrong: 2 Layers and 3 Layers were given 1 Layer's starting point, 3
      Layers was run out to 1.3e9 when it stops near 1.7e8, and 6 Layers and
      "> 6 Layers" were said to join together when the gold series starts over an
      order of magnitude later. Page 35 repeated the same false convergence for the
      identical chart, and stated an x-range of 10^6-10^9 that its own next sentence
      contradicted with a 2e5 start point. Page 32's four endpoint values were all
      slightly off, though its synthesis held — the fitted exponents really are
      -0.094 vs -0.095, so the slide's "slopes barely change" claim stands.
      SECOND internal contradiction found this pass (after page 30's). Checking an
      entry against ITSELF costs nothing and has now caught two errors.
- [x] Batch 5 — pages 36, 37, 39. COMPLETE on the fourth attempt. All three dirty,
      9 more errors, applied — 11 counting the two salvaged earlier, so page 36 alone
      cost four findings after its half-audit. Page 39 was the worst: its blue-series
      bullet had THREE errors bundled together, the most interesting being that its
      stated endpoint was actually the ORANGE line's final point, duplicated into the
      wrong bullet, and that the early dip it describes belongs to orange too. Page 37
      conflated a solid arrow and a separate dash-dot extension into one object.
      Page 36's 3D panel had four of nine sparsity ticks each off by one point.
      Recorded a CROSS-SLIDE note in the slide file's front matter: slides 36 and 55
      show the same Abnar 3D figure but embed DIFFERENT image objects (different
      MD5s), so their tick lists legitimately differ and have not been reconciled.
      Slide 36's is measured; slide 55's is not. Do not assume either is wrong.
      FOUR attempts were needed: session rate limit, server error, 600s stall, then
      success. Original partial note follows.
- [~] (superseded) Batch 5 first attempt — PARTIAL. Page 36 is HALF DONE: two findings were
      salvaged from an agent killed mid-task and are applied (panel (b)'s y-axis is
      labelled "Pretraining Loss L" ticked 2.2-3.0, so the illegibility claim was
      false — that is now SIX false illegibility claims out of six ever tested; and
      panel (c)'s x-axis has a seventh tick, 4B). Page 36's remaining panels, and
      pages 37 and 39 entirely, are NOT audited.
      THREE consecutive agent failures here, three different modes: session rate
      limit, server error mid-response, then a 600s stall. The session limit resets
      at 16:00 Asia/Bangkok and low-priority mode was on, which pauses for spare
      capacity — so these look like capacity failures, not task failures. Retrying a
      fourth time in the same window is likely to burn quota for nothing.
      The salvage did work, though, and it is worth recording as the payoff for the
      run-9 lesson: the agent told to create its findings file as its literal FIRST
      action, before reading anything, left behind a usable file; the one told merely
      to "append as you go" left nothing when it died early.
- [x] Batch 6 — pages 43 and 47 (44 handled separately, see below). BOTH dirty, 8 errors,
      applied. Two more undisclosed logarithmic y-axes — page 43's Loss (R^2 0.99997 log vs
      0.9920 linear) and page 47's Training loss (0.9998 vs 0.9797) — taking this deck to
      SEVEN mis-stated axes, by a wide margin its most repeated error class. Page 47's
      Parameters axis was given as "100M to 10B" when its printed ticks run to 1T, two
      orders out; and the "67B"/"1.5T" teal labels were placed beside the vertical
      crosshair when they actually sit at the panel's left edge, ~2,000 px away. Page 43's
      3D landscape had the data-fraction direction backwards: the red high-error ridge
      traces to log2(data fraction) = -5, so the blue low-error corner is the FULL-data
      end, not the "less-data" end the entry claimed — which contradicted the slide's own
      thesis. That one was adjudicated in the parent with a confirming crop, because the
      agent flagged it as resting partly on monotonicity rather than a direct trace.
      Page 44: NOT audited by this batch. Its panel (a) grid was verified cell by cell
      while checking the committed slide images and is exact; panels (b) and (c) were read
      at page scale only. Recorded as partial in both the slide file and kb.json.
- [x] Batch 7 — page 53. Dirty, 4 errors, applied. The best of them is this deck's THIRD
      internal contradiction: the entry capped Hoffmann's residuals at +0.06 while its own
      axis note said points sit above 0.10 — measured, two green outliers at +0.115 and
      +0.125. The "Ours" outlier was also wrong in both count and value (two points at
      +0.154/+0.169, not one at +0.11), the green/blue crossing is at 7-9e18 FLOPs rather
      than ~1e20, and the Chinchilla dot sits at 6.2e23 rather than 3-4e23 — which is
      corroborated by the 5.76e23 Gopher budget the deck itself quotes on slide 47.
- [x] After all batches — record written ONCE, at the end. This was overdue: pass 3 had
      run five batches and applied 36 corrections while the slide file's front matter and
      kb.json both still said "TWO PASSES DONE, 18 of 57", so the KB was telling readers
      that pages it had audited were provisional. Exactly the index rot AGENTS.md warns
      about, and it understated the work rather than overstating it.
      Final tally, checked arithmetically rather than by eye (an earlier draft of this
      record said 37 audited; 9 + 9 + 18 = 36): **36 of 57 pages fully audited across three
      passes, 26 of them dirty, 84 errors found and applied**, plus existence-only checks
      on 17/19/27/28 and a partial on 44. 20 pages remain unaudited.
      Also resolved the two provisional flags in wiki/compute-optimal-scaling.md: slide
      47's 1.5T token figure and slide 43's two joint-scaling equations were both confirmed
      against the page, so the footnote no longer hedges them. Checked that no wiki page
      repeats any of the 12 newly-corrected values — none does; the Epoch AI passages
      describe the finding qualitatively from the verified transcript.

- [ ] Figure audit, lecture 9: the remaining unaudited pages beyond those 19. Highest value are the ones the
      wiki quotes and could not check: 37, 38, 39 (critical batch size) and 43, 44,
      46, 47 (joint scaling, Chinchilla methods 1-2). A pass over these was killed by
      a rate limit. WHEN RE-RUNNING, tell the agent to append findings to a file per
      page — the pass that failed reported at the end and lost everything.
- [ ] Transcribe the 1 remaining PDF deck (lecture 16) — page-images, not
      source-text, and a figure-audit pass if it is as chart-dense as lecture 9.
      (Lecture 11's deck was done in run 12, lecture 15's in run 16 — the latter
      WITHOUT an audit, at the user's instruction, which is recorded in its front
      matter and in kb.json and should not become the default.)
- [ ] Transcribe the 1 remaining executable lecture (17). Check each for a
      published `var/traces/lecture_NN_stdout.txt` in the lectures repo before
      writing off its runtime values as machine-dependent — lecture 7 had one.
      (Lectures 12 and 13 were done in runs 13 and 14; neither computes anything,
      so neither had runtime values to recover.)
      Lecture 10, done in run 11, needed neither: it computes symbolically, so every
      value is reproducible from the source with sympy and nothing is
      machine-dependent. Check for that shape first — it is much the cheapest case.
- [x] Describe the figures in lectures 1, 2, 6 and 7 — DONE in run 10, and again for
      lecture 10 in run 11 (22 figures, the largest set of any executable lecture).
      What still has no description is the THIRD-PARTY layer: the figures these
      lectures display by external URL (NVIDIA docs, arXiv, Wikimedia, Springer, the
      JAX scaling book, Baseten, Anyscale) are recorded as links only and were never
      looked at, because they are not ours to copy. No wiki claim rests on one.

## Run 10 — Slide images (Step 1c) for lectures 1-9

The user opted in to full figure coverage, to the course's own images for the four
executable lectures, and to attribution in `AGENTS.md` rather than asking the course
staff first. Source repo `stanford-cs336/lectures` carries **no LICENSE file**, so the
material is public-to-read but not explicitly licensed for redistribution; that is
recorded in `AGENTS.md` and in `kb.json` caveats.

Selection method, per deck: the PDF raster test (a pasted image covering >4% of the
page) INTERSECTED with the slide file's own prose describing an actual figure, minus
title cards, outlines and section dividers. Then adjudicated by hand, because the two
signals disagree in both directions:
  - ADDED, raster test missed them (vector-drawn, no pasted raster): L3 32 (the
    hand-drawn RoPE rotation diagrams), L5 2 (an "Outline" slide that actually carries
    the matmul-throughput scatter and the FlashAttention figures).
  - DROPPED, raster present but not a figure: L3 31 (equations), L4 21, 22 (pasted
    paper tables the file transcribes cell by cell), L8 59, 66, 67, 69, 71 (same).
    The skill's rule is that a transcribed table beats a picture of one, because it can
    be quoted, searched and cited by cell.

### Render
- [x] L3 46/67 pages — NOTE: lecture 3 needs the number-map bypass. A stray "1" on
      page 61 (the numerator of the MQA arithmetic-intensity fraction) is read as a
      folio, which makes pages 61-67 resolve to slides 1-7 and collide. Pages 1-60 are
      unaffected. The slide file's headings are a plain 1..67.
- [x] L4 48/60, L5 51/55, L8 51/73, L9 40/57 — identity map, script works directly
- [x] Lectures 1, 2, 6, 7 — 23 of 25 fetched. Dropped `course-staff.png` (staff photo
      grid, no course content) and `ranks.png` (four boxes labelled Rank 0-3, which the
      prose states completely). The course's own `images/*.png` from the lectures repo.
      Third-party hotlinks (NVIDIA docs, arXiv, Wikimedia, Springer, jax-ml) are NOT
      copied; they stay as URLs in the slide files.

### Wire in
- [x] embed_slide_images.py into raw/slides/ and wiki/ for each deck lecture. 259 images
      in raw/slides/ (one per rendered page), 257 refs across 59 wiki pages. Topic-page
      anchoring needed a per-page deck assignment, since a bare "slide 44" is ambiguous
      across five decks that all print no folio: 41 pages resolved to one deck by the
      slide file they link, 13 mixed pages were adjudicated by reading every citation in
      context, and 5 of those needed a per-slide split (critical-batch-size L8+L9,
      training-stability L3+L4, transformer-hyperparameters L3+L9)
- [x] Captions. The script captions the deck renders from the slide heading; the 23
      executable-lecture figures were each looked at and given a written description,
      which is the first time this KB describes any of their contents
- [x] AGENTS.md — Images section: coverage table, the no-constructed-paths rule, what
      was rendered and what was not, attribution, the no-LICENSE note and a takedown
      procedure. Also fixed "three decks" -> "five decks" in the layout table
- [x] kb.json — materials.images with per-lecture counts, 3 new caveats, and 2 stale
      ones corrected (the method split still said 3 decks; the figures caveat still said
      lectures 1, 2, 6, 7 had no described figures)
- [x] INDEX.md — a raw/images/ entry in the source-material section, and the
      "no binaries" paragraph corrected to name its one exception

### Check
- [x] slide_number_map --verify PASSES on all five decks after the insertions (lecture 3
      via the bypass, as above)
- [x] every referenced image path exists: 259 files, 259 slide-file refs, 257 wiki refs,
      0 missing. Full repo link sweep: 1,906 relative links, 0 broken
- [x] Read two rendered images against raw/slides. BOTH CLEAN. L9 slide 44's Rosenfeld
      grid checked cell by cell across all seven rows (the file's green/red/open split is
      exact); L8 slide 40's Megatron tensor-parallel diagram checked box by box, including
      the f/g forward-backward bullets. Worth recording: the 1400px render is NOT enough to
      audit a dense panel — the first read of slide 44 at thumbnail scale miscounted row -4
      and looked like an error. Crop from the PDF before calling a discrepancy

### Licensing (added after the image work, in response to a question about reuse)
- [x] Established that CS336 has NO licence of any kind — not CC BY-NC-SA, not anything.
      Checked on 2026-09-03: no LICENSE/LICENSE.md/COPYING/NOTICE in stanford-cs336/lectures,
      no reuse terms in its README, no licence/copyright/reuse statement on
      cs336.stanford.edu, no licence text on any page of any transcribed deck and none in
      any PDF's metadata, and no Creative Commons marker on the YouTube recordings
      (consistent with the default Standard YouTube License). Default all-rights-reserved.
- [x] Checked the third-party layer against arXiv licence metadata. 9 of the 10
      most-reproduced sources are arXiv nonexclusive-distrib/1.0, which grants arXiv
      distribution rights and third parties NONE. The one exception is Wei et al.,
      Emergent Abilities (2206.07682), which is CC BY 4.0.
- [x] Measured the per-source amount, which is the evidence for the fair-use position on
      the papers: 32 distinct cited sources across the 236 rendered pages; heaviest is
      Kaplan et al. at 8 pages spread over three lectures, then Fedus et al. at 8 in one;
      only 3 of 32 appear on more than three pages, most once or twice. That is the same
      order of use the lectures themselves make.
      CORRECTION TO AN EARLIER CLAIM IN THIS BUILD: "fair use doesn't chain" was the wrong
      frame. This KB has its own independent position against each paper's rightsholder,
      judged on what it took from THAT work. The 68-92% figure is the amount relative to
      Stanford's DECK (layer 2) and must not be applied to the papers (layer 3).
- [x] LICENSE.md — scoped, three layers. CC-BY-4.0 on the compilation's own editorial
      work only; raw/transcripts/ and raw/slides/ explicitly NOT offered under it, because
      they are derivative of material that is not ours to license; layer 2 and 3 stated as
      unlicensed with the evidence; takedown procedure.
- [x] Linked from INDEX.md (the Also list and the images entry), AGENTS.md (layout table
      and the images provenance section) and kb.json (a new top-level `license` block)

### Remove the verbatim captions from the repo
- [x] `raw/transcripts/original/` gitignored and untracked — 16 files, 2.0 MB, 132k words,
      25% of the repo's text and the weakest item in it on every fair-use factor except
      purpose: a complete verbatim reproduction of nine lectures with no editorial
      contribution at all. Kept on disk locally, exactly as `raw/pdfs/` is.
- [x] Audit trail preserved as a recipe rather than a copy: `fetch_transcript.py <video_id>`
      piped through `transcript_to_md.py` reproduces them exactly, and the video id is in
      each transcript's front matter. Note lectures 8 and 9 never had a `.segments.json`
      stored, so the video is the source of truth for all nine either way.
- [x] Rewrote every reference so nothing dangles: 9 transcript headers (front matter plus
      the inline sentence, whose wording varied per file), 5 wiki pages, AGENTS.md's layout
      table, INDEX.md's source-material list, LICENSE.md and a kb.json caveat.
- [x] Checked: 1,895 relative links, 0 broken, and 0 links pointing at a file that is on
      disk but not in the repo — so a public reader hits no dead reference.
- [ ] NOT DONE, and a deliberate open question: the captions remain in git HISTORY and on
      GitHub, since they were committed and pushed before this. Removing them for real needs
      a history rewrite (git filter-repo or BFG) plus a force push, which would invalidate
      every commit SHA — including the ones this file cites — and would not purge GitHub's
      unreferenced objects without asking support. Left as-is: the material is out of the
      working tree, unbrowsable and unindexed, which is the practical benefit; a rewrite is
      a separate decision.

### Publish
- [x] Commit and push

## Run 11 — Lecture 10: Inference

Video EfM546A79aM (85 min). Percy Liang. **Executable lecture** — `lecture_10.py`,
611 lines, so `source-text`, no deck and no page numbers. Cite function names and
source line ranges, not slide numbers.

### Course material
- [x] raw/slides/10-inference.md — transcribe lecture_10.py (611 source lines ->
      1113 lines). All 279 text() literals accounted for, all 30 source URLs
      present, all 28 image() calls recorded. Every @inspect value recomputed in
      sympy 1.14 and matched against the source's own asserts: the two intensity
      limits (B and B*T), S*T/(S+T) with its S/2 and S/(S+1) specializations,
      accelerator_intensity 295.22, and the full Llama-2-13B performance table at
      B=1/64/256 and the two GQA variants. This lecture has NO machine-dependent
      numbers — the arithmetic is symbolic, so nothing had to be withheld.
      Source discrepancies recorded: (1) the GQA comments say "worse latency" when
      the computed latency improves against the row above — they are consistent
      only against the B=1 baseline; (2) the TransformerPerformanceStats docstring
      says num_params is "in bytes" when it is a count; (3) quantization says
      "higher latency" where it means better.

### Transcript
- [x] 10 — verbatim captions fetched (111 paragraphs, ~13.1k words)
- [x] 10 — copy-edited transcript (drafted by Sonnet, adjudicated here)
- [x] 10 — verify: all three checks pass, after one real fix.
      Timestamps: 111 markers, identical sequence.
      Numbers: NOTHING lost. Three digits added, each a word-to-digit
      normalisation of a mathematical or format term — "N over three" -> "N over
      3", "S over S plus one" -> "S over S plus 1", "int four" -> "int4". The
      instructed DeepSeek v4 restoration was applied at both 1:37 and 2:23 and is
      net-neutral in the count (it replaces "GPT-4").
      Word ratios: 86.0% retention. FOUND A REAL DRIFT — the sentence "Yeah, I
      guess maybe I'll say that Mamba and DeltaNet are more powerful than
      sliding-window attention. Maybe you can think about the Mambas" had been
      moved from [1:01:03] into [1:01:49], which passes both the timestamp and
      number checks and breaks exactly the citation the markers exist for. Moved
      back in the parent; both paragraphs then sit inside the band (0.85, 0.76).
      The three remaining outliers (1:00:17, 1:05:45, 1:14:54, all 0.69-0.70) were
      read in full and are pure filler removal.
      A distinctive-word sweep across all 111 paragraphs found no other
      cross-boundary movement.
      Restorations: 26 of 28 restored terms appear verbatim in the course
      material. The two that do not are "QKV" (the source writes Q, K and V
      separately, in that order — a letter-order fix, not a new name) and "Tatsu"
      (a spoken name with no printed counterpart). 4 [Ed:] notes mark genuine
      ambiguity; 11 student-question markers over 8 exchanges.

### Wiki
- [x] wiki/10-inference.md (291 lines)
- [x] Topic pages (10 new) — inference (the hub), kv-cache (the four axes:
      heads/dimension/layers/sequence), prefill-and-generation (the intensity
      table), latency-and-throughput (the Llama-2-13B performance model),
      quantization (formats, QAT/PTQ, GPTQ, AWQ), pruning-and-distillation,
      speculative-sampling (algorithm + the two-token exactness proof),
      continuous-batching (Orca, selective batching), paged-attention (vLLM),
      cross-layer-attention
- [x] Extend rather than duplicate: arithmetic-intensity gains the inference
      derivation; attention-variants gains the GQA performance table, the MQA
      verdict and the contested accuracy evidence; multi-head-latent-attention
      gains the 16384->576 ratio and Tables 8/9; sparse-attention gains DeepSeek
      v4's CSA/DSA/HCA; linear-attention and state-space-models gain the
      sliding-window comparison; flash-attention gains why it is ASSUMED by the
      inference accounting; precision-and-data-types points at the new
      quantization page; efficiency, course-map, executable-lectures updated
- [x] Fixed three stale claims found while extending: lecture 4's page said none
      of its three deferred topics were in the KB (two now are), INDEX's deck list
      omitted lecture_09.pdf, and AGENTS.md still said only 3, 4, 5 and 8 of the
      decks were done
- [x] INDEX.md — ten-lecture coverage: banner, Start here entry, a Lecture 10
      section with 10 annotated entries, raw-material section rewritten
- [x] Link sweep — 2,333 relative links, 0 broken, 0 missing anchors; all 94 wiki
      pages appear in INDEX.md; no LaTeX inside code fences
- [x] Citation checks — all 127 [MM:SS] citations in the new pages and all 32 in
      the lecture-10 sections of extended pages match a real marker.
      QUOTE CHECK, and it found real work: 104 quoted fragments were compared
      against the finished transcript and slide file. 45 did not match, because
      they had been drafted from the VERBATIM captions before the copy-edit landed
      and the editor legitimately reworded them ("produces is" -> "produces are",
      "waiting for a bus" recast, and so on). All 45 rewritten to the published
      wording. Two further slips fixed: an unmarked elision that dropped "kind of"
      from the lecture's closing line in five pages, and a citation of the Mamba
      quote at [1:01:49] that the drift fix moved to [1:01:03].

### Images (Step 1c)
- [x] Copy the 22 course-own images/*.png into raw/images/10-inference/ (3.1 MB).
      The 6 third-party hotlinks (4 jax-ml scaling book, 1 Baseten, 1 Anyscale) are
      NOT copied; they stay as URLs in the slide file.
- [x] 22 descriptions written by a Sonnet reader that opened and zoomed each image.
      Spot-checked 2 in the parent: mla-accuracy.png exact (12 numbers, param row,
      bolding, verbatim Table 8 caption); gqa-speed.png structurally right (3 series,
      GQA meets MHA at 64 groups) but its sub-1 values were stated more precisely
      than a y-axis with ticks only at 1 and 2 supports — rewritten as relative
      statements. Three source-image quirks recorded rather than silently fixed:
      "9% bettter" and "Je t'amie" are typos IN the images, and
      deepseek-v4-attention.png does not label CSA/DSA/HCA at all.
- [x] Embedded all 22 into raw/slides/10-inference.md by script (blank line either
      side, alt text, one-paragraph caption, attribution link). All paths resolve.
- [x] Wire into wiki/ pages — the 22 images live in raw/slides/10-inference.md
      under the figure marker each one belongs to. No wiki embeds: lecture 10 has
      no slide numbers, so there is no "cites slide N" anchor for the script to
      use, and the executable lectures' figures are referenced from the slide file
      the wiki links. Same shape as lectures 1, 2, 6 and 7.
- [x] AGENTS.md — images coverage table now runs 1-10, the executable-lecture
      count is five, the ownership-not-content filter for lecture 10 is stated, and
      the three source-image quirks are recorded

### Publish
- [x] Update sources.md (lecture_10.py now transcribed; the symbolic-computation
      note and the 22-of-28 image split recorded)
- [x] kb.json — coverage 10/18, 84 topic pages, executableLectures 5 of 9,
      byLecture."10" = source-text, images 281 files / 32.3 MB, 50 caveats (6 new)
- [x] Commit and push — pushed to chaimantec/cairn-kb-cs336 at 1e765f7
- n/a  kbUrl already set on the catalog entry from run 1

## Run 12 — Lecture 11: Scaling Laws (in the wild) — SLIDES + IMAGES ONLY

Video vTfEyOyzV9E (77 min). Tatsunori Hashimoto. PDF deck `lecture_11.pdf`, 58
pages. Scope this run, set by the user: **Step 1b (slide transcription) and
Step 1c (image extraction) only** — no transcript, no wiki, no publish beyond
committing what these two steps produce.

### Course material
- [x] Download lecture_11.pdf into raw/pdfs/ (gitignored) — 6.7 MB, 58 pages,
      PDF metadata author "Tatsu Hashimoto", created 2026-05-04
- [x] Numbering settled BEFORE any page was read. slide_number_map.py found no
      printed number on any page (both bottom corners scanned, bare number and
      running-footer forms). Same case as lectures 3, 4, 5, 8 and 9: `## Slide N`
      == PDF page N, a plain 1..58. Readers were told the mapping and forbidden
      from making a numbering judgment of their own; each was asked to report any
      printed folio it saw, as an independent check.
- [x] Page profile measured before reading: 50 of 58 pages carry a pasted raster
      covering >4% of the page. The 8 that do not are 2, 30, 47, 48, 49, 50, 53
      and 58 — the motivation slide, the "recent scaling law recipes" list, the
      four muP derivation pages, the robustness prose slide and the recap.
- [x] raw/slides/11-scaling-laws-in-the-wild.md — 1,908 lines, 58 sections, 210 KB,
      33.9k words, 392 table rows. The largest slide file in this build (lecture 9 was
      162 KB). Read by **six** Opus readers, not the four planned, because FOUR WERE
      KILLED by session rate limits in three waves. Incremental appending meant zero
      pages were lost and zero were re-read: the first wave delivered 1-12, 16-27,
      30-36 and 45-58, then 13-15, then 28-29 and 37-44. Merged by slide number by a
      script that refuses to assemble unless all 58 are present exactly once.
- [x] Heading-sequence check — PASS. 58 headings, 1..58, no gaps, no dupes, in order.
      `slide_number_map.py --verify` also passes (it degenerates to this check, since
      the map is a 1..58 fallback rather than something read off the pages).
- [x] Heading-vs-text-layer cross-check — 58/58 match the deck's own printed titles.
      NOTE A CHECKER BUG, the seventh in this build: the first version of this check
      took each page's heading as the first line of `get_text()`, which follows PDF
      content-stream order, not visual position. That reported page 49 as a mismatch.
      The page's real title sits at y=34 in 23.8pt ("Deriving muP (condition A2) part
      2") and the string the check picked up was body text at 15.9pt. THE READER WAS
      RIGHT AND THE CHECK WAS WRONG. Rebuilt to take the topmost band's largest-font
      spans, left to right; 58/58 then matched. Anyone rebuilding this check on another
      deck should start from position and font size, never stream order.
- [x] YAML front matter parses (13 keys); no LaTeX inside code fences (no code fences
      in the body at all)
- [x] Figure audit pass 1 — DONE, 7 pages (37, 11, 29, 41, 45, 35, 51), two agents
      because the first was killed by a rate limit after two pages. Both appended per
      page, so nothing was lost. RESULT: DIRTY — 2 clean (37, 51), 5 with errors,
      16 corrections (10 substantive, 6 minor), all applied.
      What held and what failed is the finding. EVERY measured quantity checked out:
      all four nominated axis-scale claims confirmed (the log axes on 29, 41 and 45
      that look linear because they are evenly labelled), slide 37's LR exponent
      settled three ways that never touch the 2-3 px glyph (Kaplan's published law
      predicts the slide's own drawn line to 0.3% under the 1e-3 reading and is off by
      exactly 10x under the alternative), slide 11's exponents confirmed, slide 35's
      calibration confirmed against the rejected alternative, and hundreds of values
      re-measured exactly — 30 markers on 37, all 88 loss values on 35, all 40 table
      cells on 51, all 17 points on 41, all 20 on 45.
      WHAT FAILED WAS THE INTERPRETIVE SENTENCE over correct data: which interval of a
      curve is flat (29), which series wins how often (29, 45), straight line vs curve
      (41), above vs level (41), and one inverted colour reading (35). Several were
      contradicted by the file's own table a few lines above — so check an entry
      against itself, it costs nothing.
      ONE CLAIM ABOUT THE SOURCE WITHDRAWN: slide 35's "colorbar runs opposite to
      surface height" is false. Kept in the file as a correction rather than deleted,
      with the lesson — the original had sampled the colorbar against its own tick
      rows, which fixes which end is which value but not which end the floor is drawn
      in, so a half-verified claim felt whole.
      FALSE ILLEGIBILITY again: slide 35's per-point labels, called unreadable, read
      fine at 6-14x. Still 0-for-every-instance ever tested in this build.
- [x] Figure audit pass 2 — DONE, 8 pages (40, 39, 43, 31, 34, 13, 24, 23). 1 clean (23),
      7 dirty, 15 corrections applied. Running total across both passes: 15 of 58 pages,
      3 clean, 12 dirty, 31 corrections.
      CONFIRMED AGAIN: every axis-scale claim tested was right, both passes, including all
      twelve axes on slide 13 and the three power-of-two linear axes on 31. Big blocks of
      values reproduced exactly (22 on 40, 38 on 39, slide 23's eight IsoFLOP minima).
      The slide 23 <-> 24 internal cross-check now holds at BOTH ends: the eight minima
      match the eight grey circles to 0.0013 bpb, about one pixel.
      FOUR NUMERIC FAULTS, which qualify pass 1's "only the prose fails" reading:
      slide 13's top-right list carried a PHANTOM value for a tan curve that is not drawn
      on that side of the panel (a loose colour tolerance picking up the neighbouring
      line's antialiased fringe) — and it contradicted the entry's own conclusion that
      5.0x is worst; slide 31 quoted values for Soap in a panel where Soap has no visible
      curve at all; slide 24's 67B offset was understated about twofold (-0.016, not
      -0.008); and two of slide 13's cells were 0.01 low, implying an end-of-run upturn
      that a dense trace shows does not exist.
      A DISAGREEMENT BETWEEN TWO OF OUR OWN ENTRIES, found and resolved: slides 4 and 31
      reproduce the same optimizer figure and their write-ups disagreed about Soap. Slide
      4's was right. Slides 31 and 36 also share a figure. Where this deck repeats one,
      the entries are a free second opinion — read them against each other.
      NOTE ON THE AGENT'S OWN REPORT: it summarised as "not one measured number was wrong
      in a way that mattered" while its own findings listed the four faults above. Checked
      against its evidence rather than taken at face value.
- [x] Figure audit pass 3 — **DELIBERATELY NOT RUN. The user stopped it on cost.** This is
      a decision, not an omission, and it is recorded as one so nobody later reads the file
      as unfinished business. By the build's own "a dirty sample means the sample was too
      small" rule a third pass was indicated (7 of 8 dirty in pass 2), and the rule is not
      being retracted — it was outweighed. Two passes over 15 pages had already cost more
      than the whole 58-page transcription, and the marginal find was falling: pass 1
      turned up an inverted claim about the source, pass 2 turned up refinements of tenths
      of a percent alongside its four real faults.
      What the KB does instead of a third pass is state the boundary and stop pretending:
      43 of 58 pages carry chart values no one re-checked, the slide file's front matter
      says so, and kb.json carries the same caveat. An acknowledged gap is worth more than
      an audit budget spent past the point of return.
      If anyone does resume it, the targets were: the multi-series log-log charts not yet
      touched (4, 12, 14, 17, 18, 21, 22, 26, 27, 28, 32, 36, 42, 44) and the 240-cell
      heatmap pair on 20 — concluding sentences first, then any value attributed to a
      series that may not be visible at all.

### Transcript (added after the audit was closed — the user asked for transcript and wiki next)
- [x] 11 — verbatim captions fetched, video vTfEyOyzV9E (101 paragraphs, ~15,950 words,
      2,356 raw segments, 76:55 of speech). Kept at
      `raw/transcripts/original/11-scaling-laws-in-the-wild.md`, which is gitignored, as
      every lecture's verbatim has been since run 10.
      These captions are the CLEANEST of the build so far: "StepFun", "Muon" and "Cautious
      AdamC" all survived intact, where CS224N's mangled "word2vec" into "word Tove". The
      damage is concentrated in "mu p"/"mup" (33 occurrences of muP), "Kimmy" for Kimi K2,
      and "Hess..." for Hestness.
- [x] 11 — copy-edited transcript (drafted by Sonnet, adjudicated here). 299 lines,
      13,963 words in the body.
- [x] 11 — verify: ALL THREE CHECKS PASS.
      Timestamps: 101 markers, identical sequence, in order.
      Numbers: nothing lost. Two digits added, both normalisations to the deck's own
      spelling ("Newton-Schulz five" -> "Newton-Schulz5", "MiniMax zero one" ->
      "MiniMax-01"). Two more cancel in the count and are deliberate: "96 data point per
      active parameter ratio" -> "96-to-1" (slide 27 prints it that way) and "I'll take
      1 minute" -> "one minute" (the captions had rendered a spoken word as a digit).
      Word ratios: ALL 101 paragraphs inside the 0.72-1.10 band, 86.5% retention, in
      line with the same lecturer's lecture 8 (84.0%) and 9 (84.6%). No drift.
      TWO PARENT CORRECTIONS. (1) The [Ed:] note at 23:05 had REPLACED the words it was
      commenting on, deleting "Chinchilla 2" from the lecturer's speech — the note read
      as the sentence's subject. Spoken words restored, note now follows them. This is a
      new failure mode for this build: an editorial note that deletes rather than
      annotates passes the timestamp and ratio checks and only shows up in a number diff.
      (2) One dictated formula was set as `H_{L-1}`, the ONLY LaTeX-like construct in the
      file and inconsistent with the same paragraph's own "square root of n sub L".
      Spelled out.
      RESTORATIONS: ~25 terms, all verified verbatim in the deck except three, each
      checked elsewhere rather than trusted — Hestness (10x in lecture 9's deck, 11x in
      its transcript), Newton-Schulz (the deck writes it unhyphenated, and inconsistently:
      "NewtonSchultz" in prose, "NewtonSchulz5" in the algorithm), and Marin (the captions
      heard "Moraine"; corroborated by wiki/scaling-laws.md's lecture-1 material AND by
      slide 41's own cited URL, oa.williamheld.com — the agent had flagged this one as
      outside knowledge, and it is better attested than that).

### Wiki
- [x] wiki/11-scaling-laws-in-the-wild.md (331 lines)
- [x] Topic pages (5 new) — maximal-update-parametrization (201 lines: the two
      conditions, both derivations, the prescription, and the three failure modes),
      optimizer-scaling, published-scaling-recipes, wsd-schedules, step-law
- [x] Extend rather than duplicate: learning-rate-scaling-and-mup (which had been
      promising "I'll go into much more detail in the advanced scaling lecture" since
      run 10 with nowhere to send the reader — that now resolves),
      critical-batch-size, compute-optimal-scaling, isoflop-method,
      scaling-law-methodology, training-stability, scaling-laws (hub TOC), course-map
      (unit 3 was "half covered")
- [x] INDEX.md — banner now 1-11, the scaling-laws note rewritten, a Start-here entry,
      a Lecture 11 wiki section with 6 annotated entries, the transcripts list, and the
      two raw-material notes that described the partial state
- [x] Link sweep — 2,431 relative links, 0 broken; all 100 wiki pages appear in INDEX;
      no LaTeX inside code fences
- [x] Citation check — all 58 timestamp citations across the six new/edited pages match
      a real marker.
      QUOTE CHECK, and it found real work: 80 quoted fragments compared against the
      finished transcript and slide file. THREE were misquotations of the lecturer —
      "with respect to loss" for "with respect to the target loss", "if you haven't
      played with it" for "with that before", and an elision that dropped "and a lot
      more unknown than that". All corrected to the published wording.
      FOUR more were the parent's OWN paraphrases set in quotation marks, reading as
      quotations of the lecturer when they were not — the exact failure AGENTS.md
      records from runs 8 and 9, now recurring a third time. All converted to italics.
- [x] kb.json — coverage 11/18, 89 topic pages, the course-material-only caveat replaced
      (it was false once the wiki landed) and a transcript-verification caveat added;
      58 caveats
- [x] sources.md, AGENTS.md — the "no transcript and no wiki page" notes removed

## Run order for future lectures — set by the user after run 12

**Do the transcript and the wiki immediately, before any figure auditing.** Run 12 inverted
that: it produced course material and images, then spent two audit passes, and finished with
lecture 11 having a fully transcribed 58-page deck and no transcript and no wiki page. That
is the least useful shape a lecture can be left in — the deck is the supporting material and
the wiki is the thing a learner actually reads, so the run bought the cheap half first and
ran out of budget before the valuable half.

The rule for the next lecture:

1. Slide transcription (Step 1b) — needed first, because the transcript edit is cross-checked
   against the deck.
2. **Transcript (Steps 1 and 1a), then the wiki (Step 3), immediately.**
3. Images (Step 1c) after that.
4. **ONE figure audit pass, then stop.** Two passes over 15 pages cost more than transcribing
   all 58, and the second pass's yield was mostly refinement. Audit the pages the wiki
   actually quotes, rather than the pages that look most chart-heavy — a wrong value nobody
   cites is cheaper to leave flagged than to hunt.


### Model choice
- [x] Put to the user, as the skill requires for an unusually chart-dense deck.
      **The user chose Opus readers**, against the Sonnet default used for
      lectures 3-9. The evidence for the question: this deck is 50/58 raster
      pages, the same profile as lecture 9, whose four Sonnet readers produced
      the dirtiest file in the build — 26 of 36 audited pages carried errors, 84
      in all, over three audit passes, one of which was killed by a rate limit
      and lost everything. The trade being made is transcription cost up front
      against audit passes afterwards.

### Images (Step 1c)
- [x] SUPERSEDED — these two boxes were left unticked when run 12 was interrupted,
      but the work was done: lecture 11 has had 32 images since that run (see
      kb.json's images.byLecture). Ticked in run 14 after checking the artifacts
      rather than the boxes.

## Run 13 — Lecture 12: Evaluation

Video JpAxdTWQJxM (78 min). Percy Liang. **Executable lecture** — `lecture_12.py`,
394 lines, so `source-text`, no deck and no slide numbers. It is the most
image-dense executable lecture in the course: 43 `image()` calls, 33 of them the
course's own `images/*.png` and 9 hot-linked to third parties.

Run order is the one the user set after run 12: course material first, then the
transcript and the wiki immediately, images after.

### Course material
- [x] raw/slides/12-evaluation.md — transcribe lecture_12.py (1,374 lines). Written
      here rather than delegated: at 394 lines of `text()`/`link()`/`image()` calls
      the source is cheap to read, and the parent has to read it anyway to write the
      wiki. The file records a section→source-line table, a 26-row table of every
      benchmark the lecture names with its citation, and the lecture's own text in
      order. NOTE: this lecture computes NOTHING — no @inspect values, no sympy, no
      asserts — so unlike lectures 2, 6, 7 and 10 there was nothing machine-dependent
      to withhold and nothing to recompute. Every number in the file is a claim the
      lecture makes about a published benchmark.
- [x] Image descriptions for the 33 course-repo PNGs (delegated to Sonnet, appended
      per image so an interruption would have cost one entry). All 33 returned.
- [x] Spot-check two descriptions against the images here — DONE, and both held.
      `gpt2-perplexity.png` was EXACT across all 50 table cells, and independently
      confirms the lecturer's spoken PTB figures (he says "35 compared to 46"; the
      table prints 35.76 against 46.54). Its bolding claim was imprecise, not wrong,
      and is now stated per column. `arc-agi-results.png` was structurally right —
      two series, and the two dashed era annotations correctly NOT counted as series,
      which is the failure this check exists to catch. One correction: the 2025 blue
      points were said to span 5–65%, when a dense cluster also sits near 0–6%; the
      width of that spread is the more interesting fact and now says so.
      THE READER'S OWN FLAGS WERE THE HIGHER-YIELD OUTPUT: eight places where an
      image is not the kind of object its filename implies — `clio-table4.png` is a
      bar chart with NO printed values (so no number may be quoted from it), four
      `*-results` files are plain tables rather than charts, `cybench-results.png`
      has Subtask-Guided data for only the older nine models, `gdpval.png` is a card
      grid, and `hle-examples.png` marks no correct answers. All recorded in the
      file's Figure audit section.

### Transcript
- [x] 12 — verbatim captions fetched, video JpAxdTWQJxM (102 paragraphs, ~11,980
      words). Kept at raw/transcripts/original/12-evaluation.md (gitignored).
- [x] 12 — copy-edited transcript (drafted by Sonnet, adjudicated here). 1,150 lines,
      ~10,460 words in the body.
      PROCESS NOTE WORTH KEEPING: the agent ran 56 minutes without writing a single
      byte, despite being told explicitly to append in batches, and its own final
      report claimed it "appended incrementally in ~10-paragraph batches as I worked
      through it (never held unwritten)". THAT CLAIM IS FALSE — the parent polled the
      path throughout and the file did not exist until the very end. An agent's
      self-report about its own write behaviour is not evidence; poll the artifact.
      Had a session limit landed at minute 50, the whole edit would have been lost.
- [x] 12 — verify: ALL THREE CHECKS PASS.
      Timestamps: 102 markers, identical sequence, in order.
      Numbers: nothing lost. Every difference adjudicated — three "1 billion word
      benchmark" and one "1 hour" spelled out to the deck's own forms, "out of 10" ->
      "out of ten", a "20 20 ... 2024 or 23" stutter resolved, "01/03" -> "o1/o3",
      "ARC-AGI 1" hyphenated, and a "-4" restored to "GPT-4 preview".
      Word ratios: 101/102 in the 0.72-1.10 band at 88.3% retention; the one outlier
      ([48:32], 0.70) read and confirmed as pure filler removal.
      TWO PARENT CORRECTIONS. (1) A SENTENCE HAD MOVED ACROSS A TIMESTAMP BOUNDARY.
      The draft completed a sentence split at [55:30]/[56:16] by pulling its second
      half back into [55:30] — and carried the FOLLOWING whole sentence with it. The
      ratio check flagged the pair at 1.10/0.61 and the number check caught the stray
      "2024" independently, which is the two checks corroborating each other for the
      first time in this build. ROOT CAUSE WAS THE PARENT'S OWN PROMPT: it told the
      agent to "complete it in the first and start the second where the captions did",
      which contradicts this KB's actual convention. Measured it rather than guessed —
      68 of 102 paragraphs here begin mid-sentence, against 80/102, 78/111 and 82/101
      in lectures 9, 10 and 11. NEXT RUN'S PROMPT SHOULD SAY: leave a split sentence
      split exactly where the captions split it.
      (2) A possible benchmark name ("Gia", [1:13:22]) had been DELETED as noise
      rather than flagged. Now marked [Ed: unclear]. Deleting is not an available
      option for something that might be content.
      RESTORATIONS: 35, of which 32 appear verbatim in the lecture's own material.
      The three that do not are named in the header with separate evidence —
      Hendrycks (the lecture cites MMLU by arXiv id through references.py, and that
      record's author list begins "Hendrycks, Dan", so a standing [Ed:] note was
      WITHDRAWN rather than left), Tatsu's group (the deck links tatsu-lab.github.io
      twice), and GPT-3.5 Turbo — the weakest, an editorial expansion of spoken
      shorthand rather than an attested string, and labelled as such.

### Wiki
- [x] wiki/12-evaluation.md (301 lines + 3 figures)
- [x] Topic pages (8 new), grouped by kind per the user's choice —
      perplexity-evaluation, exam-benchmarks, chat-benchmarks, agentic-benchmarks,
      reasoning-benchmarks, safety-evaluation, benchmark-contamination,
      construct-validity
- [x] Extend existing pages — course-map (which was still claiming coverage of
      lectures 1–9; unit 4 now marked half-covered with a forward note),
      SEE_ALSO (the CS224N KB's evaluation cluster, which is complementary rather
      than duplicative: it derives metrics, this lecture judges them)
- [x] INDEX.md — banner 1–12, an evaluation note, a Start-here entry, a Lecture 12
      wiki section with 9 annotated entries, the transcripts list, and the
      executable-lecture and images paragraphs in Raw material
- [x] Link sweep — 2,616 relative links, 0 broken apart from the pending transcript;
      all 109 wiki pages appear in INDEX; no LaTeX inside code fences.
      NOTE A CHECKER BUG, the eighth in this build: the anchor check reported
      wiki/pipeline-parallelism.md → torch-distributed.md#async_op-and-overlapping as
      broken. It is not. GitHub KEEPS underscores in heading slugs and the checker's
      character class was stripping them. THE LINK WAS RIGHT AND THE CHECK WAS WRONG.
      Anyone rebuilding an anchor check must allow `_` in the slug alphabet.
- [x] Citation and quote check — 250 timestamp citations across the nine pages all
      match a real marker.
      QUOTE CHECK FOUND REAL WORK, for the third time in this build: 19 of 34
      quotations failed. Eleven were genuine quotations with drifted wording — a
      sentence-initial capital, a dropped article ("a grain of salt" for "grain of
      salt"), "LLM Arena" for "LMArena" — all corrected to the published text. Eight
      were THE PARENT'S OWN PARAPHRASES SET IN QUOTATION MARKS, reading as quotations
      of the lecturer when they were not; all converted to italics. One phrase
      straddles the [9:19] marker and cannot be a contiguous quotation, so it is given
      without quotation marks and the page says why.

### Images (Step 1c)
- [x] Fetch the 33 course PNGs into raw/images/12-evaluation/ (9.1MB, all PNG)
- [x] Embed: all 33 in raw/slides/ at the point they appear, and all 33 in the wiki
      passage that discusses them, plus 3 repeated on the lecture page. Inserted by
      script, not by hand — the script walks to the end of the enclosing list before
      inserting, so no list was split, and it enforces blank lines and a single-line
      caption with no nested emphasis. Verified afterwards: 0 list-splitting
      insertions, 0 malformed captions, 0 missing files.
      Unlike every other lecture in this KB, the wiki here carries EVERY image rather
      than a subset, because the figures are the lecture's argument rather than
      illustrations of it.
- [x] Record the 9 third-party hot-links as URLs only, not redistributed

### Publish
- [x] sources.md — lecture 12 row marked transcribed
- [x] kb.json — coverage 12/18, 97 topic pages, 346 images, 44.3MB; two new caveats;
      and TWO STALE ENTRIES FIXED that were carried from run 12: the caveat still
      saying "covers lectures 1-10 of 18", and the images note still saying lecture
      11 has "32 images but no wiki page" (it has had both since run 12). INDEX also
      still claimed lecture 11's deck had had no figure audit; it has had two passes.
- [x] AGENTS.md — image coverage table, the lecture-12 note, and the dating caveat
      that leaderboard screenshots are Spring 2026 snapshots
- [x] Commit and push (kbUrl already set; no re-link)

## Run 14 — Lecture 13: Data I (Sources, Datasets)

Video -qm0ln33G24 (82 min). Percy Liang. **Executable lecture** — `lecture_13.py`,
622 lines, so `source-text`, no deck and no slide numbers. 399 `text()`, 54
`link()` and 18 `image()` calls (14 course-repo PNGs, 4 hot-linked to third
parties). Much less image-driven than lecture 12: here the figures illustrate the
argument rather than being it.

Run order is the one the user set after run 12: course material first, then the
transcript and the wiki immediately, images after, then ONE figure audit pass.

### Course material
- [x] raw/slides/13-data-sources-datasets.md — transcribe lecture_13.py. Written
      here rather than delegated, as for lecture 12: at 622 lines of
      `text()`/`link()`/`image()` the source is cheap to read and the parent has
      to read it anyway to write the wiki. Records a section→source-line table
      (26 functions) and a 25-row table of every named dataset with the size the
      lecture states and its citation. Like lecture 12 this program COMPUTES
      NOTHING — no @inspect, no sympy, no asserts — so nothing was machine
      dependent and nothing needed recomputing; every number is a claim about a
      published dataset.
      NOTED: `alpaca_2023` is imported from references.py and never used. Recorded
      in the front matter so nobody later reads its absence as a transcription gap.
- [x] Image descriptions for the 14 course-repo PNGs (delegated to Sonnet,
      appending per image). ALL 14 RETURNED, and the agent DID append per image —
      polled throughout and the file grew 4 -> 7 -> 9 -> 14 blocks. That is the
      opposite of run 13's agent, which wrote nothing for 56 minutes and then
      falsely reported incremental writing. Same instruction, different outcome:
      the instruction is worth keeping but the polling is what establishes it.
- [x] Spot-check two descriptions against the images here — DONE, and the two
      were chosen as the highest-stakes claims rather than the most chart-heavy.
      RESULT: one confirmed, one corrected.
      decline-consent.png CONFIRMED, including the claimed MISMATCH: the lecture
      says the figure examines restrictions "for URLs in common datasets (C4,
      RefinedWeb, Dolma)" and the figure breaks out NOTHING by dataset — it is
      robots.txt composition, ToS composition, and a LOG-SCALE rate chart by
      crawler ORGANIZATION. All nine legend percentages matched verbatim. The
      log axis matters: the post-ChatGPT rise is steeper than a linear read.
      comma-results.png: structure and values CONFIRMED — five series in legend
      order, eleven benchmarks, and the six gold stars fall exactly where Comma
      leads the three 1T baselines (an inference, recorded as one). But ONE
      SUMMARISING SENTENCE WAS WRONG and is corrected in place: Qwen3 is NOT
      tallest in all eleven groups — level with or below MPT on HSwag and LLaMA
      on OBQA. Run 12's finding recurs exactly: the measured values held, the
      interpretive sentence over them did not.
- [x] Figure audit section written into the slide file, plus a figure_audit key
      in the front matter. Records the mismatch, the correction, eight unprompted
      reader flags (dclm-quality and nemotron-results are TABLES not charts;
      comma-results is a CHART not a table; dclm-filter is a Sankey whose
      percentages are by DOCUMENT COUNT, 1.4% surviving; tulu's 23.3M total is
      94% one row; the stackv2 colour inconsistency) and the audit boundary.

### Transcript
- [x] 13 — verbatim captions fetched, video -qm0ln33G24 (107 paragraphs, ~12,300
      words). Kept at raw/transcripts/original/13-data-sources-datasets.md
      (gitignored). Captions are clean by this build's standards — "Llama 3",
      "Qwen 3.5", "AI2" and "Olmo" all survived intact.
- [x] 13 — copy-edited transcript (drafted by Sonnet, adjudicated here). 1,157
      lines, ~10,300 words in the body.
      THE PROMPT FIX WORKED. Run 13's prompt told the agent to complete a sentence
      split across a marker; this run told it to LEAVE A SPLIT SENTENCE SPLIT
      EXACTLY WHERE THE CAPTIONS SPLIT IT. Run 13 got a sentence dragged across a
      boundary; this run got NONE — no paragraph pair shows the 1.10/0.61 signature
      of transposition. One prompt line, one whole failure mode gone.
      The agent also appended incrementally as instructed (polled: 14 -> 26 -> 39
      -> 52 -> 64 -> 76 -> 88 -> 98 -> 107 markers), unlike run 13's.
- [x] 13 — verify: ALL THREE CHECKS PASS.
      Timestamps: 107 markers, identical sequence, in order. NOTE 29 OF THE 107 ARE
      [H:MM:SS] — the lecture runs to 1:21:47 — so the narrow `\d+:\d+` regex that
      AGENTS.md records as a past failure would have skipped 27% of this lecture.
      The checker was written against that bug and the two others on record.
      Numbers: nothing substantive lost. Ten paragraphs differ, each adjudicated:
      four year-joins the captions split ("20 um 18" -> 2018, "20 20 three" ->
      2023, two bare "20" false starts); ONE DIGIT JOIN CORROBORATED BY THE
      LECTURE'S OWN MATERIAL — "400 uh 20 million repositories" -> 420 million,
      which lecture_13.py prints as "420M+"; "11 labs" -> ElevenLabs; one spoken
      SELF-CORRECTION removed as a false start ("40 — sorry, 800 gigabytes"); and
      the rest name restorations containing digits (Books1, Books2, WebText2).
      Word ratios: 84.6% retention, in line with lectures 8 (84.0%) and 9 (84.6%).
      TWO PARENT CORRECTIONS, BOTH FOUND BY THE RATIO CHECK.
      (1) A SUBSTANTIVE SENTENCE HAD BEEN DROPPED at 44:40 — ratio 0.56, the only
      paragraph well outside the band. The draft opened "website, but still",
      deleting the lecturer's actual answer to the student question that precedes
      it: that both a book and a website are copyrighted, but a book author can
      probably protect theirs better in court BECAUSE IT IS PUBLISHED. That is the
      substance of the answer, not a disfluency. Restored; the paragraph returned
      to the band and retention rose 84.3% -> 84.6%. This is the third distinct
      way an edit has lost content in this build — after the note-that-replaces
      (run 12) and the delete-as-noise (run 13), now the drop-mid-answer.
      (2) A WORD HAD BEEN INSERTED at 25:29: "not necessarily a *settled* fact"
      for the spoken "not necessarily a fact". Reverted. The edit is not licensed
      to sharpen a claim, even one that reads better.
      The other three outliers (0.65, 0.71, 0.72) were read in full and confirmed
      as pure filler and false-start removal.
      HEADER REWRITTEN BY THE PARENT: the draft's header asserted "All three
      checks pass" with invented supporting detail, which the agent had no way to
      run. Replaced with the parent's actual results and the two corrections.
- [x] 13 — restored-proper-noun sweep against the material file: all 33
      corroborated restorations appear verbatim in lecture_13.py's transcription
      (WET, resiliparse, DataComp-LM, arXiv x98, CAPTCHA, OLMo, Qwen3, RefinedWeb,
      FineWeb, Dolma, Bibliotik, Books3, RedPajama, WebText, Books1/2, fastText,
      DCLM x67, Nemotron, OpenHermes, ELI5, PII, Pushshift, trafilatura, jusText,
      MinHash, Smashwords, Gutenberg, Software Heritage, LLVM, CommonPile, Tulu).
      The eight the agent flagged as uncorroborated are genuinely absent and are
      all spoken asides with no printed counterpart — EXCEPT TWO, WHICH THE SWEEP
      PROMOTED. The agent had been TOO CAUTIOUS, which is the opposite of this
      build's usual failure:
        - MPT, called "the weakest, an editorial reading", is one of the five
          series in comma-results.png — the legend reads "Comma v0.1-1T, LLaMA,
          MPT, RPJ-INCITE, Qwen3", read directly off the image in the figure audit.
          The lecturer is reading the chart's legend aloud. Promoted, with that
          evidence written into the header.
        - Shayne Longpre (from "Shane Lampray") is corroborated twice over: the
          material carries "Longpre et al. (2023)" for FLAN v2, and the paper the
          lecture cites at that exact point — Consent in Crisis, arXiv 2407.14933 —
          is his.

### Wiki
- [x] wiki/13-data-sources-datasets.md (476 lines, all 14 figures embedded)
- [x] Topic pages (8, grouped by kind as the user chose in run 13) —
      pretraining-datasets (the chronological hub, BooksCorpus 2015 -> CommonPile
      2025), web-crawling (Common Crawl, WARC/WET, HTML->text, robots.txt and the
      politeness/selection/re-visit policies), data-filtering (the central
      disagreement: rule-based C4/Gopher vs model-based CCNet/GPT-3/DCLM, and
      Nemotron-CC's objection that everyone over-filters), deduplication (MinHash,
      Jaccard, Bloom filters, the 51B->5B GitHub figure), copyright-and-fair-use
      (the four factors, the lawsuits, "copying is the violation"), data-licensing-
      and-consent (robots.txt/ToS decline, shadow libraries, CommonPile's three
      subtleties), code-data (GitHub, Software Heritage, Stack v1/v2, the LLVM
      bridge), synthetic-data (Nemotron-CC rephrasing; the first real treatment in
      this KB).
      CHECKED FOR OVERLAP FIRST: no existing page covers crawling, dedup,
      filtering, copyright or synthetic data — only course-map mentions them in
      passing. data-mixture-selection/data-scaling-laws/data-repetition are about
      mixture PROPORTIONS from the scaling lectures, a different question from
      SOURCES, and should be cross-linked rather than extended.
- [x] Extend existing pages — course-map: unit 4's data half was marked "still a
      preview" and is now marked HALF COVERED, naming what lecture 14 still holds.
- [x] SEE_ALSO — DELIBERATELY UNCHANGED, recorded as a decision rather than an
      omission. Checked rather than assumed: the CS224N KB's 109 pages were listed
      and its only data-adjacent ones (09-pretraining, pretraining-and-finetuning,
      preference-data) are about pretraining OBJECTIVES and finetuning, not data
      provenance. Nothing in either sibling KB bears on sources, crawling,
      copyright or filtering, so adding an entry would have been padding an
      editorial file.
- [x] INDEX.md — banner now 1-13 (and the "no wiki pages for data" note replaced
      with "data is now half covered"), a Start-here entry, a Lecture 13 wiki
      section with 9 annotated entries, and the transcripts list
- [x] Link sweep — 2,798 relative links, 0 broken; all 118 wiki pages appear in
      INDEX; 0 missing images; no LaTeX inside code fences.
      NOTE TWO CHECKER BUGS, the ninth and tenth in this build, BOTH MINE AND BOTH
      IN THE SWEEP ITSELF. (1) It reported 17 broken anchors that were all correct.
      GitHub replaces EACH space with a hyphen and does not collapse runs, so a
      heading like "Collective operations — setup" slugs to "...operations--setup"
      with TWO hyphens; my collapse of `\s+` produced one. (2) It parsed
      `add_kernel[grid](x, y, ...)` out of a Triton CODE FENCE as a markdown link.
      THE LINKS WERE RIGHT AND THE CHECK WAS WRONG, both times. Anyone rebuilding
      this must replace spaces one-for-one and strip fenced/inline code first.
- [x] Citation and quote check — 198 timestamp citations across the nine pages all
      match a real marker.
      THE QUOTE CHECK FOUND REAL WORK for the fourth consecutive run: 148 quotations
      compared against the finished transcript and material.
      SIX WERE THE PARENT'S OWN PARAPHRASES SET IN QUOTATION MARKS — "looks like an
      encyclopedia" / "looks like a helpful answer" (twice, on two pages) and "keep
      only MIT and Apache" / "keep only permissively licensed web pages". All
      converted to italics. This is the SAME failure AGENTS.md records from runs 8,
      9, 12 and 13; it is now the most persistent single defect in this build.
      THREE WERE GENUINE MISQUOTATIONS: "will probably be able to protect that
      better" for the spoken "they'll probably..." (on two pages), and "not just how
      to generate code" for "not just LEARNING how to generate code" (on two pages).
      Corrected to the published wording.
      ONE WAS AN UNMARKED ELISION — Nemotron-CC "probably one of the main datasets"
      silently dropped the lecturer's "I don't know if it's the first, but probably
      not". Now marked with an ellipsis.
      SEVEN QUOTATIONS SPAN A PARAGRAPH BREAK and are legitimate: the captions split
      contiguous speech, so the quotation is faithful even though no single [MM:SS]
      paragraph contains it. These are cited as a SPAN (e.g. 43:54-44:40) rather
      than dropped. Run 13 handled its one such case by removing the quotation
      marks; citing the span is better, because the speech really was contiguous and
      the paragraph boundary is an artifact of caption grouping, not of the lecture.
      A THIRD CHECKER BUG, also mine: the first quote regex excluded newlines, so
      every quotation wrapped across two lines went unchecked — which is most of
      them. It reported 42 exact of 59; the corrected version checks 148. The fix is
      to collapse whitespace BEFORE extracting quotes, strip apostrophes on both
      sides (so 'copyright' matches "copyright"), and compare against a
      marker-stripped body so boundary spans resolve.

Material the LECTURER adds that the program does not have, found in a structural
skim of the captions and worth carrying into the wiki:
  - a student question on voice cloning / ElevenLabs and personality rights (~30:51)
  - a student question on pirated books vs website content, and how the two differ
    in an author's expectation of compensation (~43:54)
  - the NYT complaint's own evidence — prompting ChatGPT to reproduce an article
    almost verbatim (~27:47)
  - an explicit "I am not a lawyer" disclaimer (~31:37) which the wiki should carry
  - a closing preview of lecture 14 (~1:21:47)

### Images (Step 1c)
- [x] Fetch the 14 course PNGs into raw/images/13-data-sources-datasets/ (2.3 MB)
- [x] Embed into raw/slides/ by script — 18 @@IMG@@ tokens expanded, 14 local
      figure blocks and 4 external URL-only notes. The script refuses to run if
      any token lacks a description or any file is missing.
- [x] Embed into the wiki — all 14 are in wiki/13-data-sources-datasets.md, placed
      as the page was composed rather than inserted afterwards, which sidesteps the
      list-splitting and caption-nesting failures the skill warns about. Verified: 0
      missing image files across the repo.
- [x] Record the 4 third-party hot-links as URLs only, not redistributed

### Housekeeping found this run
- [x] sources.md — the banner said "covers Lectures 1-9 of 18", STALE SINCE RUN 10;
      AGENTS.md said "Coverage: Lectures 1-7 of 18", STALE SINCE RUN 8. Both are the
      exact failure mode the skill warns about — the chat reads them and trusts
      them — and both had survived several runs because each run updated INDEX and
      kb.json and stopped there. All four coverage statements now read 1-13 and were
      verified consistent by grep. sources.md's image count was stale the same way
      (259 images / 28 MB, now 360 / 47 MB).
      THE LESSON FOR NEXT RUN: coverage lives in FOUR places — INDEX.md, sources.md,
      AGENTS.md and kb.json — and a run that updates only the first and last leaves
      two lying. Grep all four before publishing.
- [x] sources.md — mark the lecture 13 row transcribed

### Publish
- [x] kb.json — coverage 13/18, 105 topic pages, 360 images / 47 MB across 13
      lectures, byLecture["13"]="source-text", executableLectures.transcribed 6->7,
      six new caveats, and a thirdPartyNotRedistributed key naming lecture 13's four
      hot-linked images. ONE STALE ENTRY FIXED, found by grepping rather than
      assuming: caveat 0 still said "covers lectures 1-12 of 18 ... Lectures 13-18
      have no transcripts and no wiki pages". It now says 14-18 and states
      explicitly that data is HALF covered.
- [x] AGENTS.md — image coverage table extended to lecture 13, and a note that its
      four hot-linked images must be given as URLs and never described, since
      nobody here has looked at them.
- [x] Commit and push — pushed to chaimantec/cairn-kb-cs336 (2a304b4..992c596).
      kbUrl was already set on catalog 94d9c003-2193-43e7-96e3-c1cb4ed0aba8 and was
      re-verified, so no link_kb.sh run. Live raw fetches confirmed 200 for INDEX.md,
      the lecture 13 wiki page, the transcript, kb.json and a lecture 13 image.

## Run 15 — Lecture 14: Data II (Filtering, Deduplication, Mixing, Post-Training)

Video 5sxHosTLPF8 (85 min). Percy Liang. **Executable lecture** — `lecture_14.py`,
464 lines, so `source-text`, no deck and no slide numbers. 248 `text()`, 31
`link()` and 18 `image()` calls (13 course-repo PNGs, 5 hot-linked to third
parties).

UNLIKE lectures 12 and 13, **THIS ONE COMPUTES**: 22 `@inspect`/`assert` lines
covering MurmurHash, exact dedup, Jaccard, a 100-seed MinHash simulation, LSH
collision probabilities at three (b, r) settings, and the data-mixing epoch
arithmetic. All of it is deterministic and machine-independent — no GPU, no
timing, no benchmark — so every value was recomputed here and is reproduced in
full, which is the first time in this build that has been possible for a
computing lecture (lectures 2, 6, 7 and 10 all had machine-dependent values to
withhold).

Run order is the one the user set after run 12: course material first, then the
transcript and the wiki immediately, images after, then ONE figure audit pass.

### Course material
- [x] raw/slides/14-data-filtering-dedup-mixing.md — transcribe lecture_14.py.
      Written here rather than delegated, as for lectures 12 and 13. Records a
      section→source-line table (11 functions), a 30-row table of every paper,
      dataset and tool cited, and a full "Every computed value in this lecture"
      table giving all 22 inspected values recomputed by running the lecture's
      own code (mmh3 at default seed 0, so they reproduce anywhere).
      NOTED: two unused imports — `download_file` from edtrace.file_util, and
      `the_pile` (the SECTION FUNCTION from lecture_13, not the reference).
      Recorded in the front matter so nobody later reads their absence as a
      transcription gap. Also noted: `keep_document` (GPT-3's Pareto keep rule)
      is defined and never called, which is why nothing in the file is random.
- [x] Image descriptions for the 13 course-repo PNGs (delegated to Sonnet,
      appending per image). ALL 13 RETURNED and the agent DID append per image —
      polled and the file was already at 9 blocks partway through. Second
      consecutive run where the incremental instruction held.
- [x] Spot-check two descriptions against the images here — DONE, chosen as the
      two highest-stakes claims rather than the most chart-heavy.
      swezero-results.png CONFIRMED EXACTLY: all 29 labelled points (23 blue
      baselines + 6 purple SWE-Zero/SWE-Hero) match cell for cell, all three arrow
      deltas (+5.9 / +6.3 / +4.7) are right, and the reader's own "before citing"
      caveat — that the deltas measure SWE-Hero against SWE-Zero and not against
      the field — checks out.
      data-filtering-scale.png: structure and values held, TWO CORRECTIONS made.
      (1) AN INTERPRETIVE SENTENCE WAS WRONG — dclm was called "briefly the lowest
      of all series" at ~450M tokens; cropping and upscaling that region shows it
      third of five, above high_quality and med_quality. THIS IS THE FIFTH
      CONSECUTIVE RUN in which the measured values held and a summarising sentence
      over them did not. (2) A COUNT WAS WRONG — "four" 1-epoch reference lines
      followed by a list of five; the legend carries one for each of the nine
      methods, and all nine token counts are now listed.
- [x] Figure audit section written into the slide file, plus a figure_audit key in
      the front matter. Records both corrections and SIX READER FLAGS, again the
      higher-yield output: openthoughts-sources.png lists ELEVEN code-domain
      sources, not the 27 the lecture text beside it claims; four of
      data-filtering-scale.png's nine legend entries have NO plotted curve;
      marin-token-viewer.png prints no data labels at all (all 30 values are ±30-50B
      estimates); swe-rebench.png contains exactly one number and NOT the lecture's
      3.4K-repo or 450K-PR figures; data-mixing-methods.png's pink shading has no
      legend; raw-target-schema.png is NOT a Venn diagram (T is drawn disjoint from
      R). Audit boundary stated: one pass, per the user's rule — the sample came
      back one exact and one with two corrections, which is the expected rate and
      not run 12's dirty-sample signature.

### Transcript
- [x] 14 — verbatim captions fetched, video 5sxHosTLPF8 (109 paragraphs, ~12,850
      words). Kept at raw/transcripts/original/14-data-filtering-dedup-mixing.md
      (gitignored). NOTE 32 OF THE 109 MARKERS ARE [H:MM:SS] — the lecture runs to
      1:24:05 — so the narrow `\d+:\d+` regex AGENTS.md records as a past failure
      would have skipped 29% of this lecture. The checker handles both forms.
- [x] 14 — copy-edited transcript (drafted by Sonnet, adjudicated here). ~11,100
      words in the body, ~55 distinct restorations logged in its header.
      THE AGENT WROTE NOTHING FOR ~30 MINUTES. Polling caught it; ONE status message
      asking it to flush to disk and it appended steadily from then on (35 -> 47 ->
      60 -> 71 -> 93 -> 109 markers). Run 13's agent had the same silent-start
      signature and lost everything; run 14's appended from the start. THE LESSON IS
      THAT POLLING PLUS ONE NUDGE IS ENOUGH — do not wait passively, and do not
      assume a silent agent is dead.
      The split-sentence rule held for the second consecutive run: no paragraph pair
      shows the transposition signature.
- [x] 14 — verify: ALL THREE CHECKS PASS.
      Timestamps: 109 markers, identical sequence, in order, 32 in [H:MM:SS] form.
      Numbers: 10 tokens differ, every one adjudicated — five decimal restorations
      the captions had dropped ("08"->0.08, "64"->0.64, "72"->0.72), a permutation
      "4 3152" split into "4, 3, 1, 5, 2", "51 from Microsoft"->phi-1, "01"->o1,
      "30 version"->SWE-Zero version, and one stuttered "50" removed as a false
      start.
      Word ratios: THE CLEANEST RESULT IN THIS BUILD — 87.5% retention and ZERO
      paragraphs outside the 0.72-1.10 band (range 0.77-0.97), against 84.0-84.6%
      and several outliers in lectures 8, 9 and 13.
      THREE PARENT CORRECTIONS, recorded in the transcript header itself.
      (1) A DIGIT JOIN REVERTED. The agent's own restoration table correctly said
      "1.2 two million" -> 1.2 million, but its body wrote "1.22 million" three
      times. 1.2M is what the material states AND what the pipeline arithmetic
      gives (75k questions x 16 answers). Body corrected to match the agent's own
      declared intent.
      (2) A RESTORATION PROMOTED — the agent was TOO CAUTIOUS, as in run 14. It left
      "100 uh you know works" (18:00) marked unclear rather than guess. It resolves
      exactly: the chart's own printed title is "Method comparison at d512 (157M),
      N=100 WARCs vs tokens", recorded in the figure description. Restored to "100
      WARCs", which also makes "a tiny fraction of Common Crawl" in the same
      sentence exact rather than vague.
      (3) A DIGIT RESTORATION ACCEPTED AND RECORDED rather than reverted: "120 of
      them didn't execute" -> 120,000, corroborated by the material's "32K
      executable + 120K nonexecutable" and by the "32,000" in the same sentence.
- [x] 14 — restored-proper-noun sweep: all but three restorations appear verbatim in
      a course material file. FOUR RESOLVE AGAINST LECTURE 13'S MATERIAL rather than
      lecture 14's, because the lecturer refers back — One Billion Word Benchmark,
      Nemotron, OLMo, and Reddit karma (which is what confirms "star Reddit posts"
      -> high-karma). THREE ARE SPOKEN-ONLY and are now flagged in the transcript
      header and in kb.json: Michael Ryan, WebOrganizer, and o1. A fourth spoken-only
      claim, the lecture's attribution of SWE-Zero to NVIDIA, is recorded in the wiki
      as what was said rather than as fact.

### Wiki
- [x] wiki/14-data-filtering-dedup-mixing.md (331 lines, 7 of the 13 figures
      embedded — the ones the lecture page's own argument needs)
- [x] Topic pages (6 new) — html-to-text-extraction (the transformation stage, the
      DCLM extractor table, the PDF/OCR path), quality-classifiers (the T/R
      framework, KenLM vs fastText, five worked recipes, and "there is no optimal
      threshold"), minhash-and-lsh (the full algorithm with every computed value —
      the flagship page of this run), post-training-data (the recipe, the taxonomy,
      OpenThoughts), agent-trajectory-data (the four SWE papers and the
      environment problem), plus a heavy extension of data-mixture-selection.
      CHECKED FOR OVERLAP FIRST, and the check changed the shape of the run: three
      of the obvious page names were already taken by lecture 13's work
      (data-filtering, deduplication, synthetic-data), so those were EXTENDED and
      cross-linked rather than duplicated, with a pointer at the top of each saying
      which page is the history and which is the how-to.
      THE MOST INTERESTING RESULT IS A DISAGREEMENT. data-mixture-selection now
      carries both lectures and says so: lecture 9 argues a small-scale mixture
      bake-off is sound BECAUSE composition moves intercepts and not slopes, and
      lecture 14 exhibits a mechanism (epoching on a scarce source) by which the
      ranking genuinely fails to transfer. Neither page previously acknowledged the
      other's position.
- [x] Extend existing pages — data-filtering (the argument as of lecture 14),
      deduplication (the three-part design space, the 61,036-times example,
      cross-dataset dedup), synthetic-data (post-training as the default, and the
      two counterintuitive OpenThoughts findings), web-crawling (DCLM's measured
      WET result), course-map (unit 4 now complete).
- [x] INDEX.md — banner now 1-14, a Start-here entry, a Lecture 14 wiki section
      with 7 annotated entries, and the transcript list.
      FOUND A STALE ENTRY THE OTHER FOUR PLACES HID: INDEX's raw-materials list of
      executable lectures still ended at lecture_12 — LECTURE 13 WAS NEVER ADDED in
      run 14, which updated the wiki sections and the transcript list and stopped.
      Both 13 and 14 are now listed.
- [x] Link sweep — 3,171 relative links, 0 broken; 373 images, 0 missing; all 124
      wiki pages appear in INDEX; no LaTeX inside code fences; 305 anchors checked
      with the one-hyphen-per-space slugging run 14 established, 1 broken and fixed.
- [x] Citation and quote check — 210 timestamp citations across the new and extended
      pages all match a real marker.
      THE QUOTE CHECK FOUND REAL WORK FOR THE FIFTH CONSECUTIVE RUN, and this time
      the largest haul yet: 201 quotations checked, and ~40 CORRECTED. None was a
      fabrication and none was a paraphrase-in-quotation-marks (the failure that
      recurred in runs 8, 9, 12, 13 and 14 did NOT recur here — one instance,
      deduplication's "3-sentence spans / exact match / remove all but one", was
      converted to italics). They were all SMALL WORD-LEVEL DRIFT from quoting the
      captions rather than the finished transcript: "just only get the target data"
      for "just get the target data back"; "compute poor" for "compute-poor"; "if
      you want to define quality to be math" for "if you define quality to be math";
      "there's more chances" for "there are more chances"; "20 times as much data
      which were not filtered" for "which was not filtered"; "doing the lifting
      here" for "doing the lifting". THE CAUSE IS STRUCTURAL AND WORTH FIXING NEXT
      RUN: the wiki was drafted from the VERBATIM CAPTIONS while the copy-edit agent
      was still running, so every quotation was taken from text that was about to
      change. Either wait for the edited transcript before quoting, or budget the
      correction pass.
      FOUR CHECKER BUGS THIS RUN, all mine, bringing the build's total to fourteen.
      (1) The citation checker used lecture 14's markers against pages that cite
      lecture 13 and 9, reporting 46 false failures. (2) The quote extractor matched
      only curly quotes, so it found ZERO of the 225 straight-quoted passages.
      (3) Its normalizer kept hyphens, so every hyphenation difference read as a
      misquote. (4) It had no ellipsis handling, so legitimately marked elisions
      failed. The corrected version checks 201 and passes 200, the one remaining
      being run 14's deliberately marked Nemotron-CC elision.

### Images (Step 1c)
- [x] Fetch the 13 course PNGs into raw/images/14-data-filtering-dedup-mixing/ (2.3 MB)
- [x] Embed into raw/slides/ by script — 13 @@IMG@@ tokens expanded. The script
      refuses to run if any token lacks a description or any file is missing.
- [x] Embed into the wiki — COUNTED RATHER THAN ASSUMED, and the first count I
      wrote was wrong in both directions. The lecture page carries 7 of the 13; the
      six topic pages carry all 13 between them; the material file carries all 13.
      That is the split the skill predicts and says not to fight — the lecture page
      takes the figures its own argument needs, and the topic pages take the rest.
      Verified: 0 missing image files across the repo (373 referenced, 373 present).
      All images were placed as each page was composed rather than inserted
      afterwards, which sidesteps the list-splitting and caption-nesting failures.
- [x] Record the 5 third-party hot-links as URLs only, not redistributed — now also
      in kb.json under materials.thirdPartyNotRedistributed, which did not exist
      before this run and now covers lectures 13 and 14.

### Housekeeping found this run
- [x] COVERAGE LIVES IN FIVE PLACES, NOT FOUR. Run 14's lesson named INDEX.md,
      sources.md, AGENTS.md and kb.json. It missed wiki/course-map.md, whose header
      still said "covers Lectures 1–12 of 18" — STALE SINCE RUN 13 — even though run
      14 updated that same file's unit-4 note. All five now read 1–14 and were
      verified consistent by grep. Add course-map.md to the list next run.
- [x] INDEX.md's executable-lecture list was missing lecture 13 (see the Wiki
      section above). Both 13 and 14 added.
- [x] sources.md — lecture 14 row marked transcribed, the executable-lecture map
      extended, image count 360 -> 373 and 47 -> 49 MB.
- [x] AGENTS.md — image coverage table extended to lecture 14, the "lectures 14-18
      have no images" line corrected to 15-18, and a note that lecture 14's five
      hot-linked images must be given as URLs and never described.

### Publish
- [x] kb.json — coverage 14/18, 110 topic pages, 373 images / 49 MB across 14
      lectures, byLecture["14"]="source-text", executableLectures.transcribed 7->8,
      four new caveats and a new thirdPartyNotRedistributed key. Caveat 0 rewritten:
      it now says the DATA UNIT IS COMPLETE rather than half covered, and names
      15-18 as the gap.
- [x] Commit and push

### Found after publishing run 15 — a link class the sweep could not see
- [x] raw/transcripts/14-...md linked `original/14-...md` in its front matter and
      header. `raw/transcripts/original/` IS GITIGNORED, so that link 404s for the
      chat — exactly the silent failure the skill warns about. Rewritten to lecture
      12's established convention: a `verbatim_original: not committed` front-matter
      key plus the paragraph saying how to regenerate the captions with
      fetch_transcript.py. The copy-edit agent invented the link; the other 13
      transcripts do not have it.
      THE SWEEP COULD NOT HAVE CAUGHT IT. Every link check in this build tests
      `os.path.exists()` against the LOCAL filesystem, where gitignored files are
      present. The published repo is the git index, not the working tree.
      NEXT RUN: sweep against `git ls-files`, not the filesystem. Skip bare
      `#anchors` and directory links or it reports ~200 false positives. Current
      result with that fix: 0 tracked .md files link to a path outside the
      published repo.

## Run 16 — Lecture 15: Mid/Post-Training

Video 2oH6PWPrYFo (80 min). PDF deck (`lecture_15.pdf`, 65 pages) — page-images,
the seventh deck in this build and the first since lecture 11. Instructor
Tatsunori Hashimoto. **User instructions for this run: slides transcribed by
Sonnet (the skill default), and NO figure audit pass.**

### Course material
- [x] raw/slides/15-mid-post-training.md — transcribe all 65 pages (five Sonnet
      readers, 13 contiguous pages each, appending as they went; 24.2k words)
- [x] Verify heading sequence 1..65 — --verify passes, 65 headings, exact match.
      NO FIGURE AUDIT THIS RUN (user instruction). Recorded in the slide file's
      `provenance` block and in kb.json as figuresAudited false for lecture 15.
      Compensating check: four rendered images read back against their
      descriptions (slides 6, 26, 44 and the deck's structure) — slide 44's 36
      heatmap cells, slide 26's seven-series legend order and bar values, and
      slide 6's three-panel InstructGPT pipeline all matched. One hedge
      ("llama/alpaca-like figure") neutralized to what is actually legible.
      A "Known gaps" table was added listing all 17 things the readers marked
      illegible, plus three defects that are properties of the DECK itself:
      slide 38's screenshot is cut off mid-word in the source, slide 43's table
      title is cropped by its own screenshot border, and slide 45's heading is
      printed "RLFH" with the letters transposed — reproduced verbatim, not
      silently corrected.

### Transcript
- [x] 15 — verbatim captions fetched (104 paragraphs, ~15.9k words, 2420 segments)
- [x] 15 — copy-edited transcript (Sonnet draft, adjudicated here). 104
      paragraphs, 15,837 -> 13,613 words (86.0% retention), 18 questions from
      the floor marked.
- [x] 15 — verify: ALL THREE CHECKS PASS, run in the parent against the verbatim
      body (sliced from the first marker, so the header's own restoration list
      cannot produce phantom differences).
      Timestamps: 104 markers, sequence identical.
      Numbers: exactly two differences, both accounted for — "GPT-01" -> "o1"
      (a documented restoration) and "10" -> "ten" (spelling out, the same
      harmless class seen in earlier runs). Nothing lost.
      Ratios: ZERO paragraphs outside the 0.72-1.10 band, so no content moved
      across a marker boundary.
      PROPER-NOUN GREP: 15 of 15 restorations claimed as confirmed appear in the
      deck — 14 verbatim, and "Hugging Face" as the deck's one-word
      "HuggingFace". The three restored from context ALONE (o1, XSum, Bluebook)
      are genuinely absent from the deck, exactly as the agent labelled them,
      and are flagged as such in the transcript header for readers.
      Two [Ed: unclear] marks left rather than guessed: SimPO's dictated
      length-normalizer notation (1:14:49) and an unparseable student question
      (39:19).
      The transcript does NOT link into the gitignored raw/transcripts/original/
      — run 15's post-publish defect did not recur.

### Wiki
- [x] wiki/15-mid-post-training.md (~430 lines, 17 of the 38 figures embedded)
- [x] Topic pages — 15 NEW. The overlap check came back almost empty this time,
      which is itself the finding: RLHF, DPO and PPO appeared in NONE of the
      existing 124 pages except passing mentions in lectures 1 and 13 and the
      course map, so this lecture opens a genuinely new area rather than
      extending one. New: rlhf, reward-models, ppo, dpo, supervised-fine-tuning,
      instruction-tuning-datasets, midtraining, preference-data,
      human-annotation, model-based-annotation, reward-overoptimization,
      mode-collapse-and-calibration, style-and-length-bias,
      hallucination-and-knowledge-extraction, safety-tuning.
      Two pages carry a tension the lecture leaves implicit and neither page
      previously acknowledged: Schulman's argument that RL is what MAKES a model
      calibrated (hallucination-and-knowledge-extraction, ~23:55) against GPT-4's
      finding that RLHF DESTROYS calibration (mode-collapse-and-calibration,
      ~1:17:54). Both pages now state the tension and say it is unresolved.
- [x] Extend existing pages — post-training-data (a "Where lecture 15 takes this"
      section separating lecture 14's data-side treatment from lecture 15's
      procedural one), synthetic-data (the Zephyr experiment as the evidence
      behind its "almost all post-training data is synthetic" claim),
      chat-benchmarks (pointer to the training-side view of style bias),
      safety-evaluation (pointer to safety-tuning).
- [x] INDEX.md — banner now 1-15, a lecture-15 summary entry, a 16-entry annotated
      wiki section, the transcript list, and the PDF-deck list (now seven decks,
      with lecture 15 flagged as the unaudited one).
- [x] Link sweep — AGAINST `git ls-files`, as run 15 said to. 3,313 links, 0 real
      breaks; 411 images referenced, 0 untracked; all 140 wiki pages appear in
      INDEX; 0 LaTeX blocks inside code fences.
      THE SWEEP FOUND TWO PRE-EXISTING BROKEN ANCHORS, one of them real: run 15's
      lecture-14 transcript pointed at
      `slides/14-...md#there-is-no-optimal-threshold`, a heading that lives in
      wiki/quality-classifiers.md and has never existed in the slide file. Fixed
      to `#filtering`, the section that actually contains the N=100 WARCs chart.
      312 anchors now check clean.
- [x] Citation and quote check — 209 timestamp citations, all matching a real
      marker. 250 quotations checked, 19 CORRECTED.
      A NEW FAILURE CLASS THIS RUN, and it is mine rather than the captions':
      I WRITE BRITISH SPELLING AND THE LECTURER SPEAKS AMERICAN, so "behaviors"
      and "defense" had been silently anglicised INSIDE quotation marks in six
      places. The KB's prose convention is British (17 "behaviour" against 1
      "behavior" across the pre-existing pages), so the prose stays and only the
      quotations were corrected. Also fixed: five quotes that began mid-sentence
      with a substituted word ("is very trial-and-error" for "are very
      trial-and-error"; "is not representative" for "it's not representative"),
      two unmarked elisions, and ONE PARAPHRASE OF MINE IN QUOTATION MARKS
      ("SFT on good, negative SFT on bad"), converted to italics — the failure
      that recurred in runs 8, 9, 12, 13 and 14 and did not recur in 15.
      Run 15's structural fix WORKED: the wiki was drafted from the EDITED
      transcript this time, not the captions, and the caption-drift class that
      produced ~40 corrections last run produced ZERO this run.
      THREE CHECKER BUGS, bringing the build's total to seventeen. (1) The
      quote extractor paired quotation marks by regex rather than by position,
      so it read the prose BETWEEN two quotes as a quotation. (2) It compared
      against the transcript with `**[MM:SS]**` markers still in place — and
      those markers split sentences mid-flow, so every quote spanning a
      paragraph boundary read as a misquote. This one alone accounted for 26 of
      the 59 initial failures. (3) The anchor checker stripped underscores when
      slugging, which GitHub does not, so `#async_op-and-overlapping` was
      reported broken when it was correct.

### Images (Step 1c)
- [x] Render figure slides into raw/images/15-mid-post-training/ — 38 images,
      4.4 MB. THE TWO SIGNALS AGREED EXACTLY: the raster test flagged 46 pages
      and the slide file's prose describes a figure on exactly those same 46,
      with zero disagreement in either direction (the skill warns the raster
      test over-reports on decks that paste equations as images; this deck does
      that only on the eight pages below). Of the 46, eight were skipped per the
      skip list: slide 1 (title) and slides 51, 52, 53, 56, 57, 58 and 60, whose
      content is pure equations and paper-text screenshots that the slide file
      already reproduces exactly in LaTeX — a transcribed equation is better
      than a picture of one.
- [x] Embed into raw/slides/ — 38 images placed by script under their headings;
      --verify still passes and all 38 files resolve.
- [x] Embed into wiki/ — 17 of the 38 in the lecture page, the rest distributed
      across the topic pages by the passage each belongs to. Placed as each page
      was composed rather than inserted afterwards, which sidesteps the
      list-splitting and caption-nesting failures.

### Publish
- [x] Coverage in SIX places, all verified consistent by grep: INDEX.md,
      sources.md, AGENTS.md, kb.json, wiki/course-map.md, and the AGENTS.md
      image-coverage table (extended to lecture 15, and the "lectures 15-18 have
      no images" line corrected to 16-18).
- [x] kb.json — coverage 15/18, 125 topic pages, 411 images / 52.6 MB across 15
      lectures, byLecture["15"]="page-images", slideDecks.transcribed 6->7.
      figuresAudited CHANGED FROM A BOOLEAN TO A PER-LECTURE MAP, because a bare
      `true` would now be a lie: a new `figuresAuditedByLecture` records that
      decks 3, 4, 5, 8, 9 and 11 were audited and 15 was not. Two new caveats
      state the missing audit and the three defects in the deck itself.
- [x] Commit and push

### Lessons for run 17
- WAIT FOR THE EDITED TRANSCRIPT BEFORE QUOTING. Run 15 identified this and it
  worked: zero caption-drift corrections this run, against ~40 last run.
- CHECK YOUR OWN SPELLING CONVENTION AGAINST THE SPEAKER'S before quoting. The
  KB's prose is British; both lecturers are American. Quotations must follow the
  speaker, prose follows the KB.
- The quote checker is now correct in this run's script. Reuse it rather than
  rewriting it: pair quotes by position, strip `**[MM:SS]**` markers from the
  source, tolerate trailing punctuation, and preserve underscores when slugging
  anchors.
- If another deck is transcribed without an audit, repeat this run's
  compensating checks: verify the heading sequence, read back four rendered
  images against their descriptions, and require a Known gaps table.

## Run 17 — Lecture 16: Post-Training — RLVR

Video dIFAi87Ws4E (76 min). Tatsunori Hashimoto. PDF deck `lecture_16.pdf`,
61 pages, so `page-images`. User instructions for this run: transcribe the deck
with Sonnet, **run no figure audit**, and keep at most two subagents alive at a
time.

Numbering: the deck prints **no page number on any page** — the eighth CS336
deck in a row to do so (3, 4, 5, 8, 9, 11, 15 are the others). `slide_number_map.py`
scanned both bottom corners of all 61 pages, for a bare number and for one
ending a running footer, and found nothing. Mapping is a plain 1..61, page N =
slide N, settled before any page was read and handed to the readers as a
conclusion. `--verify` therefore degenerates to a heading-sequence check.

### Course material
- [x] raw/slides/16-post-training-rlvr.md — 61 pages, four Sonnet readers in two
      waves of two (1-15, 16-31, 32-46, 47-61). 1,648 lines, 17,400 words.
      All four reported no printed folio anywhere, confirming the 1..61 map.
- [x] Heading-sequence check — PASS, exactly 61 headings 1..61 in order. Also
      checked: no LaTeX trapped inside a code fence (the fences hold the two
      Python screenshots on slides 19-20 and the PPO implementation, correctly).
- n/a Figure audit — SKIPPED at the user's instruction, as in run 16 (lecture 15).
      Compensating checks instead: heading sequence, and rendered images read
      back against their descriptions.

### Transcript
- [x] 16 — verbatim captions fetched (99 paragraphs, ~14.8k words)
- [ ] 16 — copy-edited transcript (Sonnet draft, adjudicated here)
- [ ] 16 — verify: timestamps, number inventory, per-paragraph word ratios,
      proper-noun grep against the deck

### Wiki
- [ ] wiki/16-post-training-rlvr.md
- [ ] Topic pages — new and extended
- [ ] INDEX.md
- [ ] Link sweep against `git ls-files`
- [ ] Citation and quote check (reuse run 16's corrected checker)

### Images (Step 1c)
- [ ] Render figure slides into raw/images/16-post-training-rlvr/
      (raster test: 55 of 61 pages carry a raster >4%; the 6 that do not are
      17, 35, 51, 56, 57, 61)
- [ ] Embed into raw/slides/ and wiki/

### Publish
- [ ] Coverage in all six places: INDEX.md, sources.md, AGENTS.md, kb.json,
      wiki/course-map.md, AGENTS.md image-coverage table
- [ ] kb.json — coverage 16/18, figuresAuditedByLecture["16"]=false
- [ ] Commit and push
