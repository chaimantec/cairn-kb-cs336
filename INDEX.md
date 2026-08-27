# CS336 — Language Modeling from Scratch (Stanford, Spring 2026)

Stanford's CS336 teaches you to build a language model from the ground up:
tokenizer, Transformer, optimizer, training loop, GPU kernels, parallelism,
inference, scaling laws, data pipelines and alignment. It is taught by **Percy
Liang** and **Tatsunori Hashimoto**, and this is its third offering. The
organizing question, stated in the first lecture and returned to in every unit, is
**efficiency**: what is the best model you can build from a fixed budget of
compute and data?

> ## ⚠️ This knowledge base covers Lecture 1 of 18
>
> **Lecture 1 — Overview and Tokenization — is covered in depth.** Nothing else
> is. There are no transcripts and no wiki pages for architectures, GPUs, kernels,
> parallelism, scaling laws, inference, evaluation, data, mid/post-training, RLVR
> or multimodality.
>
> Where a page describes later material, it is repeating Lecture 1's *syllabus
> preview* and says so at the top. Do not cite this knowledge base as covering
> CS336 as a whole. Machine-readable coverage is in [`kb.json`](kb.json).

## Start here

- **[Lecture 1 — Overview and Tokenization](wiki/01-overview-tokenization.md)** —
  the main page. Why the course exists, why small models are not simply small
  frontier models, the bitter lesson restated as accuracy = efficiency ×
  resources, a history of language models, the five-unit syllabus, and then the
  tokenization unit in full.

## Wiki

### Covered in depth

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
- **[Efficiency](wiki/efficiency.md)** — the course's organizing principle.
  accuracy = efficiency × resources, why efficiency matters *more* at scale, the
  compute-constrained assumption and where it stops holding, and a table mapping
  each of the five units to the resource it is really about. Read this for "why is
  the course arranged this way?"
- **[Executable lectures](wiki/executable-lectures.md)** — CS336's Percy-taught
  lectures are Python programs, not slide decks. How the format works, why there
  are no slide numbers to cite, and why the worked numbers in this KB had to be
  obtained by running the code. Read this before citing any CS336 lecture material.

### Syllabus previews — Lecture 1's framing only

- **[Course map](wiki/course-map.md)** — all five units and five assignments as
  Percy presents them: basics, systems, scaling laws, data, alignment. The most
  useful page for "where in CS336 is X taught?" Substantive on each unit's
  vocabulary and concerns, but it is the preview, not the treatment.
- **[Scaling laws](wiki/scaling-laws.md)** — scaling recipes rather than single
  models, hyperparameter transfer, predictability over optimality, ISOFLOP curves
  and the $D \approx 20N$ Chinchilla rule, and why inference cost has pushed
  practice away from it. Lecture 1's preview; CS336 teaches this properly in
  Lectures 9 and 11, which are not in this KB.

## Raw material

- **[`raw/transcripts/01-overview-tokenization.md`](raw/transcripts/01-overview-tokenization.md)** —
  **the transcript to read.** Copy-edited from the auto-captions: repunctuated,
  filler removed, mis-heard technical terms restored against the lecture source.
  Every `[MM:SS]` marker is preserved in its original position, so timestamps
  quoted from it are citable. The header lists every restoration made.
- **[`raw/transcripts/original/`](raw/transcripts/original/)** — the verbatim
  auto-captions, kept as the record of what was actually said, alongside the raw
  caption segment JSON they were generated from. Consult these to check the edited
  version, not to read the lecture.
- **[`raw/slides/01-overview-tokenization.md`](raw/slides/01-overview-tokenization.md)** —
  the written course material: a full transcription of `lecture_01.py`, section by
  section, with a section-to-source-line table, every citation resolved to a URL,
  the tokenizer code verbatim, and the worked values produced by running it.
  **This is the authority for anything Percy wrote down**; the transcript is the
  authority for what he said.
- **[`sources.md`](sources.md)** — every lecture, deck, assignment and linked
  document with its canonical URL, including the material for the 17 lectures this
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
- **A section of `lecture_01.py`** cites what the lecture *wrote*, and resolves
  against `raw/slides/`. Prefer this for code, numbers and paper citations.

There are **no slide numbers** in this course's Percy-taught lectures. If a source
appears to cite one, it is wrong — see
[executable lectures](wiki/executable-lectures.md).
