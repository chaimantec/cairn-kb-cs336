# CS336 — Language Modeling from Scratch (Stanford, Spring 2026)

Stanford's CS336 teaches you to build a language model from the ground up:
tokenizer, Transformer, optimizer, training loop, GPU kernels, parallelism,
inference, scaling laws, data pipelines and alignment. It is taught by **Percy
Liang** and **Tatsunori Hashimoto**, and this is its third offering. The
organizing question, stated in the first lecture and returned to in every unit, is
**efficiency**: what is the best model you can build from a fixed budget of
compute and data?

> ## ⚠️ This knowledge base covers Lectures 1–16 of 18
>
> **Lecture 1 (Overview and Tokenization), Lecture 2 (PyTorch and Resource
> Accounting), Lecture 3 (Architectures), Lecture 4 (Attention Alternatives and
> Mixtures of Experts), Lecture 5 (GPUs and TPUs), Lecture 6 (Kernels and
> Triton), Lecture 7 (Parallelism), Lecture 8 (Parallelism, Part 2), Lecture 9
> (Scaling Laws — Basics), Lecture 10 (Inference), Lecture 11 (Scaling Laws in
> the Wild), Lecture 12 (Evaluation), Lecture 13 (Data I — Sources and
> Datasets), Lecture 14 (Data II — Filtering, Deduplication, Mixing,
> Post-Training Data), Lecture 15 (Mid/Post-Training — SFT and RLHF) and
> Lecture 16 (Post-Training — RLVR) are covered in depth.**
> Nothing else is. **The Data unit is complete**, and the post-training unit is
> now two lectures deep: Lecture 15 takes the course from a base model to
> something close to ChatGPT via supervised fine-tuning and RLHF, and
> **Lecture 16 goes from there to o1- and R1-class reasoning models** via
> reinforcement learning from verifiable rewards — PPO, GRPO, and the three open
> recipes (DeepSeek-R1, Kimi K1.5, Qwen 3) read side by side. The topics lecture
> 15 kept deferring to "next lecture" — RLVR, GRPO, reasoning models — are now
> covered. There are still no transcripts and no wiki pages for alignment and
> multimodality (Lecture 17) or the guest lecture (18).
>
> **A note specific to evaluation:** [Lecture 12](wiki/12-evaluation.md) is the
> hinge of the course — the point where model-building stops and the question
> becomes what behaviour you wanted in the first place. It surveys perplexity,
> exam, chat, agentic, reasoning and safety benchmarks, then argues that the
> survey was never the point: every benchmark is an attempt to turn an abstract
> construct into a concrete metric, and each loses something in the translation.
> It is the natural next read after the scaling-law lectures, because it is the
> general case of their warning that a clean perplexity curve need not predict
> anything a user cares about.
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

- **[Lecture 12 — Evaluation](wiki/12-evaluation.md)** — given a model, how good
  is it? The lecture that sits between building a model and choosing its data, and
  the one to read before you trust any number in a model card. Its spine is a
  single line — *abstract construct → concrete metric* — and its survey is ordered
  by what is being measured and, implicitly, by who judges: perplexity needs no
  judge and is therefore untargeted; exams have an answer key and pay for it in
  realism; chat has none and must hire a judge, inheriting the judge's biases;
  agentic benchmarks get an executable answer key back, which is the cleanest
  escape in the lecture; and safety has neither an answer key nor an agreed judge.
  Then contamination, dataset quality, and the methods-versus-models distinction
  that explains what this course's own assignments are actually measuring.

- **[Lecture 13 — Data I: Sources and Datasets](wiki/13-data-sources-datasets.md)** —
  where training data actually comes from, and what you are allowed to do with it.
  A history and a survey rather than a derivation: why "trained on the entire
  internet" is wrong four times over, what copyright and fair use permit, the four
  raw sources everything is built from, and then a chronological tour of fifteen
  named corpora from BooksCorpus (2015) to CommonPile (2025). The through-line is
  that **the sources barely change after 2019 — almost every advance is a filtering
  decision**, which is why the lecture ends by calling filtering the highest-leverage
  step and the whole process "a lot just based on vibes."

