# Tokenization

A **tokenizer** converts between raw text and the sequences of integers a language
model actually operates on. It is the first thing CS336 builds, and the first
place the course's efficiency argument shows up in concrete form.

Formally it is a pair of functions that must round-trip:

```python
class Tokenizer(ABC):
    def encode(self, string: str) -> list[int]: ...
    def decode(self, indices: list[int]) -> str: ...
```

`decode(encode(s)) == s` for every string `s`. Percy Liang is blunt about this in
[Lecture 1](01-overview-tokenization.md) at [1:07:04]: if you implement a
tokenizer that does not round-trip, you have a problem.

## The two quantities that decide everything

Every tokenizer design is a trade between two numbers.

**Compression ratio** — bytes per token:

$$\text{compression ratio} = \frac{\text{number of UTF-8 bytes in the string}}{\text{number of tokens}}$$

In the lecture's own code this is

```python
def get_compression_ratio(string: str, indices: list[int]) -> float:
    num_bytes = len(bytes(string, encoding="utf-8"))
    num_tokens = len(indices)
    return num_bytes / num_tokens
```

Higher is better, because a higher ratio means fewer tokens for the same text, and
attention costs $O(n^2)$ in the sequence length $n$. Percy makes this link
explicitly at [1:07:50].

**Vocabulary size** — the number of distinct token values. You can always raise the
compression ratio by enlarging the vocabulary, since more entries let you cover
longer chunks with a single token. But you pay in **sparsity**: every vocabulary
entry is a symbol the model has to learn an embedding for, and rare entries get
correspondingly few gradient updates. Contemporary multilingual tokenizers land
around 100k–200k tokens ([1:07:50]).

So the design problem is: get the compression ratio up without letting the
vocabulary blow up, and without leaving anything unrepresentable.

## Why tokenization exists at all

The efficiency framing from [28:36] gives two reasons, and the second is the one
people miss:

1. **Shorter sequences.** A thousand bytes become roughly 250 tokens. Since
   attention is quadratic, this is a large, direct saving.
2. **Adaptive computation.** A tokenizer lets you spend more of the model's
   capacity on the interesting parts of the input. Frequent, predictable stretches
   collapse to one token; rare or information-dense stretches stay spread across
   several, and therefore get more of the model's attention and more forward-pass
   compute.

The second point is why tokenization belongs in a course about efficiency rather
than a course about preprocessing. It is not merely a compression step — it is a
decision about where the model's compute goes.

## Three approaches that don't work

Lecture 1 builds up through three tokenizers, each of which fails in an
instructive way. All are run on the same string, `"Hello, 🌍! 你好!"` — 13 Unicode
characters, 20 UTF-8 bytes. (The values below come from running the lecture's own
code; see [the transcription](../raw/slides/01-overview-tokenization.md) for the
full traces.)

### Character-level

Map each Unicode code point to an integer with `ord`, and back with `chr`.

```python
class CharacterTokenizer(Tokenizer):
    def encode(self, string): return list(map(ord, string))
    def decode(self, indices): return "".join(map(chr, indices))
```

On the test string: 13 tokens, compression ratio ≈1.54. The vocabulary is roughly
150K — the number of Unicode characters — and in this example the observed bound
is 127,758, driven there entirely by the single 🌍 (whose code point is 127,757).

Two problems, per [1:08:35]: the vocabulary is very large, and most of it is rare,
which is an inefficient use of a vocabulary. Percy's verdict is that this is *"the
worst of both worlds — large vocabulary, low compression ratio."*

### Byte-level

Encode to UTF-8 and take the bytes. Some characters are one byte (`a` → `b"a"`);
others are several (🌍 → `b"\xf0\x9f\x8c\x8d"`).

```python
class ByteTokenizer(Tokenizer):
    def encode(self, string): return list(map(int, string.encode("utf-8")))
    def decode(self, indices): return bytes(indices).decode("utf-8")
```

The vocabulary is a tidy **256**, which is as small as it could possibly be. But
the compression ratio is **exactly 1.0** — necessarily, since tokens *are* bytes;
the lecture asserts this in code. Sequences are therefore as long as the text is
bytes, which against quadratic attention is disqualifying ([1:10:06]).

