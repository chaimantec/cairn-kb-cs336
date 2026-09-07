# Lecture 17 — Multimodality

**Percy Liang.** [Transcript](../raw/transcripts/17-multimodality.md) ·
[course material](../raw/slides/17-multimodality.md) ·
[`lecture_17.py`](https://github.com/stanford-cs336/lectures/blob/main/lecture_17.py) ·
[trace viewer](https://cs336.stanford.edu/lectures/?trace=lecture_17)

Every other lecture in CS336 builds a language model: text in, text out. This one
asks what has to change when the input is a photograph — and what would have to
change again for the output to be one. It is a late addition to the course, and
the lecturer says so in his first sentence: the plan had been more reinforcement
learning, but "this class would be a little bit incomplete without saying
something about multimodality, which is so pervasive if you look at all the major
models" (≈0:05).

> **A note on the title.** The syllabus calls this lecture "Alignment,
> multimodality." The program calls itself "Lecture 17: multimodal models," and
> the program is right — **there is no alignment content in it.** Alignment was
> [lecture 15](15-mid-post-training.md) and
> [lecture 16](16-post-training-rlvr.md). This page follows the content.

## The argument the whole lecture rests on

Before any model arrives, the lecture states a syllogism, and every one of the
eight papers below is an attempt at its last step.

> - Transformers work really well. So we gotta use them.
> - Transformers speak tokens (discrete or continuous), where a token represents some ~semantic unit of information.
> - Therefore, we must convert everything into tokens.

The first premise is empirical and faintly resigned: "despite the best efforts of
people to try other things, across all these modalities they are still, at scale,
the best thing we have. So we have to figure out how to use them" (≈1:39).
Everything else follows from what a Transformer will accept.

Two things in that middle bullet are easy to skim past and carry the lecture.

**"Discrete or continuous."** A text token is a symbol with an embedding table
behind it. An image token, for most of this lecture, is a *continuous vector*
straight out of an encoder, with no vocabulary at all. Both count as tokens
because both are things a Transformer can attend over. The lecturer widens the
word deliberately (≈2:26).

**"Some ~semantic unit."** This is the criterion that makes the problem hard.
"In natural language, tokens are subwords, and these are somewhat meaningful,
whereas a pixel is certainly not meaningful by itself" (≈2:26). You cannot feed
in pixels and call them tokens.

So image encoding is framed as the sequel to a problem the course already
solved once. [Tokenization](tokenization.md) was already a compromise for text —
"we had a BPE tokenizer, you guys implemented it, it's fine… it's not the worst
thing in the world" — and "for non-text modalities we have to scratch our heads a
lot more and figure out what is the equivalent of the BPE tokenizer that will take
an image and produce things that a Transformer can digest" (≈3:12).

The destination is the **omni model**: "the ability to take any combination of
these modalities as input and output any combination of these modalities"
(≈0:51). The lecture states it and then immediately declines to reach it.

![Four-panel grid illustrating the text, image, audio and video modalities, each with example content](../raw/images/17-multimodality/multimodality.png)

*The opening motivation: text, images, audio and video as four modalities an omni model would have to accept and emit. [Source](https://github.com/stanford-cs336/lectures/blob/main/images/multimodality.png)*

Which leaves two questions, and the lecture is explicit that it answers one:

> 1. How do we input non-text data (e.g., understand images)?
> 2. How do we output non-text data (e.g., generate audio)?

Seven of the eight papers answer question 1. [Chameleon](chameleon.md) is the only
one that touches question 2, and it does so by changing the answer to question 1.

→ [Multimodal models](multimodal-models.md) develops this framing on its own page.

## 1. Encoding images: CLIP

[CLIP](clip.md) is where the lecture starts, "back in 2021," because it "is a lot
of the foundation of modern VLMs" (≈3:59). Its contribution is not an
architecture but a **training signal**.

The context is a contrast between two fields. Language had already gone
"into this kind of foundation-model era" with GPT-2 and GPT-3, while vision "had
traditionally been based on large annotated datasets such as ImageNet" (≈4:46).
So the question was whether the trick that worked for text — scrape the internet,
accept the noise, train something big — had a visual equivalent (≈5:33).

![CLIP's three-panel diagram: contrastive pre-training over an image-text similarity matrix, then zero-shot classification built from label text](../raw/images/17-multimodality/clip.png)

*CLIP's own figure. Left: contrastive pre-training, where the highlighted diagonal of the $I_i \cdot T_j$ matrix is the aligned pairs. Right: the same encoders reused zero-shot, by embedding "A photo of a {object}" for each candidate label. [Source](https://github.com/stanford-cs336/lectures/blob/main/images/clip.png)*

**The objective.** Take a batch of image-text pairs — the lecture says 32,000,
the program says 32,768. Encode every image and every text. Then for each image,
require that the dot product with *its* caption exceed the dot product with all
the others; and symmetrically for each text. "In a nutshell, that's the CLIP
objective, and you can imagine there's basically two times N different softmax
classification problems" (≈7:07).

Two properties follow, and both matter later. The loss is a **ranking** loss, not
a reconstruction loss — nothing asks the model to produce a caption. And the
other examples in the batch *are* the negatives, which is why batch size appears
in the method rather than in the optimization details.

**The data, and a circularity worth noticing.** 400 million image-text pairs,
built by searching for 500K queries and taking ~20K pairs each. The dataset was
never released, which is why OpenCLIP exists — and OpenCLIP's replacement,
LAION-5B, "actually used CLIP for the data filtering and then trained OpenCLIP.
So there's some bootstrapping happening" (≈8:41). This is the same loop the
[data filtering](data-filtering.md) lecture records for text: a model becomes the
quality filter for the data that trains its successor.

**The 336×336 center crop, which the rest of the lecture spends its time undoing.**
Images arrive at arbitrary sizes and "neural nets… don't like things to be
dynamic; they want things to be fixed size" (≈9:27), so CLIP resizes the short
side to 336 and cuts the borders off. The lecturer flags at the time that this is
"obviously for convenience… later we'll see that you can do much better than
this," and gives the excuse that "the CLIP authors were thinking about ImageNet
and classification. So usually the object is in the middle, and you're just
trimming off some background" (≈10:12). Hold onto this: AnyRes and dynamic
resolution both exist because of it.

**The encoders.** The vision side is a [Vision Transformer](vision-transformers.md)
— ResNets were tried and lost, so "when people say CLIP, they usually mean the ViT
version" (≈12:30). The best model is **ViT-L/14** at 336px: Large, 14×14 patches.
Rather than averaging the patch vectors, CLIP uses **attention pooling** — attention
with the query set to the global average of the activations, giving "another
vector, which is maybe a little bit more informed than just a straight-up average"
(≈14:07). The text side is a small GPT-2-style Transformer, 63M parameters, whose
sentence embedding is the `[EOS]` activation at the top layer (≈16:31).

**The headline result**, and it is worth reading precisely: zero-shot CLIP beat a
ResNet-50 *trained on* ImageNet's 1.2 million labelled images. The lecturer's
gloss is about labour, not accuracy — those 1.2M labels were "many, many hours of
Amazon Mechanical Turk worker time," while CLIP leveraged annotation that the web
had already produced for free (≈17:18).

**The ablation that decides the lecture's shape.** CLIP also tried predicting the
caption from the image, as a bag of words or as a language model, and found that
"if you use a stronger model, it actually does worse, or at least is less
efficient" (≈20:26).

![Line chart of zero-shot ImageNet accuracy against images processed for three training objectives, annotated with 4X and 3X efficiency gaps](../raw/images/17-multimodality/clip-efficiency.png)

*Three objectives against images processed (**linear** x-axis). Contrastive CLIP reaches ~16% zero-shot accuracy at 33M images where bag-of-words *prediction* needs 134M (4×), which in turn needs 134M where a full Transformer LM needs 400M (3×). Generating the caption forces the model to account for the exact wording; ranking it asks only for what distinguishes it. [Source](https://github.com/stanford-cs336/lectures/blob/main/images/clip-efficiency.png)*

The lecturer's reading is careful: "again, this is ImageNet accuracy, so it's
classification… actually modeling the exact token sequences of the caption isn't
so important for getting a rough representation of the image" (≈21:12).

**And CLIP's own summary states the limitation the rest of the lecture inherits:**
its design decisions "are based on image classification, so it's not very
fine-grained" (≈21:59). Everything — 336×336, the center crop, pooling to one
vector — was validated on a coarse task. Ask it to read text in a photograph and
it was never tuned for that.

→ [CLIP](clip.md) · [Contrastive image-text pretraining](contrastive-image-text-pretraining.md) · [Vision Transformers](vision-transformers.md)

## 2. SigLIP: change the loss, and the systems problem dissolves

CLIP's technical debt is stated plainly: it "requires large batch sizes, like
30,000… and furthermore, the softmax operation operates over the full batch, so
it's not really very decomposable," unlike language-model training where "all the
sequences kind of parallelize, and you just do an aggregation at the end" (≈21:59).

[SigLIP](siglip.md) changes exactly one thing. CLIP asks *which* of these N images
goes with this text — a multiclass question, so every score must be normalized
against every other. SigLIP asks *does this image go with this text* — "basically
binary classification, where the diagonal entries are positive examples and the
off-diagonal entries are negative examples" (≈22:46). The implementation is four
lines: normalize, dot product, build labels that are $+1$ on the diagonal and
$-1$ off it, take a log-sigmoid.

Everything else follows from that. An independent per-pair decision needs no
normalizer, so no full-batch softmax, so no need to communicate every score to
every device.

![Four-panel diagram of SigLIP's ring-based chunked loss computation across three devices over three rounds](../raw/images/17-multimodality/siglip-parallelism.png)

*The loss computed in a ring across three devices. Each device first computes its own diagonal block, then rotates its chunk of text embeddings to a neighbour and computes the next block of negatives, until after three rounds every off-diagonal block is covered — and no device ever materializes the full matrix. [Source](https://github.com/stanford-cs336/lectures/blob/main/images/siglip-parallelism.png)*

The lecturer connects it explicitly to the systems half of the course: "if you
recall your [systems lecture](07-parallelism.md), you can think about it as DDP,
if you like, where each device stores a subset of the image-text pairs. However,
there are interactions between the examples, unlike in language model training,
where everything just factors" (≈26:37).

**The efficiency claim, and why the parenthetical makes it real.** CLIP took 10
days on 256 TPUv3; SigLIP took 5 days on 32 TPUv4. It would be easy to read that
as newer hardware, and the lecturer forecloses it: the v4 chips are *not* faster
per chip — "they're better because you can put more of them in a pod, and the
interconnect is better. But at this scale, they're actually not faster — actually,
like 60% slower, or something" (≈25:50). Half the wall-clock on one eighth of the
chips, on *slower* chips. The saving is the loss function, not the silicon.

**"Decouple batch size from the loss"** is the paper in one phrase. Under CLIP the
batch *is* the negative set, so "if you change the batch size, it's a different
loss function." Under a sigmoid, small batches give "more variance, but it's the
same loss in expectation" (≈28:12). SigLIP is better below ~16K, and pushing up to
a 1M batch does not help — 32K "was essentially their critical batch size,"
which the lecturer ties straight back to
[critical batch size](critical-batch-size.md) from the scaling lectures.

One further detail decides which encoder later models use. WebLI, SigLIP's
billion-pair dataset, ran **OCR over the images** and used the extracted text as
supervision (≈25:04). That is why SigLIP encoders read better than CLIP's, and why
every OCR-capable model in the rest of this lecture picks SigLIP.

→ [SigLIP](siglip.md) · [Contrastive image-text pretraining](contrastive-image-text-pretraining.md)

## 3. The VLM template: LLaVA

Now the question changes. CLIP and SigLIP produce an image embedding; neither is a
language model and neither can hold a conversation about a picture. [LLaVA](llava.md)
(2023) is the paper that shows how little it takes to join the two, and its
significance at the time was openness: the closed models "were able to do visual
reasoning, and they were able to show that they could do some visual reasoning as
well. It wasn't as good as GPT-4, of course, but it was an open model, and people
got to see what went on under the hood" (≈29:49).

The lecture frames the whole approach as post-training rather than pre-training:
"we take an existing image encoder, we take an existing LLM, and then we kind of
stitch it together, rather than training something from scratch" (≈29:01).

**The data trick, stated precisely because it is easy to misread.** MS COCO already
had human bounding boxes and captions. LLaVA prompted **GPT-4 with the textual
annotations** — not the images — to generate questions and conversations, then
paired those generations back with the original images. 158K examples. A text-only
model bootstraps a multimodal dataset, because somebody had already described the
images in text.

![LLaVA's data generation example: COCO captions and bounding boxes prompt GPT-4 into conversation, description and reasoning responses](../raw/images/17-multimodality/llava-gen.png)

*The two context types fed to GPT-4 (captions, and box coordinates) and the three response types generated from them: conversation, detailed description, and complex reasoning. The photograph is shown for reference — GPT-4 never saw it. [Source](https://github.com/stanford-cs336/lectures/blob/main/images/llava-gen.png)*

**The architecture is one matrix.** CLIP's output "isn't really in the same space,
so to speak, as the text. So then they multiply by a matrix $W$ to get another
vector, which is in the same space as the text embeddings" (≈32:09). Then image
vectors and text vectors are concatenated and "this whole sequence of vectors just
goes through a standard Transformer" — "so we're, in some sense, converting these
images into textual tokens, so we can leverage the pre-trained language model"
(≈32:54).

![LLaVA architecture: an image through a vision encoder and projection W, concatenated with instruction tokens into one language model](../raw/images/17-multimodality/llava-architecture.png)

*The template the next five papers all vary: vision encoder → projection $W$ → image tokens $H_v$, concatenated with instruction tokens $H_q$ into a single language model. The program notes that "Flamingo and Q-former are more complex"; LLaVA's claim is that a linear map suffices. [Source](https://github.com/stanford-cs336/lectures/blob/main/images/llava-architecture.png)*

**Two training stages**, and the logic is about what each stage's data can be
trusted to move. Stage 1 freezes both towers and trains only $W$ — "the goal of
training $W$ is that they come to look like natural-language token embeddings"
(≈33:40). Stage 2 unfreezes the language model but keeps the vision encoder
frozen. Note what is never trained: the vision encoder. Qwen changes that.

→ [Vision-language models](vision-language-models.md) · [LLaVA](llava.md) · [Visual instruction tuning](visual-instruction-tuning.md) · [Modality projectors](modality-projectors.md)

## 4. LLaVA-OneVision: resolution, token budgets, and transfer

[LLaVA-OneVision](llava-onevision.md) (2024) keeps the template and swaps every
part — SigLIP for CLIP, Qwen-2 72B for Vicuna, a 2-layer MLP for the linear $W$.
"It's like you have a system and you're just upgrading the parts, but it's the
same rough system" (≈36:45). Its real contributions are elsewhere.

**AnyRes, and why the center crop had to go.** OCR "needs to preserve very
fine-grained information — otherwise a 'J' looks like an 'I,' and that's not
good," and "if you have a document and you crop to 336-by-336, you can't read it"
(≈36:45, ≈37:32). AnyRes breaks the image into pieces each sized to the encoder's
native resolution, encodes each separately, and concatenates — plus one
downsampled pass over the whole image for global context. If that yields too many
tokens, interpolate down.

![AnyRes pipeline: an image split into a grid and encoded per piece alongside a resized global view, both flattened into the LLM](../raw/images/17-multimodality/llava-onevision-anyres.png)

*The two paths: split the high-resolution image into encoder-sized pieces and encode each (top), and resize the whole image to one global view (bottom); both flatten into the LLM. The grid drawn here is schematic — the deck states no value for the $a \times b$ tiling. [Source](https://github.com/stanford-cs336/lectures/blob/main/images/llava-onevision-anyres.png)*

The justification is elegant: "the Transformer is already adaptive — sentences can
be any length… So it turns out images can be any resolution — that's kind of
piggybacking on the same dynamic ability" (≈38:20).

**The three input types are a token budget.** The stated goal is to "make all of
the modalities produce roughly the same length" (≈39:55), because otherwise
long videos would dominate. Read that way the three rules are obvious: a single
image can spend the whole budget, so give it high resolution and up to nine crops;
multiple images share it, so each gets base resolution; video frames share it
among up to 32 frames, so each gets fewer tokens still. "If I have a single image,
I get to look at it more carefully; if I have multiple images, I'm just going to
look at it from afar" (≈39:55).

![Token-budget table giving the per-modality token formulas for single-image, multi-image and video inputs](../raw/images/17-multimodality/llava-onevision-modalities.png)

*The budget as arithmetic: single-image $729 + N \times 729$ (max $(1+9) \times 729 = 7290$), multi-image $N \times 729$ (max $12 \times 729 = 8748$), video $N \times 196$ (max $32 \times 196 = 6272$). Three very different input shapes landing within about 40% of each other. [Source](https://github.com/stanford-cs336/lectures/blob/main/images/llava-onevision-modalities.png)*

This is where [long context](qwen-vl-series.md) first bites: "you start running
into context-length problems for video, and later we'll see how a big part of
being able to handle multimodal is dealing with long context" (≈40:42).

**On the data, the lecture is franker than the paper.** The stated philosophy is
quality over quantity, but "another way to interpret this is that it's very
targeted data — if you look at many of these, it's very task-based… So this is
definitely post-training territory." And: "this work is also unabashedly
distilling GPT-4 models, so that they can get the best performance, which is, I
guess, not ideal, but this is what you do if you don't have an annotation budget"
(≈41:28).

![Donut chart and full dataset legend for the 1.6M OneVision mixture, split across single-image, multi-image and video](../raw/images/17-multimodality/llava-onevision-data-2.png)

*The final OneVision mixture: 31.2% single-image, 43.0% multi-image, 25.9% video, itemized dataset by dataset. Note when quoting: the printed per-dataset counts sum to about 1.32M against the stated 1.6M total — see the [figure audit](../raw/slides/17-multimodality.md#figure-audit). [Source](https://github.com/stanford-cs336/lectures/blob/main/images/llava-onevision-data-2.png)*

**The finding worth remembering is cross-modal transfer**, and all three cases have
the same shape: a capability trained in one input format appears in another that
was never trained for it (≈43:00–≈44:36).

- Chart and diagram reading, trained only on **single images**, generalizes to
  questions spanning **two** images — "at training time, it never saw an example
  where you have a table and a chart and you're asking questions about both."
- OCR from single images plus relational reasoning from multi-image data combine
  into **GUI agency** over sequences of screenshots.
- **Visual prompting** — drawing a circle on an image to point at something —
  transfers from single images to **video**, tracking the circled subject across
  frames.

![Transfer example S8: a circled football player tracked across four video frames from single-image visual prompting](../raw/images/17-multimodality/llava-onevision-transfer-s8.png)

*Visual prompting transferring to video: the highlighted player is tracked across four frames, from training data that only ever circled things in still images. [Source](https://github.com/stanford-cs336/lectures/blob/main/images/llava-onevision-transfer-s8.png)*

The lecturer's own reaction is the honest one: "when I first looked at this, I
said, oh boy, you're basically targeting each of these tasks — it's kind of like
supervised learning. But if you have enough tasks, these models seem to do some
transfer, which is, I guess, reassuring" (≈44:36).

→ [LLaVA-OneVision](llava-onevision.md) · [Image resolution and token budgets](image-resolution-and-tokens.md) · [Cross-modal transfer](cross-modal-transfer.md) · [Video understanding](video-understanding.md)

## 5. The Qwen series, read as a time series

Three papers from one lab across three years, and the lecture uses them to show
what changes and what does not. → [The Qwen-VL series](qwen-vl-series.md)

**Qwen-VL (2023)** uses an OpenCLIP encoder and, for the adapter, "one layer of
cross-attention, incorporating 2D positional embeddings, and mapped to a fixed
size of 256" — which the lecturer immediately marks as provisional: "this is
definitely not very dynamic, but neither is the vision encoder at this point"
(≈46:12). Special tokens `<img>`, `<box>` and `<ref>` let the model *name* a
region in its output, which is what makes grounding possible at all.

Its three training stages differ from LLaVA's in that **which components are
frozen changes non-monotonically**: stage 1 freezes the LM and trains the vision
side on large-scale low-quality data; stage 2 unfreezes everything at higher
resolution; stage 3 freezes the vision encoder again for instruction tuning. The
principle is that data quality should decide which parameters may move.

![Qwen-VL's three training stages, each showing which of ViT, cross-attention and QwenLM is frozen or trainable](../raw/images/17-multimodality/qwen-vl-stages.png)

*Snowflakes mark frozen components and flames trainable ones. The pattern is not monotonic: QwenLM is frozen, then trained, then trained; the ViT is trained, trained, then frozen. [Source](https://github.com/stanford-cs336/lectures/blob/main/images/qwen-vl-stages.png)*

**Qwen2-VL (2024)** brings **dynamic resolution**, which repudiates both CLIP's
fixed crop and Qwen-VL's fixed 256 tokens. "Let's say you have this picture here
that might be mapped to 11,000 tokens. This tiny picture of an equation might only
be mapped to eight tokens" (≈49:16). Each 224×224 region is encoded with a ViT/14
and every 2×2 block of patches is merged into one token.

![Qwen2-VL's dynamic resolution architecture: four native-resolution inputs producing very different token counts into one decoder](../raw/images/17-multimodality/qwen2-vl-architecture.png)

*Four inputs at native resolution producing wildly different token counts — 11,427 for a tall blog screenshot, 8 for a small equation, 1,125 for a photograph, 2,208 for a video clip. An image now costs what it is worth. [Source](https://github.com/stanford-cs336/lectures/blob/main/images/qwen2-vl-architecture.png)*

It also answers a question a student asked back in the CLIP section — whether
position embeddings should account for two dimensions rather than one (≈15:45).
**MRoPE** splits the rotary embedding across time, height and width: "each
position, each patch, is now a triple, defined by the coordinates. And then, to
compute MRoPE, for every dimension you compute the [RoPE](rope.md), and then you
concatenate" (≈51:37). → [Multimodal RoPE](multimodal-rope.md)

**Qwen3-VL (2025)** the lecturer calls "kind of a systems paper," and its four
changes are "minor but potentially important." The sharpest is a fix to MRoPE
itself. RoPE's dimensions carry different frequencies, so allocating the three
axes in contiguous blocks means "maybe all the temporal dimensions are
low-frequency and all the height dimensions are high-frequency." Interleaving them
— `[t w h t w h …]` rather than `[t t t t w w w w h h h h]` — exposes every axis to
every band (≈53:57).

The other three: **explicit video timestamps as tokens**, so time becomes
something "you can directly refer to, like, what happened after two seconds"
(≈55:31); a **square-root-normalized per-token loss**, so that long video examples
do not dominate the gradient — the same problem
[data mixing](data-mixture-selection.md) solves for text; and **DeepStack**, which
"add[s] these directly into the residual stream of the language model… a bit more
of a deep fusion of the vision encoder into the language model, as opposed to the
vision encoder being a black box that just outputs a sequence of vectors"
(≈57:04).

![Qwen3-VL architecture adding DeepStack, which injects vision tokens into several decoder blocks rather than only at the input](../raw/images/17-multimodality/qwen3-vl.png)

*DeepStack (dashed box, right) injects vision tokens between LLM blocks rather than only at the input — the fourth and last variation on LLaVA's single matrix $W$. Video frames now carry explicit `<0.0 seconds>` timestamp tokens. [Source](https://github.com/stanford-cs336/lectures/blob/main/images/qwen3-vl.png)*

> **One attribution this KB does not vouch for.** The lecturer says DeepStack is
> "a paper from the DeepSeek team" (≈57:04). The course material names no authors
> for it, so nothing here corroborates or contradicts that, and the transcript
> keeps it as spoken. Check the [cited paper](https://arxiv.org/abs/2406.04334)
> before repeating the attribution.

Pre-training is a **context-length curriculum** — train the adapter, then everything
at 8K, 32K and 256K, with "most of the tokens are in stages two and three" (≈57:50) —
and post-training is [lecture 16](16-post-training-rlvr.md) applied to a
multimodal model: long chain-of-thought SFT, distillation, then RL.

→ [The Qwen-VL series](qwen-vl-series.md) · [Modality projectors](modality-projectors.md)

## 6. Chameleon: give up continuous tokens, and generation becomes possible

Everything so far answers question 1 and cannot answer question 2, for a
structural reason rather than a lack of effort: a continuous image embedding is
not something you can sample from a softmax. "Because it's a language model, you
can only generate text — you can't generate images" (≈1:07:16).

[Chameleon](chameleon.md) (Meta, 2024) takes the other branch: **map everything
into discrete tokens.** "In some ways, aesthetically — well, maybe this reflects
that I'm a language person — this is kind of appealing, because now you can
analyze and generate images in the same way, since everything is a discrete
token" (≈1:08:02). No projector, no separate generation head, no diffusion model:
the architecture collapses back into the plain language model of
[lecture 1](01-overview-tokenization.md).

![Chameleon's two panels: mixed-modal pre-training and mixed-modal generation over one interleaved text and image token stream](../raw/images/17-multimodality/chameleon.png)

*Text (green) and image tokens (blue) in one stream, delimited by Start Image / End Image markers. Training and generation are the same operation; at generation time the image span is routed to a de-tokenizer that renders pixels. [Source](https://github.com/stanford-cs336/lectures/blob/main/images/chameleon.png)*

The discretizer is **[VQ-VAE](discrete-image-tokens.md)**, "rather old, from 2017":
map a patch to a continuous vector, round it to the nearest entry in a learned
codebook, and train by reconstruction (≈1:09:36). A 512×512 image becomes **1,024
tokens** from a codebook of **8,192**. Then — and this is the detail that makes the
unification literal — "they train a new tokenizer, now, because your data looks
different than if it were just normal natural language" (≈1:11:09):
[BPE](byte-pair-encoding.md) run over image codes.

Note how different the *supervision* is from CLIP's, and what it costs. CLIP's
encoder is trained against **text**, so it keeps what a caption would mention and
discards the rest. VQ-VAE's is trained against **reconstruction**, so it must keep
whatever rebuilds the pixels.

**The instability is the concrete finding.** "Text and images, despite occupying
the same space, just behave very differently — so just calling things discrete
tokens doesn't hide the fact that there's an image living there." Text is
relatively predictable; "image tokens have very high entropy — I don't know what
shade of blue this exact token is going to be." The result was norm growth and
loss instability, fixed with **QK norm** and **z-loss** (≈1:12:41) — both already
familiar from [architectures](03-architectures.md) and
[training stability](training-stability.md), reappearing here for a new reason.

→ [Modality balancing and stability](modality-balancing.md)

**The verdict is a genuine trade, not a win.** "Elegant… but the downside is that
it turned out this model was not really as performant, and the discretization
definitely loses information — think about OCR, again: if you discretize, very
small print, you're not going to be able to read it anymore" (≈1:13:26). That is
exactly the capability AnyRes and dynamic resolution were invented to protect: the
continuous and discrete branches of the field are optimizing against each other on
one axis.

And the branch lost, for now — for an external reason. VQ-VAE-style discretization
was popular for image generation "because you have a Transformer — how do you
generate from a Transformer? Well, you have to generate discrete things." Then
"diffusion models came out and became popular and viable for generation. So this
flavor of method is somewhat less popular than it used to be" (≈1:14:12).

## 7. What the lecture concludes

The summary's third bullet is the synthesis the eight papers were building toward:

> Comprehension and generation might demand different things (semantics versus finer-grained details)

Spelled out (≈1:15:43): "there's no one universal encoder. Let's say, for example,
when we looked at CLIP, you only cared about classification, to capture high-level
semantics, so these vectors could be fairly small… Whereas, if you wanted to do
OCR, or if you wanted to generate an image, then you need really fine-grained
detail — that's why diffusion is so good."

Which makes the last bullet a **compromise rather than a unification**:
*continuous encoders + Transformer + diffusion models for generation.* Continuous
embeddings to understand, a separate diffusion model to generate. The lecturer is
careful to label this as inference rather than knowledge: frontier models are
"touted as being natively multimodal… and, in fact, they do — but, of course,
there are no details about how these are built. I think it's probably some
combination of having a continuous encoder… and then diffusion for the generation
— but that's my speculation" (≈1:14:58).

And the fourth bullet generalizes Chameleon's instability into a rule: "video
certainly has lower information density than text, so you don't want video to
overwhelm your text" (≈1:16:30).

The closing assessment is worth quoting because of what it says about the field's
pace: "it seems like even CLIP, even though it's five years old, or similar ideas,
are still kind of the go-to way to capture the semantics of images" (≈1:16:30).

## Questions from the floor

Five exchanges add material that is nowhere in the program.

**Why train on image-*text* pairs rather than images alone?** (≈10:58) Because
self-supervision from augmentation — "I think this idea is called SimCLR" — teaches
invariance to crops and rotations, and "you won't data-augment your way from one
type of dog to another dog. So by using text, it gives you higher-level semantic
representations" (≈11:43). This is the cleanest statement in the lecture of *what
the text is for*.

**Doesn't caption noise confuse the model?** (≈18:52) It is noisier than the
question supposes: captions are scraped from surrounding text or alt text, and "if
you have an image of a dog, you don't need to say 'a dog.'" The defence is
averaging — "it's unlikely that there's always going to be a dog" — plus heavy
filtering. The lecturer's own verdict: "it's maybe somewhat surprising, but also
interesting, that it worked" (≈19:38).

**Is multimodal training harder from a systems perspective?** (≈1:01:02)
"Certainly it's not easier," and the specific answer is **data loading**: "video
data, loading it can be a bottleneck. Generally, we have not really focused on
data loading when talking about language models, because it's very cheap." The fix
is the standard one — make loading async with compute. On token counts, he pushes
back on the premise: models are trained on tens of trillions of tokens and "I
wouldn't say that the number of multimodal tokens vastly outnumbers text tokens"
(≈1:02:36).

**Must the language model be pre-trained before alignment?** (≈1:03:22)
"It definitely has to be pre-trained, because otherwise it doesn't make sense to
align it." The alignment stage is not adaptive — "you pick a token budget — say, 67
billion tokens — and you just train."

**Why is the vision encoder so much smaller?** (≈1:04:09) Because it does a local
job: "it's looking at a patch — a patch is very small, and it's just trying to
understand the patch. There's not much knowledge there — so, per patch, we're not
reasoning. So most of the capabilities of the model are still in the language
model." ViTs stay "generally less than a billion parameters."

That last exchange contains a small moment worth preserving, because it is a case
where the course material settles what the transcript leaves confused. Reading his
own slide, the lecturer says "it's a 72-billion-parameter language model, and… I
don't know why this number is — oh, this, sorry, this is 72 *million*" (≈1:05:45).
The table resolves it exactly: for the 72.7B backbone, Stage-1 trains **72.0M**
parameters, because Stage 1 trains only the projector.

![LLaVA-OneVision's three training stages and the full configuration table of resolution, data, trainable parameters and learning rates](../raw/images/17-multimodality/llava-onevision-training.png)

*The table he was reading. The Model rows give trainable parameters per backbone size: 1.8M / 20.0M / 72.0M in Stage-1, where only the projector moves, against 0.8B / 8.0B / 73.2B once the full model trains. Every cell here was verified against the source image. [Source](https://github.com/stanford-cs336/lectures/blob/main/images/llava-onevision-training.png)*

## Where this sits in the course

This lecture reuses more of CS336 than any other late lecture, and mostly without
announcement:

| From | Reappears here as |
| --- | --- |
| [Tokenization](tokenization.md), [BPE](byte-pair-encoding.md) | the framing of image encoding as "the equivalent of the BPE tokenizer"; BPE literally re-run over VQ-VAE codes |
| [RoPE](rope.md) | MRoPE, then interleaved MRoPE |
| [Training stability](training-stability.md) | QK norm and z-loss, for a new cause — entropy mismatch between modalities |
| [Mixture of experts](mixture-of-experts.md) | Qwen3-VL's 235B-A22B backbone |
| [Parallelism](07-parallelism.md) | SigLIP's ring-structured loss, framed as DDP with cross-example interaction |
| [Critical batch size](critical-batch-size.md) | SigLIP's finding that 32K suffices and 1M does not help |
| [Data mixing](data-mixture-selection.md) | square-root-normalized per-token loss, so video does not dominate |
| [Data filtering](data-filtering.md) | CLIP filtering LAION, which then trains OpenCLIP |
| [Post-training](16-post-training-rlvr.md) | Qwen3-VL's SFT-on-long-CoT, distillation and RL |
| [Long context](qwen-vl-series.md) | the 8K→32K→256K curriculum, driven by video |

There is no assignment: "I guess we don't have any homework on doing this, but if
you're curious, I would encourage you to play around with training some of these
models" (≈1:17:17).

## All 32 figures

Every figure in this lecture is described in full in the
[course material](../raw/slides/17-multimodality.md), each under the section where
it appears, and all 32 are committed to `raw/images/17-multimodality/`. This page
embeds the fifteen that its argument needs. Before quoting any figure, read the
[figure audit](../raw/slides/17-multimodality.md#figure-audit) — it records what
was checked, what was corrected, and the three figures whose content is bounded by
a reader flag.