- **[Lecture 14 — Data II: Filtering, Deduplication, Mixing, Post-Training
  Data](wiki/14-data-filtering-dedup-mixing.md)** — what you *do* to data once you
  have it, and the only lecture in the course that is mostly algorithms with
  running code. Filtering is one problem stated once — given target data T and raw
  data R, find the subset of R that resembles T — and language identification,
  quality filtering and toxicity filtering are that same problem with a different
  T. Deduplication is the linear-time construction that gets you from Jaccard
  similarity to MinHash to locality-sensitive hashing. Mixing is a regression
  problem with a trap in it: a plain 50/50 mixture over a 10T-token source and a
  10B-token source silently runs **50 epochs** on the small one. The recurring
  lesson is that **every stage has a scale-dependent optimum** — there is no best
  filtering threshold and no best mixture in the abstract, only one relative to how
  long you will train.

- **[Lecture 15 — Mid/Post-Training](wiki/15-mid-post-training.md)** — the lecture
  that gets you from GPT-3 to ChatGPT, and the first of the course's post-training
  unit. Two halves: supervised fine-tuning, which is "basically exactly the same as
  pre-training" so that everything interesting is in the data; and RLHF, which is a
  different game entirely — SFT fits a distribution, RLHF maximizes a reward, and
  that single distinction is what makes mode collapse a permitted outcome rather
  than a bug. Along the way: six generations of open SFT data and why they got
  *smaller*, the argument that **training on facts the model does not know teaches
  it to hallucinate**, midtraining dissolving the pre-training/post-training
  boundary, fifteen minutes on who the annotators are and why that changes what the
  model believes, and the DPO derivation in full. It ends by naming
  **overoptimization** as RLHF's big open problem, which is the door to lecture 16.

If you are looking for a single number or formula, the topic pages below are
usually the faster route than the lecture pages.

## Wiki

### Lecture 16 — post-training: RLVR

- **[Lecture 16 — Post-training: RLVR](wiki/16-post-training-rlvr.md)** — the
  lecture page: why RLHF has a compute ceiling and a verifier removes it, PPO's
  implementation reality, GRPO and its two flaws, then DeepSeek-R1, Kimi K1.5,
  Qwen 3 and agentic RL read side by side. Carries 21 of the lecture's 47
  figures; the topic pages below carry the rest.
- **[RLVR](wiki/rlvr.md)** — reinforcement learning from verifiable rewards: the
  idea, why it lifts RLHF's ceiling, and the two ways "verifiable" turns out to
  be weaker than it sounds.
- **[GRPO](wiki/grpo.md)** — PPO with the value function deleted and the
  baseline taken from sibling rollouts. The z-score advantage, why on-policy
  running makes the clipping vanish, and why it is not a valid policy gradient.
- **[Advantage estimation and baselines](wiki/advantage-estimation-and-baselines.md)**
  — what the policy-gradient theorem actually licenses you to subtract, the
  three ways to get a baseline, and why GAE is usually inert in language-model
  PPO.
- **[Length bias in RL](wiki/length-bias-in-rl.md)** — GRPO's length normalizer
  rewards long *wrong* answers. Read this before believing that R1's growing
  chain-of-thought plot shows the model thinking harder.
- **[Verifiable rewards](wiki/verifiable-rewards.md)** — how much a checker
  really buys. Kimi verifies mathematics with a reward model; the Lean compiler
  turned out not to be adversarially robust.
- **[Reward hacking](wiki/reward-hacking.md)** — reading the answer out of a
  repository's future commits, why that looks like emergence in a training
  curve, and why blocklisting `git log` does not close it.
- **[DeepSeek-R1](wiki/deepseek-r1.md)** — the case study: R1-Zero as a
  controlled experiment, and why both of its famous phenomena — growing CoT
  length and the "aha moment" — are overstated.
- **[Kimi K1.5](wiki/kimi-k1-5.md)** — the same destination by a different
  derivation, plus length as a cost, best-of-$k$ difficulty filtering, and the
  RL-infrastructure section.
- **[Qwen 3](wiki/qwen3.md)** — the consolidated playbook, RL on roughly 4,000
  examples, and thinking-mode fusion (which the field has since partly
  reversed).
