# See also — related knowledge bases

Other Cairn knowledge bases whose material genuinely bears on CS336. Each entry
says what that KB is good for, and what it does not cover, so it is worth deciding
*before* spending a fetch on it.

Pass the **repo URL** as the `kb` argument to `kb_read` / `kb_list`. The
`raw.githubusercontent.com` links are direct `web_fetch` targets that return plain
markdown — a `github.com/.../blob/...` URL returns rendered HTML instead.

---

- **CS224N — Natural Language Processing with Deep Learning** (Stanford,
  Christopher Manning, Spring 2024).
  KB: `https://github.com/chaimantec/cairn-kb-cs224n` — pass this as `kb`.

  The course CS336 assumes you have taken. Percy points at it directly in Lecture 1
  ([29:24]): *"if you took CS224N, the NLP class, then you've seen Transformers."*
  Where CS336 builds and measures the stack, CS224N derives it — attention and the
  Transformer from first principles, word vectors, RNNs and LSTMs, pretraining, and
  evaluation.

  Most useful here for **the derivations CS336 skips**. Reach for it when a CS336
  question is about *why* a mechanism works rather than how to make it fast.
  It also covers subword tokenization from the linguistic side, which complements
  the efficiency-first treatment in [tokenization](wiki/tokenization.md) and
  [byte-pair encoding](wiki/byte-pair-encoding.md).

  **Complete: all 23 lectures**, with slide-by-slide text for every deck.
  [INDEX](https://raw.githubusercontent.com/chaimantec/cairn-kb-cs224n/main/INDEX.md) ·
  [subword modeling / BPE](https://raw.githubusercontent.com/chaimantec/cairn-kb-cs224n/main/wiki/subword-modeling.md) ·
  [tokenizers in practice](https://raw.githubusercontent.com/chaimantec/cairn-kb-cs224n/main/wiki/tokenizers-in-practice.md) ·
  [attention](https://raw.githubusercontent.com/chaimantec/cairn-kb-cs224n/main/wiki/attention.md) ·
  [transformer](https://raw.githubusercontent.com/chaimantec/cairn-kb-cs224n/main/wiki/transformer.md)

- **CS221 — Artificial Intelligence: Principles and Techniques** (Stanford,
  Percy Liang, Autumn 2025).
  KB: `https://github.com/chaimantec/cairn-kb-cs221` — pass this as `kb`.

  Same instructor, same executable-lecture format, one layer further down. Useful
  for the machine-learning foundations CS336 takes for granted — gradient descent,
  backpropagation, linear regression — and for **tensors and einops**, which is
  exactly the ground CS336 Lecture 2 covers.

  **Partial: lectures 1–4 of 20.** Those four are the learning foundations, and
  they are the only part relevant to CS336 anyway; the rest of CS221 (search, MDPs,
  games, CSPs, logic) is not covered by that KB and would not bear on this course
  if it were.
  [INDEX](https://raw.githubusercontent.com/chaimantec/cairn-kb-cs221/main/INDEX.md) ·
  [gradient descent](https://raw.githubusercontent.com/chaimantec/cairn-kb-cs221/main/wiki/gradient-descent.md) ·
  [backpropagation](https://raw.githubusercontent.com/chaimantec/cairn-kb-cs221/main/wiki/backpropagation.md) ·
  [tensors](https://raw.githubusercontent.com/chaimantec/cairn-kb-cs221/main/wiki/tensors.md) ·
  [einops](https://raw.githubusercontent.com/chaimantec/cairn-kb-cs221/main/wiki/einops.md)
