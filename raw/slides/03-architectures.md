---
title: Lecture 3 — Everything you didn't want to know about LM architecture and hyperparameters (course material)
lecture: 3
instructor: Tatsunori Hashimoto
source_format: slide-deck-pdf
source_file: lecture_03.pdf
source_repo: https://github.com/stanford-cs336/lectures
source_url: https://github.com/stanford-cs336/lectures/blob/main/lecture_03.pdf
pages: 67
method: page-images
numbering: >
  This deck prints NO page numbers on any page. The slide labels below are
  therefore PDF page numbers, not printed slide numbers: "Slide N" means "PDF
  page N", for N = 1..67, one heading per page, in order. Cite them as page
  numbers of lecture_03.pdf. (A page-number scan reported a printed "1" on page
  61; that glyph is the numerator of the MQA arithmetic-intensity fraction, not
  a folio.)
figures: >
  This deck is figure-dense: 50 of its 67 pages carry raster images, 108 in
  total, and several of the most important pages are almost pure image. Every
  figure below was described from the rendered page image. Where a chart's axis
  label or a small number could not be resolved even at high magnification, the
  entry says "not legible" rather than guessing. Tables printed as pasted images
  (which extract as nothing from the text layer) were read off the page by eye
  and transcribed cell by cell.
math: >
  The deck's math is set in CambriaMath and extracts from the text layer as
  scattered fragments with fractions flattened onto one line. All equations
  below were transcribed from the rendered page, not from the text layer.
---

# Lecture 3 — Everything you didn't want to know about LM architecture and hyperparameters (course material)