- **[Agentic RL](wiki/agentic-rl.md)** — there is no agentic algorithm, only
  data: midtraining sources, four expert models distilled back into one, and
  SWE-bench-style environments generated from GitHub.
- **[Process versus outcome supervision](wiki/process-vs-outcome-supervision.md)**
  — why process reward models lost, and it was scalability of the supervision
  rather than accuracy.
- **[Expert iteration](wiki/expert-iteration.md)** — training on your own
  correct answers, and Kimi's ablation showing RL consistently beats it.
- **[RL infrastructure](wiki/rl-infrastructure.md)** — the straggler problem,
  moving weights between a training and an inference system, and why chasing
  utilization by reusing rollouts destabilizes training.
- **[Reasoning models](wiki/reasoning-models.md)** — what defines one, and the
  open question of whether RL is needed at all.
- **[Long chain-of-thought](wiki/long-chain-of-thought.md)** — held three ways:
  a capability, an inference cost, and an artifact of the objective.
- **[Test-time scaling](wiki/test-time-scaling.md)** — accuracy degrades
  gracefully as the thinking budget shrinks, even when the chain is truncated
  mid-thought.
- **[Thinking-mode fusion](wiki/thinking-mode-fusion.md)** — one model, two
  behaviours, switched by a prompt tag; and why later releases went back on it.
- **[Reasoning distillation](wiki/reasoning-distillation.md)** — SFT on someone
  else's chains of thought recovers much of the capability, which reframes RL as
  a source of supervision rather than an optimizer.

### Lecture 15 — mid/post-training: SFT and RLHF

- **[Lecture 15 — Mid/Post-Training](wiki/15-mid-post-training.md)** — the lecture
  page: the whole arc from base model to chat model, in the order the lecture
  builds it. Carries 17 of the lecture's 38 figures; the topic pages below carry
  the rest. Read it first if you want the shape of the lecture rather than one
  answer.
- **[Supervised fine-tuning](wiki/supervised-fine-tuning.md)** — why the method is
  deliberately boring and the data is everything. SFT as *extraction* rather than
  instruction: 500 examples measurably steer a model, adding correct data can make
  it worse, and you mostly cannot tell what pre-training already contains.
- **[Instruction-tuning datasets](wiki/instruction-tuning-datasets.md)** — FLAN,
  Self-Instruct, Alpaca, Vicuna, Open Assistant, WizardLM, Tulu 3, Nemotron. Why
  FLAN's reconstituted-benchmark examples read so strangely, why Open Assistant's
  volunteer effort stalled, and why the newest SFT data is tool calls rather than
  chat.
- **[Midtraining](wiki/midtraining.md)** — instruction data mixed into the *decay
  phase* of pre-training rather than applied after it. Why "base model" has stopped
  meaning what it meant, and why the decay phase is where mixture ablations
  actually get run.
- **[RLHF](wiki/rlhf.md)** — the pipeline and the conceptual shift from fitting a
  distribution to maximizing a reward. Also the two arguments for optimizing at
  all: the generation–verification gap, and verification being easier than
  generation.
- **[Preference data](wiki/preference-data.md)** — the comparison judgments
  themselves, and the only two public sets of annotation guidelines that exist:
  InstructGPT's helpful/truthful/harmless, and Google Bard's, which leaked.
- **[Human annotation](wiki/human-annotation.md)** — the longest section of the
  lecture. The workforce moved upmarket and pay bifurcated; annotator demographics
  measurably shift what opinions a model expresses; expertise decides which errors
  get caught; and inter-annotator agreement cannot tell consensus from everyone
  quietly using the same chatbot.
- **[Model-based annotation](wiki/model-based-annotation.md)** — the Zephyr
  experiment, where a well-resourced attempt to avoid distillation gave up and
  switched to AI feedback. Where model annotation works, and the two places it
  cannot: past the frontier, and for world knowledge only professionals have.
- **[Reward models](wiki/reward-models.md)** — the Bradley–Terry pairwise loss, why
  it is identified only up to a per-prompt shift, and why that same invariance is
  what makes DPO possible.
