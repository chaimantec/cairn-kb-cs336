# CS336 — Language Modeling from Scratch (Stanford, Spring 2026)

Stanford's CS336 teaches you to build a language model from the ground up:
tokenizer, Transformer, optimizer, training loop, GPU kernels, parallelism,
inference, scaling laws, data pipelines and alignment. It is taught by **Percy
Liang** and **Tatsunori Hashimoto**, and this is its third offering. The
organizing question, stated in the first lecture and returned to in every unit, is
**efficiency**: what is the best model you can build from a fixed budget of
compute and data?

> ## ⚠️ This knowledge base covers Lectures 1, 2, 3, 4 and 5 of 18
>
> **Lecture 1 (Overview and Tokenization), Lecture 2 (PyTorch and Resource
> Accounting), Lecture 3 (Architectures), Lecture 4 (Attention Alternatives and
> Mixtures of Experts) and Lecture 5 (GPUs and TPUs) are covered in depth.**
> Nothing else is. There are no transcripts and no wiki pages for kernels and
> Triton, parallelism, scaling laws, inference, evaluation, data, mid/post-training,
> RLVR or multimodality.
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

- **[Lecture 3 — Architectures](wiki/03-architectures.md)** — what the large
  language models have in common and what they vary. Pre- vs post-norm, RMSNorm,
  gated activations, RoPE, the hyperparameter ratios everyone converges on,
  stability tricks, and GQA/sliding-window attention. Taught from a survey of
  forty-odd models rather than from theory.

- **[Lecture 4 — Attention Alternatives and Mixtures of Experts](wiki/04-attention-alternatives.md)** —
  where modern models depart from the standard transformer *structurally*, in two
  places. Replacing quadratic attention with something linear (Mamba-2, Gated
  DeltaNet) or sparse (DeepSeek Sparse Attention), and replacing the dense
  feedforward block with a sparsely routed mixture of experts. Both are cost
  arguments, not expressiveness arguments.

- **[Lecture 5 — GPUs and TPUs](wiki/05-gpus-tpus.md)** — the first lecture of the
  systems unit, and the one that explains what the hardware is actually doing. What
  a GPU is made of and how its memory hierarchy works, the six tricks for making a
  workload fast on one (control divergence, low precision, fusion, recomputation,
  coalescing, tiling), and FlashAttention assembled out of those parts. Built around
  one benchmark plot whose strange shape it fully explains by the end.

If you are looking for a single number or formula, the topic pages below are
usually the faster route than the lecture pages.

## Wiki

### Lecture 5 — GPUs, TPUs and making them fast

- **[GPU architecture](wiki/gpu-architecture.md)** — what a GPU physically is.
  Latency-versus-throughput design, the streaming multiprocessor, the memory
  hierarchy with the A100 latency table (shared memory ~20 cycles, global memory
  290), why the whole chip is not fast memory (cost, physics, energy), and the
  widening gap between compute throughput and memory bandwidth that motivates the
  rest of the lecture. Start here for "what is an SM?" or "why is global memory
  slow?"
- **[The GPU execution model](wiki/gpu-execution-model.md)** — threads, blocks and
  32-thread warps; SIMT and what it costs. Includes control divergence: why an `if`
  makes every thread execute both branches, and why GPU code multiplies by masks
  instead of branching. Also the per-scope memory table (registers, local, shared,
  global, constant, host).
