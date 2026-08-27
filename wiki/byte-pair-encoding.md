# Byte-Pair Encoding (BPE)

BPE is the tokenizer CS336 teaches and the one Assignment 1 asks you to build. It
resolves the dilemma laid out in [tokenization](tokenization.md) — character,
byte and word tokenizers each fail on either vocabulary size or compression ratio
— by **training the vocabulary on data** instead of fixing it in advance.

The core idea, from [Lecture 1](01-overview-tokenization.md) at [1:12:27]: common
sequences of bytes are represented by a single token, rare sequences are split
into many. Because you start from bytes, nothing is ever out-of-vocabulary, which
is what kills the `UNK` problem that sinks word-level tokenization.

## Where it comes from

BPE was introduced by Philip Gage in 1994 as a **data compression** algorithm,
long before language models. It was brought into NLP by
[Sennrich et al. (2016)](https://arxiv.org/abs/1508.07909) for neural machine
translation — before which, as the lecture notes, papers had been using word-based
tokenization. The first language model to use it was
[GPT-2](https://cdn.openai.com/better-language-models/language_models_are_unsupervised_multitask_learners.pdf).

## The algorithm

Start with each byte as its own token, then repeatedly merge the most frequent
adjacent pair.

```python
def train_bpe(string: str, num_merges: int) -> BPETokenizerParams:
    indices = list(map(int, string.encode("utf-8")))
    merges: dict[tuple[int, int], int] = {}
    vocab: dict[int, bytes] = {x: bytes([x]) for x in range(256)}

    for i in range(num_merges):
        counts = count_adjacent_pairs(indices)     # pair -> frequency
        pair = max(counts, key=counts.get)          # most common pair
        new_index = 256 + i
        merges[pair] = new_index
        vocab[new_index] = vocab[pair[0]] + vocab[pair[1]]
        indices = merge(indices, pair, new_index)

    return BPETokenizerParams(vocab=vocab, merges=merges)
```

Three things to notice about the loop.

The vocabulary starts at exactly 256 entries — one per byte value — so **every
possible input is representable from the outset**. Merging only ever adds entries;
it never removes the escape hatch.

New indices are handed out sequentially from 256, so `num_merges` *is* the
vocabulary size minus 256. That makes vocabulary size a direct, chosen parameter —
the knob the other tokenizers could not offer.

And merges **compose**: `vocab[new_index]` is built from two existing vocabulary
entries, which may themselves be merged tokens. This is why the merge list has to
be applied in training order at encode time.

The two helpers:

```python
def count_adjacent_pairs(indices):
    counts = defaultdict(int)
    for index1, index2 in zip(indices, indices[1:]):
        counts[(index1, index2)] += 1
    return counts

def merge(indices, pair, new_index):
    """Return `indices`, but with all instances of `pair` replaced with `new_index`."""
    new_indices, i = [], 0
    while i < len(indices):
        if i + 1 < len(indices) and indices[i] == pair[0] and indices[i + 1] == pair[1]:
            new_indices.append(new_index)
            i += 2
        else:
            new_indices.append(indices[i])
            i += 1
    return new_indices
```

## A worked example

The lecture trains on `"the cat in the hat"` with three merges ([1:13:13]). The
string is 18 bytes, so it starts as 18 tokens:

```
[116, 104, 101, 32, 99, 97, 116, 32, 105, 110, 32, 116, 104, 101, 32, 104, 97, 116]
  t    h    e    ␣   c   a    t   ␣    i    n   ␣    t    h    e   ␣    h   a    t
```

| Merge | Pair chosen | As bytes | Count | New token | Sequence after |
| --- | --- | --- | --- | --- | --- |
| 0 | `(116, 104)` | `b't'` + `b'h'` | 2 | `256 = b'th'` | `[256, 101, 32, 99, 97, 116, 32, 105, 110, 32, 256, 101, 32, 104, 97, 116]` |
| 1 | `(256, 101)` | `b'th'` + `b'e'` | 2 | `257 = b'the'` | `[257, 32, 99, 97, 116, 32, 105, 110, 32, 257, 32, 104, 97, 116]` |
| 2 | `(257, 32)` | `b'the'` + `b' '` | 2 | `258 = b'the '` | `[258, 99, 97, 116, 32, 105, 110, 32, 258, 104, 97, 116]` |

18 tokens down to 12, so the compression ratio on the training string rises from
1.0 to **1.5**.

Three observations that the example is designed to produce:

**Merges compose across steps.** Merge 1 consumes the token merge 0 created; merge
2 consumes merge 1's. Within three steps the algorithm has built `b'the '` out of
four separate bytes.

**Ties are broken by first-seen.** At [1:13:59] Percy notes there are ties — several
pairs occur twice — and "we'll just take the first one." That is not a stated
policy so much as a consequence of `max(counts, key=counts.get)` returning the
first maximal key in insertion order. If you reimplement this, your tie-breaking
must match, or your merge list will diverge from the reference.

**It learns the leading space.** The third merge produces `b'the '` *with* the
trailing space. This is the same phenomenon as the `␣word` observation in
[tokenization](tokenization.md#three-behaviours-that-surprise-people), arrived at
from the training side: spaces are frequent neighbours, so they get absorbed.

## Encoding new text

Apply the learned merges, in order, to the byte sequence:

```python
def encode(self, string: str) -> list[int]:
    indices = list(map(int, string.encode("utf-8")))
    # Note: this is a very slow implementation
    for pair, new_index in self.params.merges.items():
        indices = merge(indices, pair, new_index)
    return indices
```

Decoding just looks each index up and concatenates the bytes:

```python
def decode(self, indices: list[int]) -> str:
    bytes_list = list(map(self.params.vocab.get, indices))
    return b"".join(bytes_list).decode("utf-8")
```

Running the three-merge tokenizer above on `"the quick brown fox"` ([1:15:34]):

```
before:  [116, 104, 101, 32, 113, 117, 105, 99, 107, 32, 98, 114, 111, 119, 110, 32, 102, 111, 120]   (19 tokens)
after:   [258, 113, 117, 105, 99, 107, 32, 98, 114, 111, 119, 110, 32, 102, 111, 120]                  (16 tokens)
```

Compression ratio 1.1875, and it round-trips. Only the leading `"the "` was
compressed — the merges were trained on `"the cat in the hat"` and know nothing
about foxes. That is BPE's data-driven character in miniature: **a tokenizer is
only as good as the distribution it was trained on**, which is why production
tokenizers are trained on enormous corpora and why a tokenizer trained on English
handles code or Chinese poorly.

## What Assignment 1 adds

The implementation above is complete and correct, and Percy is candid at [1:16:20]
that it is "extremely slow." Assignment 1 asks for four improvements ([1:16:20]–[1:17:06]):

- **Don't loop over every merge.** `encode` currently applies all merges to the
  whole sequence, and the number of merges is the vocabulary size minus 256 — so
  for a realistic vocabulary this is tens of thousands of full passes. Loop only
  over merges that can apply, which means building indices to find them.
- **Handle special tokens.** Detect and preserve things like `<|endoftext|>`.
  Percy calls this "conceptually not deep but important to building a modern
  tokenizer."
- **Pre-tokenize.** The toy version tokenizes an entire string at once; real
  implementations first break text into chunks (the GPT-2 tokenizer regex is the
  standard one) and run BPE within each chunk. Much faster, and it stops merges
  spanning word boundaries in unwanted ways.
- **Make it fast.** And at some point, he notes, you may find that Python is simply
  not fast enough — "if you want to implement it in, say, your favourite language,
  Rust or C or something, then go for it."

## Sources

- [Lecture 1](01-overview-tokenization.md), BPE section from [1:11:39]
- [`lecture_01.py` transcription](../raw/slides/01-overview-tokenization.md#bpe-tokenizer) —
  full code and the step-by-step trace
- [Edited transcript](../raw/transcripts/01-overview-tokenization.md)
- Sennrich et al., *Neural Machine Translation of Rare Words with Subword Units*
  ([arXiv:1508.07909](https://arxiv.org/abs/1508.07909))