- **[PPO](wiki/ppo.md)** — policy gradients → off-policy/TRPO → clipped surrogate,
  as three fixes to three problems. Includes the distinction between the two
  different KL terms that are easy to confuse.
- **[DPO](wiki/dpo.md)** — the full derivation in three moves, the annotated
  gradient (raise the winner, lower the loser, step size set by how wrong the
  implied reward was), the variants that "don't seem to matter very much," and the
  unresolved DPO-versus-PPO dispute.
- **[Reward overoptimization](wiki/reward-overoptimization.md)** — the lecture's
  nominated big problem. Goodhart's law with a training curve, and the observation
  that a *better* optimizer makes it worse.
- **[Mode collapse and calibration](wiki/mode-collapse-and-calibration.md)** — why
  a reward objective permits collapse, and GPT-4's still-unsolved post-RLHF
  miscalibration. Notes the tension with the hallucination page's argument that RL
  is what *produces* calibration.
- **[Style and length bias](wiki/style-and-length-bias.md)** — style is a
  deliberate data-collection decision; raters and model judges both reward it; and
  you can RLHF on length alone and score well. The rule: control style separately
  from capability.
- **[Hallucination and knowledge extraction](wiki/hallucination-and-knowledge-extraction.md)**
  — the subtlest argument in the lecture. One Open Assistant example teaches both a
  citation and the habit of citing, and only the second generalizes.
- **[Safety tuning](wiki/safety-tuning.md)** — the violation-rate versus
  false-refusal trade-off, Tulu 3's WildChat-mined pipeline, and the finding that
  ~500 examples is enough to move refusal behaviour across four benchmarks.

### Lecture 14 — data: filtering, deduplication, mixing, post-training

- **[Lecture 14 — Data II: filtering, deduplication, mixing, post-training
  data](wiki/14-data-filtering-dedup-mixing.md)** — the lecture page: the whole
  pre-training pipeline in the order data moves through it, then post-training
  data. Carries 7 of the lecture's 13 figures — the ones its own argument needs —
  and links to the six topic pages below, which carry all 13 between them. Read it first if you want the shape of the lecture rather than one
  answer.
- **[HTML-to-text extraction](wiki/html-to-text-extraction.md)** — the stage before
  everything else. Why linearizing HTML is *inherently* lossy rather than badly
  tooled, why the extractors are all rule-based, and DCLM's measured comparison:
  trafilatura and resiliparse are close to each other and **Common Crawl's own WET
  text is worse than both**. Also the PDF path — truncation, re-crawling, OCR — and
  why PDFs are worth it.
- **[Quality classifiers](wiki/quality-classifiers.md)** — how a filter is actually
  built. The target/raw framework, KenLM (a generative model of the target) versus
  fastText (a classifier), stochastic keeping, and five worked recipes: fastText
  language ID, OpenMathText's three-mechanism math pipeline, GPT-3's Pareto keep
  rule, LLaMA's use of pages *referenced by* Wikipedia, and phi-1's
  GPT-4-labels-then-random-forest. Ends with the argument that **there is no
  optimal threshold**.
- **[MinHash and LSH](wiki/minhash-and-lsh.md)** — the algorithms, worked through
  with the lecture's own computed values. Why deduplication is quadratic and must
  not be, the characteristic-matrix proof that a MinHash collides with probability
  exactly the Jaccard similarity, and how $b$ bands of $r$ hashes sharpen that into
  a phase transition. Includes the collision-probability table at three (b, r)
  settings, and why the real-world setting n=9000, b=20, r=450 is hunting for
  documents **99.3%** similar.
- **[Post-training data](wiki/post-training-data.md)** — environments, tasks, and
  responses from a teacher. OpenThoughts' three counterintuitive findings — a
  better model is not necessarily a better teacher, sixteen samples per prompt
  beats more sources, answer filtering did not help — and why its headline 1.2M
  examples is really **75,000 questions**.
- **[Agent trajectory data](wiki/agent-trajectory-data.md)** — the four SWE
  datasets, and the infrastructural problem behind all of them: most GitHub repos
  do not run. SWE-smith generates bugs into working code; SWE-Zero observes that
  strong models solve many tasks with no execution at all and builds 300K
  trajectories on that basis; SWE-rebench has a model write the install scripts;
  SWE-ZERO-12M scales it to 12M with a 1.7B generator.
