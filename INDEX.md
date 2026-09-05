# CS336 — Language Modeling from Scratch (Stanford, Spring 2026)

Stanford's CS336 teaches you to build a language model from the ground up:
tokenizer, Transformer, optimizer, training loop, GPU kernels, parallelism,
inference, scaling laws, data pipelines and alignment. It is taught by **Percy
Liang** and **Tatsunori Hashimoto**, and this is its third offering. The
organizing question, stated in the first lecture and returned to in every unit, is
**efficiency**: what is the best model you can build from a fixed budget of
compute and data?

> ## ⚠️ This knowledge base covers Lectures 1–11 of 18
>
> **Lecture 1 (Overview and Tokenization), Lecture 2 (PyTorch and Resource
> Accounting), Lecture 3 (Architectures), Lecture 4 (Attention Alternatives and
> Mixtures of Experts), Lecture 5 (GPUs and TPUs), Lecture 6 (Kernels and
> Triton), Lecture 7 (Parallelism), Lecture 8 (Parallelism, Part 2), Lecture 9
> (Scaling Laws — Basics), Lecture 10 (Inference) and Lecture 11 (Scaling Laws in
> the Wild) are covered in depth.**
> Nothing else is. There are no transcripts and no wiki pages for evaluation,
> data, mid/post-training, RLVR or multimodality.
>
> **A note specific to scaling laws:** CS336 splits them across *two* lectures,
> with inference in between, and **both are now covered in full**. Lecture 9 is the
> basics: data scaling laws, scaling laws for model engineering, and the whole
> Kaplan-versus-Chinchilla story. [Lecture 11](wiki/11-scaling-laws-in-the-wild.md)
> is the advanced treatment — the published scaling recipes of MiniCPM, DeepSeek,
> Qwen, Kimi K2, Hunyuan, LLaMA 3 and MiniMax-01 read off their own figures; WSD
> learning-rate schedules; the StepFun grid search; optimizers and Muon; and muP
> derived from its two conditions, with the three things that break it. Read them
> in order: 11 assumes 9.
>
> **A note specific to parallelism:** CS336 has *two* lectures called
> "Parallelism", and **both are now covered**. Lecture 7 is Percy Liang's
> executable lecture — collective operations and the data/tensor/pipeline cuts.
> Lecture 8 is Tatsunori Hashimoto's slide deck, and it is where FSDP and ZeRO,
> which lecture 7 repeatedly defers to, are actually explained. Read them in order.
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

- **[Lecture 6 — Kernels and Triton](wiki/06-kernels-triton.md)** — the hands-on
  half of the systems pair. How to find out where your time actually goes
  (benchmarking and profiling, with both harnesses written from scratch), and how to
  write the kernel that fixes it. Four Triton kernels of increasing difficulty —
  elementwise GeLU, softmax, a row sum too long for one block, and tiled matmul —
  plus the five hardware details the programming model hides.

- **[Lecture 8 — Parallelism (Part 2)](wiki/08-parallelism-2.md)** —
  what you actually do with those primitives on a cluster. ZeRO stages 1-3 and
  FSDP, and why two of the three are free; the pipeline bubble and how to fill it;
  what activation memory really costs and which parts tensor parallelism cannot
  touch; expert and context parallelism; and the rules for combining four or more
  strategies at once, checked against ten published training runs.
- **[Lecture 7 — Parallelism](wiki/07-parallelism.md)** — crossing the chip
  boundary. The collective operations that distributed training is built from, the
  interconnect hierarchy that carries them (NVLink, InfiniBand, Ethernet), and then
  three ways to cut a network across GPUs: data parallelism along the batch, tensor
  parallelism along the width, pipeline parallelism along the depth — each
  implemented from primitives rather than called from a library.

- **[Lecture 9 — Scaling Laws (basics)](wiki/09-scaling-laws.md)** — how to spend a
  budget you only get to spend once. Why loss is log-linear in data, parameters and
  compute; where the ≈0.1 exponent comes from and what it says about how networks
  learn; using scaling trends to settle architecture, optimizer and depth/width
  questions without training the big model; critical batch size and learning-rate
  scaling; and the Kaplan-versus-Chinchilla dispute in full — the three methods, the
  three small decisions that caused the disagreement, and why you should overtrain
  past the answer anyway.