This is the written content of CS336 Lecture 3, transcribed page by page from
[`lecture_03.pdf`](https://github.com/stanford-cs336/lectures/blob/main/lecture_03.pdf)
(Tatsunori Hashimoto, Stanford CS336, Spring 2026). The deck surveys what the
large open language models have in common architecturally, which choices vary
between them, and what evidence exists for each choice.

Headings are **PDF page numbers** — the deck prints no slide numbers. See the
`numbering` note in the front matter.

## Sections → slide ranges

| Section | Slides |
| --- | --- |
| Title and outline | 1–2 |
| Recap of the transformer, and why there are so many variants | 3–9 |
| Normalization: pre- vs post-norm, LayerNorm vs RMSNorm, bias terms | 10–19 |
| Activations and gated linear units; serial vs parallel layers | 20–29 |
| Position embeddings and RoPE | 30–35 |
| Hyperparameters: feedforward ratio, heads, aspect ratio, vocab, regularization | 36–51 |
| Stability tricks: z-loss, QK-norm, logit soft-capping | 52–56 |
| Attention heads: MQA/GQA, sparse and sliding-window attention | 57–66 |
| Recap and conclusion | 67 |

## Slide 1 — Lecture 3 (title card)

Title card. Large black heading **"Lecture 3"**, and beneath it in blue small-caps
**"Everything you didn't want to know about LM architecture and hyperparameters"**.
Below that, in grey: "CS336" and "Tatsu H". A blue band runs across the foot of
the slide; a matching blue band is the deck's running header on every subsequent
page. No figure other than the coloured bands.

## Slide 2 — Outline and goals

Three bulleted goals, each marked with a blue diamond bullet:

- Quick recap of a modern transformer (what you implement)
- What do most of the large LMs have in common?
- What are common variations to the architecture / training process?

At the bottom, in bold: **"Today's theme:** the best way to learn is hands-on
experience / the second best way is to try to learn from others' experience".

No figures.

## Slide 3 — Starting point: the 'original' transformer

**Left half — figure.** The canonical encoder–decoder block diagram from
"Attention is All You Need", drawn in the paper's own pastel colouring. The
encoder stack (left) reads bottom-to-top: "Inputs" → pink **Input Embedding** →
a $\oplus$ node fed from a sine-wave icon labelled "Positional Encoding" →
orange **Multi-Head Attention** → yellow **Add & Norm** → blue **Feed Forward**
→ yellow **Add & Norm**, the whole block bracketed and labelled "N×". The decoder
stack (right) reads: "Outputs (shifted right)" → pink **Output Embedding** →
$\oplus$ with its own sine-wave "Positional Encoding" → orange **Masked
Multi-Head Attention** → **Add & Norm** → orange **Multi-Head Attention** (with
cross-attention arrows arriving from the top of the encoder stack) → **Add &
Norm** → blue **Feed Forward** → **Add & Norm**, also bracketed "N×". Above the
decoder: purple **Linear** → green **Softmax** → "Output Probabilities".

**Right half — text and equations.**

**Review:** choices in the standard transformer

**Position embedding:** sines and cosines

$$PE_{(pos,2i)} = sin(pos/10000^{2i/d_{\mathrm{model}}})$$
$$PE_{(pos,2i+1)} = cos(pos/10000^{2i/d_{\mathrm{model}}})$$

**FFN:** ReLU

$$\mathrm{FFN}(x) = \max(0, xW_1 + b_1)W_2 + b_2$$

**Norm type:** post-norm, LayerNorm

## Slide 4 — What you implemented – simple, modern variant

**Left — figure 1.** A vertical block diagram of the assignment-1 model, drawn
in the same pastel style. Bottom to top: "Inputs" → pink **Token Embedding** →
grey **Transformer Block** → vertical ellipsis → grey **Transformer Block** →
yellow **Norm** → purple **Linear (Output Embedding)** → green **Softmax** →
"Output Probabilities".

**Middle — figure 2.** A zoom into one Transformer Block, drawn as a rounded grey
box. At the bottom, "Input tensor with shape `(batch_size, seq_len, d_model)`"
enters a vertical residual line. The line branches into a yellow **Norm** →
orange **Causal Multi-Head Self-Attention w/ RoPE**, whose output returns to a
green **Add** node on the residual line. That then branches again into a yellow
**Norm** → blue **Position-Wise Feed-Forward**, returning to a second green
**Add**. The block exits at "Output tensor with shape `(batch_size, seq_len,
d_model)`". Note that both norms sit on the branch, not on the residual line —
this is the pre-norm arrangement.

**Right — text.**

Differences:

- **LayerNorm** is in front of the block
- **Rotary position embeddings (RoPE)**
- FF layers use **SwiGLU**, not ReLU
- Linear layers (and layernorm) have **no bias** (constant) terms

Then, in bold: "Why did we pick these? What should you pick?"

## Slide 5 — How should we think about architectures?

The slide's own text — the title, the line "Lots of architecture. Just in
2024-2025.." — is largely covered by a collage of overlapping screenshots of
2024-era model paper title pages. A blue-highlighted caption band runs across
the middle of the slide: **"Over 19 new *dense* model releases, many of them with
minor architecture tweaks.."**

**Figure — a collage of 14 pasted title-page screenshots**, tilted and
overlapping so that several are partly hidden. The legible ones are:

- "Falcon2-11B Technical Report", with an author block (Quentin Malartic,
  Nilabhra Roy Chowdhury, Ruxandra Cojocaru, Mugariya Farooq, Sanath Narayan,
  Ankit Singh, Mohammed Al-Yafeai, Hamza Alobeidli, Kirill Fedyanin, Reda Alami),
  affiliated in Abu Dhabi, United Arab Emirates.
- A Meta-branded page: "The Llama 3 Herd of Models", "Llama Team, AI @ Meta", with
  the footnote "A detailed contributor list can be found in the appendix of this
  paper."
- "Nemotron-4 340B Technical Report", NVIDIA.
- "Reka Core, Flash, and Edge: A Series of Powerful Multimodal Language Models",
  with a long author list (Aitor Ormazabal, Che Zheng, Cyprien de Masson d'Autume,
  Dani Yogatama, Deyu Fu, Donovan Ong, Eric Chen, Eugenie Lamprecht, Hai Pham,
  Isaac Ong, Kaloyan Aleksiev, Lei Li, Matthew Henderson, Max Bain, Mikel Artetxe,
  Nishant Relan, Piotr Padlewski, Qi Liu, Ren Chen, Samuel Phua, …).
- A "… Technical Report" page whose authors include Harkirat Behl and Sébastien
  Bubeck.
- "InternLM2 Technical Report", with a very long author list beginning "Zheng Cai,
  Maosong Cao, Haojiong Chen, Kai Chen, Keyu Chen, …" and affiliations "Shanghai
  AI Laboratory / SenseTime Group / The Chinese University of Hong Kong / Fudan
  University", contact `internlm@pjlab.org.cn`.
- A Qwen page: a block author list beginning "An Yang, Baosong Yang, Binyuan Hui,
  Bo Zheng, Bowen Yu, Chang Zhou, Chengpeng Li, …" and ending "…, Zhifang Guo, and
  Zhihao Fan", signed "Qwen Team, Alibaba Group".
- "Phi-3 Tech… Language…", Microsoft.
- A Gemma page: "… at a Practical Size", "Gemma Team, Google DeepMind", dated
  2024-06-27.
- A page whose visible fragment reads "… of a Small Language Model", with authors
  including Gabriel Martín Blázquez, Guilherme Penedo, Agustín Piqueres Lajarín,
  Vaibhav Srivastav, Clémentine Fourrier, Ben Burtenshaw, Cyril Zakka, Mathieu
  Morlon.
- Several further title pages are visible only as fragments and cannot be
  identified.

## Slide 6 — How should we think about architectures?

Same title as slide 5, and again largely covered by a collage. The one line of
slide text visible beneath the images is "There can't be that many LLMs released
this year right?" (the title and this line are both partly obscured by the pasted
screenshots).

**Figure — a collage of 18 pasted screenshots** of 2025–2026 model announcements
and papers, overlapping and tilted. The legible ones are:

- A black OpenAI announcement page dated "August 5, 2025 · Release · Product":
  **"Introducing gpt-oss"**, subtitle "gpt-oss-120b and gpt-oss-20b push the
  frontier of open-weight reasoning model…", with buttons "Explore on Hugging
  Face" and "Read model card".
- A Meta page: "Llama 4: Leading Multimodal In…", listing three cards — **Llama 4
  Behemoth** (288B active parameters, 16 experts, 2T total parameters; "The most
  intelligent teacher model for distillation"; marked Preview), **Llama 4
  Maverick** (17B active parameters, 128 experts, 400B total parameters; "Native
  multimodal with 1M context length"; Available), and **Llama 4 Scout** (17B
  active parameters, 16 experts, 109B total parameters; "Industry leading 10M
  context length / Optimized inference"; Available).
- "MiniMax-M1: Scaling Test-Time … Efficiently with Lightning Attent…", MiniMax.
- A DeepSeek-branded page: "DeepSeek-V3.2: Pushing the Frontier of Open Large
  Language Models", DeepSeek-AI, `research@deepseek.com`.
- An IBM page: "IBM Gra… efficien… hybrid models for enterprise", published
  02 October 2025.
- A Mistral AI logo graphic.
- A Liquid AI page ("Liquid", with a nav bar reading "Try LFM · Docs · LEAP ·
  Disco…").
- "MiniMax M2 & Agent: Ingenious in Simplicity", with buttons "Access API",
  "Coding Plan", "Try Agent Now".
- "ERNIE 4.5 Technical Repo…".
- "Kimi K2: Open Agentic Intelligence", "Technical Report of Kimi K2", Kimi Team.
- A z.ai page dated "2025-12-22 · Research": "GLM-4.7: Advancing the Coding
  Capability", with link chips "Try it at Z.ai", "Call it at Z.ai", "Z.ai Coding
  Plan", "GitHub", "HuggingFace", "Tech Report".
- An NVIDIA page: "NVIDIA Nemotron 3: Efficient and …", whose body text mentions
  "…models—Nano, Super, and Ultra. These models deliver strong …capabilities. The
  Nemotron 3 family uses a Mixture-of-Experts …to provide best-in-class throughput
  and context lengths of …dels are trained with NVFP4 and incorporate LatentMoE, a
  …ity. The two larger models also include MTP layers for faster…".
- "Intern-S1: A Scientific Multimodal Foundation Model", Intern-S1 Team, Shanghai
  AI Laboratory.
- "Step-3 is Large yet Affordable: Model-system Co-design for Cost-effective
  Decoding", StepFun Inc.
- A page fragment about "…foundation models—Mixture-of-Experts …taining 424B total
  …MoE architecture,".
- A dark page with a scattered-dots graphic, and a page dated "2025-05-15"; both
  otherwise not legible.

## Slide 7 — Let's look at the data (on dense architectures)

Text: "Learn from the many other models (and papers) out there", and on the right,
in bold, "We will talk through many major architecture and hyperparameter
variants." followed by three bullets:

- What do all these models have in common?
- What parts vary?
- What can we learn from this?

**Figure — a screenshot of the instructor's own dense-model database**, rendered
as a dark-themed Notion-style table. This is the same table as on slide 9, shown
here at a smaller scale, and it is the empirical backbone of the whole lecture.
Its columns, left to right, are: **Name**, **Year**, **Vocab count**, **Norm**,
**Parallel Layer**, a checkbox column truncated in the header to "**P.**"
(slide 29 shows the same database with this column widened, where it reads
**Pre-norm**), **Position embed…**, **Activati…**, **Other tricks**, **MLP factor**,
**num_la…** (number of layers) and **model_dim**; a further numeric column begins
at the right edge and is cut off by the screenshot. Categorical cells are drawn
as coloured chips (RMSNorm blue, LayerNorm grey, Serial gold, Parallel blue, RoPE
red, Relative gold, Absolute brown, AliBi purple, SwiGLU tan, GeGLU red, GeLU
grey, ReLU gold, SqRelu magenta, Z-loss red, QK-norm gold, Logit soft capping
purple, Pre+post norm pink, Hybrid variants pink/green/orange).

The rows, in the order shown (one row above `mT5` is scrolled off the top and only
its lower edge is visible):

| Name | Year | Vocab count | Norm | Parallel Layer | Pre-norm | Position embed. | Activation | Other tricks | MLP factor | num_layers | model_dim |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| mT5 | 2020 | 250000 | RMSNorm | Serial | ✓ | Relative | GeGLU | | 2.5 | 24 | 4096 |
| GPT3 (175B) | 2020 | 50257 | LayerNorm | Serial | ✓ | Absolute | GeLU | | 4 | 96 | 12288 |
| GPTJ | 2021 | 50257 | LayerNorm | Parallel | ✓ | RoPE | GeLU | | | 28 | 4096 |
| LaMDA | 2021 | 32000 | | | ✓ | Relative | GeGLU | | 8 | 64 | 8192 |
| Anthropic LM (not claude) | 2021 | 65536 | | | ✓ | | | | 4 | 64 | 8192 |
| Gopher (280B) | 2021 | 32000 | RMSNorm | Serial | ✓ | Relative | ReLU | | 4 | 80 | 16384 |
| GPT-NeoX | 2022 | 50257 | LayerNorm | Parallel | ✓ | RoPE | GeLU | | 4 | 44 | 6144 |
| BLOOM (175B) | 2022 | 250680 | LayerNorm | Serial | ✓ | AliBi | GeLU | | 4 | 70 | 14336 |
| OPT (175B) | 2022 | 50272 | LayerNorm | Serial | ✓ | Absolute | ReLU | | 4 | 96 | 12288 |
| PaLM (540B) | 2022 | 256000 | RMSNorm | Parallel | ✓ | RoPE | SwiGLU | Z-loss | 4 | 118 | 18432 |
| Chinchilla | 2022 | 32000 | RMSNorm | Serial | ✓ | Relative | ReLU | | 4 | 80 | 8192 |
| Baichuan 2 | 2023 | 125696 | RMSNorm | Serial | ✓ | AliBi | SwiGLU | Z-loss | 2.68 | 32 | 4096 |
| Mistral (7B) | 2023 | 32000 | RMSNorm | Serial | ✓ | RoPE | SwiGLU | | 3.5 | 32 | 4096 |
| LLaMA2 (70B) | 2023 | 32000 | RMSNorm | Serial | ✓ | RoPE | SwiGLU | | 3.5 | 80 | 8192 |
| LLaMA (65B) | 2023 | 32000 | RMSNorm | Serial | ✓ | RoPE | SwiGLU | QK-norm | 2.6875 | 80 | 8192 |
| Olmo 2 | 2024 | 100000 | RMSNorm | Serial | ☐ | RoPE | SwiGLU | Z-loss, QK-norm | 2.6875 | 32 | 4096 |
| Gemma 2 (27B) | 2024 | 256128 | RMSNorm | Serial | ✓ | RoPE | GeGLU | Logit soft capping | 8 | 46 | 4608 |
| Nemotron-4 (340B) | 2024 | 256000 | LayerNorm | Serial | ✓ | RoPE | SqRelu | | 4 | 96 | 18432 |
| Qwen 2 (72b) – same for 2.5 | 2024 | 152064 | RMSNorm | Serial | ✓ | RoPE | SwiGLU | | 3.609 | 80 | 8192 |
| Falcon 2 11B | 2024 | 65024 | LayerNorm | Parallel | ✓ | RoPE | GeLU | Z-loss | 4 | 60 | 4096 |
| Phi3 (small) – same for phi4 | 2024 | 100352 | RMSNorm | Serial | ✓ | RoPE | GeGLU | | 3.5 | 32 | 4096 |
| Llama 3 (70B) | 2024 | 128000 | RMSNorm | Serial | ✓ | RoPE | SwiGLU | | 3.5 | 80 | 8192 |
| Reka Flash | 2024 | 100000 | RMSNorm | Serial | ✓ | RoPE | SwiGLU | | | | |
| Command R+ | 2024 | 256000 | LayerNorm | Parallel | ✓ | RoPE | SwiGLU | | 2.75 | 64 | 12288 |
| OLMo | 2024 | 50304 | RMSNorm | Serial | ✓ | RoPE | SwiGLU | | 2.6875 | 32 | 4096 |
| Qwen (14B) | 2024 | 152064 | RMSNorm | Serial | ✓ | RoPE | SwiGLU | | 2.675 | 40 | 5120 |
| DeepSeek (67B) | 2024 | 100000 | RMSNorm | Serial | ✓ | RoPE | SwiGLU | | 2.6875 | 95 | 8192 |
| Yi (34B) | 2024 | 64000 | RMSNorm | Serial | ✓ | RoPE | SwiGLU | | 2.857142 | 60 | 7168 |
| Marin 8B | 2025 | 128256 | RMSNorm | Serial | ✓ | RoPE | SwiGLU | | 3.5 | 32 | 4096 |
| OLMo 3 (7B) | 2025 | 100278 | RMSNorm | Serial | ☐ | Hybrid (SWA+Full) | SwiGLU | QK-norm, Z-loss | 2.6875 | 32 | 4096 |
| Qwen 3 (8B) | 2025 | 151936 | RMSNorm | Serial | ✓ | RoPE | SwiGLU | QK-norm | 3 | 36 | 4096 |
| Command A | 2025 | 255000 | LayerNorm | Parallel | ✓ | Hybrid (RoPE+No…) | SwiGLU | | | | |
| Gemma 3 | 2025 | 262000 | RMSNorm | Serial | ☐ | RoPE | GeGLU | Pre+post norm, QK-norm | 4 | 62 | 5376 |
| SmolLM2 (1.7B) | 2025 | 49152 | RMSNorm | Serial | ✓ | RoPE | SwiGLU | | 4 | 24 | 2048 |
| LFM 2.5 (1.2B) | 2026 | 65536 | RMSNorm | Serial | ✓ | Hybrid (Conv+Full) | SwiGLU | QK-norm | 4 | 16 | 2048 |
| Gemma 4 E4B (8B) | 2026 | 262144 | RMSNorm | Serial | ☐ | Hybrid (SWA+Full) | GeGLU | Logit soft capping, p-Rope, QK-norm | 4 | 42 | 2560 |
| Ministral 3 (8B) | 2026 | 131072 | RMSNorm | Serial | ✓ | Hybrid (SWA+Full) | SwiGLU | | 3.5 | 34 | 4096 |

Blank cells above are blank in the screenshot. The four rows with an unchecked
pre-norm box are Olmo 2, OLMo 3 (7B), Gemma 3 and Gemma 4 E4B (8B) — the models the
deck later identifies as using non-residual post-norm or pre+post norm.

## Slide 8 — What are we going to cover?

A plain agenda slide, no figures.

**Common architecture variations**

- Activations, FFN
- Attention variants
- Position embeddings

**Hyperparameters that (do or don't) matter**

- What is ff_dim? Do multi_head dims always sum to model_dim?
- How many vocab elements?

**Stability tricks**

## Slide 9 — Architecture variations..

Text: "Let's think about the core architecture piece", and on the right, in bold,
"High level view:" followed by two bullets:

- Dominance of 'LLaMA-like' architectures
- Trends over the years (QK-norm, Hybrid attention)

**Figure — the same dense-model database screenshot as slide 7**, here enlarged so
that the model names and chips are easier to read; the right-hand numeric columns
(`num_layers`, `model_dim`) are correspondingly pushed off the right edge of the
image. Row order, column set and cell values are identical to the table
transcribed under slide 7. The visual point the bullets make is that the lower
(more recent) two-thirds of the Norm column is almost solid blue "RMSNorm", the
Parallel Layer column is almost solid gold "Serial", the Position embed. column
turns solid red "RoPE" from 2021 onward and then shifts to pink/green "Hybrid"
chips in 2025–2026, the Activation column is dominated by tan "SwiGLU", and the
"Other tricks" column fills with "QK-norm" only in the 2024+ rows.

## Slide 10 — Pre-vs-post norm

Subtitle: "The one thing *everyone* agrees on (in 2024)".

**Left — figure (from Xiong 2020, credited beneath the diagrams as "Figure from
Xiong 2020").** Two side-by-side block diagrams of one transformer layer, each
drawn with a thick grey vertical arrow as the residual stream running from $x_l$
at the bottom to $x_{l+1}$ at the top.

- *Left diagram (post-LN).* Going up the residual stream: an "addition" box fed
  from a yellow **Multi-Head Attention**, then a green **Layer Norm** placed
  directly **on** the residual stream, then another "addition" box fed from a blue
  **FFN**, then a second green **Layer Norm** on the stream. The Layer Norms
  interrupt the residual path.
- *Right diagram (pre-LN).* Going up the residual stream: a green **Layer Norm**
  sits on the **branch** feeding the yellow **Multi-Head Attention**, whose output
  returns to an "addition" box on the stream; then a second green **Layer Norm**
  on the branch feeding the blue **FFN**, returning to another "addition". The
  residual path itself is unbroken.

**Right — figure: a two-column equation box, also from Xiong 2020**, headed
"Post-LN Transformer" and "Pre-LN Transformer". The post-LN column reads:

$$x^{post,1}_{l,i} = \mathrm{MultiHeadAtt}(x^{post}_{l,i}, [x^{post}_{l,1},\cdots,x^{post}_{l,n}])$$
$$x^{post,2}_{l,i} = x^{post}_{l,i} + x^{post,1}_{l,i}$$
$$x^{post,3}_{l,i} = \mathrm{LayerNorm}(x^{post,2}_{l,i})$$
$$x^{post,4}_{l,i} = \mathrm{ReLU}(x^{post,3}_{l,i}W^{1,l} + b^{1,l})W^{2,l} + b^{2,l}$$
$$x^{post,5}_{l,i} = x^{post,3}_{l,i} + x^{post,4}_{l,i}$$
$$x^{post}_{l+1,i} = \mathrm{LayerNorm}(x^{post,5}_{l,i})$$

The pre-LN column reads:

$$x^{pre,1}_{l,i} = \mathrm{LayerNorm}(x^{pre}_{l,i})$$
$$x^{pre,2}_{l,i} = \mathrm{MultiHeadAtt}(x^{pre,1}_{l,i}, [x^{pre,1}_{l,1},\cdots,x^{pre,1}_{l,n}])$$
$$x^{pre,3}_{l,i} = x^{pre}_{l,i} + x^{pre,2}_{l,i}$$
$$x^{pre,4}_{l,i} = \mathrm{LayerNorm}(x^{pre,3}_{l,i})$$
$$x^{pre,5}_{l,i} = \mathrm{ReLU}(x^{pre,4}_{l,i}W^{1,l} + b^{1,l})W^{2,l} + b^{2,l}$$
$$x^{pre}_{l+1,i} = x^{pre,3}_{l,i} + x^{pre,5}_{l,i}$$

with a final line spanning the box: "Final LayerNorm: $x^{pre}_{Final,i} \leftarrow \mathrm{LayerNorm}(x^{pre}_{L+1,i})$".

Slide text under the right figure: "Set up LayerNorm so that it doesn't affect the
main residual signal path (on the left)".

At the foot, in bold: "Almost all modern LMs use pre-norm (but BERT was
post-norm)", and beneath it "(One somewhat funny exception – OPT350M. I don't know
why this is post-norm)".

## Slide 11 — Pre-vs-post-norm, the data

The slide carries almost no text of its own — only the title and two credits,
"Salazar and Ngyuen 2019" (bottom left) and "Figure from Xiong 2020" (bottom
right). The three pasted charts are the slide.

**Figure 1 (left, from Salazar and Nguyen 2019) — "English-Vietnamese development
BLEU".** A line chart on a light-lavender panel. The x-axis is "epochs", ticked at
20, 40, 60, 80, 100. The y-axis is "Dev BLEU", ticked from 18 to 30 in steps of 2.
There are **five** series, all of which rise steeply over the first ~15 epochs and
then flatten:

- *Orange dash-dot-dot*, "PreNorm+ScaleNorm+FixNorm+NoWarmup": climbs from about
  20 at the left edge to roughly 26 by epoch 20, ~28.3 by epoch 40, ~28.7 by epoch
  60, and stays around 28.7–29 through epoch 100. It is the topmost curve over the
  second half of the run.
- *Blue solid*, "PreNorm+ScaleNorm+FixNorm": tracks the orange curve closely — about
  26 at epoch 20, ~28 at epoch 40, ~28.5 at epoch 60, oscillating around 28.3–29
  and ending near 28.3 just before epoch 100.
- *Green dashed*, "PreNorm+LayerNorm+FixNorm": about 26.2 at epoch 20, ~27.8 at
  epoch 40, ~28.1 at epoch 60, ending around 28.3.
- *Red dash-dot*, "PreNorm+LayerNorm": about 26 at epoch 20, ~27.8 at epoch 40,
  ~28.2 at epoch 60, ending around 28.1.
- *Purple dotted*, "PostNorm+LayerNorm": visibly the lowest curve throughout —
  about 24.7 at epoch 20, ~26.5 at epoch 40, ~27.5 at epoch 60, ~28 by epoch 80,
  and it terminates near epoch 86 at about 28.

**Figure 2 (top right, from Xiong 2020) — panel "(a) Validation Loss (IWSLT)".**
x-axis "Epochs", odd ticks 1 through 15; y-axis "Validation Loss", ticked 4
through 8. **Four** series:

- *Blue dotted with × markers*, "Post-LN (RAdam w/o warm-up)": starts ~7.9 at
  epoch 1, ~5.9 at epoch 3, ~4.9 at epoch 5, ~4.5 at epoch 7, ~4.35 at epoch 9,
  flattening to about 4.2 by epoch 15 — the highest (worst) curve throughout.
- *Orange solid with × markers*, "Pre-LN (RAdam w/o warm-up)": ~6.2 at epoch 1,
  ~4.7 at epoch 3, ~4.3 at epoch 5, ~4.15 at epoch 7, ~4.0 by epoch 9, flat at
  ~4.0 to epoch 15.
- *Green dotted with round markers*, "Post-LN (Adam w/ warm-up)": ~8.0 at epoch 1,
  ~5.6 at epoch 3, ~4.6 at epoch 5, ~4.3 at epoch 7, ~4.2 at epoch 9, converging
  onto ~4.0 by about epoch 12.
- *Red solid with round markers*, "Pre-LN (Adam w/o warm-up)": ~6.4 at epoch 1,
  ~4.75 at epoch 3, ~4.35 at epoch 5, ~4.15 at epoch 7, ~4.0 at epoch 9, flat at
  ~4.0 to epoch 15. It sits essentially on top of the orange curve.

**Figure 3 (top far right, from Xiong 2020) — panel "(b) BLEU (IWSLT)".** Same
four series and the same legend as figure 2. x-axis "Epochs", odd ticks 1–15;
y-axis "BLEU", ticked 0, 10, 20, 30.

- *Blue dotted ×*, Post-LN (RAdam w/o warm-up): ~2 at epoch 1, ~7 at epoch 2,
  ~16 at epoch 3, ~21.5 at epoch 4, ~24.6 at epoch 5, ~28.7 at epoch 7, ~30.2 at
  epoch 9, ~31.5 at epoch 11, ~32 at epoch 15 — lowest throughout.
- *Orange solid ×*, Pre-LN (RAdam w/o warm-up): ~10.7 at epoch 1, ~22 at epoch 2,
  ~25.8 at epoch 3, ~28.6 at epoch 4, ~30 at epoch 5, ~31.5 at epoch 7, ~33 at
  epoch 9, plateauing at ~33.3.
- *Green dotted ○*, Post-LN (Adam w/ warm-up): ~1.7 at epoch 1, ~9 at epoch 2,
  ~17.5 at epoch 3, ~23.5 at epoch 4, ~27 at epoch 5, ~30.7 at epoch 7, ~31.3 at
  epoch 9, ~32.9 at epoch 12, ~32.3 at epoch 15.
- *Red solid ○*, Pre-LN (Adam w/o warm-up): ~11.7 at epoch 1, ~21 at epoch 2,
  ~25.8 at epoch 3, ~28.7 at epoch 4, ~29.9 at epoch 5, ~31.5 at epoch 7, ~33 at
  epoch 9, plateauing at ~33.4 — highest throughout, essentially coincident with
  the orange curve.

**Figure 4 (bottom middle, from Xiong 2020) — panel "(a) Validation Loss on
BERT".** x-axis "Pre-training Steps (Thousands)", ticked 100, 300, 500, 700, 900;
y-axis "Validation Loss", ticked 1.6, 1.7, 1.8, 1.9, 2.0. **Two** series:

- *Blue dotted*, "Post-LN": starts above 2.05 at 100k steps, ~1.86 at 300k, ~1.755
  at 500k, ~1.68 at 700k, ~1.645 at 900k, ending near 1.64 at just over 1000k.
- *Orange solid*, "Pre-LN": ~1.92 at 100k, ~1.785 at 300k, ~1.70 at 500k, ~1.645
  at 700k, terminating at about 1.62 near 800k. It lies below the Post-LN curve
  everywhere it is plotted.

## Slide 12 — Pre-vs-post norm, explanations?

Two labelled figures side by side, with a two-line conclusion beneath.

**Left — "Gradient attenuation [Xiong 2020]".** A grouped bar chart, captioned
underneath "(a) $W^1$ in the FFN sub-layers". The x-axis is "Layer" with six
groups, 1 through 6; the y-axis is "Gradient Expectation", ticked 0.0, 0.5, 1.0
(the plot area extends a little above 1.3). **Three** series, in legend order:

- *Blue*, "Pre-LN (init)": roughly flat across depth — about 0.22 at layer 1, 0.23
  at layer 2, 0.22 at layer 3, 0.18 at layer 4, 0.17 at layer 5, 0.18 at layer 6.
- *Orange*, "Post-LN (init)": grows sharply with depth — about 0.06 at layer 1,
  0.12 at layer 2, 0.27 at layer 3, 0.37 at layer 4, 0.76 at layer 5, and about
  1.3 at layer 6.
- *Green*, "Post-LN (after warm-up)": essentially zero at every layer; the bars are
  slivers barely above the axis, growing very slightly with depth.

**Right — "Gradient spikes [Salazar and Ngyuen]".** A line chart titled "Gradient
global norm" on a light-lavender panel. The x-axis is "iteration (x100)", ticked
0, 200, 400, 600, 800, 1000, 1200; the y-axis is "Global norm (log scale)", ticked
$-0.5$ to $3.5$ in steps of $0.5$. **Four** series, in legend order:

- *Purple dotted*, "PostNorm+LayerNorm": settles around a baseline of about $0.0$
  and throws frequent tall spikes throughout the run, many reaching 1.0–1.8 and
  the largest reaching about 2.85 near iteration 1150 and about 2.2 near 1200. It
  is both the highest-baseline and the spikiest series.
- *Orange dash-dot-dot*, "PreNorm+ScaleNorm+FixNorm+NoWarmup": starts near 1.2,
  falls quickly to a baseline around $-0.25$, with occasional spikes to about
  1.1–1.2.
- *Blue solid*, "PreNorm+ScaleNorm+FixNorm": starts near 2.3, falls to a baseline
  around $-0.4$, with periodic spikes reaching roughly 0.5–1.2.
- *Red dash-dot*, "PreNorm+LayerNorm": the lowest baseline, around $-0.6$, with
  occasional spikes to about 0.8–1.0.

Beneath the figures, in bold: "**Original stated advantage** – removing warmup.
**Today** – stability and larger LRs for large networks".

## Slide 13 — New things – 'double' norm or non-residual postnorm

Text: "If putting LayerNorms in residual streams is bad.. Why not post-norm outside
the stream?"

**Figure — two block diagrams side by side**, in the same style as slide 10, each
with a thick grey residual arrow from $x_l$ up to $x_{l+1}$.

- *Left diagram.* Reading up the residual stream: an "addition" box fed by a yellow
  **Multi-Head Attention** hanging off the stream; then a green **Layer Norm**
  placed **on** the residual stream; then another "addition" box fed by a blue
  **FFN**; then a second green **Layer Norm** on the stream. (This is ordinary
  post-norm, shown for comparison.)
- *Right diagram.* Reading up: a green **Layer Norm** on the branch, feeding a
  yellow **Multi-Head Attention**, whose output passes through a *second* green
  **Layer Norm** before returning to the "addition" on the residual stream; then
  the same sandwich again — a green **Layer Norm**, a blue **FFN**, and another
  green **Layer Norm** — before the second "addition". The residual stream itself
  carries no norm at any point. This is the 'double norm' / non-residual post-norm
  arrangement.

At the foot, in bold: "**Recent models:** Grok, Gemma 2. Olmo 2 *only* does
non-residual post norm".

## Slide 14 — LayerNorm vs RMSNorm

Two rows of text-plus-equation on the left, with a "Notable models:" list opposite
each on the right.

Top: "Original transformer: **LayerNorm** – normalizes the mean and variance across
$d_{model}$"

$$y = \frac{x - \mathrm{E}[x]}{\sqrt{\mathrm{Var}[x] + \epsilon}} * \gamma + \beta$$

**Notable models:** GPT3/2/1, OPT, GPT-J, BLOOM

Bottom: "Many modern LMs: **RMSNorm** – does not subtract mean or add a bias term"

$$y = \frac{x}{\sqrt{\left\lVert x \right\rVert_2^2 + \varepsilon}} * \gamma$$

**Notable models:** LLaMA-family, PaLM, Chinchilla, T5

The only image on the page is the blue header band; there is no figure.

## Slide 15 — Why RMSNorm?

**Modern explanation** – it's faster (and just as good).

- **Fewer operations** (no mean calculation)
- **Fewer parameters** (no bias term to store)

The LayerNorm formula is repeated on the right of those bullets:

$$y = \frac{x - \mathrm{E}[x]}{\sqrt{\mathrm{Var}[x] + \epsilon}} * \gamma + \beta$$

**Does this explanation make sense?**

**Figure — a small pasted table** (from Ivanov et al 2023; the credit
"[Ivanov et al 2023]" is printed in the bottom-right corner of the slide). The
top of the table is clipped by the paste, so one header row is only half visible.
The legible content is:

| Operator class | % flop |
| --- | --- |
| △ Tensor contraction | 99.80 |
| ☐ Stat. normalization | 0.17 |
| ○ Element-wise | 0.03 |

Beneath the table, the slide's own caption: "Matrix multiplies are the *vast*
majority of FLOPs (and memory)".

## Slide 16 — Why RMSNorm (2)

**Important lesson:** FLOPS are not runtime! (we will discuss this in far more
detail later)

**Figure 1 (left) — the same Ivanov et al 2023 table, now with a second numeric
column.** Again the very top of the paste is clipped.

| Operator class | % flop | % Runtime |
| --- | --- | --- |
| △ Tensor contraction | 99.80 | 61.0 |
| ☐ Stat. normalization | 0.17 | 25.5 |
| ○ Element-wise | 0.03 | 13.5 |

Slide caption under it: "RMSNorm can still matter due to the importance of *data
movement*".

**Figure 2 (right) — a vertical dataflow diagram, also from Ivanov et al 2023.**
An input arrow labelled "X" comes down from the top, with a branch splitting off
to the left (the residual connection). The main path then passes through four
coloured boxes, each annotated with a black tag on its left (FLOPs) and a white
box on its right (FLOP-to-memory ratio):

- a pale-peach **MHA** box, tagged **43G** on the left and **153** on the right;
- an orange **Dropout** box, tagged **4M** and **1/3**, with a small circle marker
  (element-wise) at its lower left;
- a small orange **+** (addition) node where the residual branch rejoins, tagged
  **4M** and **1/3**, also with a circle marker;
- an amber **LayerNorm** box, tagged **29M** and **3.5**, with a small square
  marker (statistical normalization) at its lower left.

Below the LayerNorm a new branch splits off and the arrow continues downward. A
partially cut-off legend at the top right shows the three shape codes: △ "Ten…"
(tensor contraction), ☐ "No…" (normalization), ○ "Ele…" (element-wise).

Slide annotations printed under the diagram: "Left top ("43G") is FLOPS" and
"Right top ("153") is the FLOP-to-memory ratio". Credit at bottom right:
"[Ivanov et al 2023]".

## Slide 17 — RMSNorm - validation

Text: "**RMSNorm** runtime (and surprisingly, perf) gains have been seen in
papers". Credit at the bottom right: "Narang et al 2020".

**Figure — a pasted results table** from Narang et al 2020. Columns: Model,
Params, Ops, Step/s, Early loss, Final loss, SGLUE, XSum, WebQ, and (after a
vertical rule) WMT EnDe. The first row is a baseline separated by a rule from the
five variants below it. Bold marks the best value in a column.

| Model | Params | Ops | Step/s | Early loss | Final loss | SGLUE | XSum | WebQ | WMT EnDe |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Vanilla Transformer | 223M | 11.1T | 3.50 | 2.182 ± 0.005 | 1.838 | 71.66 | 17.78 | 23.02 | 26.62 |
| RMS Norm | 223M | 11.1T | 3.68 | 2.167 ± 0.008 | **1.821** | **75.45** | **17.94** | **24.07** | **27.14** |
| Rezero | 223M | 11.1T | 3.51 | 2.262 ± 0.003 | 1.939 | 61.69 | 15.64 | 20.90 | 26.37 |
| Rezero + LayerNorm | 223M | 11.1T | 3.26 | 2.223 ± 0.006 | 1.858 | 70.42 | 17.58 | 23.02 | 26.29 |
| Rezero + RMS Norm | 223M | 11.1T | 3.34 | 2.221 ± 0.009 | 1.875 | 70.33 | 17.32 | 23.02 | 26.19 |
| Fixup | 223M | 11.1T | 2.95 | 2.382 ± 0.012 | 2.067 | 58.56 | 14.42 | 23.02 | 26.31 |

RMS Norm is the only row whose values are bolded, and it is both faster
(3.68 steps/s vs the vanilla 3.50) and better on every quality column.

## Slide 18 — More generally: dropping bias terms

**Most modern transformers don't have bias terms.**

Original Transformer:

$$\mathrm{FFN}(x) = \max(0, xW_1 + b_1)W_2 + b_2$$

Most implementations (if they're not gated):

$$FFN(x) = \sigma(xW_1)W_2$$

**Reasons:** memory (similar to RMSnorm) and optimization stability

No figure on this page.

## Slide 19 — LayerNorm: recap

A bulleted summary slide with no figures.

- Basically everyone does non-residual norm (often prenorm)
  - Intuition – keep the good parts of residual connections
  - Observations – nicer gradient propagation, fewer spike
  - Some people add a second norm outside the residual stream
- Most people do RMSnorm
  - In practice, works as well as LayerNorm
  - But, has fewer parameters to move around, which saves on wallclock time
  - People more generally drop bias terms since the compute/param tradeoffs are
    not great.

## Slide 20 — Activations

A three-line text slide, no figures.

**A whole zoo of activations** ..

In large type, centred: "ReLU, GeLU, Swish, ELU, GLU, GeGLU, ReGLU, SeLU, SwiGLU,
LiGLU"

In bold: "What are these things? What do people use? Does it matter?"

## Slide 21 — A few of the common activations

Three rows: activation name and formula on the left, a small plot in the middle
(for the first two), and a "Notable models:" list on the right.

**ReLU**

$$FF(x) = \max(0, xW_1)\,W_2$$

*Figure 1 (middle of the ReLU row) — a matplotlib line plot titled "ReLU()".*
x-axis "Input", ticked $-6, -4, -2, 0, 2, 4, 6$; y-axis "Output", ticked $-6, -4,
-2, 0, 2, 4, 6$. **One** blue series: flat at exactly 0 for all inputs below 0,
then rising as a straight line of slope 1 through the origin, reaching about 7 at
input 7.

**Notable models:** Original transformer, T5, Gopher, Chinchilla, OPT

**GeLU**

$$FF(x) = \mathrm{GELU}(xW_1)W_2$$
$$GELU(x) \coloneqq x\,\Phi(x)$$

*Figure 2 (middle of the GeLU row) — a matplotlib line plot titled
"GELU(approximate='none')".* Same axes and ranges as the ReLU plot ("Input" on x,
"Output" on y, both $-6$ to $6$). **One** blue series: essentially 0 for inputs
below about $-2$, dipping slightly negative (to roughly $-0.17$) around input
$-0.75$, passing through the origin, and then curving smoothly up onto the
slope-1 line, reaching about 2 at input 2 and about 7 at input 7. The difference
from ReLU is the smooth knee and the small negative lobe.

**Notable models:** GPT1/2/3, GPTJ, GPT-Neox, BLOOM

**SwiGLU / GeGLU (next slide..)** — no plot for this row.

**Notable models:** Llama, PaLM, T5 v1.1, *most models post 2023*

## Slide 22 — Gated activations (*GLU)

Text and equations only; no figure. Parts of the formulas are printed in red to
mark what changes.

GLUs modify the 'first part' of a FF layer

$$FF(x) = \color{red}{\max(0, xW_1)}\,W_2$$

Instead of a linear + ReLU, augment the above with an (entrywise) linear term

$$\color{red}{\max(0, xW_1)} \rightarrow \max(0, xW_1) \otimes (xV)$$

This gives the gated variant (ReGLU) – note that we have an extra parameter (V)

$$\mathrm{FF}_{\mathrm{ReGLU}}(x) = (\max(0, xW_1) \otimes xV)\,W_2$$

## Slide 23 — Gated variants of standard FF layers

Two definitions on the left with "Notable models:" lists opposite. No figure.

**GeGLU**

$$\mathrm{FFN}_{\mathrm{GEGLU}}(x, W, V, W_2) = (\mathrm{GELU}(xW) \otimes xV)W_2$$

**Notable models:** T5 v1.1, mT5, LaMDA, Phi3, Gemma 2, Gemma 3, Gemma 4

**SwiGLU** (swish is $x * \mathrm{sigmoid}(x)$)

$$\mathrm{FFN}_{\mathrm{SwiGLU}}(x, W, V, W_2) = (\mathrm{Swish}_1(xW) \otimes xV)W_2$$

**Notable models:** LLaMa 1/2/3, PaLM, Mistral, OlMo, *most models post 2023*

Note printed at the foot: "Note: Gated models use smaller dimensions for the
$d_{ff}$ by 2/3"

## Slide 24 — Do gated linear units work?

Text: "Yes, fairly consistently so." Credit at the bottom right: "Shazeer 2020".

**Figure — a pasted results table** from Shazeer 2020. Three numeric columns:
"Score Average", "CoLA MCC", "SST-2 Acc". The rows are grouped by horizontal
rules: three non-gated FFNs, then five gated FFNs, then two reference rows. Bold
marks the best value in a column.

| | Score Average | CoLA MCC | SST-2 Acc |
| --- | --- | --- | --- |
| FFN<sub>ReLU</sub> | 83.80 | 51.32 | 94.04 |
| FFN<sub>GELU</sub> | 83.86 | 53.48 | 94.04 |
| FFN<sub>Swish</sub> | 83.60 | 49.79 | 93.69 |
| FFN<sub>GLU</sub> | 84.20 | 49.16 | 94.27 |
| FFN<sub>GEGLU</sub> | 84.12 | 53.65 | 93.92 |
| FFN<sub>Bilinear</sub> | 83.79 | 51.02 | **94.38** |
| FFN<sub>SwiGLU</sub> | 84.36 | 51.59 | 93.92 |
| FFN<sub>ReGLU</sub> | **84.67** | **56.16** | **94.38** |
| [Raffel et al., 2019] | 83.28 | 53.84 | 92.68 |
| ibid. stddev. | 0.235 | 1.111 | 0.569 |

The headline "Score Average" column: every gated row except FFN<sub>Bilinear</sub>
(83.79) beats every non-gated row, and FFN<sub>ReGLU</sub> is highest at 84.67
against 83.80 for FFN<sub>ReLU</sub>. The last two rows are the Raffel et al.
baseline and the standard deviation of that baseline, not experimental
conditions.

## Slide 25 — Do gated linear units work (2)?

Text: "Yes, with other works corroborating Shazeer 2020". Credit at bottom right:
"Narang et al 2020".

**Figure — a pasted results table** from Narang et al 2020, the same experimental
setup as slide 17 but sweeping activations. Columns: Model, Params, Ops, Step/s,
Early loss, Final loss, SGLUE, XSum, WebQ. (The WMT EnDe column that appears on
slide 17 is cropped off this paste.) A rule separates the "Vanilla Transformer"
baseline from the eleven variants; bold marks values that beat the baseline.

| Model | Params | Ops | Step/s | Early loss | Final loss | SGLUE | XSum | WebQ |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Vanilla Transformer | 223M | 11.1T | 3.50 | 2.182 ± 0.005 | 1.838 | 71.66 | 17.78 | 23.02 |
| GeLU | 223M | 11.1T | 3.58 | 2.179 ± 0.003 | 1.838 | **75.79** | **17.86** | **25.13** |
| Swish | 223M | 11.1T | 3.62 | 2.186 ± 0.003 | 1.847 | **73.77** | 17.74 | **24.34** |
| ELU | 223M | 11.1T | 3.56 | 2.270 ± 0.007 | 1.932 | 67.83 | 16.73 | 23.02 |
| GLU | 223M | 11.1T | 3.59 | 2.174 ± 0.003 | **1.814** | **74.20** | **17.42** | 24.34 |
| GeGLU | 223M | 11.1T | 3.55 | 2.130 ± 0.006 | **1.792** | **75.96** | **18.27** | **24.87** |
| ReGLU | 223M | 11.1T | 3.57 | 2.145 ± 0.004 | **1.803** | **76.17** | **18.36** | **24.87** |
| SeLU | 223M | 11.1T | 3.55 | 2.315 ± 0.004 | 1.948 | 68.76 | 16.76 | 22.75 |
| SwiGLU | 223M | 11.1T | 3.53 | 2.127 ± 0.003 | **1.789** | **76.00** | **18.20** | **24.34** |
| LiGLU | 223M | 11.1T | 3.59 | 2.149 ± 0.005 | **1.798** | **75.34** | **17.97** | **24.34** |
| Sigmoid | 223M | 11.1T | 3.63 | 2.291 ± 0.019 | 1.867 | **74.31** | 17.51 | 23.02 |
| Softplus | 223M | 11.1T | 3.47 | 2.207 ± 0.011 | 1.850 | **72.45** | 17.65 | **24.34** |

The four gated rows (GLU, GeGLU, ReGLU, SwiGLU, plus LiGLU) hold the five lowest
final losses in the table, SwiGLU lowest at 1.789.

## Slide 26 — Gating, activations

Bulleted summary, no figures.

- **Many variations (ReLU, GeLU, \*GLU) across models.**
- **\*GLU isn't *necessary* for a working model (see GPT3), but it's rare to see
  others..** — with, indented beneath it, "Some outlier models.." and, indented
  further, "Nemotron 340B (Squared ReLU)".
- **Evidence points towards somewhat consistent gains from Swi/GeGLU**

## Slide 27 — Serial vs Parallel layers

Text: "Normal transformer blocks are *serial* – they compute attention, then the
MLP", and beneath the figure: "Could we parallelize the transformer block?"

**Figure — a single-column block diagram of one serial transformer block**, drawn
as a rounded grey box with a vertical arrow entering at the bottom and leaving at
the top. Reading bottom to top, the boxes on the main column are: yellow **Norm**,
peach **Causal Multi-Head Self-Attention**, purple **Dropout**, green **Add**
(with a curved residual arrow arriving from the right, tapped off below the first
Norm); then yellow **Norm**, blue **Position-Wise Feed-Forward**, purple
**Dropout**, green **Add** (again with a curved residual arrow from the right,
tapped off below the second Norm). The attention half is computed before, and
feeds, the feed-forward half — that is what "serial" means here.

## Slide 28 — Parallel layers

Text: "A few models (GPTJ, PaLM, GPT-NeoX) do parallel layers. Originally in
GPT-J". Below the boxed quotation: "If implemented right, LayerNorm can be shared,
and matrix multiplies can be fused", and in bold "**Recent Models:** Cohere
Command A, Falcon 2 11B, Command R+".

**Figure — a pasted screenshot of a paragraph from the PaLM paper**, reproduced
inside a thin rectangular border. It reads:

> **Parallel Layers** – We use a "parallel" formulation in each Transformer block
> (Wang & Komatsuzaki, 2021), rather than the standard "serialized" formulation.
> Specifically, the standard formulation can be written as:
>
> $$y = x + \mathrm{MLP}(\mathrm{LayerNorm}(x + \mathrm{Attention}(\mathrm{LayerNorm}(x))))$$
>
> Whereas the parallel formulation can be written as:
>
> $$y = x + \mathrm{MLP}(\mathrm{LayerNorm}(x)) + \mathrm{Attention}(\mathrm{LayerNorm}(x))$$
>
> The parallel formulation results in roughly 15% faster training speed at large
> scales, since the MLP and Attention input matrix multiplications can be fused.
> Ablation experiments showed a small quality degradation at 8B scale but no
> quality degradation at 62B scale, so we extrapolated that the effect of parallel
> layers should be quality neutral at the 540B scale.

The citation "Wang & Komatsuzaki, 2021" is rendered as a blue hyperlink within the
screenshot.

## Slide 29 — Summary: architectures

Four headed bullet pairs on the left:

**Pre-vs-post norm:**

- Everyone does non-residual norm (except OPT350M), likely with good reason.

**Layer vs RMSnorm:**

- RMSnorm has clear compute wins, sometimes even performance

**Gating:**

- GLUs are consensus now

**Serial vs parallel layers:**

- Most models now use serial layers

**Figure — a third screenshot of the dense-model database**, filling the right
half of the slide. This view is sorted by year and shows only six columns:
**Name**, **Year**, **Norm**, **Parallel Layer**, **Pre-norm** (a checkbox — this
is the column that appears truncated as "P." on slides 7 and 9), **Position
embedding**, and **Activations**. It includes several early rows that the
slide 7/9 view had scrolled past. Transcribed in full:

| Name | Year | Norm | Parallel Layer | Pre-norm | Position embedding | Activations |
| --- | --- | --- | --- | --- | --- | --- |
| Original transformer | 2017 | LayerNorm | Serial | ☐ | Sine | ReLU |
| GPT | 2018 | LayerNorm | Serial | ☐ | Absolute | GeLU |
| T5 (11B) | 2019 | RMSNorm | Serial | ✓ | Relative | ReLU |
| GPT2 | 2019 | LayerNorm | Serial | ✓ | Absolute | GeLU |
| T5 (XXL 11B) v1.1 | 2020 | RMSNorm | Serial | ✓ | Relative | GeGLU |
| mT5 | 2020 | RMSNorm | Serial | ✓ | Relative | GeGLU |
| GPT3 (175B) | 2020 | LayerNorm | Serial | ✓ | Absolute | GeLU |
| GPTJ | 2021 | LayerNorm | Parallel | ✓ | RoPE | GeLU |
| LaMDA | 2021 | | | ✓ | Relative | GeGLU |
| Anthropic LM (not claude) | 2021 | | | ✓ | | |
| Gopher (280B) | 2021 | RMSNorm | Serial | ✓ | Relative | ReLU |
| GPT-NeoX | 2022 | LayerNorm | Parallel | ✓ | RoPE | GeLU |
| BLOOM (175B) | 2022 | LayerNorm | Serial | ✓ | AliBi | GeLU |
| OPT (175B) | 2022 | LayerNorm | Serial | ✓ | Absolute | ReLU |
| PaLM (540B) | 2022 | RMSNorm | Parallel | ✓ | RoPE | SwiGLU |
| Chinchilla | 2022 | RMSNorm | Serial | ✓ | Relative | ReLU |
| Mistral (7B) | 2023 | RMSNorm | Serial | ✓ | RoPE | SwiGLU |
| LLaMA2 (70B) | 2023 | RMSNorm | Serial | ✓ | RoPE | SwiGLU |
| LLaMA (65B) | 2023 | RMSNorm | Serial | ✓ | RoPE | SwiGLU |
| GPT4 | 2023 | | | ☐ | | |
| Baichuan 2 | 2023 | RMSNorm | Serial | ✓ | AliBi | SwiGLU |
| Olmo 2 | 2024 | RMSNorm | Serial | ☐ | RoPE | SwiGLU |
| Gemma 2 (27B) | 2024 | RMSNorm | Serial | ✓ | RoPE | GeGLU |
| Nemotron-4 (340B) | 2024 | LayerNorm | Serial | ✓ | RoPE | SqRelu |
| Qwen 2 (72b) – same for 2.5 | 2024 | RMSNorm | Serial | ✓ | RoPE | SwiGLU |
| Falcon 2 11B | 2024 | LayerNorm | Parallel | ✓ | RoPE | GeLU |
| Phi3 (small) – same for phi4 | 2024 | RMSNorm | Serial | ✓ | RoPE | GeGLU |
| Llama 3 (70B) | 2024 | RMSNorm | Serial | ✓ | RoPE | SwiGLU |
| Reka Flash | 2024 | RMSNorm | Serial | ✓ | RoPE | SwiGLU |
| Command R+ | 2024 | LayerNorm | Parallel | ✓ | RoPE | SwiGLU |
| OLMo | 2024 | RMSNorm | Serial | ✓ | RoPE | SwiGLU |
| Qwen (14B) | 2024 | RMSNorm | Serial | ✓ | RoPE | SwiGLU |
| DeepSeek (67B) | 2024 | RMSNorm | Serial | ✓ | RoPE | SwiGLU |
| Yi (34B) | 2024 | RMSNorm | Serial | ✓ | RoPE | SwiGLU |
| Mixtral of Experts | 2024 | | | ☐ | | |
| Command A | 2025 | LayerNorm | Parallel | ✓ | Hybrid (RoPE+NoPE) | SwiGLU |
| Gemma 3 | 2025 | RMSNorm | Serial | ☐ | RoPE | GeGLU |
| SmolLM2 (1.7B) | 2025 | RMSNorm | Serial | ✓ | RoPE | SwiGLU |

Blank cells are blank in the screenshot. The list ends at SmolLM2 — the 2026 rows
present on slides 7 and 9 are not shown in this view.

## Slide 30 — Many variations in position embeddings

Four labelled schemes down the left, each with its "Notable models:" list opposite.

**Sine embeddings:** add sines and cosines that enable localization

$$Embed(x, i) = v_x + PE_{pos}$$

*Figure 1 — a small pasted box* reproducing the original transformer's positional
encoding formulas:

$$PE_{(pos,2i)} = sin(pos/10000^{2i/d_{\mathrm{model}}})$$
$$PE_{(pos,2i+1)} = cos(pos/10000^{2i/d_{\mathrm{model}}})$$

**Notable models:** Original transformer

**Absolute embeddings:** add a position vector to the embedding

$$Embed(x, i) = v_x + u_i$$

**Notable models:** GPT1/2/3, OPT

**Relative embeddings:** add a vector to the *attention computation*

*Figure 2 — a pasted equation image* (rendered in a serif math face, unlike the
slide's own typeface):

$$e_{ij} = \frac{x_i W^Q (x_j W^K + a^K_{ij})^T}{\sqrt{d_z}}$$

**Notable models:** T5, Gopher, Chinchilla

**Rope embeddings** (next slides..)

**Notable models:** GPTJ, PaLM, LLaMA / *Most 2024+ models*

## Slide 31 — RoPE: rotary position embeddings

**High level thought process:** a *relative* position embedding should be some
$f(x, i)$ s.t.

$$\langle f(x, i), f(y, j) \rangle = g(x, y, i - j)$$

That is, the attention function *only* gets to depend on the relative position
(i-j). How do existing embeddings not fulfill this goal?

- **Sine:** Has various cross-terms that are not relative
  $$\langle Embed(x, i), Embed(y, i) \rangle = \langle v_x, v_y \rangle + \langle PE_i, v_y \rangle \ldots$$
- **Absolute:** obviously not relative
- **Relative embeddings:** *(the same pasted equation image as slide 30)*
  $$e_{ij} = \frac{x_i W^Q (x_j W^K + a^K_{ij})^T}{\sqrt{d_z}}$$
  "is not an inner product"

The only image on this page is that pasted relative-attention equation.

## Slide 32 — RoPE: rotary position embeddings

**How can we solve this problem?**

- We want our embeddings to be invariant to absolute position
- We know that inner products are invariant to arbitrary rotation.

**Figure — three hand-drawn vector diagrams side by side**, all in the deck's own
blue, illustrating that rotating both vectors by the same amount preserves the
angle between them. There are no axes or numbers; each diagram is a pair of
arrows from a common origin, labelled with the words "we" and "know".

- *Left diagram.* Two dark-blue arrows from one origin: a nearly vertical one
  labelled "we" and a shorter one, roughly 40° to its right, labelled "know".
  Caption below: "Position independent embedding".
- *Middle diagram.* The same two dark-blue arrows in the same orientation, plus a
  pale-blue ghost arrow and a small curved arrow showing "know" being rotated.
  Caption below: "Embedding "we know that"", then "Rotate we by '0 positions'" and
  "know by '1 positions'".
- *Right diagram.* Both arrows now rotated anticlockwise so that "know" is near
  vertical and "we" points up and to the left, with pale-blue ghost arrows and two
  curved arrows showing the rotations. Caption below: "Embedding "of course we
  know"", then "Rotate we by '2 positions'" and "Rotate know by '3 positions'".

The angle between the "we" and "know" arrows is visibly the same in all three
diagrams.

## Slide 33 — RoPE: rotary position embeddings

**There are many rotations, which one do you pick?**

Below the figures, the slide's own line: "Just pair up the coordinates and rotate
them in 2d (motivation: complex numbers)". Credit at the right: "[Su et al 2021]".
A separate margin note at the far right reads "Gemma 4 alternative: just first 2".

**Figure 1 (top left, inside a grey dashed border labelled "d=2") — the
two-dimensional case.** On the left, a two-cell green box labelled
"$(x_1, x_2)$" with the caption "Query / Key"; above it, a green
"$\boldsymbol{\theta}_1$" labelled "Constant", and below it a red "$\mathbf{m}$"
labelled "Position", both connected to the box by curved grey arrows. A grey
arrow leads right to a set of 2-D axes: a pale-green arrow from the origin to the
point $(x_1, x_2)$ (dashed guide lines mark $x_1$ on the horizontal axis and
$x_2$ on the vertical), and a second pale-green dashed arrow to the rotated point
$(x'_1, x'_2)$, with a black curved arrow between them labelled in red
"$\mathbf{m\theta_1}$". A further grey arrow leads right to a two-cell box shaded
green at the top and red at the bottom, labelled "$(x'_1, x'_2)$" with the caption
"Position Encoded Query / Key".

**Figure 2 (bottom, inside a solid black border) — the general $d$-dimensional
case.** Six rows, one per token of the sentence "Enhanced / Transformer / with /
Rotary / Position / Embedding" (the words label the rows down the left). Each row
shows, under the heading "Query / Key", a strip of coloured cells: two green cells
under the label "$\boldsymbol{\theta}_1$", two pink cells under
"$\boldsymbol{\theta}_2$", an ellipsis, then two peach cells under
"$\boldsymbol{\theta}_{d/2-1}$" and two blue cells under
"$\boldsymbol{\theta}_{d/2}$". The middle column, headed "Position", gives each
row an index in a different colour: **1** (red), **2** (olive), **3** (green),
**4** (teal), **5** (cyan), **6** (dark blue). A large black arrow points right to
the third column, headed "Position Encoded Query / Key", where the same cell
strips are reproduced with a vertical colour gradient blending each cell into its
row's position colour — row 1's cells wash to red, row 2's to olive, and so on
down to row 6's dark blue. Grey callout lines connect the first row's leading
green pair, and its position index "1", up into the $d=2$ diagram above,
indicating that the top figure is a zoom on one coordinate pair of one token.

**Figure 3 (right margin) — a separate two-panel graphic on partial rotation.**
Top panel: an eight-cell greyscale strip labelled on its left "the / m = 5" and on
its right "**original** query"; under the strip, four brackets group the cells in
pairs, labelled $\boldsymbol{\theta}_1$, $\boldsymbol{\theta}_2$,
$\boldsymbol{\theta}_3$, $\boldsymbol{\theta}_4$. Middle: a small pair of axes
with a black arrow rotated anticlockwise into a purple arrow, annotated "only
rotate high frequency pairs". Bottom panel: the same eight-cell strip, labelled
"the / (rotated)" on the left and "**partially rotated** query" on the right, where
only the first two cells carry a purple diagonal wash; the bracket under the first
two cells reads "some **positional** information" and the bracket under the
remaining six reads "only **semantic** information".

## Slide 34 — The actual RoPE math

Text: "Multiply with sines and cosines".

**Figure — a pasted page fragment from Su et al 2021** carrying two numbered
equations. Equation (14):

$$f_{\{q,k\}}(\boldsymbol{x}_m, m) = \boldsymbol{R}^d_{\Theta,m} \boldsymbol{W}_{\{q,k\}} \boldsymbol{x}_m$$

Equation (15) defines the block-diagonal rotation matrix:

$$\boldsymbol{R}^d_{\Theta,m} = \begin{pmatrix}
\cos m\theta_1 & -\sin m\theta_1 & 0 & 0 & \cdots & 0 & 0 \\
\sin m\theta_1 & \cos m\theta_1 & 0 & 0 & \cdots & 0 & 0 \\
0 & 0 & \cos m\theta_2 & -\sin m\theta_2 & \cdots & 0 & 0 \\
0 & 0 & \sin m\theta_2 & \cos m\theta_2 & \cdots & 0 & 0 \\
\vdots & \vdots & \vdots & \vdots & \ddots & \vdots & \vdots \\
0 & 0 & 0 & 0 & \cdots & \cos m\theta_{d/2} & -\sin m\theta_{d/2} \\
0 & 0 & 0 & 0 & \cdots & \sin m\theta_{d/2} & \cos m\theta_{d/2}
\end{pmatrix}$$

The equation numbers "(14)" and "(15)" are printed flush right in the pasted
fragment.

Below the figure: "Difference with sine embeddings – not additive, no cross terms"

## Slide 35 — Implementation and code for RoPE

**Figure — a pasted screenshot of syntax-highlighted Python** (the HuggingFace
LLaMA attention forward pass), with three hand-added blue arrows and margin
labels pointing into it. The code reads:

```python
query_states = self.q_proj(hidden_states)
key_states = self.k_proj(hidden_states)
value_states = self.v_proj(hidden_states)

