# CS336 — Language Modeling from Scratch (Stanford, Spring 2026)

Stanford's CS336 teaches you to build a language model from the ground up:
tokenizer, Transformer, optimizer, training loop, GPU kernels, parallelism,
inference, scaling laws, data pipelines and alignment. It is taught by **Percy
Liang** and **Tatsunori Hashimoto**, and this is its third offering. The
organizing question, stated in the first lecture and returned to in every unit, is
**efficiency**: what is the best model you can build from a fixed budget of
compute and data?

> ## ⚠️ This knowledge base covers Lectures 1 and 2 of 18
>
> **Lecture 1 (Overview and Tokenization) and Lecture 2 (PyTorch and Resource
> Accounting) are covered in depth.** Nothing else is. There are no transcripts
> and no wiki pages for architectures, GPU hardware, kernels, parallelism, scaling
> laws, inference, evaluation, data, mid/post-training, RLVR or multimodality.
>
> Where a page describes later material, it is repeating Lecture 1's *syllabus
> preview* and says so at the top. Do not cite this knowledge base as covering
> CS336 as a whole. Machine-readable coverage is in [`kb.json`](kb.json).

## Start here

- **[Lecture 1 — Overview and Tokenization](wiki/01-overview-tokenization.md)** —
  why the course exists, why small models are not simply small frontier models,
  the bitter lesson restated as accuracy = efficiency × resources, a history of
  language models, the five-unit syllabus, and then the tokenization unit in full.
- **[Lecture 2 — PyTorch and Resource Accounting](wiki/02-pytorch-resource-accounting.md)** —
  how to work out what a computation costs before running it. Tensors and
  floating-point formats, einops, counting FLOPs, MFU, arithmetic intensity and
  the roofline, the $C = 6ND$ training rule, memory per parameter, and the two
  techniques that trade compute for memory.

If you are looking for a single number or formula, the topic pages below are
usually the faster route than the lecture pages.

## Wiki

### Lecture 2 — resource accounting

- **[Resource accounting](wiki/resource-accounting.md)** — the hub for Lecture 2
  and the mindset it teaches: napkin math before profilers. Works both of the
  lecture's motivating questions end to end — *144 days to train a 70B model on
  1024 H100s*, *53B parameters on 8 H100s with AdamW* — and says what each
  ingredient is and which page covers it. Read this first for "how do I estimate
  what this will cost?"
- **[FLOPs, FLOP/s and MFU](wiki/flops-and-mfu.md)** — the units, and why they are
  confusable. Why you always halve the H100's 1979 teraFLOP/s datasheet figure,
  the $2BDK$ matmul count, peak throughput per GPU and dtype (A100/H100/B200), and
  what model FLOPs utilization measures. Read this for "how many FLOPs is this?"
  or "is 0.5 MFU good?" (yes).
- **[Arithmetic intensity and roofline](wiki/arithmetic-intensity.md)** — the
  central idea of the lecture. An H100 needs ~295 FLOPs of work per byte moved
  before arithmetic becomes the limit; five operations worked with real numbers
  show ReLU at 0.25, GeLU at 5, matrix–vector at ~1, and only large matmul at 341.
  Explains why ReLU is not faster than GeLU, why inference is memory-bound, and
  why MFU is not 1. Read this for "am I compute-bound or memory-bound?"
- **[Training FLOPs and the 6ND rule](wiki/training-flops.md)** — $C = 6ND$
  derived rather than asserted: 2ND forward, 4ND backward, and why the backward
  pass is exactly twice the forward one. Also what the rule assumes and where it
  breaks (long context). Read this for "how much compute does this training run
  need?"
- **[Memory accounting for training](wiki/memory-accounting-for-training.md)** —
  the four consumers of GPU memory and their cost in bytes per parameter, the
  2 + 2 + (4 + 4) = 12 breakdown behind the "largest model that fits" question,
  the optimizer family as a sequence of additions (momentum → AdaGrad → RMSProp →
  Adam) and why Adam costs 8 bytes where AdaGrad costs 4. Read this for "will this
  fit?"
- **[Precision and floating-point data types](wiki/precision-and-data-types.md)** —
  fp32, fp16, bf16, fp8 (E4M3/E5M2) and nvfp4, organized around the one axis that
  matters: dynamic range beats resolution in deep learning. Why fp16 underflows at
  1e-8 and bf16 does not, and the mixed-precision recipe — bf16 for parameters,
  activations and gradients, fp32 for optimizer state — that every byte count in
  this KB assumes. Read this for "which dtype, and why?"
- **[einops](wiki/einops.md)** — named tensor dimensions: `einsum`, `reduce` and
  `rearrange`, the "unnamed dimensions are summed over" rule, and packing/unpacking
  a head dimension with parentheses. CS336 uses einops for the rest of the course,
  so read this before any later lecture's code.
- **[Activation checkpointing and gradient accumulation](wiki/activation-checkpointing.md)** —
  the two ways to trade compute for memory. Why training must store every layer's
  activations and inference need not, why gradient accumulation gives a
  *bit-identical* update at a quarter of the activation memory, and the
  $O(L)$ / $O(1)$ / $O(\sqrt{L})$ checkpointing tradeoff. Read this for "I am out
  of memory."

### Lecture 1 — tokenization

