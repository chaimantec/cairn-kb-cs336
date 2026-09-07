---
title: Multimodality
lecture: 17
video: https://www.youtube.com/watch?v=26FtD08ZpOU
source: copy-edited from the auto-generated YouTube captions
---

# Lecture 17 — Multimodality (transcript)

This is a copy-edited version of the auto-generated YouTube captions for this
lecture: punctuation, capitalization and sentence boundaries have been added,
filler ("um," "uh," filler "you know," filler "sort of") and false starts have
been removed, and student questions have been marked. No substantive content has
been added, removed or reordered, and all 100 `**[MM:SS]**` timestamp markers
are preserved in their original positions.

**One class of exception, stated precisely.** Student questions are asked off
microphone, and the caption engine renders them as noise rather than as words —
at [10:58], for instance, it produced "they provide text every type of email
generation that we follow will have some sort of it." Where that happened, the
noise has been replaced by an explicit `*[Question from the floor, inaudible]*`
marker rather than preserved. **Nothing is lost by this**, because the lecturer
restates every such question before answering it, and the restatement is kept
verbatim. Two paragraphs — [10:58] and [59:28] — differ from the original in
length mainly for this reason, in opposite directions: one replaced garble with
a short marker, the other added a one-line gloss of what the restatement shows
the question to have been. The verbatim caption file this was
edited from is not kept in this repository; the terminology restorations below
were cross-checked against `raw/slides/17-multimodality.md`, the transcription
of the lecture's own source program (`lecture_17.py`), which is authoritative
for model names, dataset names, numbers and citations.