# Flash attention requires the input to have the shape
# batch_size x seq_length x head_dim x hidden_dim
# therefore we just need to keep the original shape
query_states = query_states.view(bsz, q_len, self.num_heads, self.head_dim).transpose(1, 2)
key_states = key_states.view(bsz, q_len, self.num_key_value_heads, self.head_dim).transpose(1, 2)
value_states = value_states.view(bsz, q_len, self.num_key_value_heads, self.head_dim).transpose(1, 2)

cos, sin = self.rotary_emb(value_states, position_ids)
query_states, key_states = apply_rotary_pos_emb(query_states, key_states, cos, sin)
```

In the screenshot the argument `position_ids` is highlighted with a yellow
background. A vertical bar in the left margin brackets the first block of lines,
annotated "Usual attention stuff". A blue arrow points at the `cos, sin =
self.rotary_emb(...)` line, annotated "Get the RoPE matrix cos/sin". A second blue
arrow points at the `apply_rotary_pos_emb(...)` line, annotated "Multiply
query/key inputs". Beneath the code, an ellipsis and the line "Same stuff as the
usual multi-head self attention below" stand for the rest of the function.

At the foot, in bold: "**Note:** embedding at *each attention operation* to
enforce position invariance".

## Slide 36 — Hyperparameters

Text slide, no figures.

**Transformer hyperparameter questions you might have had in 224n..**

- How much bigger should the feedforward size be compared to hidden size?
- How many heads, and should num_heads always divide hidden size?
- What should my vocab size be?

And other model setting questions

- Do people even regularize these huge LMs?
- How do people scale these models - very deep or very wide?

## Slide 37 — Surprising (?) consensus hyperparameter 1

Text: "Feedforward – model dimension ratio."

**Figure — a pasted equation image** (in a serif math face) repeating the original
FFN definition:

$$\mathrm{FFN}(x) = \max(0, xW_1 + b_1)W_2 + b_2$$

"There are two dimensions that are relevant – the feedforward dim ($d_{ff}$) and
model dim ($d_{model}$). What should their relationship be?"

Centred in bold:

$$\boldsymbol{d_{ff} = 4\,d_{model}}$$

"This is *almost always* true. There's just a few exceptions."

## Slide 38 — Exception #1 – GLU variants

Text: "Remember that GLU variants scale down by 2/3rd. This means most GLU
variants have $d_{ff} = \frac{8}{3} d_{model}$. This is mostly what happens. Some
notable such examples."

**Table** (drawn with the deck's own blue header and alternating grey/white
rows — this one is native slide content, not a paste):

| Model | $d_{ff}/d_{model}$ |
| --- | --- |
| PaLM | 4 |
| Mistral 7B | 3.5 |
| LLaMA-2 70B | 3.5 |
| LLaMA 70B | 2.68 |
| Qwen 14B | 2.67 |
| DeepSeek 67B | 2.68 |
| Yi 34B | 2.85 |
| T5 v1.1 | 2.5 |

Caption under the table: "Models are roughly in this range, though PaLM, LLaMA2
and Mistral are slightly larger"

## Slide 39 — Exception #2 – T5

"As we have (and will) see, most LMs are have boring, conservative hyperparameters.
One exception is T5 [Raffel et al 2020] which has some *very bold* settings."

"In particular, for the 11B model, they set"

$$d_{ff} = 65{,}536$$
$$d_{model} = 1024$$

"For an astounding 64-times multiplier."

**Figure — a pasted quotation from the T5 paper**, inside a thin rectangular
border:

> for "11B" we use $d_{\mathrm{ff}} = 65{,}536$ with 128-headed attention producing
> a model with about 11 billion parameters. We chose to scale up $d_{\mathrm{ff}}$
> specifically because modern accelerators (such as the TPUs we train our models
> on) are most efficient for large dense matrix multiplications like those in the
> Transformer's feed-forward networks.

At the foot: "Other, recent exceptions – Gemma 2 (8x), SmolLM/Gemma 3/Gemma 4 (4x,
GLU)"

## Slide 40 — Why this range of multipliers?

Text: "Empirically, there's a basin between 1-10 where this hyperparameter is
near-optimal". Credit at the bottom right: "[Kaplan+ 2020]".

**Figure — a pasted line chart from Kaplan et al 2020.** The x-axis is
**"Feed-Forward Ratio ($d_{ff}$ / $d_{model}$)"** on a log scale, with major ticks
at $10^0$ and $10^1$ and a sub-caption "50M Parameters" beneath the axis label.
The y-axis is **"Loss Increase"**, ticked 0%, 2%, 4%, 6%, 8%, 10%. There are
**two** series, and they lie almost exactly on top of each other over most of the
range:

- *Orange line with round markers*, "$d_{\mathrm{model}}/n_{\mathrm{head}} = 64$":
  about 0.75% at a ratio of roughly 0.5, dropping to 0% at ratio 1, ~0.05% at
  ratio 2, ~0.3% at ratio 4, ~1.8% at ratio 8, ~4.8% at ratio ~25, and ~8.4% at
  the rightmost point (ratio ~50).
- *Blue line with × markers*, "$n_{\mathrm{head}} = 8$": only plotted over the
  upper part of the range — ~1.5% at ratio 8, ~4.9% at ratio ~25, and ~7.8% at the
  rightmost point.

The curve is flat (near-zero loss increase) from about ratio 1 to ratio 4 and then
rises steeply — this flat region is the "basin" the slide text refers to.

## Slide 41 — What can we learn from the model-dim hyperparam?

Three bullets, no figures.

- The 'default' choices of $d_{ff} = 4d_{model}$ and $d_{ff} = 2.66 d_{model}$ have
  worked well for nearly all modern LLMs.
- But T5 does show that even radical choices of $d_{ff} = 64 d_{model}$ can work.
  This hyperparameter choice isn't written in stone.
- That said, T5 has a follow-up model (T5 v1.1) that is 'improved' and uses a much
  more standard 2.5 multiplier on GeGLU, so the 64-times multiplier is likely
  suboptimal.

## Slide 42 — Surprising (?) consensus hyperparameter 2

Text: "Head-dim*num-heads to model-dim ratio. As a reminder, slide from 224n."

**Figure — a pasted screenshot of a CS224n slide**, inside a grey-bordered box.
Its heading is "**Multi-head self-attention is computationally efficient**" and its
body reads:

> - Even though we compute $h$ many attention heads, it's not <ins>really more</ins>
>   costly.
>   - We compute $XQ \in \mathbb{R}^{n \times d}$, and then reshape to
>     $\mathbb{R}^{n \times h \times d/h}$. (<ins>Likewise</ins> for $XK$, $XV$.)
>   - Then we transpose to $\mathbb{R}^{h \times n \times d/h}$; now the head axis
>     is like a batch axis.
>   - Almost everything else is identical, and the **matrices are the same sizes.**

(The words "really more" and "Likewise" carry the red squiggly underline of a
spell-checker in the screenshot.)

Below the box: "This doesn't *have to* be true: we can have head-dimensions >
model-dim / num-heads." and, centred, "But most models do follow this guideline"

## Slide 43 — How many heads, whats the model dim?

Text: "Some examples of this hyperparameter".

**Table** (native slide content with the deck's blue header):

| Model | Num heads | Head dim | Model dim | Ratio |
| --- | --- | --- | --- | --- |
| GPT3 | 96 | 128 | 12288 | 1 |
| T5 | 128 | 128 | 1024 | 16 |
| T5 v1.1 | 64 | 64 | 4096 | 1 |
| LaMDA | 128 | 128 | 8192 | 2 |
| PaLM | 48 | 258 | 18432 | 1.48 |
| LLaMA2 | 64 | 128 | 8192 | 1 |
| Qwen 3.5 (27B) | 24 | 256 | 5120 | 1.2 |

(The PaLM head dim is printed as 258, not 256.) The "Ratio" column is
(num heads × head dim) / model dim.

Caption under the table: "Most models have ratios around 1 – notable exceptions by
some google models."

## Slide 44 — Aspect ratios

Text: "Should my model be deep or wide? *How* deep and how wide?" and "Most models
are surprisingly consistent on this one too!"

**Table** (native slide content, blue header):

| Model | $d_{model}/n_{layer}$ |
| --- | --- |
| BLOOM | 205 |
| T5 v1.1 | 171 |
| PaLM (540B) | 156 |
| GPT3/OPT/Mistral/Qwen/OLMo 3 | 128 |
| LLaMA / LLaMA2 | 102 |
| Gemma 3 | 87 |
| Gemma 4 | 61 |
| T5 (11B) | 33 |

A short vertical blue bar is drawn in the left margin spanning the rows from
"GPT3/OPT/Mistral/Qwen/OLMo 3" (128) up through "Gemma 3" (87), labelled "Sweet
spot?".

## Slide 45 — Considerations about aspect ratio

Text: "**Extremely deep models are harder to parallelize and have higher latency**".
Credit at the right: "[Tay et al 2021]".

**Figure 1 — a pasted quotation from Tay et al 2021**, inside a thin rectangular
border. The paste is cut off mid-sentence at its right/bottom edge:

> **The Limits of Depth vs Width**  We note an obvious limitation with our advice.
> Scaling depth has an obvious limiter, i.e., they are non-parallelizable across
> different machines or devices and every computation has to always wait for the
> previous layer. This is unlike width, which can be easily parallelizable over
> thousands or hundreds of thousands of devices. Within the limitation of scaling

**Figure 2 — a small pipeline-parallelism diagram.** Four coloured boxes in a row,
labelled "Layer 0" (light blue), "Layer 1" (pale yellow), "Layer 2" (pink) and
"Layer 3" (peach). Red arrows run left to right across the top of the boxes,
labelled "forward"; blue arrows run right to left below them, labelled "backward".
Beneath each box sits a cartoon graphics-card icon, captioned "GPU 0", "GPU 1",
"GPU 2" and "GPU 3" respectively — one layer per device, so each device must wait
on the one before it.

## Slide 46 — Evidence on aspect ratio scaling

The slide has almost no text of its own: only the title and two credits,
"[Kaplan et al 2020]" (bottom left) and "[Tay et al 2021]" (bottom right). The
three pasted figures are the slide.

**Figure 1 (left, Kaplan et al 2020) — loss against aspect ratio.** The x-axis is
"**Aspect Ratio** ($d_{model}$ / $n_{layer}$)" on a log scale with major ticks at
$10^1$, $10^2$ and $10^3$. The y-axis carries horizontal gridlines but **no tick
labels and no axis title in this paste** — it is a loss axis, increasing upward,
with no numbers readable. Two black vertical rules are drawn on the plot at
roughly aspect ratio 9 and aspect ratio 330, and the annotation between them
reads "A wide range of architectures achieve similar performance". There are
**three** data series (the two vertical rules and the annotation are not series):

- *Blue line with round markers*, "50M Params": highest at the far left (aspect
  ratio ≈ 4), falling through ratio ≈ 11 and ≈ 30 to a minimum around ratio 50–90,
  then rising again through ratio ≈ 250 and ≈ 400 to a high point at ratio ≈ 750.
- *Orange line with × markers*, "274M Params": starts at the far left (ratio ≈ 2.5)
  at a much lower value than the blue curve, declines to a minimum around ratio
  25, tracks the others through the flat region, and then rises very steeply,
  ending highest of all at ratio ≈ 1800.
- *Green line with star markers*, "1.5B Params": starts at ratio ≈ 4 below the blue
  curve, falls to its minimum around ratio 30, stays flat through the middle, and
  rises the least steeply of the three, ending at ratio ≈ 750 well below the other
  two.

All three curves are close together and nearly flat between the two vertical
rules, which is the slide's point.

**Figure 2 (top right, Tay et al 2021) — a pair of scatter panels titled "DM" and
"NL", with y-axis "Negative Log-Perplexity" and x-axis "FLOPS".** A shared legend
on the right, headed "Params", encodes marker size: a dot for 0.0e+0, and open
circles of increasing size for 2.0e+8, 4.0e+8, 6.0e+8 and 8.0e+8. In both panels a
single red-outlined circle marks the "Base" configuration and grey lines connect
it to the variants.

- *Left panel "DM"* (varying $d_{model}$). y-axis ticked $-2.00$ to $-1.55$ in
  steps of 0.05; x-axis ticked 0.0e+0 to 4.0e+13. Five green circles, labelled
  DM256 at about (0.2e+13, $-1.985$), DM512 at about (0.7e+13, $-1.81$), Base
  (red) at about (1.15e+13, $-1.752$), DM1K at about (1.5e+13, $-1.72$), and DM2K
  at about (2.9e+13, $-1.653$), with marker size growing with parameter count.
- *Right panel "NL"* (varying number of layers). Same axes. Eight green circles
  radiating from the red Base marker: NL4 at about (0.35e+13, $-1.97$), NL8 at
  about (0.75e+13, $-1.82$), NL12 = Base (red) at about (1.15e+13, $-1.75$), NL16
  at about (1.5e+13, $-1.715$), NL24 at about (2.05e+13, $-1.672$), NL32 at about
  (2.55e+13, $-1.655$), NL36 at about (2.8e+13, $-1.645$), NL40 at about (3.4e+13,
  $-1.673$) and NL48 at about (4.4e+13, $-1.622$).

**Figure 3 (bottom right, Tay et al 2021) — the same pair of panels with y-axis
"SuperGlue Accuracy"** instead of negative log-perplexity, ticked 60 to 78 in
steps of 2; x-axis "FLOPS" ticked 0.0e+0 to 4.0e+13; same "Params" size legend.

- *Left panel "DM"*: DM256 at about (0.2e+13, 64.8), DM512 at about (0.7e+13,
  69.7), Base (red) at about (1.15e+13, 69.8), DM1K at about (1.5e+13, 71.0),
  DM2K at about (2.9e+13, 71.9).
- *Right panel "NL"*: NL4 at about (0.35e+13, 61.7), NL8 at about (0.75e+13,
  69.1), NL12 = Base (red) at about (1.15e+13, 69.8), NL16 at about (1.5e+13,
  73.4), NL24 at about (2.05e+13, 74.7), NL32 at about (2.55e+13, 73.0), NL36 at
  about (2.8e+13, 75.5), NL40 at about (3.4e+13, 72.2), NL48 at about (4.4e+13,
  75.4).

## Slide 47 — What are typical vocabulary sizes?

Two tables side by side, both native slide content with blue headers.

Left, headed "Monolingual models – 30-50k vocab":

| Model | Token count |
| --- | --- |
| Original transformer | 37000 |
| GPT | 40257 |
| GPT2/3 | 50257 |
| T5/T5v1.1 | 32128 |
| LLaMA | 32000 |

Right, headed "Multilingual / production systems 100-250k":

| Model | Token count |
| --- | --- |
| mT5 | 250000 |
| PaLM | 256000 |
| GPT4 | 100276 |
| Gemma 4 | 262144 |
| DeepSeek | 100000 |
| Qwen 15B | 152064 |
| Yi | 64000 |

Caption beneath both: "Monolingual vocabs don't need to be huge, but multilingual
ones do"

## Slide 48 — Dropout and other regularization

Text slide, no figures.

Centred: "Do we need regularization during pretraining?"

**Arguments against:**

- There is *a lot* of data (trillions of tokens), more than parameters.
- SGD only does a single pass on a corpus (hard to memorize)

"This is all quite reasonable.. but what do people do in practice?"

## Slide 49 — Dropout and weight decay in practice

**Table** (native slide content, blue header):

| Model | Dropout* | Weight decay |
| --- | --- | --- |
| Original transformer | 0.1 | 0 |
| GPT2 | 0.1 | 0.1 |
| T5 | 0.1 | 0 |
| GPT3 | 0.1 | 0.1 |
| T5 v1.1 | 0 | 0 |
| PaLM | 0 | (variable) |
| OPT | 0.1 | 0.1 |
| LLaMA | 0 | 0.1 |
| Qwen 14B | 0.1 | 0.1 |

To the right of the table: "Many older models used dropout during pretraining" and
"Newer models (except Qwen) rely only on weight decay".

Footnote at the bottom of the slide: "* Most of the times papers just don't discuss
dropout. On open models, this closely matches not doing dropout. This may not be
true of closed models."

No figures beyond the table.

## Slide 50 — Why weight decay LLMs?

Text: "[Andriushchenko et al 2023] has interesting observations about LLM weight
decay". Below the figures, two captions: "It's not to control overfitting" (under
the left chart) and "Weight decay interacts with learning rates (cosine schedule)"
(under the two right charts).

**Figure 1 (left) — a scatter plot of validation loss against training loss.**
x-axis "Training loss", ticked 3.3, 3.4, 3.5, 3.6, 3.7, 3.8; y-axis "Validation
loss", ticked 3.3, 3.4, 3.5, 3.6, 3.7, 3.8. **Three** series of dots, distinguished
only by colour:

- *Red*, "$\lambda_{WD} = 0.0$"
- *Green*, "$\lambda_{WD} = 0.1$"
- *Blue*, "$\lambda_{WD} = 0.3$"

All three colours lie interleaved along a single tight diagonal band running from
about (3.26, 3.26) to (3.8, 3.8). No colour sits systematically above or below the
diagonal — which is the "not to control overfitting" point.

**Figure 2 (middle) — a training-loss curve panel titled "10× cosine LR decay".**
x-axis "Iteration", ticked 10000, 20000, 30000, 40000, 50000, 60000; y-axis
"Training loss", ticked 3.2 to 3.7 in steps of 0.1. **Six** series, three solid and
three dashed:

- *Red solid*, "$\lambda_{WD} = 0.0$, $w_t$": ~3.66 at 10k, ~3.50 at 20k, ~3.44 at
  30k, ~3.35 at 40k, ~3.31 at 50k, ~3.30 at 60k.
- *Green solid*, "$\lambda_{WD} = 0.1$, $w_t$": ~3.68 at 10k, ~3.55 at 20k, ~3.48
  at 30k, ~3.37 at 40k, ~3.31 at 50k, ~3.28 at 60k.
- *Blue solid*, "$\lambda_{WD} = 0.3$, $w_t$": ~3.72 at 10k, ~3.63 at 20k, ~3.56 at
  30k, ~3.43 at 40k, ~3.32 at 50k, ~3.28 at 60k — the highest of the solid curves
  for most of the run, converging with the others by the end.
- *Red dashed*, "$\lambda_{WD} = 0.0$, $w_t \rightarrow$ tiny LR": branches below
  the solid curves early, ~3.50 at 15k, ~3.38 at 25k, ~3.33 at 40k, ending ~3.27.
- *Green dashed*, "$\lambda_{WD} = 0.1$, $w_t \rightarrow$ tiny LR": similar path,
  ending ~3.24.
- *Blue dashed*, "$\lambda_{WD} = 0.3$, $w_t \rightarrow$ tiny LR": similar path,
  ending lowest at ~3.23.

**Figure 3 (right) — the same six series, titled "Constant LR".** Same axes and
ranges. Here the dashed "tiny LR" runs branch off the solid runs at 20k, 30k, 40k
and 50k in turn and each drops sharply, producing a staircase of dashed curves
below the solid ones.

- *Red solid*: ~3.66 at 10k, ~3.50 at 20k, ~3.45 at 30k, ~3.40 at 40k, ~3.38 at
  50k, flattening.
- *Green solid*: ~3.68 at 10k, ~3.55 at 20k, ~3.51 at 30k, ~3.47 at 40k, flat at
  ~3.47 to 50k.
- *Blue solid*: ~3.73 at 10k, ~3.64 at 20k, ~3.62 at 30k, ending ~3.60 at ~35k —
  the highest curve throughout.
- The three dashed "tiny LR" branches drop steeply where they split off; the red
  dashed reaches ~3.29 by 60k, the green dashed ~3.31, and the blue dashed ~3.36.

## Slide 51 — Summary: hyperparameters

Four headed bullets on the left:

**Feedforward**

- Factor-of-4 rule of thumb (8/3 for GLUs) is standard (with some evidence)

**Head dim**

- Head dim*Num head = D model is standard – but low to no validation

**Aspect ratio**

- Wide range of 'good' values (100-200). Systems concerns dictate the value

**Regularization**

- You still 'regularize' LMs but its effects are primarily on optimization dynamics

**Figure — a fourth view of the dense-model database screenshot**, filling the
right half of the slide. Columns: **Name**, **Year**, **MLP factor**, **Aspect
ratio (d/layer)**, **weight decay**, **drop_rate**. Transcribed in full (blank
cells are blank in the screenshot):

| Name | Year | MLP factor | Aspect ratio (d/layer) | weight decay | drop_rate |
| --- | --- | --- | --- | --- | --- |
| Original transformer | 2017 | 4 | 85 | 0 | 0.1 |
| GPT | 2018 | 4 | 64 | 0.1 | 0.1 |
| T5 (11B) | 2019 | 64 | 43 | 0 | 0.1 |
| GPT2 | 2019 | 4 | 33 | 0.1 | 0.1 |
| T5 (XXL 11B) v1.1 | 2020 | 2.5 | 171 | 0 | 0 |
| mT5 | 2020 | 2.5 | 171 | 0 | 0 |
| GPT3 (175B) | 2020 | 4 | 128 | 0.1 | 0.1 |
| GPTJ | 2021 | | 146 | 0.1 | 0 |
| LaMDA | 2021 | 8 | 128 | | |
| Anthropic LM (not claude) | 2021 | 4 | 128 | | |
| Gopher (280B) | 2021 | 4 | 205 | | |
| GPT-NeoX | 2022 | 4 | 140 | 0.01 | 0 |
| BLOOM (175B) | 2022 | 4 | 205 | 0.1 | 0 |
| OPT (175B) | 2022 | 4 | 128 | 0.1 | 0.1 |
| PaLM (540B) | 2022 | 4 | 156 | | 0 |
| Chinchilla | 2022 | 4 | 102 | | |
| Baichuan 2 | 2023 | 2.68 | 128 | 0.1 | 0 |
| Mistral (7B) | 2023 | 3.5 | 128 | 0.1 | 0 |
| LLaMA2 (70B) | 2023 | 3.5 | 102 | 0.1 | 0 |
| LLaMA (65B) | 2023 | 2.6875 | 102 | 0.1 | 0 |
| GPT4 | 2023 | | 0 | | |
| Olmo 2 | 2024 | 2.6875 | 128 | | |
| Gemma 2 (27B) | 2024 | 8 | 100 | | |
| Nemotron-4 (340B) | 2024 | 4 | 192 | | 0 |
| Qwen 2 (72b) – same for 2.5 | 2024 | 3.609 | 102 | | |
| Falcon 2 11B | 2024 | 4 | 68 | 0.1 | |
| Phi3 (small) – same for phi4 | 2024 | 3.5 | 128 | | |
| Llama 3 (70B) | 2024 | 3.5 | 102 | | 0 |
| Reka Flash | 2024 | | 0 | | |
| Command R+ | 2024 | 2.75 | 192 | | |
| OLMo | 2024 | 2.6875 | 128 | 0.1 | 0 |
| Qwen (14B) | 2024 | 2.675 | 128 | 0.1 | 0.1 |
| DeepSeek (67B) | 2024 | 2.6875 | 86 | 0.1 | 0 |
| Yi (34B) | 2024 | 2.857142 | 119 | 0.1 | 0 |
| Mixtral of Experts | 2024 | | 0 | | |
| Command A | 2025 | | 0 | | |
| Gemma 3 | 2025 | 4 | 87 | | |
| SmolLM2 (1.7B) | 2025 | 4 | 85 | | |

## Slide 52 — Stability tricks

Text: "Recently, lots of attention on *stable training*", and beneath the figure:
"Don't train models that look like the blue curve!"

**Figure — a two-panel training-run plot with a shared x-axis** (training step,
ticked 0, 100000, 200000, 300000, 400000, 500000, 600000; the axis itself is
unlabelled). Both panels share one legend, printed in the top panel, with **two**
series:

- *Blue*, "OLMo 0424 7B"
- *Orange*, "OLMo 2 1124 7B"

*Top panel*, y-axis "loss", ticked 2.0 to 3.0 in steps of 0.2. The blue curve
falls from above 3.0 to about 2.2 by 100k steps and drifts down to about 2.1 by
600k; it is the **lower** of the two curves throughout, but it is punctuated by
tall thin spikes that shoot off the top of the panel, at roughly 50k, 150k, 160k,
185k, 205k, 210k, 245k, 300k, 355k, 435k, 475k and 570k steps. The orange curve
falls to about 2.45 by 100k and drifts down to about 2.28 by 600k — consistently
higher loss, but visibly smooth, with no spikes.

*Bottom panel*, y-axis "L2 norm of the gradient", ticked 0.0 to 3.0 in steps of
0.5. The blue trace settles at a baseline of roughly 0.25 but throws hundreds of
spikes, most of them reaching 1.0–3.0 and many clipping off the top of the panel;
the spike density visibly increases with training step, becoming almost solid
after about 350k. The orange trace drops to a baseline of roughly 0.08 within the
first few thousand steps and stays there for the whole run, with only a scattering
of small spikes to about 0.5–1.0.

So the "blue curve" the slide warns against is the one with the *lower* loss but
the unstable gradients.

## Slide 53 — Where do the issues arise? Beware of softmaxes!

Text: "**Softmaxes** – can be ill-behaved due to exponentials / divison by zero"
(the typo "divison" is on the slide).

**Figure — the same two block diagrams as slide 4**, reproduced here to point at
where softmaxes occur. On the left, the whole-model column: "Inputs" → pink
**Token Embedding** → grey **Transformer Block** → ellipsis → grey **Transformer
Block** → yellow **Norm** → purple **Linear (Output Embedding)** → green
**Softmax** → "Output Probabilities". On the right, the single-block zoom: input
tensor with shape `(batch_size, seq_len, d_model)` → yellow **Norm** → orange
**Causal Multi-Head Self-Attention w/ RoPE** → green **Add**; → yellow **Norm** →
blue **Position-Wise Feed-Forward** → green **Add** → output tensor with shape
`(batch_size, seq_len, d_model)`. The two softmaxes at issue are the output
softmax (the green box, top left) and the attention softmax (inside the orange
box, right).

## Slide 54 — Output softmax stability – the 'z-loss'

Text: "Recall the softmax calculation". Credit under the equation boxes: "[From
Devlin 2014]".

**Figure 1 (left) — a pasted equation box:**

$$\log(P(x)) = \log\left(\frac{e^{U_r(x)}}{Z(x)}\right)$$
$$= U_r(x) - \log(Z(x))$$
$$Z(x) = \Sigma^{|V|}_{r'=1} e^{U_{r'}(x)}$$

**Figure 2 (right) — a second pasted equation box:**

$$L = \sum_i \left[\log(P(x_i)) - \alpha(\log(Z(x_i)) - 0)^2\right]$$
$$= \sum_i \left[\log(P(x_i)) - \alpha \log^2(Z(x_i))\right]$$

Text: "This is useful for stability! PaLM used this 'z loss' trick."

**Figure 3 — a pasted quotation from the PaLM paper**, in a thin border:

> We additionally use an auxiliary loss of $z\_loss = 10^{-4} \cdot \log^2 Z$ to
> encourage the softmax normalizer $\log(Z)$ to be close to 0, which we found
> increases the stability of training.

At the foot, in bold: "**Other examples:** Baichuan 2 (2023), DCLM (2024), OLMo 2
(2025), OLMo 3 (2025)"

## Slide 55 — Attention softmax stability – the 'QK norm'

Slide text: "The query and keys are Layer (RMS) normed before going into the
softmax operation." — note that the **middle of this sentence is covered by a
pasted image**, so the words between "are" and "softmax operation" are hidden on
the rendered page.

**Figure 1 — a pasted architecture diagram of a transformer block with QK-norm**,
occupying the upper half of the slide and largely covered by figure 2. What
remains visible: an outer box labelled "Multihead attention" at the top right; a
dashed residual path; a cyan **LN** box feeding a green **QKV** box (the label is
partly hidden), a line labelled **V** running across the top to the right, a green
**Proj** box at the right, and a $\oplus$ node where the residual rejoins. Below
that, a second dashed sub-block with another cyan **LN** feeding a green **FC**
box (also partly hidden).

**Figure 2 — a pasted hand-drawn meme**, sitting on top of figure 1 and hiding its
middle. It is a crude ink drawing of a stick figure in a jester's hat with a
speech-shaped blob reading "**STACK MORE LAYER Norms**" — the words "STACK MORE
LAYER" are part of the drawing, in blocky all-caps, while "**Norms**" is slide-native
text in the deck's own proportional sans-serif, layered on top of the pasted image
inside the speech bubble so that it completes the pun (turning "stack more layers"
into "stack more LayerNorms"); behind the figure a
banner reads "NEURAL NETWORKS" and a chart with an axis labelled "LAYERS"
(vertically) and "LAYERS" (horizontally) shows a jagged green line rising to the
right.

At the foot, in bold: "**Other examples: DCLM, OLMo2, Gemma 2, Qwen3, OLMo 3,
Gemma 4**", and beneath it "Originally from vision and multimodal models
[Dehgani 2023, Idefcs, Chameleon]" (spellings as printed).

## Slide 56 — Logit soft-capping.

Text: "**Soft-capping** the logits to some maximum value via Tanh"

**Figure 1 — a pasted quotation from the Gemma 2 paper**, in a thin rectangular
border:

> **Logit soft-capping.** We cap logits (Bello et al., 2016) in each attention
> layer and the final layer such that the value of the logits stays between
> $-$soft_cap and $+$soft_cap. More specifically, we cap the logits with the
> following function:
>
> $$\mathrm{logits} \leftarrow \mathrm{soft\_cap} * \tanh(\mathrm{logits}/\mathrm{soft\_cap}).$$
>
> We set the soft_cap parameter to 50.0 for the self-attention layers and to 30.0
> for the final layer.

("Bello et al., 2016" is rendered as a blue hyperlink inside the screenshot.)

Slide text: "Prevents logits from blowing up, but also might have perf issues?"

**Figure 2 — a small pasted results table**, captioned "Table 4: Models perplexity
with confidence interval $\pm 0.1$ at 95% level." A single-row table with six
columns:

| bf16 baseline | *soft_cap* | *QKV_norm* | *QK_norm_cap* | *QK_norm* | *QK_FC_norm* |
| --- | --- | --- | --- | --- | --- |
| 11.19 | 11.24 | 10.85 | 11.00 | 10.84 | 10.87 |

Soft-capping alone (11.24) is the only column *worse* than the bf16 baseline
(11.19); the QK-norm variants are all better, the best being QK_norm at 10.84.

## Slide 57 — Attention heads

Text slide, no figures.

"Most models don't touch the attention heads much at all with a few minor
exceptions.."

- **GQA / MQA** : Saving inference costs by reducing the number of heads
- **Sparse or sliding window attention (GPT4/Mistral):** restricting the attention
  pattern to reduce compute cost
- **Exotic SSM stuff (Jamba, Falcon 3, Qwen 3.5, etc):** next lecture!

## Slide 58 — GQA/MQA – Reducing attention head cost

Text: "**Let's think about the compute involved for attention**"

**Figure — a hand-built diagram of batched multi-head attention**, drawn with
coloured slab shapes. Top row: three vertical pink slabs labelled $XQ$, times a
stack of three horizontal orange slabs labelled $K^\top X^\top$, equals a stack of
three grey squares labelled $XQK^\top X^\top$, annotated
"$\in \mathbb{R}^{3 \times n \times n}$" and, in teal, "3 sets of all pairs of
attention scores!". A curved arrow carries that stack down to the second row.
Second row: $\mathrm{softmax}\big(\,$ the stack of grey squares labelled
$XQK^\top X^\top\,\big)$ times three vertical teal slabs labelled $XV$, equals a
stack of three narrow grey slabs, then times a small grey square labelled $P$
(annotated "mix"), equals one grey slab annotated
"output $\in \mathbb{R}^{n \times d}$".

A margin key at the right of the slide defines the symbols:

- d = hidden dim
- b = batch
- n = length (<d)
- h = heads
- k = head dim (d/h)

Below the diagram, in bold with small labels printed above the individual terms
("X" over $bnd$, "softmax" over $bhn^2$, "projection" over $d^2$):

**Total arithmetric operations** $(bnd^2)$, **total memory accesses**
$(bnd + bhn^2 + d^2)$

("arithmetric" is the slide's own spelling.)

Then: "Arithmetic intensity (compute/memory) is high
$O\left(\left(\frac{1}{k} + \frac{1}{bn}\right)^{-1}\right)$ - we can keep our GPUs
running"

## Slide 59 — GQA/MQA – Reducing attention head cost

Text: "What about the *incremental* case when we generate text?", then in bold
"**Key difference:** can't parallelize the generation process – needs to be step by
step", then "In this case – we need to incrementally re-compute/update attention
via the 'KV cache'".

**Figure — a pasted frame of a KV-caching animation**, headed "**Step 1**". It is
split into two horizontal bands by a dashed line, labelled down the left in blue
"Without cache" (upper) and "With cache" (lower). Each band shows the same
left-to-right computation with column headings **Q**, **K$^\top$**, **QK$^\top$**,
**V**, **Attention**:

- a yellow box "Query Token 1" (shape annotated `(1, emb_size)`),
- "x" a green vertical box "Key Token 1" (shape `(emb_size, 1)`),
- "=" a small green box labelled $q_1k_1$ (shape `(1, 1)`),
- "x" an orange box "Value Token 1" (shape `(1, emb_size)`),
- "=" a green box "Token 1" (shape `(1, emb_size)`).

At step 1 the two bands are identical, since there is nothing cached yet. A legend
at the foot gives two swatches: a white square for "Values that will be masked" and
a blue square for "Values that will be taken from cache".

Credit at bottom right: "[Animation from
https://medium.com/@joaolages/kv-caching-explained-276520203249]"

## Slide 60 — GQA/MQA – Reducing attention head cost

Text only; no figure.

"What's the incremental arithmetic intensity?"

In bold, with small labels printed above the memory terms ("K, V" over $bn^2d$,
"projection" over $nd^2$):

**Total arithmetric operations** $(bnd^2)$, **total memory accesses**
$(bn^2d + nd^2)$

"Arithmetic intensity is not good
$O\left(\left(\frac{n}{d} + \frac{1}{b}\right)^{-1}\right)$ - need large batches +
short seq length (n) or big model dimensions (d)"

"Is there some way around this? The n/d term is difficult to reduce."

## Slide 61 — MQA – just have fewer key dimensions.

Text: "**Key idea** – have multiple queries, but just one dimension for keys and
values"

**Figure — a pasted MQA dataflow diagram** (from the Fireworks AI blog). A pink box
"Embedded input sequence" at the top left feeds a blue rounded box "QKV projection
+ split" at the top right. That splits into three yellow boxes in a row: "K
(shared)", "V (shared)" and a stack of "Q (per head)". Two pink boxes down the
left, "K Cache" and "V Cache", are labelled with a blue annotation "Cache current
token" and each has a small blue tab on its right edge; green arrows labelled
"broadcast" (drawn as dashed green ellipses) carry the single shared K and V out
to a stack of blue rounded boxes on the right labelled "Dot product attention (per
head)". That feeds a stack of yellow boxes "Emb (per head)", which feeds an arrow
labelled in blue "output projection, …".

Text: "We have much fewer items to move in and out of memory (KV Cache)"

In bold: **Total memory access** $(bnd + bn^2k + nd^2)$, **Arithmetic intensity**
$O\left(\left(\color{red}{\frac{1}{d}} + \frac{n}{d\color{red}{h}} + \frac{1}{b}\right)^{-1}\right)$

(In the rendered formula the numerator "1" of $\frac{1}{d}$ and the "h" in the
denominator $dh$ are printed in red, marking what changed relative to slide 60.)

Credit at the foot: "[figure from
https://blog.fireworks.ai/multi-query-attention-is-all-you-need-db072e758055]"

## Slide 62 — Additional extensions – GQA

Text: "Don't go all the way to one dimension of KV – have fewer dims"

**Figure — the standard GQA schematic from Ainslie et al 2023**, three panels side
by side titled "Multi-head", "**Grouped-query**" (bold) and "Multi-query". Each
panel has three labelled rows, named down the left of the leftmost panel:
**Values** (orange rectangles, top), **Keys** (pink rectangles, middle) and
**Queries** (blue rectangles, bottom), with dotted lines connecting keys/values to
the queries they serve.

- *Multi-head:* eight value rectangles, eight key rectangles and eight query
  rectangles, connected one-to-one by short dotted vertical lines.
- *Grouped-query:* four value rectangles and four key rectangles, but eight query
  rectangles; each key/value fans out by dotted lines to two queries.
- *Multi-query:* a single value rectangle and a single key rectangle, fanning out
  by dotted lines to all eight query rectangles.

Text below: "Simple knob to control expressiveness (key-query ratio) and inference
efficiency", and in bold "**More recently –** MLA (multihead latent attention) from
deepseek v2"

## Slide 63 — Does MQA hurt? Sometimes..

The slide's only text is the title and two figure captions: "Small PPL hit w/ MQA
[Shazeer 2019]" over the left figure and "Low/no hit w/ GQA [Ainslie 2023]" over
the right ones. The three pasted figures are the slide.

**Figure 1 (left, Shazeer 2019) — a pasted results table**, captioned "Table 3:
Billion-Word LM Benchmark Results." Columns: Attention, $h$, $d_k, d_v$, $d_{ff}$,
and (after a vertical rule) dev-PPL. A horizontal rule separates the first two rows
from the last four.

| Attention | $h$ | $d_k, d_v$ | $d_{ff}$ | dev-PPL |
| --- | --- | --- | --- | --- |
| multi-head | 8 | 128 | 8192 | **29.9** |
| multi-query | 8 | 128 | 9088 | 30.2 |
| multi-head | 1 | 128 | 9984 | 31.2 |
| multi-head | 2 | 64 | 9984 | 31.1 |
| multi-head | 4 | 32 | 9984 | 31.0 |
| multi-head | 8 | 16 | 9984 | 30.9 |

Multi-query costs 0.3 perplexity against multi-head (30.2 vs the bolded 29.9), but
is much better than any of the four reduced-head multi-head baselines below the
rule.

**Figure 2 (top right, Ainslie et al 2023) — a scatter plot** with x-axis "Time per
sample (ms)" (ticked 0, 0.5, 1, 1.5) and y-axis "Performance" (ticked 46, 46.5,
47). **Four** labelled points, no legend:

- *Blue dot*, "GQA-XXL", at about (0.28 ms, 47.2) — top left, i.e. fast and best.
- *Pink dot*, "MHA-XXL", at about (1.5 ms, 47.3) — top right, essentially the same
  performance for roughly five times the latency.
- *Orange dot*, "MQA-XXL", at about (0.24 ms, 46.6) — fastest, but below GQA on
  performance.
- *Pink dot*, "MHA-Large", at about (0.38 ms, 45.95) — the lowest performing point.

**Figure 3 (bottom right, Ainslie et al 2023) — a line chart** with x-axis "GQA
groups" (ticked 1, 4, 8, 16, 32, 64) and y-axis "Time per sample (s)" (ticked 1
and 2, with the plot extending to about 2.6). **Three** series:

- *Pink dotted horizontal line*, "MHA": constant at about 2.6 s across the whole
  x-range — a reference level, not a swept curve.
- *Blue solid line with square markers*, "GQA": about 0.42 s at 1 group, 0.42 s at
  4, 0.45 s at 8, 0.5 s at 16, 0.8 s at 32, and about 2.6 s at 64 groups, where it
  meets the MHA line.
- *Orange dotted horizontal line*, "MQA": constant at about 0.40 s across the whole
  x-range — again a reference level.

The point is that GQA stays near MQA's latency until the number of groups
approaches the full head count, at which point it degenerates to MHA.

## Slide 64 — Sparse / sliding window attention

Text: "**Attending to the entire context can be expensive (quadratic).**" and
"Build sparse / structured attention that trades off expressiveness vs runtime
(GPT3, GPT-OSS, Gemma4)". Credit at bottom right: "[Child et al 2019]".

**Figure — the three-panel attention-mask figure from Child et al 2019**, drawn as
blue-on-grey grids. Each panel has two small grids on top (showing the receptive
field of one query, and the mask for a single position) and one large square grid
below (the full attention mask, with the diagonal in dark blue and the attended
cells in mid blue on a grey background). The three panels are captioned:

- "(a) Transformer" — the large grid is a solid lower-triangular block: every
  position attends to all previous positions.
- "(b) Sparse Transformer (strided)" — the large grid keeps a narrow band along the
  diagonal plus regularly spaced diagonal stripes running back through the
  sequence.
- "(c) Sparse Transformer (fixed)" — the large grid keeps a local block near the
  diagonal plus fixed vertical columns of attended positions, giving a staircase of
  blocks.

## Slide 65 — Current standard trick – interleave 'full' and 'LR' attention

Text: "From Cohere Command A – Every 4$^{th}$ layer is a full attention"

**Figure — a pasted architecture diagram of Cohere's Command A.** On the left, a
grey panel headed "**Command A**" stacks, top to bottom: a "**Tokenizer**" box
annotated "· Vocabulary size: 255,000 / · Multilingual / · Special tokens (for chat
turns, tool calls…)"; an "Input embeddings" bar; then rows "Transformer Block 1",
"Transformer Block 2", "Transformer Block 3" each showing "Self-Attention (SWA)"
beside "MLP"; then "Transformer Block 4" showing "Self-Attention (full)" beside
"MLP"; a row of ellipsis dots annotated "Interleaved SWA and full attention (3:1
ratio)"; then "Transformer Block N" with "Self-Attention (full)" and "MLP"; and
finally "LM Head" with "Output embeddings". A vertical annotation runs down the
right edge of the stack. The SWA blocks' attention chips are tinted purple and the
full-attention ones green.

On the right, two exploded block diagrams. The upper one is headed "Command A
Transformer Block (SWA)" and contains a purple-outlined "**Sliding Window
Self-Attention**" box annotated "· Grouped-query attention / · RoPE positional
embeddings", beside a grey "**MLP**" box annotated "· SwiGLU activation / · No bias
terms". The lower one is headed "Command A Transformer Block (Full)" and contains a
grey "**Full Self-Attention**" box annotated "· Grouped-query attention / · No
positional embeddings", beside the same "**MLP**" box with "· SwiGLU activation / ·
No bias terms".

Text below: "Long-range info via NoPE, short-range info via RoPE + SWA." and, in
bold, "**Other models –** LLaMA 4, Gemma 3, Gemma 4, OLMo 3 does SWA+Full RoPE."

## Slide 66 — Other recent examples of interleaved attention

Three pasted figures side by side, each with a caption printed beneath it in the
slide's own font: "Gemma 4" (left), "Olmo 3" (middle), "Qwen 3.5 / Qwen 3 Next"
(right). Source URL at the bottom right of the slide:
`https://newsletter.maartengrootendorst.com/p/a-visual-guide-to-gemma-4`