- **[Tokenization](wiki/tokenization.md)** — what a tokenizer is and why the two
  numbers that matter are compression ratio and vocabulary size. Walks the three
  approaches that fail — character, byte and word level — with the real measured
  values for each, then what a production tokenizer (`o200k_base`, 200,019
  entries) actually does. Read this for "why not just feed the model bytes?"
- **[Byte-pair encoding](wiki/byte-pair-encoding.md)** — the algorithm CS336
  teaches and Assignment 1 asks you to build. Training loop and encode/decode in
  code, a fully worked three-merge example on `"the cat in the hat"` with the
  merge table and resulting compression ratios, why tie-breaking matters, and the
  four things Assignment 1 adds to the toy version. Read this for "how does BPE
  actually work?"

### Across the course

- **[Efficiency](wiki/efficiency.md)** — the course's organizing principle.
  accuracy = efficiency × resources, why efficiency matters *more* at scale, the
  compute-constrained assumption and where it stops holding, a table mapping each
  of the five units to the resource it is really about, and why being able to
  *count* a resource is the precondition for optimizing it. Read this for "why is
  the course arranged this way?"
- **[Executable lectures](wiki/executable-lectures.md)** — CS336's Percy-taught
  lectures are Python programs, not slide decks. How the format works, why there
  are no slide numbers to cite, and the difference between a value this KB
  recomputed and one that is a measurement of the lecturer's own GPU. Read this
  before citing any CS336 lecture material.

### Syllabus previews — Lecture 1's framing only

- **[Course map](wiki/course-map.md)** — all five units and five assignments as
  Percy presents them: basics, systems, scaling laws, data, alignment. The most
  useful page for "where in CS336 is X taught?" Substantive on each unit's
  vocabulary and concerns, but it is the preview, not the treatment — except the
  resource-accounting part of Unit 2, which Lecture 2 delivers and this KB covers.
- **[Scaling laws](wiki/scaling-laws.md)** — scaling recipes rather than single
  models, hyperparameter transfer, predictability over optimality, ISOFLOP curves
  and the $D \approx 20N$ Chinchilla rule, and why inference cost has pushed
  practice away from it. Lecture 1's preview; CS336 teaches this properly in
  Lectures 9 and 11, which are not in this KB.

## Raw material

- **[`raw/transcripts/`](raw/transcripts/)** — **the transcripts to read**, one per
  covered lecture:
  [Lecture 1](raw/transcripts/01-overview-tokenization.md),
  [Lecture 2](raw/transcripts/02-pytorch-resource-accounting.md).
  Copy-edited from the auto-captions: repunctuated, filler removed, mis-heard
  technical terms restored against the lecture source. Every `[MM:SS]` marker is
  preserved in its original position, so timestamps quoted from them are citable.
  Each header lists every restoration made.
- **[`raw/transcripts/original/`](raw/transcripts/original/)** — the verbatim
  auto-captions, kept as the record of what was actually said, alongside the raw
  caption segment JSON they were generated from. Consult these to check the edited
  version, not to read the lecture.
- **[`raw/slides/`](raw/slides/)** — the written course material, transcribed from
  the lectures' own source programs:
  [`lecture_01.py`](raw/slides/01-overview-tokenization.md),
  [`lecture_02.py`](raw/slides/02-pytorch-resource-accounting.md). Each has a
  section-to-source-line table, every citation resolved to a URL, and the code
  verbatim. **This is the authority for anything Percy wrote down**; the transcript
  is the authority for what he said.
- **[`sources.md`](sources.md)** — every lecture, deck, assignment and linked
  document with its canonical URL, including the material for the 16 lectures this
  KB does not yet cover. Explains how CS336 splits between executable lectures and
  PDF decks.

`raw/pdfs/` is empty by design — no binaries are committed. The course's decks are
5–7 MB each and live at the URLs in `sources.md`.

## Also

- **[`SEE_ALSO.md`](SEE_ALSO.md)** — sibling knowledge bases worth reading:
  **CS224N** (complete, all 23 lectures) for the attention and Transformer
  derivations CS336 assumes, and **CS221** (lectures 1–4) for gradient descent,
  backpropagation, tensors and einops.
- **[`kb.json`](kb.json)** — machine-readable coverage and provenance. Read this to
  know how far to trust a citation from here.
- **[`AGENTS.md`](AGENTS.md)** — how this wiki is organized, for future maintainers.
- **[`TODO.md`](TODO.md)** — build tracker. Unchecked boxes are outstanding work.

## Citing this material

Two kinds of citation, and they are not interchangeable:

- **A timestamp** — `(≈1:13:13)` — cites what Percy *said*, and resolves against
  the transcript.
- **A section of `lecture_01.py` or `lecture_02.py`** cites what the lecture
  *wrote*, and resolves against `raw/slides/`. Prefer this for code, numbers and
  paper citations.

There are **no slide numbers** in this course's Percy-taught lectures. If a source
appears to cite one, it is wrong — see
[executable lectures](wiki/executable-lectures.md).

One further distinction, specific to Lecture 2: a number this KB marks
**"(computed)"** was recomputed from the lecture's own arithmetic and is exact,
while a value marked **"machine-dependent, not reproduced"** — wall-clock timings,
measured FLOP/s, MFU, peak-memory readings — is a fact about the GPU the lecture
ran on, and no number is given for it here. Do not supply one.