- **[TPUs](wiki/tpus.md)** — the alternative evolution, and what the comparison
  teaches. Near-identical memory hierarchy and the same systolic-array matmul
  circuit; the difference is granularity (2 processors and 8 matmul units against an
  H100's ~132 and 528) and, above the chip, networking. Includes the tensor-core
  naming collision that catches everyone.
- **[Tensor cores](wiki/tensor-cores.md)** — since the V100 there is dedicated
  matmul hardware, and it runs **more than 10× faster** than any other floating-point
  operation. This is why every architecture that scales has a matrix multiply at its
  centre. Read this for why hardware constrains architecture design.
- **[Microscaling formats](wiki/microscaling-formats.md)** — MXFP8 and MXFP4. One
  E8M0 scale factor per 32 elements instead of one per tensor, why that makes a
  transpose expensive enough to keep two quantized copies of every matrix, and the
  realistic payoff (20–30% on matmuls, not 2×). Note: the deck contradicts itself on
  MXFP4's scale format and the page says so.
- **[Operator fusion](wiki/operator-fusion.md)** — the factory-and-conveyor-belt
  argument. Why five chained pointwise ops cost five round trips to global memory
  and one fused kernel costs one, what `torch.compile` and JAX do for you
  automatically, and where fusion stops being automatic.
- **[Memory coalescing](wiki/memory-coalescing.md)** — DRAM returns a whole
  ~128-byte burst section per read. A warp whose 32 addresses land inside one gets
  its data free; one whose addresses scatter pays 32 bursts and wastes most of each.
  Explains why row-major layout makes one loop order fast and the other slow.
- **[Tiling](wiki/tiling.md)** — the most important single technique. Cut matrices
  into tiles, load each into shared memory once, reuse it: global-memory reads drop
  from $N$ per input to $N/T$. Also tile-size selection, `max-autotune`, burst
  alignment, and why padding nanoGPT's vocabulary from 50257 to 50304 gave a 25%
  speedup.
- **[Wave quantization](wiki/wave-quantization.md)** — why a matmul gets
  dramatically slower going from $N=1792$ to $N=1793$. The tile count crosses from
  98 to 120 against an A100's 108 SMs, so a second wave runs with most of the GPU
  idle. The clearest evidence in the course that hardware details show up in
  benchmarks.
- **[FlashAttention](wiki/flash-attention.md)** — the lecture's finale. Standard
  attention, computed exactly, made fast purely by moving less data. The online
  softmax that makes a global operation tileable, the two-tile worked trace, and
  recomputation in the backward pass so the $N \times N$ score matrix is never
  materialized.

### Lecture 4 — attention alternatives and mixture of experts

- **[Mixture of experts](wiki/mixture-of-experts.md)** — the hub for the second half
  of Lecture 4. What an MoE is (replace one FFN with $N$ and route each token to a
  few), the distinction between total and *active* parameters that every MoE claim
  turns on, the evidence that they win, why the field took until 2024 to adopt them,
  and why "expert" is a misnomer — the routers are a single matrix multiply and there
  is no semantic specialization. Read this first for "what is an MoE and why?"
- **[MoE routing](wiki/moe-routing.md)** — how tokens get assigned to experts. Token
  choice versus expert choice, the four routing algorithms (top-$k$, hashing, RL,
  linear assignment) and why top-$k$ won on cost rather than elegance, the
  DeepSeekMoE router as equations, and fine-grained and shared experts — on which
  DeepSeek's and OLMoE's careful ablations *disagree*. Carries the table of what
  twelve real MoEs use. Read this for "how does top-$k$ routing work?"
- **[Load balancing losses](wiki/load-balancing-losses.md)** — the heuristic that
  makes MoEs trainable at all. Expert collapse as a rich-get-richer dynamic, the
  Switch Transformer auxiliary loss and the trick of reading its *gradient* rather
  than its objective, DeepSeek's per-device variant and why it needs its own loss,
  V3's aux-loss-free biases (which are not quite aux-loss-free), and the ablation
  showing two experts taking nearly all tokens without it. Read this for "what is a
  load balancing loss?"
- **[Expert parallelism](wiki/expert-parallelism.md)** — the systems half. Why data
  and model parallelism each saturate and experts give a third axis, the
  structured-sparsity hardware fit, communication as the price and Nemotron 3's
  down-projection trick for lowering it, and the historical bug where an overloaded
  expert would silently drop your token because of *another user's* queue. Read this
  for "how are MoEs sharded?"
- **[Linear attention](wiki/linear-attention.md)** — the associativity trick that
  makes sub-quadratic attention possible: reorder $(QK^\top)V$ to $Q(K^\top V)$ and
  the $n^2$ term disappears. The recurrent form, the *duality* between parallel
  training and fixed-state inference that makes the family practical, and a clear
  statement of which step is lossy — dropping the softmax, not the recurrence. Read
  this for "what is linear attention?"
- **[State space models](wiki/state-space-models.md)** — Mamba-2 and Gated DeltaNet
  as elaborations of linear attention's state update, with the rule that governs
  them: gates may depend on the input, never on the state, or duality breaks. The
  erase term that makes DeltaNet overwrite rather than accumulate, why every
  deployed model is a hybrid with periodic full attention, and the state-size
  bottleneck that remains. Read this for "how does Mamba work?"
- **[Sparse attention](wiki/sparse-attention.md)** — DeepSeek Sparse Attention: a
  cheap indexer scores every token, top-$k$ survive, full attention runs on those.
  Includes the correction the lecture makes explicitly — **this is not linear time**,
  it is quadratic with much better constants — and why it can be bolted on after
  pretraining. Read this for "what is DSA?"
- **[Multi-head latent attention](wiki/multi-head-latent-attention.md)** — DeepSeek's
  KV-cache reduction: cache one low-dimensional latent per token and reconstruct keys
  and values from it, with the up-projection merged into the query projection so the
  keys are never materialized. Also why RoPE breaks that merge and the split-dimension
  fix. Read this for "what is MLA?"
- **[Multi-token prediction](wiki/multi-token-prediction.md)** — predicting several
  tokens ahead with lightweight chained modules, and the systems argument that is the
  real payoff: the model becomes its own speculative decoder. Read this for "what is
  MTP?"
- **[Upcycling](wiki/upcycling.md)** — building an MoE by cloning a trained dense
  model's MLP into experts, why the randomly initialized router is what breaks the
  symmetry between identical copies, and why the technique fell out of use when MoE
  became the default rather than because it stopped working.

### Lecture 3 — architectures

- **[The model architecture survey](wiki/model-architecture-survey.md)** — the hub
  for Lecture 3. The database of ~44 dense models the whole lecture is read off,
  which of its five deck views carries which columns, what is settled (RMSNorm,
  non-residual norms, serial blocks, gated activations) and what is still moving
  (position embeddings, stability tricks). Read this first for "what does model X
  actually use?" or "is this choice consensus or contested?"
- **[Pre-norm and post-norm](wiki/pre-norm-and-post-norm.md)** — where the
  normalization layer goes, and the one thing everyone agrees the 2017 paper got
  wrong. Both arrangements as equations, the gradient-attenuation and
  gradient-spike evidence, why "keep your residual stream clean" is the operative
  heuristic, and the modern double-norm variant that leaves four models' pre-norm
  boxes unticked. Read this for "why is the layer norm at the front?"
- **[RMSNorm and dropping bias terms](wiki/rmsnorm.md)** — why models dropped a
  strictly more expressive normalizer. Normalization is 0.17% of a transformer's
  FLOPs and 25.5% of its runtime; the whole argument is data movement, and it is
  the clearest link from Lecture 3 back to Lecture 2's arithmetic intensity. Read
  this for "why RMSNorm?" or "why no bias terms?"
- **[Gated activations](wiki/gated-activations.md)** — GLU, ReGLU, GeGLU and
  SwiGLU, with the naming rule that makes them easy. The gating equation, the two
  parameter-matched studies that agree gating helps, the Google/LLaMA split, and
  the 2/3 rule that explains why feedforward multipliers cluster at 2.67. Read
  this for "what is SwiGLU and why?"
- **[RoPE](wiki/rope.md)** — rotary position embeddings, derived rather than
  asserted. The relative-position constraint no earlier scheme satisfies, why
  rotation satisfies it, the worked "we know that" / "of course we know" example,
  the block-diagonal rotation matrix, why multiplying avoids the cross terms that
  adding creates, and the p-RoPE and NoPE variants. Read this for "how does RoPE
  work?"
- **[Transformer hyperparameters](wiki/transformer-hyperparameters.md)** — the
  numbers you must pick: feedforward ratio, head dimension, aspect ratio, vocab
  size, dropout and weight decay. Every rule of thumb with the table of what
  models do and the sweep showing how forgiving it is, plus the finding that weight
  decay in pretraining is not a regularizer at all. Read this for "what should I
  set $d_{ff}$ to?"
- **[Training stability](wiki/training-stability.md)** — why runs blow up and the
  three fixes. Reading a loss curve against a gradient-norm curve, the z-loss for
  the output softmax, QK norm for the attention softmax, and logit soft-capping —
  which is the only one of the three that measurably costs quality. Read this for
  "what is z-loss?" or "why QK norm?"
- **[Attention variants](wiki/attention-variants.md)** — MQA, GQA, sliding-window
  and interleaved attention, all motivated by inference cost rather than quality.
  The arithmetic-intensity accounting for prefill against incremental decoding,
  why the KV cache creates the problem GQA solves, and the local/global
  interleaving that current long-context models use. Read this for "why do models
  use GQA?"

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
  [Lecture 2](raw/transcripts/02-pytorch-resource-accounting.md),
  [Lecture 3](raw/transcripts/03-architectures.md),
  [Lecture 4](raw/transcripts/04-attention-alternatives.md),
  [Lecture 5](raw/transcripts/05-gpus-tpus.md).
  Copy-edited from the auto-captions: repunctuated, filler removed, mis-heard
  technical terms restored against the lecture material. Every `[MM:SS]` marker is
  preserved in its original position, so timestamps quoted from them are citable.
  Each header lists every restoration made and every place left marked unclear.
- **[`raw/transcripts/original/`](raw/transcripts/original/)** — the verbatim
  auto-captions, kept as the record of what was actually said, alongside the raw
  caption segment JSON they were generated from. Consult these to check the edited
  version, not to read the lecture.
- **[`raw/slides/`](raw/slides/)** — the written course material. **This is the
  authority for anything the lecturer wrote down**; the transcript is the authority
  for what was said. CS336 supplies it in two very different forms:
  - [`lecture_01.py`](raw/slides/01-overview-tokenization.md) and
    [`lecture_02.py`](raw/slides/02-pytorch-resource-accounting.md) are Percy
    Liang's *executable lectures* — Python programs, transcribed from source text,
    each with a section-to-source-line table and the code verbatim. There are no
    slide numbers to cite.
  - [`lecture_03.pdf`](raw/slides/03-architectures.md) (67 pages),
    [`lecture_04.pdf`](raw/slides/04-attention-alternatives.md) (60 pages) and
    [`lecture_05.pdf`](raw/slides/05-gpus-tpus.md) (55 pages) are Tatsunori
    Hashimoto's slide decks, transcribed from the rendered page images, with every
    figure described in prose and every table transcribed cell by cell. Slide
    numbers in all three are **PDF page numbers**, because none of the decks prints
    any of its own. Lectures 4 and 5 are the figure-dependent ones — 102 images
    across 60 pages and 83 across 55, most pages carrying only 30–40 words of their
    own text — so the figure descriptions there are not a supplement to the content,
    they *are* the content. Each deck's front matter records which pages were
    audited against the PDF and what the audit found, including the places where a
    deck contradicts itself.
- **[`sources.md`](sources.md)** — every lecture, deck, assignment and linked
  document with its canonical URL, including the material for the 13 lectures this
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

- **A timestamp** — `(≈1:13:13)` — cites what the lecturer *said*, and resolves
  against the transcript.
- **A section of `lecture_01.py` or `lecture_02.py`**, or **a slide number of
  `lecture_03.pdf` or `lecture_04.pdf`**, cites what the lecture *wrote*, and
  resolves against
  `raw/slides/`. Prefer this for code, equations, numbers, tables and paper
  citations.

For Lectures 3 and 4, "slide N" means PDF page N of `lecture_03.pdf` or
`lecture_04.pdf`. Neither deck prints page numbers, so there is no printed numbering
to disagree with the page count — but say "page" rather than "printed slide" if the
distinction could matter to a reader opening the file.

There are **no slide numbers** in this course's Percy-taught lectures. If a source
appears to cite one, it is wrong — see
[executable lectures](wiki/executable-lectures.md).

One further distinction, specific to Lecture 2: a number this KB marks
**"(computed)"** was recomputed from the lecture's own arithmetic and is exact,
while a value marked **"machine-dependent, not reproduced"** — wall-clock timings,
measured FLOP/s, MFU, peak-memory readings — is a fact about the GPU the lecture
ran on, and no number is given for it here. Do not supply one.