**Figure 1 (left, "Gemma 4") — a diagram of Gemma 4's layer stack.** Down the
centre runs a vertical arrow through six rounded boxes: four green "**Local
Attention**" boxes, then a pink "**Global Attention**" box, then a vertical
ellipsis, then a final pink "**Global Attention**" box. Callouts around it:

- Top left, a row of token chips reading "Gemma / 4 / is / a / great / **tool**"
  (the last chip purple), labelled "**Sliding Window** Attention", with a leader
  line to the second "Local Attention" box.
- Bottom left, an eight-cell greyscale strip whose first two cells carry a purple
  diagonal wash, labelled "**p-RoPE**" above and "positional information" beneath a
  bracket under those first two cells, with a leader line to the first "Global
  Attention" box.
- Right, a tall two-tone purple column annotated "dimensionality x2" fanning out by
  dotted lines to eight shorter purple columns, labelled "**Keys = Values** (8
  Queries per Key)", with a leader line to the first "Global Attention" box.
- Bottom right, "last layer is always **global attention**." pointing at the final
  pink box.

**Figure 2 (middle, "Olmo 3") — a pasted hyperparameter table** from the OLMo 3
report, with alternating pale-blue row shading:

| | |
| --- | --- |
| Gradient clipping | 1.0 |
| Z-loss weight | $10^{-5}$ |
| Weight decay on embeddings | No |
| Sliding window attention | 3/4 of layers; 4,096 tokens |
| RoPE scaling | YaRN on full attn. layers |
| RoPE $\theta$ | $5 \cdot 10^5$ |
| Layer norm applied to | Outputs |

