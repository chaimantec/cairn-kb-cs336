# The Qwen-VL series

Three multimodal models from one lab across three years, which
[Lecture 17](17-multimodality.md) reads as a **time series** rather than as three
separate systems: same problem, same team, successive answers. "Hopefully I'll go
through this quickly, because you'll hopefully see the pattern" (≈46:12).

The pattern is that the [template](vision-language-models.md) never changes and
everything around it does.

## Qwen-VL (2023)

- **Vision encoder:** OpenCLIP's ViT-bigC (as the deck prints it; the cited paper
  names it ViT-bigG), 14×14 patches.
- **Adapter:** one layer of cross-attention with 2D positional embeddings, mapping
  to a **fixed length of 256** tokens. → [Modality projectors](modality-projectors.md)
- **Special tokens:** `<img>`, `<box>` for bounding boxes, `<ref>` for description
  (≈46:59). These let the model name a region in its output, which is what makes
  grounding possible.

The fixed 256 is flagged as provisional the moment it appears: "this is definitely
not very dynamic, but neither is the vision encoder at this point."

**Three training stages, in which what is frozen changes non-monotonically:**

| Stage | Data | ViT | Adapter | LM |
| --- | --- | --- | --- | --- |
| 1 | large-scale, low quality (5B → 1.4B after cleaning) | **train** | train | frozen |
| 2 | higher-quality, task-specific; higher resolution | train | train | **train** |
| 3 | instruction tuning | **frozen** | train | train |

"So this is a bit different from the LLaVA models, in that they train the vision
encoder" (≈47:44). The stage-1 data table is worth a look for how aggressive the
cleaning is — LAION-en goes from 2B pairs to 280M, a 14% survival rate, and the
whole mixture from 5B to 1.4B.

## Qwen2-VL (2024)

The headline is **dynamic resolution**, which repudiates two earlier decisions at
once: [CLIP](clip.md)'s fixed 336×336 crop and Qwen-VL's own fixed 256 tokens.

> Let's say you have this picture here that might be mapped to 11,000 tokens. This
> tiny picture of an equation might only be mapped to eight tokens. (≈49:16)

An image now costs what it is worth. The mechanism is AnyRes-like: each 224×224
region is encoded with a ViT/14, and every 2×2 block of adjacent patches is merged
into one token to control context length.

*(The arithmetic gives 64 tokens per region; the lecture prints **66**. The
[course material](../raw/slides/17-multimodality.md#qwen2-vl) transcribes it as
printed and declines to explain the difference.)*

Also new:

- **A larger vision encoder** — 675M parameters.
- **MRoPE**, splitting rotary position embedding across time, height and width.
  → [Multimodal RoPE](multimodal-rope.md)
- **Video sampling** at 2 frames/second, capped at 16,384 tokens.
  → [Video understanding](video-understanding.md)

## Qwen3-VL (2025)

The lecturer calls this "kind of a systems paper" and its changes "minor but
potentially important" — but three of the four are worth understanding.

**Interleaved MRoPE.** A fix to Qwen2-VL's own scheme. RoPE's dimensions carry
different frequencies, so allocating the three axes in contiguous blocks means
"maybe all the temporal dimensions are low-frequency and all the height dimensions
are high-frequency." Interleaving — `[t w h t w h …]` instead of
`[t t t t w w w w h h h h]` — gives every axis a share of every band (≈53:57).

**Explicit video timestamps.** Time moves out of the positional encoding and into
the sequence as tokens, so it becomes "something you can directly refer to, like,
what happened after two seconds" (≈55:31).

**Square-root-normalized per-token loss.** Video examples are enormously longer
than text ones, so under a plain per-token loss they dominate the gradient.
Dividing by $\sqrt{\text{length}}$ rather than length damps that without flattening
long examples entirely. → [Modality balancing](modality-balancing.md)

**DeepStack** as the adapter, injecting vision tokens into multiple decoder layers.
→ [Modality projectors](modality-projectors.md)

The backbone is [Qwen3](qwen3.md), "dense and MoE models up to 235B-A22B" — 235B
total parameters with 22B active, in the
[mixture-of-experts](mixture-of-experts.md) notation of lecture 4 — with context
up to **256K**, "which is really important if you're trying to do long video."

**Pre-training is a context-length curriculum:**

| Stage | Objective | Trains | Tokens | Sequence length |
| --- | --- | --- | --- | --- |
| S0 | vision-language alignment | merger only | 67B | 8,192 |
| S1 | multimodal pre-training | all | ~1T | 8,192 |
| S2 | long-context pre-training | all | ~1T | 32,768 |
| S3 | ultra-long-context adaptation | all | 100B | 262,144 |

Long context is bought last and incrementally, and "most of the tokens are in
stages two and three" (≈57:50). Post-training is
[lecture 16](16-post-training-rlvr.md) applied to a multimodal model: SFT on
[long chain-of-thought](long-chain-of-thought.md) data,
[knowledge distillation](reasoning-distillation.md), then RL.

## What the series shows

The lecturer's own summary of three years of change: "from Qwen to Qwen-2 to
Qwen-3, many of these changes were — there's some kind of change there, but the
overall framework remains the same, and mostly it's scaling up, curating more
datasets, noticing that you have to handle long context, and maybe sharpening your
image processing" (≈1:06:30).

And on the reports themselves — the same complaint the
[evaluation](12-evaluation.md) and
[scaling](11-scaling-laws-in-the-wild.md) lectures make: "there's a lot of data
work that happens, not too many details about the data and the data mix in these
later Qwen models."

Qwen3-VL's results table compares it against Gemini 2.5 Pro, GPT-5 and Claude Opus
4.1 across eleven benchmark categories, and "the Qwen models are actually quite
strong" (≈58:38). It is reproduced cell by cell in the
[course material](../raw/slides/17-multimodality.md#qwen3-vl).

→ [Vision-language models](vision-language-models.md) ·
[Lecture 17](17-multimodality.md)