**Terminology restorations, confirmed against the slide file** (the term
appears in `raw/slides/17-multimodality.md`): CLIP; SigLIP (from "cichlip" /
"cig" / "sig"); SigLIP-2 (from "cichlip 2"); LLaVA (from "Lava" / "LA");
LLaVA-OneVision (from "Lava 1 vision"); LLaVA 1.5; LLaVA-Next; Qwen-VL (from
"QuenVL"); Qwen2-VL (from "Quinn 2" / "Quen 2"); Qwen3-VL (from "Quen 3VL" /
"quant three"); Qwen-2 (from "Quen 2," LLaVA-OneVision's text decoder); Qwen-3
(from "quen 3 models"); ViT / ViT-L/14 (from "vit" / "vit l14"); ResNet /
ResNets (from "resnats"); VQ-VAE / Vector Quantized Variational Autoencoder
(from "vector quantized variation encoder"); MRoPE / Multimodal Rotary
Position Embedding (from "M rope" / "multimodal rope"); RoPE; DeepStack (from
"deep stack"); AnyRes (from "any res" and, separately, "the NRA idea");
Vicuna (from "Vunia"); LLaMA; ShareGPT (from "shared GPT"); WebLI (from "web
li"); MS COCO (from "MS Coco" / "MSCO"); Mechanical Turk; LAION-5B (from "lion
5B"); OpenCLIP (from "open clip"); ImageNet (from "imageet"); GPT-4 (from
"GPD4" / "GP4"); GPT-2 (from "GP2"); TPUv3, TPUv4 (from "TPU v3s" / "v4s");
BPE (from "BP tokenizer"); QK norm; z-loss (from "Z loss regularization");
attention pooling; codebook (from "code book"); omni model (from
"omnimodel"); and the special tokens `<img>`, `<box>`, `<ref>`.

**Restored from context only** (not printed anywhere in the slide file — an
aside the lecturer spoke but never wrote, or a name outside the source
program's scope): SimCLR (from "sim clear," the data-augmentation contrastive
method he contrasts CLIP with); GPT-3 (from "GPD3," in the historical-context
aside — the slide file never mentions GPT-3); ChatGPT (from "chat GPT").

*Two names were originally listed here and have been moved up to the confirmed
group:* **GPT-5** (from "GPD5") and **Claude Opus 4.1** (from "Opus 4.1"). Both
are printed in the slide file after all — as column headers of the Qwen3-VL
benchmark table on `qwen3-vl-results.png`, whose description was written and
merged into the slide file after this transcript was edited. The classification
was right when it was made and is now a confirmation rather than a
context-only restoration.

**Left unclear:** two spots, both marked inline with `*[Ed: unclear]*` —
one where the captions trail off after "you can do OCR, you can compare" (an
inaudible closing remark before the Qwen-VL capabilities section), and one
where "the AI2 has a mobile paper" could not be confidently resolved (most
likely AI2's Molmo paper, but this is a guess, not a restoration, so it is
flagged rather than silently substituted).

**[0:05]** Originally the plan was to talk more about reinforcement learning, but since I only had one lecture to give, I thought this class would be a little bit incomplete without saying something about multimodality, which is so pervasive if you look at all the major models. So this will be an overview of multimodal models — of course, this could be a whole class in itself, and I'll try to do what I can in one lecture. Okay, so let's dive into multimodal models. So far in this class we've exclusively focused on language models. And language models are already pretty general — they give you the ability to go from any piece of text to any text. So this could be any language, code, you know,

**[0:51]** poetry, anything you like, even, let's say, DNA, for example. But the world is multimodal — you have text, you have images, audio, video — and if you think about a north star of where we want to be, it's what people call an omni model: the ability to take any combination of these modalities as input and output any combination of these modalities. So, for example, I can give you an image and a video and ask a question about the image and the video. I can generate images. I can convert the audio into an image. You can imagine all sorts of different applications by having an omni model.

**[1:39]** In this class I'm not going to build up to a whole omni model, but I'll give you some of the pieces that at least allow you to ingest a little bit of images, which we'll mostly focus on today. So how do you build such a thing? This is sort of a weird way to motivate it, but the reality is that Transformers work really well, and despite the best efforts of people to try other things, across all these modalities they are still, at scale, the best thing we have. So we have to figure out how to use them. Now, Transformers were designed for text, so they have this property that they speak tokens — they take as input

**[2:26]** a bunch of tokens and then output some tokens. Here I'm expanding the notion of token to be not just a discrete token, as we've seen in text, but also potentially continuous tokens — you can think of these as embeddings of tokens. The key thing is that a token should represent some sort of semantic unit of information. For example, in natural language, tokens are subwords, and these are somewhat meaningful, whereas a pixel is certainly not meaningful by itself. So somehow we must convert everything, including audio and images, into either discrete or

**[3:12]** continuous tokens. Now notice that we've already had to do this even with text — in the first lecture we talked about tokenization, and there it wasn't too bad. I mean, we had a BPE tokenizer, you guys implemented it, it's fine. One could wish for a better tokenizer, or no tokenizer at all, something that doesn't have tokenizers, but it's not the worst thing in the world. Now, for non-text modalities we have to scratch our heads a lot more and figure out what is the equivalent of the BPE tokenizer that will take an image and produce things that a Transformer can digest. Okay, so

**[3:59]** you have to answer two questions here. One is: for non-text data, for example images or videos or audio, how do you input these into the Transformer? And then, how do you generate them out? This lecture will mostly focus on question number one. So let's start — rewind the clock back to 2021. The CLIP model, which some of you might be familiar with, is a lot of the foundation of modern VLMs, so we can go through this in more detail. CLIP stands for Contrastive Language-Image Pre-training, and a bit of historical context is

**[4:46]** around that time: GPT-3 had already happened, GPT-2 — all the language models had already gone into this kind of foundation-model era. Vision was still — there were some efforts, but it had traditionally been based on large annotated datasets such as ImageNet, and training models such as ResNet on these datasets, and doing a bunch of other things like data augmentation, and getting really good performance. And so the researchers at OpenAI were wondering: is it possible to leverage the large amount of images and textual captions out there? So, inspired by

**[5:33]** this idea in language, where you could just scrape the internet — you have all this text, it's very noisy, but somehow if you train a large enough model you can make sense out of this and generate something useful. So what was the equivalent of that for images? As a result, CLIP was born, and the idea of CLIP is fairly simple and straightforward. Suppose I give you a bunch of image-text pairs — let's say I give you 32,000 of these. What I'm going to do is, for each image, I'm going to encode it using an image encoder, which I'll explain later, into a vector. So

**[6:20]** I'm going to have 32,000 of these image encodings. I'm going to also do the same thing with the corresponding text, so that'll give me t1 through tn — these are the embeddings of the text, these are the embeddings of the images. Now the objective I want to set up is: I want — let's take I1. I1 is associated with, aligned with, t1. I want this alignment, the dot product between these two embeddings, to be much larger than the dot product between I1 and all the other texts. And conversely, I also want

**[7:07]** for this text to say that this dot product is larger than the dot product between T1 and all the other images. So, in a nutshell, that's the CLIP objective, and you can imagine there's basically two times N different softmax classification problems. So here's the code: you get an m-by-d matrix for the image encodings and m-by-d for the textual embeddings. You normalize, and you take the dot product, with some temperature, and then you compute the cross entropy.

**[7:54]** So it's basically like a multiclass classification problem where the examples are structured in this n-by-n matrix form. Any questions about that? All right. So, some more details about how CLIP was built. Where does the data come from? There aren't too many details in the paper about the data. Roughly, you take a bunch of queries from somewhere and then you go search

**[8:41]** online for these, and you mine a bunch of image-text pairs, and this resulted in 400 million image-text pairs. The dataset wasn't released, but there was an effort by a paper called OpenCLIP, which replicated and further extended the CLIP ideas — there was this dataset called LAION-5B, which was released, five billion images with textual descriptions, and there was a bunch of processing that happened. Interestingly, they actually used CLIP for the data filtering and then trained OpenCLIP. So there's some bootstrapping happening, but at least

**[9:27]** this is a model where you can point to the dataset it was trained on, and the code as well. So, a few other details. Images come in all sorts of resolutions, right? If you find an internet image, it might be long and skinny, or tall — sorry, long and skinny or the other way — so, arbitrary dimensions. And one thing you learn about neural nets is that they don't like things to be dynamic; they want things to be fixed size. So there is a somewhat heuristic processing that happens:

**[10:12]** they resize the image so that the shorter side is 336 — or whatever the target size could be, 224 as well — and then they center-crop to get a square. This is obviously for convenience. Later we'll see that you can do much better than this, but this is just for expediency, and also, at this time, the CLIP authors were thinking about ImageNet and classification. So usually the object is in the middle, and you're just trimming off some background, so it doesn't matter too much. So, what is the vision encoder? This is the general

**[10:58]** algorithm, but I haven't specified what the image and text encoders are.

*[Question from the floor, inaudible]*

So the question, I think, is: why do they train on image-text pairs, as opposed to just images? Right — so there is a line of work that tries to encode images into representations based on data augmentation. So, for

**[11:43]** example, if you take an image — I think this idea is called SimCLR — you augment it, you crop it, you randomly perturb it, rotate it, and you want to say that that is the same image, i.e. they should have similar embeddings. Now, that's useful for low-level details, but the problem is that you won't data-augment your way from one type of dog to another dog. So by using text, it gives you higher-level semantic representations of the images. Yeah, that's a good question, and later we'll see how these semantic representations are useful if you look

**[12:30]** at the types of tasks that people are trying to solve. Okay, so what is the vision encoder that CLIP uses? They actually experimented with a bunch of different ones — ResNets and Vision Transformers, which had just come out — and they found that the Vision Transformers performed best. So when people say CLIP, they usually mean the ViT version. A Vision Transformer does the following: you take an image and break it up into patches — I think the original ViT paper used 16-by-16, and CLIP used 14-by-14.

**[13:18]** But anyway, you have these little patches, which are basically vectors. So each patch is, in some sense, a token for this Vision Transformer. You add the positional embeddings, just like you would if you were training a language model, and then you just put it through a standard Transformer. And if you want to classify, then — okay, so the Transformer encoder produces a sequence of vectors, and the CLIP paper does something a

**[14:07]** little bit different: you can average all these vectors, which gives you a single vector, but they found it was better to do attention pooling, which means you take the global average of all the activations, but then you also do another round of attention, with that as the query against the keys and values of each position, and that gives you another vector, which is maybe a little bit more informed than just a straight-up average. Okay, so that's the ViT, and the CLIP paper found that the best model is what's called ViT-L/14. To decipher this: this is a Large ViT — I think it has 24

**[14:56]** layers or so, not quite sure about that — and 14-by-14 patches. Each patch is RGB, so it has three channels, and they trained on 336-by-336-resolution images. Actually, they trained on lower resolution and then went up — I guess for speed, because high-resolution images take longer — and for the latter part of training they train on higher resolution.

*[Question from the floor: For the position embedding, would it be more complicated than text? For example, text only has one dimension, but images have two.]*

Yeah, so the question is, can you be

**[15:45]** smarter about the position embeddings, because this is just, you know, linear, zero through nine. I think in the CLIP paper they actually tried some 2D version of this, and the 1D one — they found it doesn't really matter that much. I think you also always have to take these results with the idea that they had classification in mind. So for classification, maybe it doesn't matter. Later we'll see fancier positional embeddings that do take into account the spatial structure. Okay, so the text encoder is a standard Transformer. It's a GPT-2-style

**[16:31]** Transformer, since that was from the same group that developed GPT-2. And in order to get a single vector out of a sequence, they prepended [BOS] and appended [EOS], and then they took the [EOS] activation at the highest layer as the representation of that whole sequence. Okay, so you have the vision encoder, you have the text encoder, and then you can do this process to train: you pick a batch, you encode all the text, you encode all the images, and then you form these two-N cross-entropy losses, and then you go from there.

**[17:18]** Okay, so the headline result, which got a lot of people excited about CLIP back in 2021, is that on the ImageNet benchmark, zero-shot CLIP outperformed a ResNet that was trained on 1.2 million ImageNet images. So if you think about this — 1.2 million ImageNet images — this was many, many hours of Amazon Mechanical Turk worker time to do this annotation. And now you have CLIP, which was trained on more organic web data. Of course, this was the labor of a bunch of people on the web, but if you're able to leverage that existing data already, then you can just do it zero-shot. And the zero-shot technique is

**[18:06]** basically — to explain, I don't have slide support here, but you take an image, and then you have a bunch of text, and then you basically take the dot product with the various labels and see which one's the highest. Oh yeah, it is — sorry, it is on this part of the slide. Okay, good point. So this is a zero-shot prediction, where you take an image and then — yeah, what I said. Cool. Any questions?

*[Question from the floor: this type of self-supervised version — is it actually an issue if, for example, there's a dog image, and, for all

**[18:52]** the caption — but maybe not just this simple, this dog; maybe other captions also have a dog — would that confuse the model?]*

Yeah, so the question is: if you have a dog, and there are maybe other captions that also have dogs, would that confuse the model? In general, this process is going to be noisy. It's okay if there's another dog, because on average it's unlikely that there's always going to be a dog — sometimes it's going to be an apple or a cat or something. But also, another point is that these images are basically scraped from the web, with text next to them, or in the alt text of the image. And so they're extremely noisy.

**[19:38]** There have been studies that look at, when you have a caption of an image, it doesn't necessarily just verbatim say what's in the image, right? Because if you have an image of a dog, you don't need to say "a dog." So this process is very noisy. So it's maybe somewhat surprising, but also interesting, that it worked. I mean, there was a lot of data filtering that was needed — if you take arbitrary images and text off the web, it's probably going to be way too noisy. One other thing to note, which relates to a point I'll come back to at the very end, is they also tried an alternative where, instead of doing this ranking, they predict text from the images.

**[20:26]** So they can set an objective where they try to take the images and predict the text, either as a bag of words or as a language model, and they show that, somewhat surprisingly, if you use a stronger model, it actually does worse, or at least is less efficient, compared to using a bag-of-words CLIP. So I think this is saying — well, again, this is ImageNet accuracy, so it's classification — that actually modeling the exact token

**[21:12]** sequences of the caption isn't so important for getting a rough representation of the image. Okay, we'll come back to this point at the end. So what have we learned so far? We've encoded images using CLIP, and it captures the semantics of the image because it's paired with text, and text generally talks about the semantics. It's important to note that the design decisions here are based on image classification, so it's not very fine-grained. But yet it still, as we'll see, serves as a kind of robust starting point for everything

**[21:59]** we're going to do later. One technical downside of CLIP is that it requires large batch sizes, like 30,000. If you have a batch size of one, clearly it doesn't work; even 10 doesn't work. And furthermore, the softmax operation operates over the full batch, so it's not really very decomposable. Whereas if you think about normal language model training, a batch of examples — all the sequences kind of parallelize, and you just do an aggregation at the end. So this point will be addressed by another paper from Google that came out, called SigLIP. So think

**[22:46]** about this as basically an improved version of CLIP. It stands for Sigmoid Loss for Language-Image Pre-training. And here the main difference is that CLIP does multiclass classification — it says, I have this aligned text and image, and I'm going to say this is positive against all the other alternative images, or against all the alternative texts. SigLIP, in some sense, is a lot simpler: it basically says, for any given image-text pair, are they aligned or not? So if I reference this diagram up here, it's basically binary classification, where the diagonal entries are positive examples and the off-diagonal entries are negative examples.

**[23:33]** That's it. So here's the algorithm: you take the embeddings, you normalize, you take the dot product as usual, but now you're forming the labels, which are minus one on the off-diagonal and one on the diagonal, and then you do a log-sigmoid. So the objective is very simple.

*[Question from the floor, inaudible — something about the sampling structure, since most of the samples are not quite random]*

Yeah, so the question is: does this

**[24:19]** require any sophisticated sampling strategies? At least in the initial paper, they were just operating on literally the same type of matrix. You could imagine — you're probably thinking, in general, for these kinds of contrastive methods, the negative examples sometimes need to be balanced between positives and negatives, or the negatives need to be tight negatives, so that you're not biased. But at least in the initial version, this was fairly straightforward. So, to speak about the data — this was done at Google, this was a dataset they used [from] another paper, which

**[25:04]** was another image-language model around 2022, which got a WebLI dataset with on the order of a billion image-text pairs. They also did some extra work — for example, for images that had text in them, they did OCR, and that's another way to form text-image pairs — and did some filtering; this was multilingual. The main point of this paper was that it was much more efficient to train than CLIP. So CLIP was trained for 10 days on 256 TPUv3s, and SigLIP was 5 days on 32 TPUv4s, and you might think, oh, TPUv4s should actually be faster than TPUv3s in terms of

**[25:50]** FLOPS per second — they're actually not. They're better because you can put more of them in a pod, and the interconnect is better. But at this scale, they're actually not faster — actually, like 60% slower, or something. So the way that this ends up faster is — of course, I think CLIP probably did not try to optimize the code for maximum throughput, it was just kind of "let's get this thing to work." But the SigLIP paper showed that you can parallelize this as follows. So, basically — and if you recall your systems lecture — you can think

**[26:37]** about it as DDP, if you like, where each device stores a subset of the image-text pairs. However, there are interactions between the examples, unlike in language model training, where everything just factors. So, in the first pass, each device just computes all the losses on whatever image-text pairs it has locally, and then you send the text. So, basically, device one gets T5 through T8, and now it's able to

**[27:25]** compute the negatives here, and then it gets T9 through T12. So you kind of rotate around until you cover all the off-diagonal block entries. Another nice thing about SigLIP is that you effectively decouple batch size from the loss, whereas in CLIP the loss is tied to the batch size — if you change the batch size, it's a different loss function. So they were able to experiment with much smaller batch sizes, less than, say, 16K, and in that regime it's much better than CLIP, because the loss function, if you have too small a batch size, just degrades. Whereas for smaller

**[28:12]** batch sizes for SigLIP, you have more variance, but it's the same loss in expectation. They also show you can go up to much larger batch sizes, but that doesn't really help — we saw that critical batch sizes hit some limit, and 32K was essentially their critical batch size. Okay, so now we have CLIP and SigLIP. These are image encoders that take an image — in this version, just a fixed-size image, 336-by-336, let's say — and map it into a vector, which contains some semantics. So now let's start building VLMs,

**[29:01]** vision-language models. And I'm going to talk about two families of models — LLaVA and Qwen. The two are very similar in the broad template; there are some details that are different, which we'll go through. And the basic idea is that we're going to take these embeddings and then inject them into a language model. So this is going to be more of a flavor of mid-training or post-training, where we take an existing image encoder, we take an existing LLM, and then we kind of stitch it together, rather than training something from scratch. So the LLaVA paper came out in 2023, and this got people excited because

**[29:49]** around this time all the closed models, like — I think it was GPT-4 — were able to do visual reasoning, and they were able to show that they could do some visual reasoning as well. It wasn't as good as GPT-4, of course, but it was an open model, and people got to see what went on under the hood. So for VLMs, there are essentially a few pieces here. There's a vision encoder — they use CLIP here. For the text decoder, they use a language model called Vicuna, which is the first LLaMA model that was fine-tuned on some ShareGPT conversations. So these are conversations that people had with

**[30:35]** ChatGPT and shared them on this website — I don't think this is up anymore. So the data that they used was based on MS COCO. This was an annotated dataset where they had Mechanical Turk workers go and annotate a bunch of images with bounding boxes and captions describing what was in the image. So they basically synthesized a dataset for their model: they prompted GPT-4 with the captions or the detected objects, and asked GPT-4 to generate questions or conversations. For example, this is an image in the MS COCO dataset — this was an annotated

**[31:22]** caption done by a human, and also here is — also annotated by a human. And then you ask GPT-4, generate me a conversation, and you might get a question and an answer. You might ask GPT-4, generate me a detailed description — it generates something, I guess, similar to the caption. And then you can ask GPT-4 to generate complex reasoning, and it does something like that. Okay, so this is synthesized data, 158,000 examples, and then they trained a model on this. So the model — so they

**[32:09]** they use CLIP, in particular the highest-performing CLIP version, ViT-L/14, so 14-by-14 patches. And then — let's just look at this image — they take images, send them through this vision encoder, CLIP, and that gives you a vector, but this vector isn't really in the same space, so to speak, as the text. So then they multiply by a matrix W to get another vector, which is in the same space as the text embeddings. And for text, you basically just use your standard embeddings to get vectors.

**[32:54]** So what's basically happening is the text gets encoded into these vectors, and the image also gets encoded into these vectors, and then this whole sequence of vectors just goes through a standard Transformer and produces output. So we're, in some sense, converting these images into textual tokens, so we can leverage the pre-trained language model. To train this model, they do it in two stages. The first stage is that they freeze the vision encoder and the language model, and only train this matrix W. And this is what they

**[33:40]** call the alignment phase, which is essentially making sure that the image gets mapped into the same space, because if you just use a random W, these images are not going to be — I mean, certainly they're not embeddings representing any sort of natural-language token. So the goal of training W is that they come to look like natural-language token embeddings. And the second stage is that they still freeze the vision encoder, and then train W and the language model — so they're fine-tuning the language model on their data. So remember, their data is basically: here's an image,

**[34:26]** and here is a conversation, or description, or complex reasoning example. So this is all image-plus-text to text. So this is one example from their paper — it's kind of interesting. You have an input image and a conversation: the user says, "what's unusual about this image?" And their model is able to tell you that you don't usually iron on the back of a minivan. And they also make a point that their model is pretty good because, even if your user prompt isn't really prompting for the unusualness, it still talks about it — whereas, of course, GPT-4 is

**[35:13]** able to do this, but other models at the time were not able to. Okay, any questions about LLaVA? Okay, so that was 2023. And I'm going to skip forward to LLaVA-OneVision, which is 2024. There was a sequence of papers after the LLaVA paper came out — LLaVA 1.5, LLaVA-Next — and I'll capture all of these innovations, I think, in what I'll describe next. The main thing that they did — it's essentially the same recipe, but they were trying to be more ambitious in terms of the types of

**[36:00]** multimodal applications they could do. So they were able to handle multiple images and videos. Here's the same sort of diagram — now they can take an image, a single image, or multiple images, or a video, which is essentially a sequence of images corresponding to a sampling of the frames. And here the vision encoder they upgraded to SigLIP. The text decoder, they use Qwen-2 now, which was probably the best language model out at that time. And then, for the projector, or the adapter, which is the thing that takes the output of the

**[36:45]** vision encoder and turns it into the input of the language model, it's a two-layer MLP instead of a linear projection. So, relatively — it's like you have a system and you're just upgrading the parts, but it's the same rough system. So let's talk a little bit about data processing. One thing that they wanted to do is OCR, and the thing with OCR is that you need to preserve very fine-grained information — otherwise a "J" looks like an "I," and that's not good. So remember that CLIP resizes and crops to 336-by-336. And clearly, if you have a document

**[37:32]** and you crop to 336-by-336, you can't read it. So their solution is this idea called AnyRes, which was actually introduced in the LLaVA 1.5 paper, and the idea is fairly straightforward: you break up an image into multiple pieces, and the idea is that each piece is the resolution of whatever the vision encoder is going to take — 336-by-336, for example. So you encode all the pieces into a bunch of vectors, and then you concatenate the vectors. So you're basically noticing that your vision encoder can't handle high resolution. So, rather than downsampling,

**[38:20]** you're just going to crop and look at different parts of the image. And now, if you have really super-high-resolution images or videos, then you get too many tokens, so then they have to downsample. But the idea behind AnyRes is that it's adaptive, and this is nice because the Transformer is already adaptive — sentences can be any length, and the Transformer already handles that pretty well. So it turns out images can be any resolution — that's kind of piggybacking on the same dynamic ability. So here is what they actually do for images here. Let's say you have this image of a paper. They

**[39:07]** do a downsample, and have one path which is basically a downsampled encoding of the entire paper. But then they break it up into these chunks, where each chunk is the resolution that the vision encoder expects, and encode each of these separately. And then you concatenate all these patches, and if you have too many, then you reduce the number of patches by interpolation. Okay, so then they're able to handle images, multiple images, and video. So you'd think, well, all of these are technically reducible to images, but they put their thumb on the scale a little bit, because they want to make

**[39:55]** sure all the modalities are roughly comparable, because videos can be very long — they don't want their dataset to be dominated by a bunch of repetitive frames. So here's what they do: for a single image, they basically have the full image downsampled, plus a bunch of crops, up to n crops, up to nine. And if you're presented with multiple images, then they just use the base resolution. So if I have a single image, I get to look at it more carefully; if I have multiple images, I'm just going to look at it from afar. And for video, we're going to use even

**[40:42]** lower resolution, or fewer tokens, to represent each frame, because the idea is — well, videos can be long, they only use up to 32 frames — but you start running into context-length problems for video, and later we'll see how a big part of being able to handle multimodal is dealing with long context. So the data here that they use — I would say they state the philosophy as: they want to curate high-quality data, over quantity. Another way to interpret

**[41:28]** this is that it's very targeted data — if you look at many of these, it's very task-based, whether you're doing visual question answering, or answering questions about tables. So this is definitely post-training territory, where you're trying to get your model to do these tasks, and therefore you create a bunch of these tasks. And this work is also unabashedly distilling GPT-4 models, so that they can get the best performance, which is, I guess, not ideal, but this is what you do if you don't have an annotation budget.

**[42:14]** So they have single images, multiple images, and videos — there's a lot of different pieces. Some of these are very specific, like: you give two images and you're trying to spot the difference. But one thing — okay, I'll come back to this later. So, for training, it's roughly the same idea as before, where the first stage is focused on alignment — in particular, you only train the projector, which is the adapter. And then, in the second stage, you

**[43:00]** train — and the third stage, you train the full model. So before we had two stages, now we have three stages. I'm not sure there's any particular principled reason for this, except that the second stage is trying to put in high-quality data, but focusing more on knowledge, and the final stage is focusing on examples that look like your downstream tasks. So one thing that they found in this paper, which was kind of interesting, is that you get transfer between modalities. So, even though they only have single-image data for diagrams and charts,

**[43:46]** this actually generalizes to multiple images. So, at training time, it never saw an example where you have a table and a chart and you're asking questions about both. But at test time, you have two images, one for the table and one for the chart, and it's able to have some sort of conversation about it. At training time, you only have OCR data on single-image data, and you only have relational reasoning for multiple images, but it can generalize to cases which are useful for powering GUI agents, where you show it a bunch of screenshots — this is multiple images — and it can generate a description of the screenshots.

**[44:36]** Another example is that they have this idea of visual prompting, where you have a circle, which is drawn onto the image, that's meant to highlight the part of the image you're supposed to do something about. This type of data only exists for single images, and now it can generalize to videos, where the user says, "describe the player highlighted in the video," and this player is across multiple frames. So, when I first looked at this, I said, oh boy, you're basically targeting each of these tasks — it's kind of like supervised learning. But if you have enough tasks, these models seem to do some transfer, which is, I guess,

**[45:25]** reassuring. Okay, so again, this is a fairly standard VLM template: vision encoder plus the projector into the language model. There's a lot of work that goes into the data curation, and they lean pretty heavily into synthesized, task-specific data. The nice thing about the LLaVA series is that it's one of the few works that open-sources not just the model weights, but also the data, so you can really replicate and study this stuff. Okay, now let's talk about Qwen models. Qwen also started training multimodal models in 2023. So

**[46:12]** the first version was Qwen-VL, and hopefully I'll go through this quickly, because you'll hopefully see the pattern. The vision encoder they used OpenCLIP — remember, OpenCLIP was an open reproduction of a CLIP model, so it's basically a CLIP encoder. And for the adapter, they did one layer of cross-attention, incorporating 2D positional embeddings, and mapped to a fixed size of 256, which is a little — I guess this will be changed later, because this is definitely not very dynamic, but neither is the vision encoder at this point. I guess this

**[46:59]** got cut off, but the special tokens are — they have an `<img>` tag, and a `<box>` tag for bounding box, and a `<ref>` tag for description. This is what happens when your slides are in HTML. So, for training, they have three stages, just like with the LLaVA models. In the first stage — the idea is, I mean, they call it pre-training, but it's really — you're not pre-training from scratch — this is generally large-scale, low-quality data. They freeze the language model and train the vision encoder and adapter. So this is a bit different from the LLaVA

**[47:44]** models, in that they train the vision encoder, and at this point — you can see some of the datasets they're training on, 1.4 billion examples. Stage two, they move to higher-quality, task-specific data, and you can see some of the usual suspects, like VQA datasets, chart question answering, and so on, and then they train all the parameters here. And finally, stage three is instruction-tuning data — you freeze the visual encoder and train the adapter and the language model. Here are some examples of the type of things that they're able to do.

**[48:30]** So, of course, this is Qwen, so some of these examples are in Chinese. Their interest is also in making a model that subsumes a language model — a language model can do code, so they want the vision model to be good at code as well. And you have this example, if you can see it — it's an image of Spider-Man fighting the Hulk, and it can output bounding boxes for them as well. It doesn't actually output an image, it just outputs a bounding box. And then you can do OCR, you can compare, you know. So, *[Ed: unclear]*.

**[49:16]** Okay, so then Qwen2-VL was, again, a kind of upgrade — there are some new ideas here, which I'll go through. So, they use a larger vision encoder. They also started doing dynamic resolution. I think this was the main thing — remember, when we went from LLaVA to LLaVA-OneVision, they realized, oh, we need to handle images of different sizes, and if you try to do video, it's clear you need some sort of dynamic resolution. So the model is basically — let's say you have this picture here that might be mapped to 11,000 tokens. This tiny

**[50:01]** picture of an equation might only be mapped to eight tokens, and so on and so forth. So how do they do this? Well, basically, it's the same idea as the AnyRes idea from LLaVA, where each 224-by-224 patch is encoded with a ViT — and remember, the ViT they started with is the OpenCLIP ViT, but this gets fine-tuned — and then, to try to reduce the context size, they compress every 2-by-2 into one. So that, ultimately, every patch generates 66

**[50:49]** tokens. For video, they sample two frames a second, but max out at 16,000 tokens. Okay, so, one thing, coming back to this question about positional embeddings — they did do something different here with Qwen2-VL. They use this idea called Multimodal Rotary Position Embedding, or MRoPE. So remember, we saw RoPE before, and the idea behind RoPE is that you have an embedding such that the inner product between the vectors depends only on the distance — the distance is defined, in 1D, by just the number of tokens away from it. So now

**[51:37]** the multi-dimensional version is essentially the same thing, except now you have 3D — you have height, you have width, and you have time. So each position, each patch, is now a triple, defined by the coordinates. And then, to compute MRoPE, for every dimension you compute the RoPE, and then you concatenate. So it's a fairly straightforward idea, although later, with Qwen3-VL, we'll see how this is actually a bit sub-optimal. Okay, so they initialize the language model with — this is the Qwen2

**[52:23]** model — so, of course, they're initializing with Qwen2. And then they have three stages of training, very similar to what we did before. And then they were able to show a bunch of different capabilities — some video understanding, it can do math and code, function calling, and so on and so forth. Okay, so, quickly, I'll talk about Qwen3-VL. Again, there are a few changes here — I wouldn't say these are big structural changes, but they're probably changes that do impact the quality of the model. So this is from last year's Qwen3-VL report. And

**[53:12]** it's roughly the same diagram here — I'll point out, maybe I'll come back to this diagram, I'll go through some of the changes. So, one is that now they're using Qwen-3 models — they have a series of dense and MoE models, and the Qwen-3 models are really, really good, and it really helps the quality of the final model. One thing that they're invested in is long-context understanding — their context length can go up to 256K, which is really important if you're trying to do long video. Let's talk about the vision encoder: they use SigLIP-2, which is basically an improved version of SigLIP. It's actually an identical architecture, designed to be

**[53:57]** backward compatible. And then they improved their RoPE, and the way to explain this is: before, you had the three dimensions — time, width, and height — and basically you'd allocate the first block of dimensions to time, then the next block to width, and the next to height. And the problem, if you remember RoPE, is that each component represents a different frequency. So this would mean that maybe all the temporal dimensions are low-frequency and all the height dimensions are high-frequency. So they

**[54:44]** basically interleave them, once they realized that, and now all the axes are exposed to both low and high frequency. Another thing they did was explicit video timestamps. So, if you look closely at this — sorry, this was — wait, okay, this is Qwen3. Oops. So, if you look — this is probably a little too small — before, the timestamp was kind of implicit in the positional encodings, right, each frame in a video intrinsically gets a

**[55:31]** different — there's a notion of time just because it's in a different position. But here they made the time explicit — so, a token like "0 seconds" is now a token that represents the time. And they found this to be helpful, presumably because it's something you can directly refer to, like, what happened after two seconds. So that's one of the main changes. They also did this square-root-normalized per-token loss. So the idea is that some examples that involve video are very long, and some examples

**[56:17]** that involve a single image are very short, and the normal thing is that every token is treated the same, and this would mean that the video examples are going to dominate. So, the details aren't too clear, but I believe they normalized each example by the square root of the length, to downweight the impact of really long examples. The final thing worth noting, which is interesting, is that they did something interesting with the adapter. So, remember, the adapter is the thing that connects the vision encoder to the language model, and LLaVA was the simplest — it was just a linear projection. Then we got an MLP, then we got some cross-attention, and

**[57:04]** DeepStack, which is actually a paper from the DeepSeek team, but what Qwen used here is more sophisticated. They noticed that the vision encoder already computes a stack of vision embeddings, and they're basically going to add these directly into the residual stream of the language model. So this is a bit more of a deep fusion of the vision encoder into the language model, as opposed to the vision encoder being a black box that just outputs a sequence of

**[57:50]** vectors. Okay, so training — they have two phases. Pre-training actually has four stages, post-training has three stages. So you can see these pipelines are getting quite complicated now. They first train the adapter — this is the same as all these models — and then they have three stages where they progressively train on longer and longer sequences, from 8K to 32K to 256K. Most of the tokens are in stages two and three. And then, for post-training, they do SFT on long chain-of-thought data,

**[58:38]** knowledge distillation, and then some reinforcement learning. At this point, it's really kind of a systems paper — I highlighted the core new ideas, but there are a lot of details that are different. And if you look at the final results, this is a pretty good model — if you look at these benchmarks, as compared to the closed models, Gemini, GPT-5, and Opus 4.1, these numbers — the bold means it's the best number in that row. So the Qwen models are actually quite strong. Okay, maybe — yeah, question.

**[59:28]** So, we're going to have — yeah?

*[Question from the floor, inaudible — about video generation, and how the model knows whether the output should be video or text]*

So the question is: what about video generation, and how do you know whether the output should be a video or text? So, first of all, these models don't generate video or images — all the multimodal stuff is on the input side. You're always generating text, and so

**[1:00:15]** then, in most of these stages, except for RL, you're supervising every token. So there's no LM-as-a-judge, or quality signal on how well the description is working — it's just whatever is in your dataset. Now, with RL, you can play around with different rewards.

*[Question from the floor: I want to ask, is training a multimodal model harder than training a pure LM, from a systems perspective? For systems, I think the most bandwidth would be used for images, videos

**[1:01:02]** that [aspect too], and from the systems side, I think images and videos — they have more [data] — so that might be why.]*

Yeah, so the two questions are: are training multimodal models harder on the systems side? And, I mean, certainly it's not easier. I think you pointed out that the datasets are larger — video data, loading it can be a bottleneck. Generally, we have not really focused on data loading when talking about language models, because it's very cheap, so that needs to be taken into account. And, you know, all

**[1:01:48]** the same principles, of making sure that your data loading is happening async with actual computation, need to be taken into account. And the second part was — sorry, what was the second question?

*[Question from the floor: They have more tokens.]*

Oh, yeah — so videos and images can have more tokens, especially video. So that's one reason we saw this kind of normalization, to make sure that we're not giving more weight to video. I mean, you can always downweight — if you have more tokens, if you don't like that, you can downweight. We talked, last time, or two weeks ago, about data mixtures — you can always weight things according to whatever makes sense. But there's

**[1:02:36]** also a lot of text tokens out there. So, I think, in general, if you look at — yeah, the models are trained on tens of trillions of tokens. I wouldn't say that the number of multimodal tokens vastly outnumbers text tokens.

*[Question from the floor, inaudible]*

Oh, I think you were first.

*[Question from the floor, inaudible]*

**[1:03:22]** Yeah. So the question is: when you do this alignment, how does it know — is the language model pre-trained? And it definitely has to be pre-trained, because otherwise it doesn't make sense to align it. So the language model is frozen, and you're just training the adapter to connect the given vision encoder with the given language model. And the way you train it is, well, you pick a token budget — say, 67 billion tokens — and you just train. There's not an adaptive threshold here.

*[Question from the floor: Also, let me know about the difference in the parameter numbers — I noticed the size of the vision [encoder]... that... so

**[1:04:09]** ...]*

Yeah, so the question is: are the number of parameters in the vision coder much smaller? And the answer is, yes, in general. I'm trying to find — where is the other place — I — not here, okay. The reason is that the vision encoder is

**[1:04:54]** in some sense doing a very local operation. It's looking at a patch — a patch is very small, and it's just trying to understand the patch. There's not much knowledge there — so, per patch, we're not reasoning. So most of the capabilities of the model are still in the language model. I'm trying to find — okay, where — maybe it was up here. Okay, I can't — oh yeah. So if you look at the LLaVA-OneVision model — so the full, so it's a 72-billion-parameter language model, and — show you what — okay, I don't know why this number is

**[1:05:45]** — oh, this, sorry, this is 72 million. Okay, so this is much smaller — the projector is much smaller, and the ViT is generally less than a billion parameters. Yeah, I guess, mostly because it's fairly a local operation — I guess that's the easy way to say it. Okay, I'm going to take some other questions — I'm going to try to move on now. So, I guess Qwen3-VL is kind of the last vision-language model I'll talk about — this got state-of-the-art performance. There's a lot of data work that happens, not too many details about

**[1:06:30]** the data and the data mix in these later Qwen models. So you'll have to look at the LLaVA paper, or — AI2 has a *[Ed: unclear — possibly "Molmo"]* paper, which I didn't get a chance to talk about, that has more details here. And, from Qwen to Qwen-2 to Qwen-3, many of these changes were — there's some kind of change there, but the overall framework remains the same, and mostly it's scaling up, curating more datasets, noticing that you have to handle long context, and maybe sharpening your image

**[1:07:16]** processing, and so on. Okay, so I'm going to jump to something quite different — there's this paper called Chameleon, from Meta, in 2024. So far, VLMs encode images into these vectors and then inject them into a language model, and because it's a language model, you can only generate text — you can't generate images. There are many ways you can go about fixing this — you can just take a VLM, you can also attach a diffusion head, and now you can generate images. But I want to talk about this

**[1:08:02]** idea here, which is: Chameleon says, what if we mapped everything into discrete tokens? And, in some ways, aesthetically — well, maybe this reflects that I'm a language person — this is kind of appealing, because now you can analyze and generate images in the same way, since everything is a discrete token. If you want to — let's say you want to generate images, you can say, here's a text prompt, and then these are just tokens that you can generate; or you can flip the language and the text around, as

**[1:08:48]** well. They have these examples, and, from the Chameleon paper, you can prompt, saying, "I'm bored, can you show me some birds?" and then it would be text, and then there'd be images, and then some more text and some images — so it's interleaved. So, in some sense, the vision of an omni model is that text and images truly live in the same space, and this is accomplished by making everything look like text — that's one way to do it. Okay, so I'll talk a little bit about how this is done. So the thing you need to do now is to map images into discrete tokens. So there's this idea, rather old, from

**[1:09:36]** 2017, which is called VQ-VAE — Vector Quantized Variational Autoencoder — and the basic idea here is to map an image into a discrete code. What you learn is a mapping that maps this into a continuous vector, and this gets rounded to the nearest code. So you have a codebook, which has, let's say, 8,000 codes — these are like prototypical vectors that correspond to patches — and then you pick the one that's closest, and that would be your representation. So that's your encoder: it goes through a continuous phase, and then you

**[1:10:22]** round it to the nearest code. And now the decoder — you take that code, and you try to reconstruct the image, and then you train these VQ-VAEs by basically minimizing the reconstruction loss. There are some other terms that you add, because this is not differentiable, which I won't have time to get into. Okay, but at the end of the day, you have 512-by-512 images that are converted into 1,024 tokens, and each token comes from a vocabulary of 8,000. So now you basically have text, you have your images, which are

**[1:11:09]** basically these codes. They train a new tokenizer, now, because your data looks different than if it were just normal natural language. The training is now actually very straightforward — this is just normal language model training, right? There's no adapter, there's no — you're not turning the vision encoder — there's just a language model now. So this is the part that's kind of appealing. So there are still two stages, but even for language model training there's usually two or more stages, where the first stage, the bulk of training, is large unsupervised

**[1:11:55]** — they have both text and text-and-image — and then stage two, they mix in high-quality data. Okay, so this is really great and elegant. There are a few problems with this. One is that they found the training was not stable, and the reason is that text and images, despite occupying the same space, just behave very differently — so just calling things discrete tokens doesn't hide the fact that there's an image living there. And, in particular, text tokens — if you think about predicting the next word, this has relatively lower entropy. Most

**[1:12:41]** words are kind of predictable, whereas image tokens have very high entropy — I don't know what shade of blue this exact token is going to be. And so, as a result, they found that training with this kind of mixture led to the norms of the parameters growing, and to loss instability. They were able to mitigate, or fix, this to some extent via QK norm and z-loss regularization, which controls the norm growth. So, I'm not even going to show the results, but I wanted to highlight this work, because there's a certain elegance here, because it's just one

**[1:13:26]** model that treats all modalities the same. But the downside is that it turned out this model was not really as performant, and the discretization definitely loses information — think about OCR, again: if you discretize, very small print, you're not going to be able to read it anymore. And, finally, training with multiple modalities is also tricky — we saw this a little bit with the Qwen models, where we had to play around with the weights, but this is more exacerbated here. So VQ-VAEs were kind of popular for a while, and, in fact,

**[1:14:12]** many people were using this for image generation. The main reason to do that is, well, you have a Transformer — how do you generate from a Transformer? Well, you have to generate discrete things, so you basically put all your data into this discrete form. But then diffusion models came out and became popular and viable for generation. So this flavor of method is somewhat less popular than it used to be. Okay, so let me just reflect and summarize here. Frontier models these days are expected to be multimodal, or, even more strongly, natively multimodal, or omni models.

**[1:14:58]** Unfortunately, I think when Gemini comes out, or GPT comes out, they're touted as being natively multimodal and handling all these modalities, and, in fact, they do — but, of course, there are no details about how these are built. I think it's probably some combination of having a continuous encoder, because you don't want to lose information, and then diffusion for the generation — but that's my speculation. The fundamental challenge, I think, when you're dealing with multimodality, is: how do you handle non-text modalities? It's also interesting that there's a symmetry between understanding

**[1:15:43]** the modality and generation, and there's — there's no one universal encoder. Let's say, for example, when we looked at CLIP, you only cared about classification, to capture high-level semantics, so these vectors could be fairly small and capture the high-level semantics. Whereas, if you wanted to do OCR, or if you wanted to generate an image, then you need really fine-grained detail — that's why diffusion is so good, because it can really micro-optimize the low- and high-frequency information. I think, in general, when you're dealing with multimodality, you have to think carefully about weighing them properly,

**[1:16:30]** for example, video certainly has lower information density than text, so you don't want video to overwhelm your text. And, like I said before, perhaps at least the current best thing is having continuous encoders — it seems like even CLIP, even though it's five years old, or similar ideas, are still kind of the go-to way to capture the semantics of images. Transformers are still there. And then diffusion models, which I didn't talk about, are great for generation. Okay, so I will stop there. That's

**[1:17:17]** it for the lecture on multimodality. I guess we don't have any homework on doing this, but if you're curious, I would encourage you to play around with training some of these models.
