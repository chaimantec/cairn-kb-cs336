# Modality projectors

The **projector** (or adapter) is the component that maps a
[vision encoder](vision-transformers.md)'s output into something a language model
will accept. It is the only genuinely new part of a
[vision-language model](vision-language-models.md) — the encoder and the LM both
already exist — and in [Lecture 17](17-multimodality.md) the sequence of
projector designs *is* the architectural story.

## The problem it solves

A vision encoder produces vectors in its own space. The language model expects
vectors in its embedding space. Those are not the same space, and a randomly
initialized map produces something that is "certainly not embeddings representing
any sort of natural-language token" (≈33:40).

So the projector's job is a **coordinate change**, and the alignment training
stage exists to learn it, with both towers frozen so that nothing else moves.

## Four designs, in order

### 1. A linear matrix — [LLaVA](llava.md)

One matrix $W$. Image features in, embedding-space vectors out, concatenated with
the text tokens. The program notes the alternatives of the day were more elaborate
— "Flamingo and Q-former are more complex" — and LLaVA's claim is that this
suffices.

### 2. A 2-layer MLP — [LLaVA-OneVision](llava-onevision.md)

The same idea with a nonlinearity. Part of the general upgrade of every component;
the lecture treats it as unremarkable.

### 3. Cross-attention to a fixed length — [Qwen-VL](qwen-vl-series.md)

"One layer of cross-attention, incorporating 2D positional embeddings, and mapped
to a fixed size of 256" (≈46:12). Two things are different in kind here:

- **Cross-attention rather than projection**, with learnable query embeddings.
- **A fixed output length**, whatever the input. This bounds the compute the LM
  spends on an image — and discards the ability to spend more on a more detailed
  one.

The lecturer flags the fixed length as provisional the moment he states it: "this
is definitely not very dynamic, but neither is the vision encoder at this point."
Qwen2-VL abandons it for
[dynamic resolution](image-resolution-and-tokens.md), where an image's token count
scales with its size.

The 2D positional embeddings are there because flattening a patch grid into a
sequence throws away which patch sat above which. Qwen-VL also adds special tokens
`<img>`, `<box>` and `<ref>`, which let the model **name a region in its output** —
the mechanism that makes grounding and pointing possible at all.

### 4. DeepStack — [Qwen3-VL](qwen-vl-series.md)

Every design above injects the image **once**, at the input layer. DeepStack
injects it into **several layers** of the language model.
[Paper](https://arxiv.org/abs/2406.04334).

> They noticed that the vision encoder already computes a stack of vision
> embeddings, and they're basically going to add these directly into the residual
> stream of the language model. So this is a bit more of a deep fusion of the
> vision encoder into the language model, as opposed to the vision encoder being a
> black box that just outputs a sequence of vectors. (≈57:04)

The consequence is that deep layers get visual information directly, rather than
only whatever survived the layers below.

> **Attribution caveat.** The lecturer describes DeepStack as "a paper from the
> DeepSeek team." No CS336 course material names its authors, so this KB neither
> confirms nor contradicts it. Check the paper before repeating the attribution.

## The trajectory

| | Injection point | Output length | Fusion depth |
| --- | --- | --- | --- |
| Linear $W$ | input only | proportional to patches | shallow |
| 2-layer MLP | input only | proportional to patches | shallow |
| Cross-attention | input only | **fixed at 256** | shallow |
| 2×2 patch merge (Qwen2-VL) | input only | **dynamic** | shallow |
| DeepStack | **many layers** | dynamic | **deep** |

Two independent axes are moving. One is *how much* the image is allowed to cost —
fixed, then dynamic. The other is *how deeply* it is mixed in — once at the input,
then throughout. The field went from treating the encoder as a black box producing
a sequence, to treating it as a stack whose intermediate representations belong
inside the language model.

→ [Vision-language models](vision-language-models.md) ·
[Lecture 17](17-multimodality.md)
