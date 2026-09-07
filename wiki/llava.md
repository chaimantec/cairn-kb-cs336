# LLaVA

**LLaVA** (Large Language and Vision Assistant, 2023) is the paper that shows how
little it takes to turn an image encoder and a language model into something that
can hold a conversation about a picture.
[Paper](https://arxiv.org/abs/2304.08485). Covered in
[Lecture 17](17-multimodality.md).

Its importance at the time was **openness**. The closed models — "I think it was
GPT-4" — could already do visual reasoning; LLaVA showed comparable behaviour in
a model whose construction was public. "It wasn't as good as GPT-4, of course, but
it was an open model, and people got to see what went on under the hood" (≈29:49).

## The parts, all off the shelf

- **Vision encoder:** [CLIP](clip.md), specifically ViT-L/14.
- **Text decoder:** Vicuna — the first LLaMA fine-tuned on ShareGPT conversations,
  which were "conversations that people had with ChatGPT and shared them on this
  website — I don't think this is up anymore" (≈30:35).
- **Projector:** a single matrix.

Neither tower is trained from scratch. → [Vision-language models](vision-language-models.md)

## The projector is one linear map

CLIP's output vector "isn't really in the same space, so to speak, as the text. So
then they multiply by a matrix $W$ to get another vector, which is in the same
space as the text embeddings" (≈32:09). Image vectors and text vectors are then
concatenated into one sequence and run through the language model unchanged.

The program positions this against the alternatives of the time — "Flamingo and
Q-former are more complex" — and LLaVA's claim is that **a linear map suffices**.
Every later model in the lecture varies this one component.
→ [Modality projectors](modality-projectors.md)

## The data trick

This is the part most easily misread. MS COCO already carried human annotations:
bounding boxes and Mechanical Turk captions. LLaVA prompted **GPT-4 with those
textual annotations** — *not* with the images — to generate questions and
conversations, then paired the generations back with the original images.

So a text-only model bootstraps a multimodal instruction dataset, because somebody
had already described the images in text. 158K examples, in three response types:
conversation, detailed description, and complex reasoning.
→ [Visual instruction tuning](visual-instruction-tuning.md)

## Two-stage training

| Stage | Vision encoder | $W$ | LM | What it learns |
| --- | --- | --- | --- | --- |
| 1 — alignment | frozen | **train** | frozen | a coordinate change: "the goal of training $W$ is that they come to look like natural-language token embeddings" (≈33:40) |
| 2 — fine-tuning | frozen | train | **train** | the instruction-following tasks |

**The vision encoder is never trained.** [Qwen-VL](qwen-vl-series.md) changes
that, training it in two of its three stages.

## The example everybody remembers

The paper's "Extreme Ironing" figure — a man ironing on the back of a vehicle in
traffic — asked "what is unusual about this image?" LLaVA answers correctly, and
the paper's point is that it does so even when the prompt does *not* ask for the
unusualness: asked merely "what's happening in the scene?", it still volunteers
that the setup is unconventional. GPT-4 handles it too; BLIP-2 and OpenFlamingo,
the open models of the moment, do not (≈34:26).

## What came next

LLaVA 1.5 and LLaVA-Next followed, and the lecture folds their contributions into
its treatment of [LLaVA-OneVision](llava-onevision.md) rather than covering them
separately — with one exception worth knowing: **AnyRes was introduced in LLaVA
1.5**, not in OneVision.
→ [Image resolution and token budgets](image-resolution-and-tokens.md)

→ [Lecture 17](17-multimodality.md)