**Figure 3 (right, "Qwen 3.5 / Qwen 3 Next") — a two-block architecture diagram.**
The left column shows two stacked block types: an upper one, marked "1×" in the
margin, containing a blue "Mixture of Experts" box over a yellow "Zero-Centered
RMSNorm" bar, and a blue "Gated Attention" box over another "Zero-Centered
RMSNorm" bar; and a lower one, marked "3×", containing a blue "Mixture of Experts"
box over a "Zero-Centered RMSNorm" bar, and a blue "Gated DeltaNet" box over a
"Zero-Centered RMSNorm" bar. The right column expands each. The upper expansion
shows a "Linear" box feeding an "Output Gate" (marked "Sigmoid"), a "Scaled Dot
Product Attention" box fed by $q$, $k$, $v$ paths through "Partial RoPE" and
"Zero-Centered RMSNorm" boxes, and four "Linear" boxes at the bottom. The lower
expansion shows a "Linear" box, an "Output Gate" marked "SiLU", a "Gated Delta
Rule" box fed by $q$, $k$, $v$, $a$, $\beta$ paths, an "L2" normalisation node, two
"Conv" boxes and "Linear" boxes at the bottom.

## Slide 67 — Recap, conclusion, etc.

Text: "Many aspects (arch, hparams) of transformers are in common across the big
LMs", and beneath the figure, "Major differences? Position embeddings, activations,
tokenization"

