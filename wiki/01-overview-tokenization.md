# Lecture 1 — Overview and Tokenization

Percy Liang opens the third offering of CS336 by arguing for why the course
exists, mapping the five units that make up the quarter, and then teaching the
first unit: tokenization. The through-line, stated at [10:50] and returned to at
[1:02:23], is that everything in the course is about **efficiency** — building the
best model you can from a fixed budget of compute and data. Tokenization is the
first place that principle bites, because the elegant choice (feed the model raw
bytes) is the compute-inefficient one.

By the end you should be able to say what a tokenizer is, why character-, byte-
and word-level tokenizers are each unusable, and how byte-pair encoding is trained
and applied. Everything after roughly [1:04:46] is the technical core; everything
before it is the course's argument for itself.

- **Video:** [Lecture 1: Overview, Tokenization](https://www.youtube.com/watch?v=JuoVZkPBiKk) (79 min)
- **Course material:** [`lecture_01.py`](../raw/slides/01-overview-tokenization.md) — an
  [executable lecture](executable-lectures.md), not a slide deck.
  Run it live at [the trace viewer](https://cs336.stanford.edu/lectures/?trace=lecture_01).
- **Transcript:** [edited](../raw/transcripts/01-overview-tokenization.md) ·
  [verbatim captions](../raw/transcripts/original/01-overview-tokenization.md)

## Why the course exists

The problem Percy names at [3:08] is that researchers have drifted away from the
technology they study. He gives the drift as three dates: in 2016 researchers
implemented and trained their own models; by 2018 they downloaded a pre-trained
model such as BERT and fine-tuned it; today they prompt an API model.

Moving up levels of abstraction is normally a good trade — but his objection at
[3:54] is specific: **these abstractions are leaky**, in a way that programming
languages and operating systems are not. When a prompted model cannot do what you
want, there is no recourse, because you have no access to the layer where the
problem lives. So for fundamental research you have to "tear up the whole stack,"
and the course's philosophy is *understanding via building*.

The complication, from [4:40], is that the thing worth understanding has been
industrialized beyond the reach of a classroom. GPT-4 reportedly cost $100M to
train; xAI has built a 230K-GPU cluster for Grok. And the frontier labs publish
nothing: the GPT-4 technical report says explicitly that, given the competitive
landscape and safety implications, it will disclose nothing about how the model
was built.

### Why small models are not simply small frontier models

The obvious workaround — build a model under 1B parameters and learn from it —
carries a real risk, and Percy gives two examples of things that only appear at
scale ([5:27]–[6:14]):

1. **The FLOP budget shifts with scale.** The fraction of FLOPs spent in the MLP
   layers is around 44% at small scale and about 80% at 175B. So an optimization
   that pays off on attention at small scale may be worth much less at the scale
   you care about.
2. **Behaviour emerges with scale.** On zero-shot and few-shot tasks, small models
   look like nothing is working at all, and improvement appears only past a
   critical size.

### What does transfer

His answer at [7:01] is to split knowledge into three kinds, and to be honest
about which the course can deliver:

| Kind | What it is | Does it transfer? |
| --- | --- | --- |
| **Mechanics** | How things work — what a Transformer is, how model parallelism works | Yes — taught here |
| **Mindset** | Squeezing the most out of hardware; taking scaling seriously | Yes — taught here |
| **Intuitions** | Which data and modelling decisions actually yield good accuracy | Only partly; these need scale |

He is unusually candid about the third at [8:33]. Some design decisions are simply
not justifiable from principle and come only from experimentation — his example is
the conclusion of the Noam Shazeer paper that introduced SwiGLU, which says of its
own architectures: *"We offer no explanation as to why these architectures seem to
work; we attribute their success, as all else, to divine benevolence."*

### The bitter lesson, restated

At [9:19] Percy corrects what he considers a common misreading. The wrong
interpretation is that scale is all that matters and algorithms don't. The right
one is that **algorithms that scale are what matter**. The compact form he uses is

$$\text{accuracy} = \text{efficiency} \times \text{resources}$$

where efficiency is output over input and resources is the input. Efficiency
matters *more* as scale grows, not less: at small scale a run that takes twice as
long just means waiting; at frontier scale it can mean hundreds of millions of
dollars, and even a 5% improvement is a big deal. He backs this with a 2020 OpenAI
result showing 44× algorithmic efficiency improvement on ImageNet between 2012 and
2019 — a gain that compounds with hardware improvement rather than competing with
it. See [efficiency as the organizing principle](efficiency.md).

## A short history of language models

From [11:36], Percy sketches the lineage, which is worth having in one place:

- **Pre-neural.** Shannon used a language model to measure the entropy of English
  in 1950. N-gram models were then a workhorse component inside machine
  translation and speech recognition systems for decades — not the whole system,
  but the part that made the output fluent.
- **Neural ingredients (2010s).** LSTMs; Yoshua Bengio's 2003 neural language
  model (a feedforward network over a small context, not an LSTM); sequence-to-
  sequence modelling, which "boldly said we can compress a whole sentence into a
  vector"; Adam; attention, developed for machine translation; the Transformer,
  built on top of it and also for machine translation; then mixture of experts and
  model parallelism.
- **Early foundation models.** ELMo and BERT — models you pre-train on a lot of
  text and then fine-tune for a downstream task. Then T5, which Percy describes at
  [13:09] as foreshadowing "prompt in, response out."
- **Embracing scaling.** OpenAI "opened the floodgates": GPT-2, then scaling laws,
  then GPT-3 at 175B with in-context learning. Google answered with PaLM (540B),
  which turned out to be under-trained — while DeepMind, then not integrated with
  Google, had worked out compute-optimal scaling ([14:40]).
- **Open models.** EleutherAI's grassroots datasets and models; Meta's 175B
  replication attempt, which hit many hardware problems; Hugging Face/BigScience's
  BLOOM. Then, in the last three years, Llama, Mistral, and a wave of Chinese
  models — DeepSeek, Qwen, Kimi, GLM and others ([15:26]).

Two distinctions he draws are worth keeping straight, because the course depends
on them. **Open-weight** models ship weights and a paper. **Open-source** models
ship weights, paper, code *and* data — AI2's Olmo, NVIDIA's Nemotron, and Marin,
the project Percy works on. At [16:12] he says plainly why he emphasizes this:
*"this course would not be possible without these models."* Even incomplete open
papers let you triangulate how frontier systems are built; he notes the data
mixture is the detail that is still almost never disclosed.

### What "language model" has meant

A neat framing from [17:00]: the word has meant something different every few
years — in 2018 something you fine-tune, in 2020 something you prompt, in 2022
something you talk to, and by 2026 something that acts autonomously. His judgement
at [18:32] is that the *fundamentals* have barely moved — GPUs and kernels,
stochastic-gradient optimization, the Transformer and attention — while the
*specs* have: longer context, and inference efficiency mattering far more.

## How the course is organized

Five units, each with an assignment. This is covered from [27:04]; the detail
lives on [the course map](course-map.md).

| Unit | Assignment | Covers |
| --- | --- | --- |
| Basics | 1 | Tokenization, architecture, training loop |
| Systems | 2 | Kernels (Triton), parallelism, inference |
| Scaling laws | 3 | Scaling recipes, extrapolation |
| Data | 4 | Evaluation, curation, filtering, dedup, mixing |
| Alignment | 5 | SFT, RLHF, DPO, GRPO |

At [1:02:23] he closes the loop by showing that each unit is the efficiency
principle applied to a different resource — systems to compute, tokenization to
sequence length, architecture to memory and FLOPs, data filtering to not wasting
gradient steps on bad data, and scaling laws to doing hyperparameter search on
small models instead of large ones. The last line of the syllabus section is a
forward reference worth noting: *"Tomorrow, we will become data-constrained."*
Today, the course assumes compute is the binding constraint.

A remark on logistics that is genuinely load-bearing for a self-study reader: at
[20:12] Percy quotes a course evaluation saying the *first* assignment was
comparable to all five CS224N assignments plus the final project, then adds he's
been told this is exaggerated. Budget accordingly.

## Tokenization

The technical unit starts at [1:04:46] and Percy flags Andrej Karpathy's
[tokenization video](https://www.youtube.com/watch?v=zduSFxRajkE) as the
inspiration for it.

### The problem

Raw text is Unicode strings. A language model, though, places a probability
distribution over sequences of **tokens** — integer indices. So you need an
`encode` that maps strings to indices and a `decode` that maps back, and the pair
must **round-trip**: at [1:07:04] he is blunt that a tokenizer which does not
round-trip is broken.

```python
class Tokenizer(ABC):
    def encode(self, string: str) -> list[int]: ...
    def decode(self, indices: list[int]) -> str: ...
```

The metric that governs every design choice here is the **compression ratio**:

$$\text{compression ratio} = \frac{\text{number of UTF-8 bytes in the string}}{\text{number of tokens}}$$

A higher ratio means a shorter sequence for the same text, which matters because
attention is quadratic in sequence length ([1:07:50]). You can always buy a higher
ratio by enlarging the vocabulary, but you pay in sparsity — every vocabulary
entry is a distinct symbol the model must learn about, and rare ones get few
gradient updates. Contemporary multilingual tokenizers sit at roughly 100k–200k
tokens.

Full treatment: [tokenization](tokenization.md).

### The four tokenizers, by the numbers

The lecture builds up through three bad tokenizers before arriving at BPE. All
four are run on the same string, `"Hello, 🌍! 你好!"` — 13 characters, 20 UTF-8
bytes.

| Tokenizer | Tokens | Vocabulary | Compression ratio | Verdict |
| --- | --- | --- | --- | --- |
| Character (`ord`) | 13 | ~150K Unicode characters (127,758 here) | ≈1.54 | "The worst of both worlds" |
| Byte (UTF-8) | 20 | 256 | exactly 1.0 | Tiny vocab, hopeless sequence length |
| Word (regex) | 8 (on a different string) | Unbounded | 5.5 | Good ratio, no fixed vocab, UNK problem |
| **BPE** | — | Chosen by you | Data-dependent | The working answer |
| GPT-5 (`o200k_base`) | 8 | 200,019 | 2.5 | What production actually uses |

Each row is worth a sentence:

**Character-level** ([1:08:35]) maps each Unicode code point through `ord`. The
vocabulary is enormous — around 150K characters — and most of it is rare. The
single 🌍 in the test string is what pushes the observed vocabulary bound to
127,758, which is exactly the pathology: you pay for a huge vocabulary and still
only get 1.54 bytes per token.

**Byte-level** ([1:10:06]) is the mirror image. Every index is in 0–255, so the
vocabulary is a tidy 256 — but the compression ratio is *exactly* 1.0 by
construction, so sequences are as long as the text is bytes. Given quadratic
attention, that is disqualifying.

**Word-level** ([1:10:51]) splits on a regex (`\w+|.`). Tokens are meaningful,
because humans invented words, and the ratio is good — 5.5 on Percy's example.
But the vocabulary is the number of distinct chunks in the training data, which is
not merely large but *unbounded*: at test time you meet a word you have never
seen. The classical fix, an `UNK` token, he calls "really ugly," and it corrupts
perplexity calculations ([1:11:39]).

**BPE** ([1:11:39] onward) resolves the dilemma by *training* the vocabulary on
data: common byte sequences become single tokens, rare ones stay split into
several, and nothing is ever out-of-vocabulary. See
[byte-pair encoding](byte-pair-encoding.md) for the algorithm and a fully worked
example.

### What the GPT-5 tokenizer actually does

Running `tiktoken`'s `o200k_base` on the test string gives 8 tokens for 20 bytes —
a compression ratio of 2.5 — from a 200,019-entry vocabulary ([1:07:04]). Two
details in that output repay attention, and they are the concrete form of the
"adaptive computation" argument Percy makes at [28:36]:

- `你好` — two Chinese characters, six UTF-8 bytes — is a **single token**.
- 🌍 is **two** tokens, neither of which is valid UTF-8 alone.

Common things get cheap; rare things get expensive. That is the whole point.

He also draws out three observations about tokenizer behaviour at [1:06:19] that
tend to surprise people, and which are why he says tokenizers are "kind of
annoying":

- A word and its **preceding space** form one token, so most tokens you see are
  really `␣word`.
- The same word gets **different indices** at the start of a string versus mid-
  string — `hello` and `␣hello` are unrelated integers.
- **Numbers** are split every few digits, sometimes unpredictably. Some tokenizers
  force one digit per token, but that inflates sequence length — another instance
  of the same trade.

### The dream of no tokenizer

At [29:24] Percy says he hopes each year not to have to teach this, because the
goal is an end-to-end architecture over raw bytes — the lecture cites ByT5,
MegaByte, BLT, T-FREE and H-Net. But none has been scaled to the frontier, and
frontier models still use tokenizers, so it stays in the syllabus.

His closing argument at [1:17:54] is the part worth remembering, because it
survives whatever replaces BPE. Any end-to-end solution still has to satisfy two
properties:

1. The model must operate on **chunks** — abstractions over the sequence — not raw
   units. This is clearest for video or DNA, where individual bytes carry very
   little signal.
2. Those chunks must be **variable**, so that computation is allocated adaptively.
   Not all bytes deserve equal treatment.

## Where this goes next

The lecture ends by pointing at Lecture 2, on **resource accounting** — "a baby
systems," in his words — followed by architectures. The formula previewed at
[36:18], $C = 6ND$ for the training FLOPs of a model with $N$ parameters over $D$
tokens, is derived there.

Note that Lecture 1 only *previews* the systems, scaling-law, data and alignment
material; the substance arrives in later lectures, which this knowledge base does
not yet cover. [Scaling laws](scaling-laws.md) records what Lecture 1 says about
them and marks clearly where the preview stops.