- **[Data mixture selection](wiki/data-mixture-selection.md)** — extended this run,
  and now the page where lectures 9 and 14 **disagree**. Lecture 9 argues a
  small-scale bake-off is sound because composition moves intercepts and not slopes;
  lecture 14 exhibits an effect that genuinely fails to transfer, and fixes it by
  simulated epoching. Also the three baselines (including "vibes"), the epoching
  trap, UniMax's cap, and RegMix.

### Lecture 13 — data: sources and datasets

- **[Lecture 13 — Data I: sources and datasets](wiki/13-data-sources-datasets.md)** —
  the lecture page: why data is the thing model developers will not disclose, the
  four reasons a crawler cannot get the web, copyright in enough depth to read the
  lawsuits, the raw sources, the dataset chronology, and CommonPile's test of
  whether permissive licensing is enough. Carries all 14 of the lecture's own
  figures.
- **[Pre-training datasets](wiki/pretraining-datasets.md)** — the chronology in one
  place, with a table of every named corpus, what it was built from and the size
  the lecture gives it. Also the books lineage (BooksCorpus → Gutenberg → Books3)
  and the web lineage (WebText → C4 → GPT-3 → The Pile → LLaMA → RefinedWeb →
  Dolma → DCLM). Three of these datasets no longer exist, each for a different
  legal reason.
- **[Data filtering](wiki/data-filtering.md)** — the central technical argument:
  rules versus models, held as a real disagreement from 2019 until DCLM produced
  numbers in 2024. Retention rates run from C4's ~11% to DCLM's 1.4%, and
  Nemotron-CC's whole position is that the field pushed that too low. Includes C4's
  curly-brace rule, which silently deleted code from the web before anyone wanted
  code models.
- **[Web crawling](wiki/web-crawling.md)** — live servers to corpus. Dynamic
  content, walled gardens, `robots.txt` as a convention rather than a law, the
  three crawler policies, Common Crawl's scale, and why WARC-vs-WET is a modelling
  decision rather than a file-format detail.
- **[Deduplication](wiki/deduplication.md)** — MinHash, Bloom filters and Jaccard
  similarity, and why ~90% of GitHub by file count is duplicate. Also the point
  where a data pipeline's deduplication and an evaluation's contamination check
  turn out to be the same operation.
- **[Copyright and fair use](wiki/copyright-and-fair-use.md)** — everything is
  copyrighted, so there are exactly two ways to use a work. The four fair-use
  factors, why copyright is "about semantics, not n-gram overlap", and what the
  Anthropic and Meta judgments actually held — training was fair use, acquisition
  by piracy was not, and it was the acquisition that cost $1.5B.
- **[Data licensing and consent](wiki/data-licensing-and-consent.md)** — the four
  independent layers of permission, the sharp post-2023 decline in what sites allow,
  shadow libraries, and the three ways "permissively licensed" fails to mean what it
  says — including why you cannot trust a Hugging Face dataset's license field.
- **[Code data](wiki/code-data.md)** — GitHub, Software Heritage, The Stack and
  Stack v2. The LLVM bridge that transfers from data-rich to data-poor programming
  languages, and how a pull request gets linearized into tokens so a model learns
  the development process rather than only the syntax.
- **[Synthetic data](wiki/synthetic-data.md)** — rewriting data instead of
  discarding it, as Nemotron-CC does; and the unresolved licensing question, where
  the likely legal answer ("probably fine") and the consistent one (CommonPile
  refused it as laundering) point in opposite directions.

### Lecture 12 — evaluation

- **[Lecture 12 — evaluation](wiki/12-evaluation.md)** — the lecture page: the core
  challenge, the four rival answers to "what makes a model good", the five families
  of benchmark in the order the lecture gives them, then realism, validity and the
  methods-versus-models distinction. Also the three questions from the floor, one of
  which fills a real gap the lecture leaves — how a multiple-choice answer is
  actually extracted from a model's output, and how sensitive scores are to it.