**Figure — the complete dense-model database screenshot**, shown at full width with
every column visible. Its columns, left to right, are: **Name**, **Has pa…** (a
"has paper?" select column whose chips read "Yes" in purple, "Kind of" in red, or
"Ac…" for GPT4), **Link** (a URL column showing `arxiv.org/abs/…`,
`cdn.openai.com/res…er.pdf`, `huggingface.co/…`, `github…`, `cohere.com/…`),
**Year**, **Vocab count**, **Norm**, **Parallel Layer**, **Pre-norm**, **Position
embedding**, **Activations**, **MoE** (a checkbox column, checked on exactly two
rows — **GPT4** and **Mixtral of Experts** — and unchecked on every other row),
**Parametrization** (empty on every row except **Phi3 (small) – same for phi4**,
which carries a filled tag reading **MuP**), **Other tricks**,
**MLP factor**, **num_layers**, **model_dim**, **Aspect ratio (d/layer)**,
**num_heads**, and **drop_rate**. The categorical columns repeat the values
transcribed under slides 7, 9, 29 and 51; the numeric columns, including
**num_heads** which appears only here, are:

| Name | MLP factor | num_layers | model_dim | Aspect ratio (d/layer) | num_heads | drop_rate |
| --- | --- | --- | --- | --- | --- | --- |
| Original transformer | 4 | 6 | 512 | 85 | 8 | 0.1 |
| GPT | 4 | 12 | 768 | 64 | 12 | 0.1 |
| GPT2 | 4 | 48 | 1600 | 33 | 12 | 0.1 |
| T5 (11B) | 64 | 24 | 1024 | 43 | 128 | 0.1 |
| GPT3 (175B) | 4 | 96 | 12288 | 128 | 96 | 0.1 |
| mT5 | 2.5 | 24 | 4096 | 171 | 64 | 0 |
| T5 (XXL 11B) v1.1 | 2.5 | 24 | 4096 | 171 | 64 | 0 |
| Gopher (280B) | 4 | 80 | 16384 | 205 | 128 | |
| Anthropic LM (not claude) | 4 | 64 | 8192 | 128 | | |
| LaMDA | 8 | 64 | 8192 | 128 | 128 | |
| GPTJ | | 28 | 4096 | 146 | 16 | 0 |
| Chinchilla | 4 | 80 | 8192 | 102 | 64 | |
| PaLM (540B) | 4 | 118 | 18432 | 156 | 48 | 0 |
| OPT (175B) | 4 | 96 | 12288 | 128 | 96 | 0.1 |
| BLOOM (175B) | 4 | 70 | 14336 | 205 | 112 | 0 |
| GPT-NeoX | 4 | 44 | 6144 | 140 | 64 | 0 |
| GPT4 | | | | 0 | | |
| LLaMA (65B) | 2.6875 | 80 | 8192 | 102 | 64 | 0 |
| LLaMA2 (70B) | 3.5 | 80 | 8192 | 102 | 64 | 0 |
| Mistral (7B) | 3.5 | 32 | 4096 | 128 | 32 | 0 |
| Baichuan 2 | 2.68 | 32 | 4096 | 128 | 32 | 0 |
| Mixtral of Experts | | | | 0 | | |
| Yi (34B) | 2.857142 | 60 | 7168 | 119 | 56 | 0 |
| DeepSeek (67B) | 2.6875 | 95 | 8192 | 86 | 64 | 0 |
| Qwen (14B) | 2.675 | 40 | 5120 | 128 | 40 | 0.1 |
| OLMo | 2.6875 | 32 | 4096 | 128 | 32 | 0 |
| Command R+ | 2.75 | 64 | 12288 | 192 | 96 | |
| Reka Flash | | | | 0 | | |
| Llama 3 (70B) | 3.5 | 80 | 8192 | 102 | 64 | 0 |
| Phi3 (small) – same for phi4 | 3.5 | 32 | 4096 | 128 | 32 | |
| Falcon 2 11B | 4 | 60 | 4096 | 68 | 32 | |
| Qwen 2 (72b) – same for 2.5 | 3.609 | 80 | 8192 | 102 | 64 | |
| Nemotron-4 (340B) | 4 | 96 | 18432 | 192 | 96 | 0 |
| Gemma 2 (27B) | 8 | 46 | 4608 | 100 | 32 | |
| Olmo 2 | 2.6875 | 32 | 4096 | 128 | 32 | |
| SmolLM2 (1.7B) | 4 | 24 | 2048 | 85 | 32 | |
| Gemma 3 | 4 | 62 | 5376 | 87 | 32 | |
| Command A | | | | 0 | | |
| Qwen 3 (8B) | 3 | 36 | 4096 | 114 | 32 | |
| OLMo 3 (7B) | 2.6875 | 32 | 4096 | 128 | 32 | |
| Marin 8B | 3.5 | 32 | 4096 | 128 | 32 | |
| Ministral 3 (8B) | 3.5 | 34 | 4096 | 120 | | |
| LFM 2.5 (1.2B) | 4 | 16 | 2048 | 128 | 32 | 0.1 |
| Gemma 4 E4B (8B) | 4 | 42 | 2560 | 61 | 8 | |

Blank cells are blank in the screenshot; an aspect ratio of 0 marks a row whose
`num_layers`/`model_dim` were never filled in (GPT4, Mixtral, Reka Flash, Command
A). This is the last slide of the deck.