- **[Lecture 10 — Inference](wiki/10-inference.md)** — the only lecture about what
  happens *after* training, and the one that explains why "inference is memory
  bound" is true rather than folklore. It derives the arithmetic intensity of both
  phases (prefill is compute-bound, generation is not, and batching cannot rescue
  attention because every sequence carries its own KV cache), turns that into a
  performance model for Llama 2 13B on an H100, and then spends the rest of the
  hour buying memory traffic back: KV-cache reductions, quantization, pruning with
  distillation, speculative sampling, continuous batching and PagedAttention.

- **[Lecture 11 — Scaling laws in the wild](wiki/11-scaling-laws-in-the-wild.md)** —
  the practical companion to Lecture 9, and the lecture to read before you spend
  money on a training run. It asks whether the Chinchilla approach survives contact
  with people actually training frontier models, and answers by reading eight
  published recipes off their own figures. Its spine is a genuine fork: either make
  the optimal hyperparameters stop moving as you scale (muP, MiniCPM's route) or
  accept that they move and fit a scaling law to where they move to (DeepSeek's).
  Along the way: why fitting a scaling law honestly costs $n^2$ and how WSD
  schedules make it linear, why most published optimizer comparisons are confounded
  by scale, and a worked case where a flawless-looking fit diverged two and a half
  decades out.

If you are looking for a single number or formula, the topic pages below are
usually the faster route than the lecture pages.

## Wiki

### Lecture 11 — scaling laws in the wild

- **[Lecture 11 — scaling laws in the wild](wiki/11-scaling-laws-in-the-wild.md)** —
  the lecture page: the two strategies, the two headline recipes, the rapid tour,
  Step Law, optimizers, and the muP derivation, in the order the lecture gives them.
- **[Published scaling recipes](wiki/published-scaling-recipes.md)** — MiniCPM and
  DeepSeek set side by side, plus Qwen, Kimi K2, Hunyuan, LLaMA 3 and MiniMax-01.
  Read this for what practitioners actually do, and for the two disagreeing answers
  to the tokens-per-parameter question: MiniCPM's joint fit says 95.60 and DeepSeek's
  IsoFLOP analysis splits compute almost evenly, close to Chinchilla.
- **[Maximal update parametrization](wiki/maximal-update-parametrization.md)** — muP
  in full: the two conditions, both derivations, the prescription for SGD and Adam,
  the evidence that the optimum stays put from 2M to 10B parameters, and the three
  things that break it — RMSNorm gains, Lion, and strong weight decay.
- **[WSD schedules](wiki/wsd-schedules.md)** — warmup–stable–decay. Why a cosine
  schedule forces you to retrain from scratch for every point on a scaling curve,
  making the fit cost $n^2$, and how branching off a stable trunk makes it linear.
  Includes the caveat the slide's own title hides: only the ~10% decays beat cosine.
- **[Step Law and hyperparameter scaling](wiki/step-law.md)** — StepFun's brute-force
  grid search. The published laws disagree about what the optimum is even a function
  of; the sweep finds the loss surface convex in both, and batch size depending
  primarily on dataset size while learning rate needs both $N$ and $D$.
- **[Optimizer scaling](wiki/optimizer-scaling.md)** — Muon, and the three reasons
  optimizer comparisons mislead: the hyperparameters are usually mistuned, the
  advantage shrinks about fourfold over a decade of model size, and a clean-looking
  scaling fit can still diverge out of sample.

### Lecture 10 — inference

- **[Inference](wiki/inference.md)** — the hub. Why serving is a repeated cost that
  now rivals training in aggregate compute, why agents removed the ceiling on how
  much speed is worth buying, the three metrics (TTFT, latency, throughput) and
  which application each belongs to, the serving landscape (vLLM, SGLang,
  TensorRT-LLM, llama.cpp), and the three-way split of the techniques.
- **[KV cache](wiki/kv-cache.md)** — the object everything in this lecture is
  trying to shrink. Why it exists (naive generation is $O(T^3)$), its exact size
  formula, why one 1024-token request costs 0.84 GB against 26 GB of Llama 2 13B
  weights, and the **four axes** — heads, dimension, layers, sequence — that GQA,
  MLA, CLA and local attention each cut. **Start here if you want the one table
  that organizes the whole lecture.**
- **[Prefill and generation](wiki/prefill-and-generation.md)** — the two phases and
  their opposite bottlenecks, with the full intensity table: prefill is
  compute-bound at $BS$ and $S/2$, generation is memory-bound at $B$ and
  $S/(S+1) < 1$ against hardware that wants 295. Also why batching rescues the MLP
  and cannot rescue attention, which is the structural fact the rest follows from.
- **[Latency and throughput](wiki/latency-and-throughput.md)** — the tradeoff, and
  the symbolic performance model behind it. Llama 2 13B on an H100 at batch 1, 64
  and 256: latency linear in $B$, throughput asymptoting, and 240 GB of memory
  demanded by a configuration an 80 GB card cannot run. Also why reducing memory
  escapes the tradeoff entirely while increasing batch size does not.
- **[Speculative sampling](wiki/speculative-sampling.md)** — the lossless one. The
  draft-and-verify algorithm in full, the two-token proof that the output
  distribution is *exactly* the target model's, the measured 1.92–2.46× speedups,
  why $K \approx 3$–4 is the sweet spot, and Medusa and EAGLE as ways to improve
  the draft model.
- **[Quantization](wiki/quantization.md)** — scale and zero point worked through
  one number, the format table from fp32 to int4 with the ranges that explain why
  int8 is inference-only, quantization-aware training versus post-training
  quantization, GPTQ's error propagation, and AWQ's finding that *activations*
  decide which weights deserve precision.
- **[Pruning and distillation](wiki/pruning-and-distillation.md)** — rip pieces out
  of a trained model and heal it. NVIDIA's importance-rank-trim-distill loop, how
  importance is actually measured (and the student question about a
  constant-output neuron that gets the right answer), and Minitron 8B reaching a
  15B model's neighbourhood for about 40× less training.
- **[Continuous batching](wiki/continuous-batching.md)** — Orca's iteration-level
  scheduling, which edits the batch between decode steps so an arriving request
  never waits, plus selective batching, which splits attention from the MLP along
  exactly the line the intensity derivation predicted.
- **[PagedAttention](wiki/paged-attention.md)** — virtual memory for the KV cache.
  Internal and external fragmentation (2,038 reserved-and-never-used slots in the
  paper's own figure), fixed-size blocks with a block table, and prefix sharing
  with copy-on-write for system prompts and multi-sample generation. The core idea
  of vLLM.
- **[Cross-layer attention](wiki/cross-layer-attention.md)** — the least-known of
  the four cache cuts, and the cleanest illustration of the idea: share keys and
  values *down the layer stack*, just as GQA shares them across heads.

### Lecture 9 — scaling laws

- **[Compute-optimal scaling](wiki/compute-optimal-scaling.md)** — the
  Kaplan-versus-Chinchilla story end to end: the joint model-data scaling forms,
  Kaplan's $N \propto C^{0.73}$ and the era of trillion-parameter dense models,
  Chinchilla's three methods and the 20-tokens-per-parameter rule, the three small
  methodological decisions that separated them, the fitting error in Chinchilla's
  own method 3, and why production models deliberately overtrain past 20:1.
  **Start here if you came looking for Chinchilla.**
- **[Data scaling laws](wiki/data-scaling-laws.md)** — the univariate law. The
  mean-estimation derivation of why error decays polynomially, why classical
  statistics predicts a slope of $-1$, why neural exponents come out near $-0.1$
  instead, and the non-parametric argument that reads that exponent as "learning at
  the rate of a ten-dimensional smoother". Also the result reused everywhere else in
  the lecture: interventions move intercepts, not slopes.
- **[Scaling law methodology](wiki/scaling-law-methodology.md)** — how not to fool
  yourself. Predictability is engineered rather than observed; a scaling law is a
  lower bound on a *recipe*, so it inherits every defect of the runs underneath it;
  choosing the right x-axis; scale-invariant quantities; and why a narrow compute
  range cannot distinguish a polynomial from an exponential.
- **[The IsoFLOP method](wiki/isoflop-method.md)** — fix the compute budget, sweep
  everything else, read the minimum off the curve. Chinchilla's method 2, the most
  robust of the three, and the one tool from this lecture that keeps working
  elsewhere — diffusion models, MoE sparsity surfaces.
- **[Upstream vs downstream](wiki/upstream-vs-downstream.md)** — where the
  predictability stops. Perplexity scales beautifully and benchmark accuracy does
  not; the model that wins downstream is mid-table upstream. Also which measurements
  are clean enough to fit from a single run and which are not.
- **[Critical batch size](wiki/critical-batch-size.md)** — now carrying both
  lectures' treatments. Lecture 8's systems view (batch size as a consumable
  resource that caps data parallelism) plus Lecture 9's optimisation view: the
  noise-limited and bias-limited regimes, the estimation recipe, $B_{crit} =
  E_{min}/S_{min}$, and the power law by which it grows as your loss target falls.
- **[Learning rate scaling and muP](wiki/learning-rate-scaling-and-mup.md)** — the
  other hyperparameter you cannot inherit. Why the optimum shifts with width, the
  $1/\text{width}$ rule of thumb, and the two competing philosophies: predict where
  the minimum goes, or reparameterise so it stops moving.
- **[Data repetition](wiki/data-repetition.md)** — four epochs are free, forty are
  worthless, and the effective-data formula in between. Plus the finding that
  optimal data *filtering* loosens as compute grows, so filter aggressiveness is a
  function of scale rather than a property of the corpus.
- **[Data mixture selection](wiki/data-mixture-selection.md)** — fitting mixture
  scaling laws, why practitioners mostly skip it and just bake off small models, and
  why that shortcut is justified by the same theory that motivated the laws.

### Lecture 8 — parallelism at cluster scale

- **[ZeRO and FSDP](wiki/zero-and-fsdp.md)** — the three sharding stages, what each
  one shards, and the collective identity that makes stages 1 and 2 cost *nothing*
  extra over plain DDP. How stage 3 hides its extra all-gather by overlapping
  communication with computation, why it is not pipelining, and where it stops.
  **Start here if you came looking for FSDP.**
- **[Activation memory](wiki/activation-memory.md)** — the $sbh(34 + 5as/h)$
  per-layer accounting, why memory peaks *after* the forward pass, which 24 of the
  34 terms tensor parallelism divides and which stubborn 10 it does not, and the
  five-row table ending in the practical lower bound $sbh \cdot 34/t$.
- **[Sequence parallelism](wiki/sequence-parallelism.md)** — splitting LayerNorm,
  dropout and residual activations along the *sequence* axis to remove the term
  tensor parallelism leaves behind. Why the name is misleading, and the
  forward/backward collective swap it shares with FSDP.
- **[Context parallelism](wiki/context-parallelism.md)** — ring attention: split the
  sequence itself across devices. Used for long-context extension and serving, and
  the technique that actually deserves the name "sequence parallel".
- **[Zero-bubble pipelining](wiki/zero-bubble-pipelining.md)** — separate the
  backward pass into propagating partials (on the critical path) and computing
  weight gradients (a leaf that can wait), then fill the bubble with the deferred
  half.
- **[Critical batch size](wiki/critical-batch-size.md)** — why batch size is a
  budget rather than a free parameter, what data parallelism and pipelines each
  spend it on, and how recomputation buys it back.
- **[Network topology: mesh vs tree](wiki/network-topology.md)** — TPU toroidal mesh
  against GPU fat tree, why constant node degree matters, the Huawei Ascend
  brute-force corner and its 4× power bill, and the convergent evolution visible in
  TPU8i/8t.
- **[3D (and 4D) parallelism](wiki/3d-parallelism.md)** — the composition rules: cut
  until it fits, tensor or expert parallel on the fast interconnect, pipeline or
  FSDP the rest of the way, then data-parallel everything left. With NVIDIA's own
  guidelines and the Narayanan 2021 evidence.
- **[Parallelism case studies](wiki/parallelism-case-studies.md)** — what ten real
  runs chose, including slide 72's full configuration table. OLMo on FSDP alone,
  Llama 3 405B's per-stage breakdown, Gemma 2 with no pipeline at all, DeepSeek V3's
  64-way expert parallelism.

### Lecture 7 — parallelism across GPUs

- **[Collective operations](wiki/collective-operations.md)** — the eight primitives
  (broadcast, scatter, gather, reduce, all-gather, reduce-scatter, all-reduce,
  all-to-all), each with the lecture's own worked four-rank example, plus the naming
  rule that makes them memorable and the all-reduce = reduce-scatter + all-gather
  identity that FSDP depends on.
- **[GPU interconnect](wiki/gpu-interconnect.md)** — NVLink, NVSwitch, InfiniBand,
  Ethernet and PCIe, with the bandwidth of each and why the tiers exist at all.
  RDMA and what it bypasses, NVL72, RoCE. Start here to work out which parallelism
  strategy your hardware can support.
- **[torch.distributed and NCCL](wiki/torch-distributed.md)** — the software stack:
  what NCCL does for you, the nccl/gloo backends, process groups, and the two
  independent kinds of asynchrony (CUDA kernels and processes) that make barrier
  ordering matter.
- **[Data parallelism](wiki/data-parallelism.md)** — DDP: slice the batch,
  all-reduce the gradients, and change one line of an ordinary training loop. Why
  losses differ across ranks but parameters never do, and the three things that cap
  it — memory, divisibility, and the critical batch size.
- **[Tensor parallelism](wiki/tensor-parallelism.md)** — column-parallel: shard each
  weight matrix down its columns and all-gather the activations after every layer.
  Why the nonlinearity can be applied before the gather, and why this one needs
  NVLink.
- **[Pipeline parallelism](wiki/pipeline-parallelism.md)** — shard the depth and
  pass activations rank to rank with point-to-point send/recv. Pipeline bubbles,
  micro-batching, and why this is the strategy that tolerates a bad network.
- **[Sharding, replication and recomputation](wiki/sharding-vs-replication.md)** —
  the lecture's closing generalization, and the most portable idea in the systems
  unit: recompute it, store it, or store it on another GPU and communicate it.

### Lecture 6 — kernels and Triton

- **[Triton](wiki/triton.md)** — the language CS336 writes kernels in. Why you
  program the *thread block* rather than the thread, the skeleton every kernel
  shares (wake up, find your index, load, compute, store), and the four things a
  PyTorch programmer has to unlearn: you get pointers not tensors, there is no
  return value, `tl.program_id` is your identity, and masking is not optional. Also
  what Triton decides for you — where a value lives, whether you get the tensor
  cores — and what the alternatives are.
- **[PTX](wiki/ptx.md)** — the assembly Triton compiles to, and what reading it
  tells you. The thread block is gone, `%ctaid.x` and `%tid.x` are how one compiled
  body serves every thread, and the compiler quietly gave each thread eight elements
  instead of one. Start here for "what does my kernel actually become?"
- **[Benchmarking](wiki/benchmarking.md)** — the three gotchas that make a naive
  timing loop wrong on a GPU: warm up, synchronize, and time with CUDA events over
  several trials. Plus what a scaling sweep shows — matmul time is *constant* below
  about 2000 dimensions before it turns cubic.
- **[Profiling](wiki/profiling.md)** — where the time went, and what PyTorch is
  really doing. How to read a CUDA kernel name (`cutlass`, `sm100`, `f32`,
  `64x64x16`), why the same `a @ b` dispatches to different kernels at different
  sizes, and how the profiler diagnoses the GeLU race.
- **[torch.compile](wiki/torch-compile.md)** — what compilation does to a
  computation graph, why the result is a *Triton* kernel, and the honest scoreboard:
  on the day, compiled beat the naive version and lost to the hand-written built-in.
- **[Warp occupancy](wiki/warp-occupancy.md)** — the register budget that decides
  how many warps fit on an SM, worked through to 18%, and why low occupancy is not
  automatically bad. Thread coarsening, and what occupancy is really for (slack for
  the scheduler to hide stalls with).
- **[Bank conflicts](wiki/bank-conflicts.md)** — 32 banks of 4 bytes, one access
  each per cycle, and why 32 threads reading a matrix column serialize completely.
  Why matmul cannot avoid it, what swizzling does, and how this differs from memory
  coalescing.
- **[Fused softmax](wiki/fused-softmax.md)** — the reduction kernel, and the
  clearest fusion arithmetic in the course: $5MN + M$ reads naively against the $MN$
  a fused kernel needs. One row per block, why the mask pads with $-\infty$, and
  what to do when a row does not fit.

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
- **[Scaling laws](wiki/scaling-laws.md)** — the **hub page** for the scaling-law
  thread, and no longer a preview-only page. It keeps Percy's Lecture 1 framing —
  scaling recipes rather than single models, hyperparameter transfer, predictability
  over optimality — and indexes the Lecture 9 treatment, with a section reading the
  two against each other, and now indexing the Lecture 11 treatment as well.

## Raw material

- **[`raw/transcripts/`](raw/transcripts/)** — **the transcripts to read**, one per
  covered lecture:
  [Lecture 1](raw/transcripts/01-overview-tokenization.md),
  [Lecture 2](raw/transcripts/02-pytorch-resource-accounting.md),
  [Lecture 3](raw/transcripts/03-architectures.md),
  [Lecture 4](raw/transcripts/04-attention-alternatives.md),
  [Lecture 5](raw/transcripts/05-gpus-tpus.md),
  [Lecture 6](raw/transcripts/06-kernels-triton.md),
  [Lecture 7](raw/transcripts/07-parallelism.md),
  [Lecture 8](raw/transcripts/08-parallelism-2.md),
  [Lecture 9](raw/transcripts/09-scaling-laws.md),
  [Lecture 10](raw/transcripts/10-inference.md),
  [Lecture 11](raw/transcripts/11-scaling-laws-in-the-wild.md).
  Copy-edited from the auto-captions: repunctuated, filler removed, mis-heard
  technical terms restored against the lecture material. Every `[MM:SS]` marker is
  preserved in its original position, so timestamps quoted from them are citable.
  Each header lists every restoration made and every place left marked unclear.
- **`raw/transcripts/original/`** — *not in this repo.* The verbatim auto-captions
  are a complete reproduction of each lecture with none of this KB's own work in
  them, so they are deliberately not published here. They remain reproducible by
  anyone who wants to audit an edit: run the `cairn-kb` skill's
  `fetch_transcript.py` on the video id in the transcript's front matter and pipe it
  through `transcript_to_md.py`. See [`LICENSE.md`](LICENSE.md).
- **[`raw/slides/`](raw/slides/)** — the written course material. **This is the
  authority for anything the lecturer wrote down**; the transcript is the authority
  for what was said. CS336 supplies it in two very different forms:
  - [`lecture_01.py`](raw/slides/01-overview-tokenization.md),
    [`lecture_02.py`](raw/slides/02-pytorch-resource-accounting.md),
    [`lecture_06.py`](raw/slides/06-kernels-triton.md),
    [`lecture_07.py`](raw/slides/07-parallelism.md) and
    [`lecture_10.py`](raw/slides/10-inference.md) are Percy
    Liang's *executable lectures* — Python programs, transcribed from source text,
    each with a section-to-source-line table and the code verbatim. There are no
    slide numbers to cite. Where such a lecture computes a number at runtime, a
    deterministic one is recomputed and marked "(computed)", while a measurement of
    the lecturer's own GPU — a timing, a profiler table — is marked
    machine-dependent and no value is given. Lecture 6 is mostly the second kind.
    **Lecture 7 is the exception**: the course publishes that one program's own
    standard output from a real four-GPU run, so its measured bandwidths and
    per-rank losses are quoted and marked "(recorded run)" — measurements of that
    machine, not of yours. **Lecture 10 is the opposite extreme**: it computes
    symbolically with sympy, so it has no machine-dependent values at all — every
    quantity was reproduced by evaluating the lecture's own expression, and each
    matches the `assert` the source makes about it. Its Llama 2 13B latency and
    throughput figures are theoretical maxima under a stated
    perfect-overlap assumption, not benchmarks.
  - [`lecture_03.pdf`](raw/slides/03-architectures.md) (67 pages),
    [`lecture_04.pdf`](raw/slides/04-attention-alternatives.md) (60 pages),
    [`lecture_05.pdf`](raw/slides/05-gpus-tpus.md) (55 pages),
    [`lecture_08.pdf`](raw/slides/08-parallelism-2.md) (73 pages) and
    [`lecture_09.pdf`](raw/slides/09-scaling-laws.md) (57 pages) and
    [`lecture_11.pdf`](raw/slides/11-scaling-laws-in-the-wild.md) (58 pages) are
    Tatsunori
    Hashimoto's slide decks, transcribed from the rendered page images, with every
    figure described in prose and every table transcribed cell by cell. Slide
    numbers in all six are **PDF page numbers**, because none of the decks prints
    any of its own. **Lecture 11's deck is the odd one out**: it is the only deck read
    at Opus rather than Sonnet — a choice made because at 33 words of native text per page
    it is the most figure-dependent deck in the course. It has had no figure audit
    yet, and its front matter states the boundary that follows: slide text and
    tables reliable, chart values provisional. Lectures 4, 5 and 8 are the figure-dependent ones — 102 images
    across 60 pages, 83 across 55, and 86 across 73, most pages carrying only 30–40
    words of their own text — so the figure descriptions there are not a supplement
    to the content, they *are* the content. Each deck's front matter records which
    pages were audited against the PDF and what the audit found, including the
    places where a deck contradicts itself. Lecture 8's deck is the most heavily
    audited: twelve pages checked across two passes, plus a sweep of every
    cross-slide claim in the file.
- **[`sources.md`](sources.md)** — every lecture, deck, assignment and linked
  document with its canonical URL, including the material for the 8 lectures this
  KB does not yet cover. Explains how CS336 splits between executable lectures and
  PDF decks.

- **`raw/images/NN-<slug>/`** — pictures, so an answer can *show* a figure rather than
  only describe it. All ten covered lectures have them, and so does lecture 11: 32–51
  images each for the six PDF-deck lectures (3, 4, 5, 8, 9, 11), one for every
  figure-bearing page; and 4–22 each for
  the five executable lectures (1, 2, 6, 7, 10), which have no deck, so these are the
  figures the course serves from its own repo. Lecture 10 is much the richest of those
  five, with 22 — it is a heavily illustrated lecture whose figures are mostly
  reproduced tables and charts from the papers it discusses. Each image sits beside the slide it shows in
  `raw/slides/`, and in `wiki/` wherever a page cites that slide. About a third of every
  deck was deliberately *not* rendered — title cards, outlines, dividers, and the tables
  and equations `raw/slides/` already reproduces cell by cell — so **read an image path
  out of a file rather than constructing one**. `AGENTS.md` has the conventions and the
  attribution; [`LICENSE.md`](LICENSE.md) has the copyright position, which is not the
  same for these as for the rest of the repo.

`raw/pdfs/` is empty by design — no binaries are committed. The course's decks are
5–7 MB each and live at the URLs in `sources.md`. The rendered slide images in
`raw/images/` (31 MB) are the one exception to "no binaries": they are committed, because
a picture is the thing a prose description cannot replace.

## Also

- **[`SEE_ALSO.md`](SEE_ALSO.md)** — sibling knowledge bases worth reading:
  **CS224N** (complete, all 23 lectures) for the attention and Transformer
  derivations CS336 assumes, and **CS221** (lectures 1–4) for gradient descent,
  backpropagation, tensors and einops.
- **[`kb.json`](kb.json)** — machine-readable coverage and provenance. Read this to
  know how far to trust a citation from here.
- **[`LICENSE.md`](LICENSE.md)** — **read this before reusing anything.** This repo has
  no single licence: the explanatory writing is CC BY 4.0, but the CS336 course material
  and the paper figures reproduced inside the decks are not mine to license, and are
  included for study under fair use. That reliance does not transfer to you.
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
