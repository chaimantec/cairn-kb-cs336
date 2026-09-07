# SigLIP

**SigLIP** (Sigmoid Loss for Language Image Pre-training, Google, 2023) changes
exactly one thing about [CLIP](clip.md) — the loss — and
[Lecture 17](17-multimodality.md) presents it entirely through the consequences.
[Paper](https://arxiv.org/abs/2303.15343).

It is the encoder every later model in the lecture actually uses:
[LLaVA-OneVision](llava-onevision.md) takes SigLIP, and
[Qwen3-VL](qwen-vl-series.md) takes SigLIP-2.

## The change

| | Question asked | Consequence |
| --- | --- | --- |
| CLIP | *which* of these $N$ images goes with this text? | multiclass — every score must be normalized against every other |
| SigLIP | *does* this image go with this text? | binary — each pair is decided independently |

In the lecturer's words: "SigLIP, in some sense, is a lot simpler: it basically
says, for any given image-text pair, are they aligned or not?… the diagonal
entries are positive examples and the off-diagonal entries are negative examples"
(≈22:46).

The implementation is four lines. Normalize both embeddings, take the scaled dot
product with a learned temperature $t$ **and a learned bias $b$**, build a label
matrix that is $+1$ on the diagonal and $-1$ off it, and apply a log-sigmoid:

$$\mathcal{L} = -\frac{1}{N}\sum_{i}\sum_{j} \log \sigma\!\big(z_{ij}\,(\mathbf{x}_i \cdot \mathbf{y}_j \, t + b)\big), \qquad z_{ij} = \begin{cases} +1 & i = j \\ -1 & i \neq j\end{cases}$$

## Why that dissolves a systems problem

An independent per-pair decision needs no normalizer, so there is no full-batch
softmax, so no device needs every other device's scores.

The lecture ties this straight to [parallelism](07-parallelism.md): "you can think
about it as DDP, if you like, where each device stores a subset of the image-text
pairs. However, there are interactions between the examples, unlike in language
model training, where everything just factors" (≈26:37).

The resolution is a **ring**. Each device first computes the loss on its own local
block, then passes its chunk of text embeddings to a neighbour and computes the
next block of negatives, "so you kind of rotate around until you cover all the
off-diagonal block entries" (≈27:25). With three devices it takes three rounds,
and no device ever materializes the full $N \times N$ matrix.

## The efficiency claim, and the detail that makes it real

**CLIP: 10 days on 256 TPUv3. SigLIP: 5 days on 32 TPUv4.** Half the time on one
eighth of the chips.

It would be easy to read that as a hardware upgrade, and the lecture forecloses
it. TPUv4 chips are *not* faster per chip than TPUv3: "they're better because you
can put more of them in a pod, and the interconnect is better. But at this scale,
they're actually not faster — actually, like 60% slower, or something" (≈25:50).

So the saving is the loss function, on *slower* silicon. The lecturer does add one
honest caveat — CLIP "probably did not try to optimize the code for maximum
throughput, it was just kind of 'let's get this thing to work'" — so the
comparison is not purely algorithmic.

## Decoupling batch size from the loss

This is the paper in one phrase, and it is the deeper point. Under CLIP the batch
*is* the negative set, so "if you change the batch size, it's a different loss
function." Under a sigmoid, batch size returns to being an optimization knob:
smaller batches give "more variance, but it's the same loss in expectation"
(≈28:12).

Two empirical results follow:

- **Below ~16K, SigLIP is clearly better**, because CLIP's loss "if you have too
  small a batch size, just degrades."
- **Above 32K, more does not help.** They went up to a **1M** batch and found 32K
  sufficed — "we saw that [critical batch sizes](critical-batch-size.md) hit some
  limit, and 32K was essentially their critical batch size." This retires the
  assumption that contrastive training keeps improving with batch size.

## WebLI, and the reason SigLIP reads better

SigLIP trains on **WebLI**: on the order of a billion image-text pairs, scraped,
multilingual (100 languages), filtered to the top 10% by quality.

The detail worth noticing is easy to skim past. WebLI **ran OCR over the images**
and used the extracted text as additional supervision — "for images that had text
in them, they did OCR, and that's another way to form text-image pairs" (≈25:04).

That is why SigLIP encoders are better at reading than CLIP's, and it is why every
OCR-capable model later in the lecture chooses SigLIP.

→ [CLIP](clip.md) ·
[Contrastive image-text pretraining](contrastive-image-text-pretraining.md) ·
[Lecture 17](17-multimodality.md)
