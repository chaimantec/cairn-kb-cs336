---
title: "Lecture 2: PyTorch, Resource Accounting"
lecture: 2
video: https://www.youtube.com/watch?v=kuYAsz7zspQ
source: copy-edited from the YouTube auto-captions
verbatim_original: original/02-pytorch-resource-accounting.md
material: ../slides/02-pytorch-resource-accounting.md
---

# Lecture 2: PyTorch, Resource Accounting — transcript

**This is the edited transcript.** The auto-captions have been repunctuated,
segmented into sentences, stripped of filler, and had mis-heard technical terms
restored. No content was added, removed or reordered, and every `[MM:SS]` marker
is preserved in its original position — content that was under a marker is still
under that marker. That was checked mechanically against the verbatim captions
three ways: the 101 timestamp markers match exactly in order and form, the
inventory of numbers in the two bodies matches token for token, and every
paragraph's word count was compared with its original to catch content drifting
across a marker boundary. The verbatim captions are kept at
[`original/02-pytorch-resource-accounting.md`](original/02-pytorch-resource-accounting.md);
that file is the record of what was actually said, and this one is the readable
version of it.

Terminology was cross-checked against
[the lecture's source program](../slides/02-pytorch-resource-accounting.md)
(`lecture_02.py`), which is the authority for names and numbers written down.
This is an "executable lecture" — the source program's own text is the written
form of what Percy narrates, so its variable names (`seq1`, `hidden1`, `n`, `B`)
and constants (1979 teraFLOP/s, 8 H100s, 53 billion parameters) are the ground
truth used below.

**Restorations made.** These are places where the captions produced a *wrong
word* and the text now reads differently. Each was confirmed against
[the lecture source](../slides/02-pytorch-resource-accounting.md) unless the
note says otherwise.

| Caption | Restored | Confirmed by |
| --- | --- | --- |
| "the Marine project had a which was running" | the Marin project had a run which was running | source (`Marin 1e23 FLOPs run finished`) |
| "the computer optimal point" | the compute-optimal point | context (standard term; same garble restored the same way in Lecture 1) |
| "we ran the loss the model and it got lost within 0.05" | we ran the model, and the loss came within 0.05 | context |
| "on 1024 Actually, this should be H100 cell" | on 1024 H100s | source (motivating question: "on 1024 H100s") |
| "any number of entities" | any number of dimensions | source ("rank, which is the number of dimensions") |
| "the DeepSeek 3.2 model" | the DeepSeek-V3.2 model | source ("DeepSeek v3.2 model"; "DeepSeek-V3.2, 2025") |
| "in GP3, which is a fairly old model" | in GPT-3, which is a fairly old model | source ("One matrix in the feedforward layer of GPT-3") |
| "train on H1 eight H100s" | train on 8 H100s | source ("train on 8 H100s using AdamW") |
| "of the 1989 number" | of the 1979 number | source (same spec) and the lecturer's own earlier, correct "1979" |
| "seek one" / "seek two" / "seek ones" | `seq1` / `seq2` | source (`"seq1 hidden, hidden seq2 -> seq1 seq2"`) |
| "hidden one" / "hidden two" | `hidden1` / `hidden2` | source (`"heads hidden1"`, `"hidden1 hidden2"`) |
| "it will come up in the assignment once" | it will come up in assignment 1 | context (matches "assignment one" = assignment 1 used elsewhere) |
| "you have M points" | you have B points | source (`B = 16384 # Number of points`) |
| "W is a M by M matrix" / "a M by M matrix, another M by M matrix" | an N by N matrix / an N by N matrix, another N by N matrix | source (`n = 1024`) |
| "compute a value on this" / "like value or dot products" / "element-wise value" / "a point-wise value activation" / "ignore this value for now" | compute a ReLU on this / like ReLU or dot products / element-wise ReLU / a point-wise ReLU activation / ignore the ReLU for now | source (`Block.forward` applies `F.relu(x)`); the transcript itself self-corrects right after the first instance ("...a value on this. So, remember ReLU is just...") |
| "an X which is BID" | an X which is B by D | source (`x = torch.ones(B, D, ...)`) |
| "I'm going to call these retain gradients for debugging purposes" | I'm going to call `retain_grad()` on these, for debugging purposes | source (`h1.retain_grad()`, `h2.retain_grad()`) |
| "you have to call uh CUDA synchronize" | you have to call `torch.cuda.synchronize()` | source (`torch.cuda.synchronize()`) |
| "each parameter takes two bytes if we're storing an FP16" | each parameter takes two bytes if we're storing a BF16 | source (`parameter_memory = 2 * num_parameters # (2 bytes for bf16)`) |
| "by default everything we're doing here is FP...16" | by default everything we're doing here is BF16 | source ("arithmetic/accelerator intensity also depends on the precision (bf16 versus fp32)"; every worked example uses `torch.bfloat16`) |
| "you store the first order bit and the second order moments" | you store the first-order moment and the second-order moment | source ("Adam: 8 bytes/parameter for storing first and second moments") |
| "the number of flops is D times number of tokens...times number of parameters" | the number of flops is 6 times the number of tokens...times the number of parameters | source (`flops = 6 * B * num_parameters`) and the lecturer's own 6ND formula stated minutes earlier |
| "which allows you to save on compute" | which allows you to save on memory | source (Summary: "reduce memory to use bigger batch sizes") |

**Capitalization and formatting only.** The following were already correct in
the captions and were only cased, hyphenated or code-formatted — they are not
restorations, and a reader comparing the two files should expect them:
float32, float64, FP16, FP32, BF16, FP8, FP4, NVFP4, HBM, MFU, AdamW, AdaGrad,
NVIDIA, IsoFLOPs, sub-cubic, back-of-the-envelope, row-major, column-major,
compute-bound, memory-bound, `dim=-1`, `w.grad`, `h1.grad`, `h2.grad`,
`loss.backward()`, `torch.utils.checkpoint`, `g2`, `g3`, `h2` (as checkpoint
labels), and the "d loss / d h1" chain-rule notation (spoken as "D loss D H1"
and written here with the slash, matching the source's own
`h1.grad = d loss / d h1` convention).

**Numeral conventions.** Matching Lecture 1's convention, assignment numbers are
written as digits: "assignment one" → "assignment 1" (three plain instances,
distinct from the "assignment once" garble listed above, which is a restoration).
Spoken dimension names became their written forms — "seek one" → `seq1`, "hidden
two" → `hidden2` — and spoken exponents were joined up: "5e 21" → "5e21", "1979
E12" → "1979e12". The 6 in the 6ND formula, spoken as "six", is written as a digit.

One caption stutter was dropped: at ≈37:45 the captions read "potentially like 80
uh 0.8 um even", where "80" is a mis-segmentation of the "0.8" that immediately
follows rather than a separate quantity, so the text reads "potentially 0.8, even".
No other quantity, budget, model size, FLOP count, or measured value was altered,
rounded, or converted.

**Marked as unclear.** Eight spots carry an inline `*[Ed: …]*` note rather than a
guess. Two are garbled captions; the rest are places where what was said and what
the lecture's own program computes do not agree, and the transcript records what
was said.

Garbles left unresolved:

- "this glossy's spec sheet" (≈29:16) — kept as "this glossy spec sheet"
  (a plausible informal term for a GPU maker's marketing spec sheet), flagged
  because it's not confirmed by the source text.
- "big just big" (≈55:22), part of a student's question about accelerator
  design — the sentence appears to be missing a word and is left as heard.
- "800 989 teraflops" (≈28:30) — left as the captions have it. This lecture uses
  **both** 1979 teraFLOP/s (with sparsity, at 29:16) and 989 teraFLOP/s (dense, at
  36:12), so either could be meant; "about 989" is the likeliest reading but is
  not certain enough to write in.
- "So float 30 16 a BF 16" (≈10:52) — read as BF16, with the stray tokens flagged.

Said-versus-computed discrepancies, recorded as spoken:

- **GPT-3's FLOP count** (≈28:30): "3e25, or whatever it is" — he hedges, and the
  lecture source says 3.14e23 for GPT-3 (2e25 is its speculated GPT-4 figure).
- **8 H100s for two weeks** (≈30:03): "about 5e21", where the source expression
  computes 9.575e21.
- **Matmul arithmetic intensity** (≈51:33): "It's 300 — woohoo. Okay, so 340",
  where the source computes ≈341.17. Both spoken numbers are kept, including the
  one he corrects a moment later.
- **Checkpointing every √L layers** (≈1:15:28): he says the recomputation overhead
  is "also square root of L", where the source states O(√L) activation memory and
  **O(L)** recomputation.

None of these were altered in the body. Where this knowledge base's wiki pages
quote a number, they follow the lecture *source*, which is the authority for what
was written down — see [`AGENTS.md`](../../AGENTS.md).

**Speakers.** Lecture 2 is taught by Percy Liang. Student questions are run
together with his speech in the auto-captions with no speaker labels; where the
recording makes clear that he is answering or repeating a question from the
floor, the text carries a `*[Question from the floor]*` marker (or
`*[Interjection from the floor, inaudible]*` where the captions only register a
sound — a laugh, a cough, a snort — at a marked speaker change, following the
same convention used in Lecture 1's transcript).

---

**[0:05]** I hope everyone is staying dry. I'm not. So, as I mentioned last time, the Marin project had a run which was running, and it finished, and it actually matched the forecast. Remember, we were running these — each of these curves is essentially an IsoFLOPs curve, which is a bunch of smaller model runs, and you try to find the compute-optimal point. You fit a scaling law, and this was the point where we predicted the loss, and we ran the model, and the loss came within 0.05. So, I thought that was pretty cool, and if you extrapolate out to GPT-5-level performance, this is the loss you get. Of course, your mileage might

**[0:52]** vary depending on how these scaling laws are. Okay, just wanted to share that news. So, last lecture I gave an overview of the entire class, and we are talking about tokenization, which is going to be on the first assignment. Today I want to talk about resource accounting, which is going to be more on the systems side of things. So, to recall, the main thing we're trying to do is train the best model we can given a finite set of resources, which could be compute, memory, sometimes data — but that's not really going to be a limiting factor for us in this class.

**[1:39]** And our goal is simply to maximize the computational efficiency of our training. So, before you can optimize the computational efficiency, we need to understand the efficiency of a given computation, and for that we need to understand the compute and memory characteristics. Just to give you a taste of the type of questions you will hopefully be able to answer by the end of the class — here's a question. How long would it take to train a 70 billion parameter model on 15 trillion tokens on 1024 H100s? So, how do you answer that? Well, there's a formula, which we'll talk about, for how you can get the number of flops to be six times the number of parameters

**[2:24]** times the number of tokens. We can look up the spec sheet to see how fast the H100 is. We have this thing called MFU, which we'll talk about — 0.5. Then you can estimate the number of flops that the hardware gives you per day, and then you can compute the number of days. So, the number of days is 143. Okay, here's another question. What's the largest model you can train on 8 H100s using AdamW? Well, H100s have 80 GB of HBM memory. The number of bytes per parameter,

**[3:13]** which is 2 + 2 + 4 + 4. We'll explain where that comes from. And then the number of parameters you can get is going to be about 53 billion. So, there's some caveats here — we don't count the activations, which depends on the batch size and the sequence length. So, this is all very rough back-of-the-envelope calculations, but hopefully by the end of this class you'll understand where these come from, and the point is not to precisely calculate every single thing, but just get the rough shape of things. Okay. So, last time I talked about knowledge and what you can take away

**[3:59]** from this class: mechanics, which are how things work. So, today that will be pretty straightforward — the mechanics are just how PyTorch works, how tensors work. There's no magic here. The mindset I want to impart on you is that resource accounting is going to be very crucial, and I want everyone to get into the habit — whenever you write a line of code — of thinking about the performance characteristics. And then finally, intuitions: here we're just going to get a sense of the resources, how they're spent. There's going to be no ML magic today. I'll leave that to Tatsu for the next lecture. Okay.

**[4:44]** So, let's get into things. Let's start bottom-up and start building up. So, what is at the bottom? The bottom are tensors. Tensors are the building block for storing everything. If you have parameters, gradients, optimizer states, data, activations — everything essentially is a tensor. For example, you can take a look at the DeepSeek-V3.2 model, and you see that the model itself is a bunch of different tensors. Each tensor has some shape and also some precision, which I'll talk about later. And tensors subsume vectors, matrices, and

**[5:29]** can generalize to any number of dimensions. Okay, so let's talk about how much tensors take to store. It depends on the type of tensor. In general, we're going to be dealing with tensors that store floating point, but tensors can also store integers and other types. For floating point, typically whenever you talk about float, I think the standard people refer to is what's called float32. A float32, if you break it down, has 32 bits. One of the bits is a sign,

**[6:15]** eight of the bits are the exponent, which gives you dynamic range, and the rest is the mantissa, or the fraction, which gives you variation. Okay, this is also known as FP32, or single precision. The term single precision comes from the fact that back in the day, when you were doing scientific computing, float32 was sort of like a baseline — if someone gave you a float, you would kind of expect it to be at least single precision, and if you wanted more precision, you could get double precision, that's float64. But in deep learning, we're kind of going the other way, because even 32 is a

**[7:01]** lot, and the types of computations that we want to do don't demand the high precision that some kind of numeric simulations do. Okay, so before we get to other types, let's just look at float32. Let's construct a 4-by-8 matrix. By default, the type of a tensor you create is float32, so if you want something else, you should declare it. The memory usage is just the number of elements times the element size, which is 4 bytes for a 32-bit number, and that's going to be 128 bytes. Just to give you some perspective — in GPT-3, which is a fairly old model,

**[7:46]** one of the matrices in the feedforward layer is about 2.3 GB. So, these tensors can get quite big, and this is by far even the biggest one that one can imagine. Since we're interested in efficiency, we want to generally reduce the amount of storage, and we'll see that as you reduce the precision, you actually save memory, and you also save time, because operating on 16 bits is going to be faster — let's say twice as fast, but not always, it sort of

**[8:32]** depends. And then, by reducing memory, we'll see later that actually reducing memory can save time as well, which is maybe less obvious, but it will hopefully become clear. So, the obvious thing is you say, okay, let's take away half the bits — now you have float16. Float16 says you have a sign, you have only five bits of exponent, and then the rest is the mantissa. So, fine, float16 is good, except its dynamic range is kind of poor, right? So, even if you try to construct a 1e-8 tensor, then that is actually

**[9:17]** just zero. So, you can't really represent very big numbers, and you can't represent very small numbers. And the reason is that this exponent is only five bits of exponent, compared to eight. So, if you train with FP16, which people did back in the day, you will get instability — you get underflow, you get overflow, you'll get NaNs. It was pretty challenging. So, then BFloat16 was invented. This was developed in 2018 to address this issue.

**[10:03]** And the observation was, well, let's not compromise on the number of bits — the number of bits is going to be the same as FP16, but we're going to shift some of the bits from the mantissa to the exponent. That means it has more dynamic range than float16. It actually has the same dynamic range as float32. But of course the resolution is worse, because there's no free lunch here. But it turns out that in a lot of deep learning applications, this is well worth the trade-off. You want the dynamic range to not overflow and underflow, and because things are kind of sloppy and stochastic anyway, you don't need that much resolution.

**[10:52]** Okay, so let me actually skip over this part. So, to summarize, what are the implications for training? You can absolutely train with float32, and if you're training a small model, you probably don't want to worry about it — float32 is fine. But it requires four bytes of memory per float, and that can take up a lot of memory. And if you train with float16, then that's going to be too risky. So, BF16 *[Ed: captions read “So float 30 16 a BF 16”]* is

**[11:39]** sort of a sweet spot. Even BF16 can be kind of risky as well — we'll maybe see that in a little bit. One thing that has become kind of common practice is to use mixed precision training. And actually — let me compile this again, I think these are stale. So, mixed precision training is where some of the computations use some precision, and for some other computations you use other precision. So, as a general rule, BF16 is

**[12:24]** what you would use for parameters, activations, and gradients. And for optimizer states, you would use FP32. We'll discuss more about that later. And to invoke mixed precision training, PyTorch has an AMP library. We're not going to talk too much about this, but you basically wrap your code, and the library takes care of it automatically for you, in the sense that it tries to cast things into BF16 when it's safe. For example, matmuls are generally safe, but if you try to do exponentiation, then it will try to leave things as FP32. Okay, so BF16, I think, is probably where this class will end. But

**[13:11]** if you're feeling very adventurous, you can go farther. So, FP8 — this was introduced four years ago, and it's actually been standardized. If you look at FP8, there are actually two versions, depending on whether you need more dynamic range or more resolution. We're not going to talk about that, but NVIDIA's transformer engine supports FP8. And, well, most recently, you can actually go down to FP4. Last year NVIDIA developed NVFP4, and there's only four bits per value. So, just so we're on the same page, four bits is not a lot — I can write all the

**[13:57]** values down on a single line here, between minus six and six. So that's not very much precision. Now there's a little bit of a cheat here, right? Because if you just naively only use these values, you're not going to be able to train very well. So what this actually means is that every value has this four bits of freedom, but there are blocks, and each block can be scaled up and down accordingly. So you can actually represent more values, but you can't represent the full dynamic range for every single value. And there was a model released this year, Nemotron 3 Super, which was trained in FP4, which I think

**[14:42]** is pretty cool. Okay, so some of this is just good to know, and some of this you can't even touch — it's not like you create a tensor and call it FP4. A lot of this is done under the hood by NVIDIA's software stack. Okay, this was a bit of a long digression, but it's maybe helpful to appreciate the intricacies of precision here. Any questions before I move on? Yeah.

*[Question from the floor]*

Sorry — there's a question: when you have a block, you're scaling all those same values, like, you have a tensor, and you scale by that block or something like that — the ratios of all of them will still be

**[15:28]** in FP4? Yeah, so the question is just about scaling with the maximum value to get more specificity.

Yeah, so — let me explain the block a bit more. Say you have a block. Within that block, you can vary within the four bits. And in addition, all those values can be scaled up and down according to your scaling factor. So if you look at an individual value, you actually get more than four bits of dynamic range, but you can't have this value be way over here and the next neighboring value way down there.

*[Question from the floor]*

Okay, so, one question — what about one bit? Couldn't BF16 be pushed down to one bit

**[16:15]** and all of that? Yeah, so there's a problem with training, obviously. Right, right — so, maybe this is a good point. The question is about one bit, because you can't really go lower than that. There's a difference between training and inference. A lot of the low-bit stuff — we'll talk about quantization later — is that you train a model at, say, BF16, and then you quantize it into, let's say, one or two bits. And that is much easier than training a one-bit language model, which I don't think anyone has done. Maybe it's possible, but I don't think anyone has trained anything credible there. Okay, let's go on. So we talked about

**[17:03]** tensors, and the memory calculus is pretty simple — just the number of elements times however much memory each element takes up. By default, the tensors you create in PyTorch are going to be on CPU, and of course, if you want things to go fast, you want to move them to GPU. Actually, I have a slight issue with the slides — they were executed on my laptop, which means I don't have a GPU. So some of the code I'll just show but not execute. Let's see — this is maybe not that interesting, but I think everyone knows how to move

**[17:50]** tensors to GPU. Just remember to do that, otherwise you won't get your speedups. Okay, so we talked about memory of tensors, which is very straightforward. Now let's talk about computing with tensors. Before talking about flops accounting for tensor operations, let's take a little bit of a digression to talk about einsums. How many of you are familiar with einsums? Okay, so like maybe two-thirds — okay, good. The motivation behind einsums, for those of you who might not be indoctrinated, is that it's very easy to mess up — I find it very confusing to

**[18:35]** look at code such as this: you have X and Y, and then you have transpose minus two, minus one, and you're trying to figure out what minus two and minus one is. So this is maybe the motivation for using variable names rather than indices. Einsums is a library for manipulating tensors where the dimensions are named. This is inspired by Einstein summation notation. There's a nice tutorial you can go through — I'm just going to cover some of the basics. Basically, the way to think about einsum is a generalized matrix

**[19:21]** multiplication with good bookkeeping. Here's an example: I have a matrix, 3 by 4, and a 4-by-3 matrix, and if I do the matmul — this is actually pretty nice, it's pretty easy to understand. In einsums, basically you say X has two dimensions, the row and the column, which I'm going to name `seq1` and `hidden`. I'm going to have Y, a matrix which also has rows and columns, which I'm going to name `hidden` and `seq2`. And I'm going to produce a tensor — here, a matrix — where the dimensions are indexed by

**[20:07]** `seq1` and `seq2`. And anything that is not mentioned here — the hidden — gets summed out. So, the way this works is: I'm going to enumerate over all possible values of all the variables that occur here, which are `seq1`, `hidden`, and `seq2`, and I'm going to index into X, index into Y, multiply them, and accumulate them into the result, Z sub `seq1 seq2`. All right — this is much easier than this, right? Okay, let's try a more complicated

**[20:52]** example. Here's the example I showed before — now we have a tensor, 2 by 3 by 4, and another tensor, 2 by 3 by 4. If I were doing things the old way, basically what I'm doing is transposing the last two, and then it implicitly batches the dimensions that are not the matmul dimensions, and then you get the answer. So, you have to kind of reason about this a bit. Einops makes this very clear — it says there's a batch dimension, there's `seq1 hidden`, `seq2 hidden`, and I'm just going to produce `batch seq1 seq2`.

**[21:37]** And notice that there's no transpose, because in some sense I've done the transpose by the naming. If I had `hidden` and `seq2`, then it would be a no-transpose. I always get confused by transposes, and the fact that I don't have to think about transposing makes me happy. Okay, so if you want to get fancy, you can say, well, `batch` — I'm just going to replace it with `...`. This means that if I had, let's say, a rank-10 tensor with eight different batching dimensions, I can just write `...` without enumerating all of them. This comes up in language modeling, because you might have

**[22:24]** a batch dimension, you might have a sequence dimension, you might have a head dimension, and you're trying to do this matrix operation for all of them, and you might not want to have to worry about this. The nice thing is that you can write modular code where you can write this `...` without worrying about even the shape of the tensor that comes in. All right, so that's einsum, which is in the einops library. There's also reduce — this is a generalization of sum, mean, max, and min. For example, if you have, let's say, this tensor,

**[23:09]** and you want to sum according to `dim=-1`, which means sum along the last dimension here. Again, I don't like this notation, but what you can do is call reduce. Basically, what you say is that there are some batching dimensions — in this case, the first two — and then there's a hidden dimension, which doesn't appear on the right side, which means it gets summed. Here I put `sum`, which means the aggregation reduction operation is sum, but you can replace it with mean, max, or min.

*[Question from the floor]*

Is there some speedup to this?

Is there some speedup to this? This basically reduces to the same type of primitive operations. You can think

**[23:55]** about it as just sugar, so it should be the same. Okay, so the final thing I'll talk about is rearrange. I think this is a pretty powerful tool — it'll come up in assignment 1. Sometimes you have a dimension that actually represents two dimensions, and you want to operate on one of them. The reason this happens is that sometimes you have a matrix and you flatten it, and then you want to maybe unflatten and flatten again. So, this is the way that works here. Imagine I have a matrix, 3 by 8, but where this

**[24:40]** dimension eight actually represents a 2-by-4 matrix. I want to multiply that 2-by-4 matrix by this 4-by-4 matrix. So, what I'm going to do is call rearrange. What I'm doing here is saying: look, this is some number of batch dimensions, which here just corresponds to the first element. And then here I use parentheses to say that this H actually represents the product of `heads` and `hidden1` — obviously there are multiple ways to decompose this, could be 2 by 4, 4 by 2. So, I said the number of heads is two,

**[25:25]** which means that `hidden1` is four. And then I can break that up into two dimensions, `heads` and `hidden`. So, this creates — before, X looked like this, and now X looks like this — this might be a little hard to see. Then you can perform your transformation on W. This is sort of what we've seen before, where you have just a standard matmul, with some number of batching dimensions for X here. This is `hidden1` times a `hidden1`-by-`hidden2` matrix.

**[26:11]** And then once you've done that transformation, you can rearrange it back. This is straightforward — you basically look at two dimensions, and then you group them into one dimension.

*[Question from the floor]*

If you take a two-dimensional thing and shift it into one dimension, are there two ways to do it — row-major or column-major?

So, the question is: if you have a two-dimensional thing and you shift it into a one-dimensional thing, which way do you do it? Well, the order you do it in is specified by the order here. Oh, okay. Yeah, okay.

**[26:57]** Sometimes I find it takes a bit of time to get used to, but it's well worth it, because once you have einops, you just think in a different way, and all the transposes and reductions become more fluid. You just have to think through these more bespoke primitives. All right, so we're going to use einops — we'll see it a little bit later. Now let's return back to the resource accounting question. So, I have tensors — we've talked about how they take memory. So, how much compute do they take? The thing we're going to use to measure computation cost is the number of flops.

**[27:43]** A flop is a floating-point operation, and we're going to assume it's a basic operation like addition or multiplication. Now, there are other things GPUs can do, but for the most part we're just going to ignore them, because these are the bread and butter and are going to eat up most of your time. One thing that's kind of a pet peeve of mine is that if I say the word "flops," it's actually ambiguous what I mean. There's FLOPs, which is the number of floating-point operations, usually written FLOPs with a lowercase s — this is a measure of the amount of computation done. And then there's FLOP/s, which is floating-point operations per second. Sometimes it's also very confusingly

**[28:30]** written as FLOPS with an uppercase S, but I'm going to always write /s to make it clear that this is measuring the speed of hardware. So, if you go and see that H100s have 800 989 *[Ed: unclear — the captions read “800 989”; this lecture uses both 1979 teraFLOP/s (the with-sparsity figure, at 29:16) and 989 teraFLOP/s (the dense figure, at 36:12), and this is plausibly “about 989”]* teraflops, that's the latter. And when I say that GPT-3 took, you know, 3e25, or whatever it is, FLOPs, that's the former *[Ed: the lecture source gives 3.14e23 FLOPs for GPT-3; 2e25 is its speculated figure for GPT-4]*. Okay, just to get that out of the way. Just to give you an order of magnitude — when I talk about 1e22, or 23, or 25, these are kind of referring implicitly to the amount of compute, or the scale of some of these models. So, if you look at H100s,

**[29:16]** if you look at this glossy spec sheet *[Ed: unclear — captions read "glossy's spec sheet"; kept as heard]* — actually, it's not on this page, okay, forget it — there is a spec sheet, and it'll tell you that for BF16, the number of FLOP/s is 1979. And then you go and you benchmark, and it's like, wait a minute, that's not actually what I'm getting. And then you go read the fine print, and there's a footnote that says this is with sparsity — so, sparse matrix — and for dense it's over two. So, you always have to take these numbers and divide by two. That's why you see this divide-by-two. Okay.

**[30:03]** So, this allows us to build intuition — so, if you have eight H100s, one node, for two weeks — that's the number of seconds in two weeks. Actually, this looks like it's one week — okay, fine, it's one week — times the number of flops you get per second. So, that's about 5e21 *[Ed: the lecture's own code computes 9.575e21 for this expression]*. Okay, so this is just building intuition for the number of flops that certain types of hardware have, and how many flops certain types of models require. It's nothing fancy, it's just math — napkin math.

**[30:49]** Okay, so now let's do something more mechanical. Suppose you have a linear model. It turns out that a lot of this calculus of counting flops is actually going to be, at the core, about linear matmuls, so this is actually not without much loss of generality. So, you have B points, each point is D-dimensional, and we're going to map each of these D-dimensional vectors to a K-dimensional output. So, B is going to be the number of points, D is the number of input dimensions, and K is the number of output dimensions. So, let's construct some

**[31:36]** X, which is the data matrix, B by D. The weight matrix is D by K, and we do the matmul. The question is, how many flops is that? It turns out that this is going to be two times the product of all three dimensions. And the way to see that is that we have one multiplication for each triple, and then also one addition. So, there's a minus one, because you actually have to add D minus one times, but let's ignore that.

**[32:23]** Okay, I'll come back to this if it was a bit fast — there's another way to derive this. So, what about the flops of other operations? Element-wise operations are just the size of the matrix, I think that's fairly clear — addition also requires NM flops. In general, no other operation you encounter is as expensive as matrix multiplication, for large enough matrices, so in general we're just going to focus on what matmuls are doing, with the important caveat of when we talk about memory.

*[Question from the floor]*

This is just for interest — there are some other algorithms for doing matrix multiplication. Is this only for, just, doing it

**[33:08]** normally?

So, the question is that there are other algorithms for doing matrix multiplication — you mean like sub-cubic algorithms. In general, the optimization that algorithms people are going to explore for most matrix multiplications are going to be much more about how you co-design with the systems, rather than these more asymptotic algorithms. Yeah.

*[Question from the floor]*

We're considering addition and multiplication in the same way — is it not possible to do addition more efficiently than multiplication?

Yeah, so I think the way the hardware is built, the two are basically the same. Intuitively it seems like you could do addition faster than multiplication, but

**[33:53]** the way the hardware is built, they're kind of the same. Okay, so you can think about this matmul as: B is the number of data points, and DK is the number of parameters — remember, X is B by D and W is D by K. So, another way to think about this formula is that the number of flops to do a forward pass of this linear matrix is actually two times the number of tokens, or data points, times the number of parameters. It turns out that this actually generalizes to transformers, which, if you remember the

**[34:39]** 6 times N times D formula, you can kind of see the shape of that forming. Okay, so unfortunately these calculations are not going to be very meaningful, because I'm doing this on CPU, but I'll just walk through the code here. So, what we've done so far is measure flops — this is independent of hardware, it's just the number of calculations you need to do for your model. Now the question is, how long does it actually take on hardware? One way to find out is you just time it. In this class, I think in a few lectures we're going to talk more about benchmarking, but here's a

**[35:25]** little preview. In general, when you time things, especially on GPU, you have to call `torch.cuda.synchronize()` to make sure — because the GPU is running asynchronously — that you have this synchronization point. Then you perform the operation, and after the operation, you also have to have the synchronization barrier. If you omit this, you're going to find that, wow, your timings are really fast, and that's because this is a non-blocking call, it just returns. And it's often good practice to try this multiple times and take the average.

*[Interjection from the floor, inaudible]*

Okay. So,

**[36:12]** the actual flops per second is basically the number of flops you did, times the time that you recorded on your hardware. And, remember, there's also a number — the GPU has a spec sheet. Let's see if I can pull it up on this link. I feel like maybe I have the wrong link. I'll have to fix that after class. That gives you a number of flops, which was 989 teraFLOP/s. So, in general, the number of actual

**[36:58]** flops per second is going to be different from the promised flops per second. And the way to think about the discrepancy directly is something called model FLOPs utilization, or MFU. The definition of MFU is the actual flops per second divided by the promised flops per second, and here, this is ignoring the communication and other overhead. So, you basically take the actual and divide by the promised. In general, it's rare that you get more than what's

**[37:45]** — you just never get anything more than what you were promised. Often you get less. In general, if you get about an MFU of 0.5 for modern models, you should be pretty happy with yourself. If you have just a straight-up matmul, you can get maybe potentially 0.8, even, but you usually can't get that high. And sometimes, if something's really wrong, you'll get something like 0.1, which means you should do something about it. So, whenever you write your model, you now know how to calculate MFU, through a combination of counting the number of

**[38:31]** logical flops that your model needs to do, and then looking at the wall-clock time and dividing.

Was there a question over there?

*[Question from the floor]*

What is the promised flops number?

That is, in the spec sheet — it's already divided by two, of the 1979 number. And then, on top of that, you only get like 0.5 of that, in general, depending on your computation. You're getting 50 percent —

*[Question from the floor]*

Why are you getting only 50 percent

**[39:17]** MFU?

That's a good question — I'll come back to that when we talk about memory bottlenecks. Okay, so to summarize here: matrix multiplications dominate the computations, generally, and that's sort of by design. The number of flops per second depends on the hardware — better hardware leads to more flops — and it also depends on the data type, which means that if you look at the spec sheet, different data types will have different flops. If you try to do float32, nowadays it's

**[40:02]** going to be really slow, because they're not really optimizing for that workload, whereas now BF16 or FP8 are going to be much faster. And MFU — now you know what MFU is, it's the actual flops divided by promised flops. All right, so, to go back to the question of why is MFU 0.5 — to understand that, I'm going to have to introduce this idea of arithmetic intensity. And the reason is that, well, it's not just doing a bunch of matmuls and then you're done, looking at how long the matmuls take.

**[40:49]** This is my very cartoon version of what hardware looks like. You have high-bandwidth memory, and then you have where the compute cores, or the accelerator chips, are. And, so, how do you compute? Well, you have to send your inputs — your matrices, your tensors, are sitting down here — from the memory to the accelerator. You do the computation, and then you send it back. So, if you want to measure how long this takes, it depends on two things. One is the accelerator speed, which is what we've talked about just now.

**[41:34]** But the other thing that matters is the memory bandwidth of your hardware, which we haven't talked about. And, if you look at the spec sheet, we talked about how the flops per second was 1979e12 / 2. And the bytes per second, which is the memory bandwidth, this is 3.3 terabytes per second. And remember why we were looking at memory — how much things take to store — it's most obviously that, well, if you have a model that's too big, it doesn't fit in your memory, that's not going to be fun. But also, it turns out that the memory you need to move — this

**[42:19]** movement — takes time. So, actually, the size of how large things are influences speed as well. Okay, so I'm going to talk through some operations, and compute how long things are going to take, and introduce this idea of arithmetic intensity. Suppose I have a million-dimensional vector of BF16, and I'm going to compute a ReLU on this. Remember, ReLU is just max of X and zero, done element-wise on the entire vector. So, I count two things — one is the number of bytes that were moved.

**[43:05]** I have to read X — copy it into the accelerator — and this is going to be two, because BF16 is two bytes per float, times N floats, so that's 2N. And then I'm going to write Y back, so that's another 2N. So, that's the number of bytes that have to be moved. And then, how many flops were done? Well, each of these elements, I'm just comparing with zero, and that's it, so that's N comparisons. Now I look at the communication time, which is the number of bytes I needed to move,

**[43:50]** divided by the speed of that movement, and that gives me the time, which is 1e-6 seconds. And what about the computation time? That's the flops divided by flops per second, so that's 1e-9. There's also another important assumption, which we generally try to hold, which is that we overlap communication and computation. We'll talk more about that when we talk more deeply about GPUs, but the idea is that, in this case, we don't sit here waiting for the things to move — as soon as they're there, we start computing them, and then we move them back. So, this

**[44:38]** movement, and also the compute, is happening at the same time. So, mathematically, we're just going to assume that the total time is the max of the two, because we assume we can perfectly overlap them. In practice, it's not going to be perfectly overlapped, there's going to be some overhead, but this is good enough for now. So, the total time, as we see here, is 1e-6. So, if you ask what the bottleneck is here: when the communication time is greater than the computation time, we call the algorithm memory-bound, because you're spending most of your time just waiting for bits to show up. And when the computation time is greater than the communication time, that's compute-bound,

**[45:24]** because then your bottleneck is actually doing the compute. So, in this case — what is ReLU? Rectified linear unit, oh, sorry — is it memory-bound or compute-bound? Memory-bound.

*[Interjection from the floor, inaudible]*

Memory-bound, yeah. And it's clear, right, because the compute is way less than the communication. Okay, so here's another way to see it, and this is where I'm going to define intensity. The intensity of an accelerator is essentially how much work the accelerator can do per byte transferred. And for any given accelerator, based on

**[46:11]** the spec sheet, you basically have the flops per second divided by the bytes per second — how much useful work can you do per byte that's moved? For H100s, that's 295. So, that means for every byte, you can do 295 floating-point operations. That's an intuitive number to have in your head — about 300. Okay, so now, the arithmetic intensity of an algorithm is how much actual work is done per byte for this workload. And if you look at it for the ReLU computation, it's flops over bytes,

**[46:56]** and it's actually — this is actually a quarter, I guess, not half. So, the point is that it's very small, 0.25. Okay, so now we can talk about bottlenecks through the language of intensity: something is memory-bound if the arithmetic intensity is smaller than the accelerator intensity, and compute-bound if it's greater than the accelerator intensity. All right, so these are equivalent — if you look at the algebra, you have two fractions, and you just multiply and divide to switch the terms around. So, in this case, we're memory-bound.

**[47:42]** In general, we're going to find ourselves in a situation where we're memory-bound, because data movement is expensive, so if you can get higher arithmetic intensity, that's good. So, 0.25 — if you see that number, if someone tells you your arithmetic intensity is 0.25, you should say, "Oh, this is really bad."

*[Question from the floor]*

What's some typical arithmetic intensity for—

Yeah, we'll get to that. Okay. All right, so one way to think about increasing arithmetic intensity is: let's just try to do more work per unit of byte moved. So, GELU is another activation, it kind of looks like this. It's

**[48:27]** some formula that's more — it doesn't have zeros. And if you do this calculation, the number of bytes moved back and forth is still 2N + 2N, and the flops here — it's about 20 flops per element-wise GELU operation, so it's 20N. So, the arithmetic intensity is, let's say, five — this is a kind of crude estimate. And in this case, are we memory-bound or compute-bound? Memory-bound. Memory-bound, right, because five is still smaller

**[49:13]** than 295 — way smaller. So, even though GELU does a lot more work than ReLU, in the way things are structured, it's still memory-bound. Which means that if you were just computing ReLU and GELU, you'd think, well, GELU is so complicated, it must be really expensive — but actually it's exactly the same, because that's not where the bottleneck is. Okay, so now let's look at some linear operations — dot product. You have a vector X, a vector W, size N, and you take the dot product. So, how many bytes are moved? You read X, which is 2N,

**[50:00]** read W, which is 2N, and then you write Y, which is a scalar, which is two. And the number of flops: you do N multiplications and N minus one additions, so that's 2N minus one. So, what's the arithmetic intensity? For this one, it's about half — which is also pretty bad, which means, hopefully you get the idea, it's memory-bound. So, what about a matrix-vector product? X is a vector, W is an N-by-N matrix, and you perform this product. How many of you think this will be compute-bound?

**[50:46]** How many of you think memory-bound? Okay, let's see. I'm going to read X — read, which is 2N — read W, which is 2N squared, and write Y, which is 2N. And the number of flops is basically doing N dot products, so that's N times the cost of doing a dot product. And the arithmetic intensity is barely higher. So, this is also memory-bound. Now let's talk about matrix multiplication — this is where things get interesting. I have an N-by-N matrix, another N-by-N matrix, and I multiply them. And the number of bytes that were

**[51:33]** moved was 2N squared, plus 2N squared, and then I have to write 2N squared bytes back to Y. And the number of flops here is N squared dot products. So, what's the arithmetic intensity? It's 300 — woohoo. Okay, so 340 *[Ed: the lecture's own code computes ≈341.17 for this case]*. And in general it's roughly N over three. And intuitively this makes sense, because you're sending N-squared things, but you're computing N-cubed things — so, the number of things you're computing over the number of things you're sending is order N. And this gets better the larger you make the matrices.

**[52:19]** So, in general, this is why when you hear people talk about, "Oh, we need to make large batch sizes," or, "have large matrices" — it's exactly this. If you're under the accelerator intensity, making things smaller doesn't actually speed things up, it's all kind of the same. Whereas, if you get to the point where you're over this, then you're actually saturating your GPUs. So, finally, this is compute-bound. So, as long as we have large matrices, we're actually pretty good — we're compute-bound, saturating the accelerator. And the question from earlier, about what about transformers — well, it turns out, we'll see

**[53:05]** in both your assignment, but also in the next lecture, that transformers are essentially big matrix multiplications with some things sprinkled in between. So, that's good news from an arithmetic-intensity standpoint, and this is by design — transformers are designed in a certain way to have high arithmetic intensity. And one comment here, just to foreshadow the inference lecture: matrix-vector products are essentially what goes on when you're doing transformer inference, because during inference you're generating one token at a time, so it's like a vector that you're trying to dot-product with a matrix, and that's where, as we saw, it's memory-bound. Whereas in training time, you get this

**[53:51]** whole sequence, and you process it all at once. Okay, and note also that the intensity depends on the precision — by default, everything we're doing here is BF16. Okay, and to tie this to the other question about MFU: the reason you might be getting low MFU is that while you might be doing —

*[Interjection from the floor, inaudible]*

— so, MFU is, sorry, actual over promised. So, if you have really large memory bandwidth bottlenecks, then you're not actually going to get very good throughput, even though

**[54:37]** the promise is: if you didn't have memory bottlenecks, you'd just be going through and doing all the computations of your model. Okay, final thing on arithmetic intensity — roofline plots. There's a nice way to — yeah?

*[Question from the floor]*

Generally speaking, these models tend to be memory-bound for most of the computation, but yet the accelerators — 50 percent is not as good, and then maybe you get 70, maybe 80, that's really good — meaning the accelerator is outsized compared to memory bandwidth. Why is it like that? Why do these accelerators

**[55:22]** need to be big, just big? *[Ed: unclear — captions read "big just big"; likely a dropped word]* Well, they're just idling or waiting for memory.

So, the question is: could you design accelerators that had better characteristics? Yeah, maybe when we talk about GPUs, and understand a bit more how they work, we can talk about why this is. But if you have an answer, you should tell Jensen, and maybe he can design better hardware.

*[Interjection from the floor, inaudible]*

Okay, so let's visualize the relationship between arithmetic intensity and performance. This plot basically plots the arithmetic intensity on the X-axis — every vertical slice here corresponds to a particular, let's say, algorithm. And then we have these lines here, and

**[56:09]** each of these lines corresponds to, let's say, a particular accelerator — maybe H100s or B200s or so. And what this shows — on the Y-axis is the flops per second that are realized. So, if your algorithm has low arithmetic intensity, like ReLU or dot products, you're going to be over here, which means the flops per second realized is going to be not as high as your peak accelerator. And as you increase the arithmetic intensity, things are going to be better — you're going to be able to saturate your hardware, up until a certain point.

**[56:55]** And after a certain point, you're compute-bound, and obviously you can't exceed the peak flops. All right, so let's go on. Now I'm going to go back and talk about memory and compute for the operations that we need in training. So far we've done tensor operations, basically matmuls, and we saw how much memory it took and how much compute it took, and the interaction between them. So, now let's actually think about what it takes to train. So, here is our running example — actually, it's

**[57:42]** not a linear network, just a deep network. I'm going to consider a case where I have an input, which is a B-by-D input, and a number of layers, where each layer is a D-by-D matmul. That produces some set of pre-activations, and then element-wise ReLU, which produces some activations of the first layer, and then this is repeated again and again. And the output is just going to be the same size. So, the deep network — just to see what it looks like in PyTorch — is basically a set of blocks. Each block

**[58:27]** has a weight matrix associated with it. So, the number of parameters here is D squared times the number of layers, L. And when you run the model on a batch of data, what we're doing is going through the layers, and each layer we basically apply the linear transformation, and then a point-wise ReLU activation, and we do that for all the layers. So, this is hopefully straightforward, a very simple model.

*[Interjection from the floor, inaudible]*

Okay, so now let's talk about

**[59:13]** gradients. Let's use an even simpler example — a simple linear model regression. We have a vector, 1, 2, 3, a weight vector, 1, 1, 1, and we take a dot product and form this MSE loss. And what happens when we take the backward pass is that each of the variables involved in this computation graph has a gradient that is either set or not set. So, `w.grad` gets set to 1, 2, 3.

**[1:00:00]** So, this is just basic mechanics of PyTorch, I think it should be familiar to all of you. So now, the question is: how much compute do gradients take? Let's count flops for computing gradients.

*[Interjection from the floor, inaudible]*

Let's take a simplified model, where you have an X which is B by D, times a W1 matrix — well, we're going to ignore the ReLU for now, just for simplicity — times W2, which is the same D-by-D shape matrix. I'm going to use einsum here, for reasons that will become clear.

**[1:00:47]** So, the first thing you do in this linear network is take X and W1 — X is batch by input dimension, W1 is in by out, and you get a batch-by-out. And H2 takes H1 and W2, and it's the same kind of story, these are just matmuls. And then you form a loss — I'm just setting it to something arbitrary, just to get a number out. So, what happens in the backward pass? I'm going to call `retain_grad()` on these, for debugging purposes, which you'll see later. You take the backward pass on the loss, and the question is: how much work did that take, how many floating-point operations was that? Let's zoom in on one layer.

**[1:01:33]** Let's focus on the second layer here — this layer. The second layer takes H1 and just multiplies by a matrix W2 to get H2, and that's it. So, if you look at the flops in the forward pass, this is just a matmul, and we just take the three dimensions and multiply them together — BF16, that's two bytes, sorry, that's irrelevant here, this is just two because it's an addition and a multiplication. So, in the backward pass, what does this look like? If you remember your chain rule and your backprop algorithm, you have to compute two things. You have

**[1:02:19]** to compute the backward message — the gradient of the loss with respect to your input — and you also have to compute the gradient with respect to your parameters. And what do those gradients look like? This is just, by the definition of these — I guess it's just the chain rule here, written in einsum notation — you take d loss / d H2, which is `h2.grad`, that's a batch-by-out matrix, and you multiply it by W2, which is in-by-out.

**[1:03:04]** And that gives you `h1.grad`, which is d loss / d H1, and which is batch-in. I always get confused whenever you see this — if you've learned calculus, there's, like, one of them has a transpose, and I always forget which order to put them in. And einsum, I think, makes it very clear, because you can remember, even just by looking at the scalar case, that it has to look like `h1.grad` is `h2.grad` times W2, and the only question is: how do you index things? And you index things by basically looking at the shape of these named dimensions. And in this case, it's just a matrix multiplication where you are summing

**[1:03:51]** over the output dimension. And when you do that, we can check that `h1.grad` was the thing computed by `loss.backward()`, and this is indeed the same thing that I wrote out here, just as a sanity check. So, the other thing you have to compute is W2's gradient with respect to the parameters, and that's d loss / d H2 times H1. So, it's the same kind of backward message coming back, but multiplied with the other thing that you're not taking the gradient with respect to. And you just write out the dimensions, and in this case, you're summing over the batch dimension.

**[1:04:37]** So, the nice thing is that if you just look at this, we know how expensive this is. The number of flops is essentially the product of all the dimensions — it doesn't matter which ones you're batching or not. You basically enumerate over the i, j, k, and the way you aggregate is different, but the flops is the same. So, notice that the backward pass is exactly twice as expensive as the forward pass, and this is because you have to compute two gradients — one with respect to the parameters, for each parameter, and then one with respect to the input, the

**[1:05:23]** other thing, that's not the parameter. Maybe I'll pause for any questions about that.

*[Interjection from the floor, inaudible]*

Okay, so, that was just for W2. You just need to apply this to all the parameters in the network, and if you put it all together, you'll see that for this network, the forward pass is two times the number of data points — which is B — times the number of parameters, flops. And the backward pass is twice that, which is four times the number of data points times the number of parameters, flops. And so, the grand total is 2 + 4 is 6,

**[1:06:09]** 6, times the number of data points and parameters. So, this is where the 6ND comes from, that you might have seen in various places — it's just counting forward and backward. So, we did this for just these deep networks, but it turns out that this is actually a good approximation for transformers as well, as long as the context length isn't too large. If the context length is too large, then you get the context length squared, and that's more flops that isn't in this kind of accounting. Okay, so let's talk a bit about — so, now we have gradients, now we have the other piece. When you

**[1:06:54]** do training, we have to do optimization. So, here's our deep network — I'm going to, so that I'm not just giving you what's in assignment 1, use the AdaGrad optimizer, which is from — this is from 2011, this predates Adam. You can think of it as somewhere in between SGD and Adam. Basically, it's SGD where you look at the second moment of the gradient. Momentum is where you look at the first moment of the gradient, and Adam is where you combine the two of them. I'm not going to have time to go into the optimizer details here, since we're focusing mostly on the

**[1:07:41]** usage. So, we can define an optimizer, and when you compute the gradients — you compute gradients, and then you take an optimizer step. If you're defining your own optimizer — so, in assignment 1, you're going to implement Adam — for each of the parameter groups, which, in this case, are W1 or W2, there's something called the optimizer state, which is storage that the optimizer uses while it's running. And what we're computing in AdaGrad

**[1:08:29]** is the squared gradients — the sum of the squares of the gradients. So, here we're getting it from the optimizer state, updating it with the current gradient, and storing it back. And then, after you update `g2`, you update the parameters.

*[Interjection from the floor, inaudible]*

Okay, so in AdaGrad, you basically divide by the square root of — average gradient squared, or sum of gradient squared, rather. Okay, let's see, I'm actually going to maybe skip the training loop. Actually,

**[1:09:14]** wait, hold on — I think I actually skipped something I didn't mean to. Okay, so that was the optimizer state. Now let's look at how much memory the optimizer is using, or, in general, what the memory usage is. So, for parameters in this network, there's D squared parameters for each of the L layers, and each parameter takes two bytes if we're storing a BF16, so that's the number of parameters, times two. Activations — this is two times, which is for BF16, batch times D times the number of layers — for every layer, you have an activation. You have gradients, which is basically a copy of all the

**[1:10:01]** parameters, and this is all BF16. And the optimizer state is for every — actually, this should be — I think this should be, sorry, not this, I'll fix this, it's a typo — this should be the number of parameters, not parameter memory. And this should be number of parameters. So, number of parameters times two, and then this is four times the number of parameters. And the reason for this is that it's customary to use FP32 for the optimizer states, for stability reasons. Obviously, people have tried using BF16, and what ends up happening is that, now you're taking squares and averaging over all those steps, and it's not very stable. And for AdaGrad, here, we have four bytes

**[1:10:48]** per parameter for storing the optimizer state. Adam — you store the first-order moment and the second-order moment, so that's eight bytes. So, if you think about it, the optimizer state is actually a lot of the memory used. One note is that memory serves kind of two purposes: one is how much you have to store this thing in your HBM, but the other thing is that it has to be shipped to the accelerators. And, in general, the optimizer state is not really the bottleneck for compute, so the amount of memory here is not really so important for performance, in terms of speed, but it is

**[1:11:36]** means that you can't fit large models in your memory.

*[Interjection from the floor, inaudible]*

Okay, so, to put it together, as we mentioned, the number of parameters here is D times D times L, and the number of flops is 6 times the number of tokens, or number of data points, times the number of parameters. So, for transformers, this is going to be a bit more complicated, but you're going to do that in assignment 1, and do it more carefully. Okay, I'm going to skip the training loop, this is just a sort of general review. Two things I want to quickly touch on before we conclude: one is that, as we see, memory has an important effect, both on the ability to store large models, but also

**[1:12:21]** sometimes your speed. So, in general, you want to reduce your memory usage. There are two things that people typically do — one is gradient accumulation. In general, you want to use batch sizes that are large enough to improve stability, up to a critical batch size, which Tatsu will talk about later. And, as we saw, activation memory scales with batch size, so you might run out of memory at some point if you have too large a batch size. So, gradient accumulation says: well, you compute — you have these micro-batches, and you compute the gradient on the micro-batches, and then you just accumulate the gradients. You don't zero out the gradients, and every batch-size-over-micro-batch-size

**[1:13:08]** steps, you update the parameters and zero out the gradients. So, this is actually a very simple code change, which allows you to save on memory. The other thing I'll quickly mention is activation checkpointing. In training, we need to store, in general, the activations of all the layers — this is done by default. Actually, interestingly, for inference we don't need to compute the gradients, so we only need to store the current layer's activations. But for training, the memory usage is B times D times two times L. And

**[1:13:53]** the question is: can you reduce this? So, activation checkpointing — also known as gradient checkpointing, or rematerialization — the key idea is that, in the forward pass, you just keep the activations for only a subset of the layers, and in the backward pass, you recompute the missing activations from the last checkpoint — that's why it's called checkpointing. This is a general trick you see in systems, which is trading off: if you want to reduce memory, you can just recompute things. So, here we're going to define this, to operationalize it — this is actually fairly easy. Okay, so basically, what you do here is you have

**[1:14:41]** the same model, except you just add `torch.utils.checkpoint` on a layer. So, that means: do this computation, but don't store any of the intermediate activations, only store what's needed. So, this means that — if you store all the activations, remember, in your deep network you have the thing that's pre-ReLU, and the thing that's after the ReLU, so that's a lot of storage — and instead, if you do activation checkpointing on those blocks, where each block has a linear and a ReLU, you don't store the pre-ReLU, and you save basically half — you can get away with half the memory. And then, when you're doing the backward pass,

**[1:15:28]** you need `g3`, but you can compute `g3` easily from `h2`. Okay, you can go farther than this — you can say, well, I can store all the layers, but, in the extreme case, I can just store no layers, and that will be maximally memory-efficient. The only thing is the compute is going to be L-squared, because for every one of these layers you have to start from the beginning. And maybe a sweet spot is if you store the checkpoints every square-root-of-L layers — that means your activation memory is square root of L, and your recomputation overhead is also square root of L *[Ed: the lecture source states this case as O(sqrt(L)) activation memory and O(L) recomputation]*. So, that's balanced.

**[1:16:15]** Okay, so, to summarize this lecture: everything operates on tensors — parameters, gradients, activations, optimizer states, data. We introduced einops, which hopefully you can embrace as a way to think about tensor operations. 6 times the number of data points times the number of parameters is a formula which we've now demystified as the number of flops per — for a training step, and, actually, if this is a training step, this should be batch size. And then we talked about arithmetic intensity and roofline analysis, which allows us to diagnose whether a computation is memory-bound or compute-bound. Matrix multiplications are compute-bound, basically everything else is memory-bound.

**[1:17:01]** And then, finally, gradient accumulation and activation checkpointing are ways to reduce the memory, and by reducing the memory, that allows you to use bigger batch sizes. Okay, so that's it for today's lecture. Next week, Tatsu will talk about architectures.