- **[Perplexity as an evaluation](wiki/perplexity-evaluation.md)** — the metric this
  course has been optimising since lecture 1, examined as an evaluation. Both sides
  of the argument: *perplexity is all you need* (best achievable is $H(t)$, attained
  iff $p=t$; the lecture calls it more faith than science) against *perplexity is
  more than you need* (it charges for every token, including the ones nobody cares
  about). GPT-2's zero-shot break with in-distribution evaluation, the benchmarks
  that are perplexity in disguise, and why a perplexity leaderboard can be won by a
  distribution that does not sum to one.
- **[Exam benchmarks](wiki/exam-benchmarks.md)** — MMLU, MMLU-Pro, GPQA and
  Humanity's Last Exam in chronological order, because the order is the argument:
  each exists because its predecessor saturated. GPQA's three numbers — experts 65%,
  non-experts with Google 34%, GPT-4 39% — are the clearest statement of what an
  exam benchmark is for. Includes the defence of multiple choice, and the point that
  a saturated benchmark is dead as a leaderboard and alive as a development signal.
- **[Chat benchmarks](wiki/chat-benchmarks.md)** — how to evaluate a response with no
  right answer. Chatbot Arena's ELO and the property that makes it affordable (no
  model has to see every prompt); AlpacaEval's length bias, the leaderboard gaming it
  caused and the regression that fixed it; WildBench's per-prompt checklist. And the
  sharpest question in the lecture: how do you evaluate a *metric*?
- **[Agentic benchmarks](wiki/agentic-benchmarks.md)** — evaluating what a model
  *does*. SWEBench and the executable answer key, TerminalBench, CyBench's
  human-calibrated first-solve times, MLEBench. Why *agent = model + scaffold* is not
  a definition but a warning: an agentic score is a property of the pair, and two
  agents on the same model post different numbers.
- **[Pure reasoning benchmarks](wiki/reasoning-benchmarks.md)** — ARC-AGI, and the one
  trajectory in the course where scaling pre-training did nothing and a change of
  method did everything. Also its constitutional limit: validity comes from being
  100% human-solvable, which means it cannot detect superhuman reasoning.
- **[Safety evaluation](wiki/safety-evaluation.md)** — why there is no crash-test
  rating for AI. HarmBench and AIR-Bench get their lists of harms from two different
  institutions because there is no first principle to derive one from; GCG shows
  safety training is defeasible by optimisation *and transfers to closed models*; and
  the risks relate to capability in opposite directions, which is why safety cannot
  be one number.
- **[Benchmark contamination](wiki/benchmark-contamination.md)** — what foundation
  models destroyed. The old guarantee was a file layout, not an argument, and the
  four routes to rebuilding it — infer overlap from the model via exchangeability,
  reporting norms, fresh evals, private evals — trade off so that none is
  simultaneously outsider-verifiable, durable and public.
