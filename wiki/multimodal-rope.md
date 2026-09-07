# Multimodal RoPE (MRoPE)

**MRoPE** extends [rotary position embedding](rope.md) from one dimension to
three, so that a token's position can express *where in an image* or *when in a
video* it came from, rather than only where in the sequence.
Introduced in [Qwen2-VL](qwen-vl-series.md) and revised in Qwen3-VL; covered in
[Lecture 17](17-multimodality.md).

## The problem

RoPE encodes position so that "the inner product between the vectors depends only
on the distance — the distance is defined, in 1D, by just the number of tokens away
from it" (≈50:49). Text has one dimension, so that is enough.

An image has two spatial dimensions and a video adds time. Flattening a patch grid
into a sequence throws away which patch was above which — and once resolution is
[dynamic](image-resolution-and-tokens.md), so that the number of tokens per image
varies, sequence position stops carrying reliable spatial information at all.

The question was raised by a student back in the [CLIP](clip.md) section (≈15:45).
CLIP had tried a 2D variant and "found it doesn't really matter that much," with
the caveat that CLIP only cared about classification. MRoPE is where it gets a
real answer.

## The construction

Each patch gets a **triple** rather than an index:

$$\text{position} = (t, h, w)$$

"So each position, each patch, is now a triple, defined by the coordinates. And
then, to compute MRoPE, for every dimension you compute the RoPE, and then you
concatenate" (≈51:37).

Text tokens, which have no spatial extent, take the same value in all three
components — so a text token following a video continues from one past the video's
maximum coordinate, with $t = h = w$.

## The flaw, and Qwen3-VL's fix

Qwen2-VL assigns the three axes to **contiguous blocks** of the embedding
dimensions: `[t t t t w w w w h h h h]`. The lecture explains why that is
sub-optimal, and the reason is a property of RoPE itself:

> The problem, if you remember RoPE, is that each component represents a
> different frequency. So this would mean that maybe all the temporal dimensions
> are low-frequency and all the height dimensions are high-frequency. (≈53:57)

Low-index RoPE dimensions rotate fast and encode fine local position; high-index
ones rotate slowly and encode coarse global position. Blocking the axes therefore
hands each axis a *different band* — time gets only fine resolution, height only
coarse, or vice versa, depending on the order.

**Interleaved MRoPE** distributes them instead:

```
Qwen2-VL:   [t t t t  w w w w  h h h h]     each axis gets one frequency band
Qwen3-VL:   [t w h t w h t w h t w h]       every axis gets every band
```

"They basically interleave them, once they realized that, and now all the axes are
exposed to both low and high frequency."

## Explicit timestamps as a separate mechanism

Qwen3-VL also takes time *out* of the positional encoding for one purpose. Before,
"the timestamp was kind of implicit in the positional encodings — each frame in a
video intrinsically gets a different… notion of time just because it's in a
different position." Qwen3-VL adds tokens that literally say the time, "so a token
like '0 seconds' is now a token that represents the time" (≈55:31).

The benefit is that it becomes referable content rather than geometry: "it's
something you can directly refer to, like, what happened after two seconds."

→ [RoPE](rope.md) · [Video understanding](video-understanding.md) ·
[Lecture 17](17-multimodality.md)