This is the elegant option, and its failure is the clearest statement of the
course's thesis: raw bytes are the principled representation and are ruled out
purely on compute grounds. Hence the interest in
[tokenizer-free architectures](#the-tokenizer-free-dream), which try to make
byte-level work by changing the model instead.

### Word-level

Split on a regex — the lecture uses `\w+|.`, which keeps alphanumeric runs
together:

```python
chunks = regex.findall(r"\w+|.", "I'll say supercalifragilisticexpialidocious!")
# ['I', "'", 'll', ' ', 'say', ' ', 'supercalifragilisticexpialidocious', '!']
```

Compression ratio 5.5 on that string — the best of the three — and the tokens are
meaningful, because humans invented words and words have stable semantics.

It fails on the vocabulary side, and worse than it first appears ([1:11:39]). The
vocabulary is "the number of distinct chunks in the training data," which is not a
number the method determines. Rare words get very few updates. And critically, the
vocabulary is effectively **unbounded**: at test time you will meet a word you have
never seen. The classical answer is a special `UNK` token, which Percy calls
"really ugly" and which corrupts perplexity calculations — you cannot assign
sensible probability mass to a symbol that stands for "something else."

## What works: BPE

[Byte-pair encoding](byte-pair-encoding.md) resolves the trilemma by *training*
the vocabulary on data rather than fixing it by fiat. Start from bytes — so the
vocabulary begins at 256 and nothing is ever out-of-vocabulary — then repeatedly
merge the most frequent adjacent pair into a new token. Common sequences end up as
single tokens; rare ones stay split into several.

You choose the vocabulary size directly, by choosing how many merges to run. That
is exactly the knob the other three approaches failed to provide.

## What a real tokenizer looks like

OpenAI's `o200k_base` (the GPT-5 tokenizer, reached in the lecture through
`tiktoken.get_encoding("o200k_base")`) has **200,019** entries. On the test string
it produces 8 tokens for 20 bytes — a compression ratio of **2.5**.

The token-by-token output is worth reading, because it is adaptive computation
made visible:

| Token ids | Decoded |
| --- | --- |
| `13225` | `Hello` |
| `11` | `,` |
| `130321, 235` | 🌍 — **two** tokens, neither valid UTF-8 alone |
| `0` | `!` |
| `220` | space |
| `177519` | `你好` — **one** token for two characters, six bytes |
| `0` | `!` |

A common CJK greeting has earned a single token. A rare emoji costs two. This is
the tokenizer spending its vocabulary where the data is.

### Three behaviours that surprise people

From [1:06:19], and each a consequence of training on raw text:

- **Leading spaces belong to the token.** Most tokens you will see are really
  `␣word`, not `word`.
- **Position changes the index.** `hello` at the start of a string and `␣hello`
  mid-string are two completely different integers with, as Percy puts it, nothing
  to do with each other.
- **Numbers fragment.** They are split every few digits, and not always
  predictably — GPT-2 tokenizes `123456789` as `123|45|67|89` but `1234567890` as
  `123|45|678|90`. Some tokenizers force one digit per token for arithmetic
  reliability, at the cost of sequence length.

These are the concrete reasons Percy calls tokenizers "kind of annoying" and why
people want to be rid of them.

## The tokenizer-free dream

At [29:24] Percy says he hopes each year not to have to teach this. The goal is a
model that operates end-to-end on bytes, and Lecture 1 cites five attempts —
[ByT5](https://arxiv.org/abs/2105.13626),
[MegaByte](https://arxiv.org/pdf/2305.07185.pdf),
[BLT](https://arxiv.org/abs/2412.09871),
[T-FREE](https://arxiv.org/abs/2406.19223) and
[H-Net](https://arxiv.org/abs/2507.07955). His assessment: promising, but not yet
scaled to the frontier — and since frontier models still use tokenizers, it stays
on the syllabus.

The durable part of his argument is what any replacement must still do
([1:17:54]). Two requirements survive the death of BPE:

1. **The model must operate on chunks**, meaning abstractions over the sequence
   rather than raw units. He points out this is clearest outside text: for video
   or DNA, individual bytes have very low signal-to-noise, and you need some
   abstraction step to lift them somewhere modelling can happen.
2. **Chunks must be variable-length**, so computation is allocated adaptively. Not
   all bytes deserve the same treatment, and a scheme that treats them equally will
   be suboptimal.

So even a tokenizer-free future inherits tokenization's job description. What
changes is whether the chunking is a separate trained artifact or a learned part
of the model.

## In this course

Assignment 1 has you implement a BPE tokenizer, with the practical extensions
Lecture 1's toy version omits — see
[byte-pair encoding](byte-pair-encoding.md#what-assignment-1-adds).

## Sources

- [Lecture 1](01-overview-tokenization.md), tokenization unit from [1:04:46]
- [`lecture_01.py` transcription](../raw/slides/01-overview-tokenization.md) —
  code, worked values and the citation list
- [Edited transcript](../raw/transcripts/01-overview-tokenization.md)