- **[Construct validity](wiki/construct-validity.md)** — the lecture's argument, in
  one page: abstract construct → concrete metric, ecological validity (GDPVal,
  MedHELM, Clio, and why realism fights privacy), dataset quality (SWE-Bench
  Verified, broken questions, the agent that scores 38% by returning nothing), the
  four purposes of evaluation, and methods versus models.

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
  [Lecture 11](raw/transcripts/11-scaling-laws-in-the-wild.md),
  [Lecture 12](raw/transcripts/12-evaluation.md),
  [Lecture 13](raw/transcripts/13-data-sources-datasets.md),
  [Lecture 14](raw/transcripts/14-data-filtering-dedup-mixing.md),
  [Lecture 15](raw/transcripts/15-mid-post-training.md),
  [Lecture 16](raw/transcripts/16-post-training-rlvr.md).
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
    [`lecture_07.py`](raw/slides/07-parallelism.md),
    [`lecture_10.py`](raw/slides/10-inference.md),
    [`lecture_12.py`](raw/slides/12-evaluation.md),
    [`lecture_13.py`](raw/slides/13-data-sources-datasets.md) and
    [`lecture_14.py`](raw/slides/14-data-filtering-dedup-mixing.md) are Percy
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
    perfect-overlap assumption, not benchmarks. **Lecture 12 is different again**:
    it computes *nothing at all* — no `@inspect` values, no benchmarks, no sympy, no
    asserts — so every number in it is a claim about a published benchmark rather
    than a measurement. What it has instead is figures: 43 of them, the most in the
    course, and they carry the argument rather than illustrating it.
    **Lecture 13 is the same kind** — it computes nothing either, and every number
    in it is a claim about a published dataset. **Lecture 14 goes back the other
    way, and is the cleanest case in the course**: it has 22 inspected values —
    MurmurHash, exact deduplication, Jaccard, a 100-seed MinHash simulation, LSH
    collision probabilities and the data-mixing epoch arithmetic — and *none* of
    them touches a GPU, a clock or a benchmark, so unlike lectures 2, 6, 7 and 10
    there was nothing machine-dependent to withhold. Every one is reproduced, and
    they reproduce on any machine.
  - [`lecture_03.pdf`](raw/slides/03-architectures.md) (67 pages),
    [`lecture_04.pdf`](raw/slides/04-attention-alternatives.md) (60 pages),
    [`lecture_05.pdf`](raw/slides/05-gpus-tpus.md) (55 pages),
    [`lecture_08.pdf`](raw/slides/08-parallelism-2.md) (73 pages) and
    [`lecture_09.pdf`](raw/slides/09-scaling-laws.md) (57 pages) and
    [`lecture_11.pdf`](raw/slides/11-scaling-laws-in-the-wild.md) (58 pages) and
    [`lecture_15.pdf`](raw/slides/15-mid-post-training.md) (65 pages) and
    [`lecture_16.pdf`](raw/slides/16-post-training-rlvr.md) (61 pages) are
    Tatsunori
    Hashimoto's slide decks, transcribed from the rendered page images, with every
    figure described in prose and every table transcribed cell by cell. Slide
    numbers in all eight are **PDF page numbers**, because none of the decks prints
    any of its own.
    **Lectures 15 and 16 are the two decks to read with the most caution**: at
    the user's instruction both were transcribed by Sonnet readers with **no
    independent figure audit**, and they are the only two page-image decks in
    this KB without one. Both front matters say so and both carry a *Known gaps*
    section. Lecture 15's lists all seventeen passages a reader marked illegible,
    plus three defects in the deck itself including a heading printed "RLFH" for
    RLHF, reproduced rather than corrected; four of its rendered images were read
    back against their descriptions and matched exactly. Lecture 16's readers
    needed no illegibility marks — every dense page resolved after re-rendering
    at 400-500 dpi — and its seven known gaps are all properties of the deck: a
    hidden text-layer string behind slide 8's caption that never renders, three
    quote boxes cropped mid-word by the slide's own edges, one merged table cell,
    and two typos printed on the slides. Two of its rendered images were read
    back and matched, and two of its readers went past looking — RGB-sampling a
    legend's swatches to pin down slide 40's five series, and recovering slide
    60's cropped axis range from an adjacent crop. **Lecture 11's deck is the odd one out**: it is the only deck read
    at Opus rather than Sonnet — a choice made because at 33 words of native text per page
    it is the most figure-dependent deck in the course. It has since had **two figure
    audit passes over 15 of its 58 pages** — 3 clean, 12 dirty, 31 corrections applied —
    and a third was deliberately declined on cost, so its front matter states the
    boundary that remains: 43 of 58 pages carry chart values nobody re-checked. Lectures 4, 5 and 8 are the figure-dependent ones — 102 images
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
  only describe it. **All twelve covered lectures have them**: 32–51
  images each for the six PDF-deck lectures (3, 4, 5, 8, 9, 11), one for every
  figure-bearing page; and 4–33 each for
  the six executable lectures (1, 2, 6, 7, 10, 12), which have no deck, so these are the
  figures the course serves from its own repo. **Lecture 12 is the richest of those
  six, with 33** — it is the most image-dense lecture in the course, and its figures
  (leaderboard screenshots, benchmark example questions, results charts) *are* its
  content rather than a supplement to it. Lecture 10 is next with 22, mostly
  reproduced tables and charts from the papers it discusses. Nine further images in
  lecture 12 are hot-linked by the course to third-party sites and are **not** copied
  here; the slide file records their URLs at the point they appear. Each image sits beside the slide it shows in
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
