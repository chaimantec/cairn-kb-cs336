---
title: Lecture 1: Overview, Tokenization
lecture: 1
video: https://www.youtube.com/watch?v=JuoVZkPBiKk
source: copy-edited from the YouTube auto-captions
verbatim_original: original/01-overview-tokenization.md
material: ../slides/01-overview-tokenization.md
---

# Lecture 1: Overview, Tokenization — transcript

**This is the edited transcript.** The auto-captions have been repunctuated,
segmented into sentences, stripped of filler, and had mis-heard technical terms
restored. No content was added, removed or reordered, and every `[MM:SS]` marker
is preserved in its original position — content that was under a marker is still
under that marker. The verbatim captions are kept at
[`original/01-overview-tokenization.md`](original/01-overview-tokenization.md);
that file is the record of what was actually said, and this one is the readable
version of it.

Terminology was cross-checked against
[the lecture's source program](../slides/01-overview-tokenization.md)
(`lecture_01.py`), which is the authority for names and numbers written down.

**Restorations made.** These are places where the captions produced a *wrong word*
and the text now reads differently. Each was confirmed against
[the lecture source](../slides/01-overview-tokenization.md) unless the note says
otherwise.

| Caption | Restored | Confirmed by |
| --- | --- | --- |
| "CS 33336" | CS336 | course code |
| "seek-to-seek modeling" | sequence-to-sequence modeling | deck ("Sequence-to-sequence modeling") |
| "Joshua Bengio" | Yoshua Bengio | deck (Bengio et al. 2003) |
| "a grassroots organization called Luther" | EleutherAI | deck ("EleutherAI's open datasets") |
| "the Marine project" (×2) | the Marin project | deck ("Marin's models") |
| "the Kimmy K2 models" | the Kimi K2 models | deck ("Moonshot's Kimi models") |
| "this H net work" | this H-Net work | deck (H-Net, arXiv:2507.07955) |
| "genetic traces" | agentic traces | deck ("agentic traces with tool calling") |
| "score them either with a human or verify or LM judge" | …or a verifier, or an LM judge | deck ("{human, verifier, LM judge}") |
| "ecologically validity" | ecological validity | deck ("ecological validity matters") |
| "these two things get completed" | …get conflated | context |
| "web pages are called from the internet" | webpages crawled from the internet | deck ("webpages crawled from the Internet") |
| "archive papers" | arXiv papers | deck ("arXiv papers") |
| "these beta sets like tiny stories and open web text" | these datasets like TinyStories and OpenWebText | deck ("Train on TinyStories and OpenWebText") |
| "computer optimal scaling laws" (×2) | compute-optimal scaling laws | deck ("compute-optimal scaling laws") |
| "how you normalize the different layers to prefer blow up" | …to prevent blow-up | context |
| "reducing the projecting our low dimensional space" | projecting into a lower-dimensional space | context |
| "the shorter the sentence" | the shorter the sequence | deck ("the shorter the sequence"); the captions self-correct to "sequence" one clause later |
| "in the full truly supervised case" | in the fully supervised case | context |
| "that are occurred the most frequently" | that occur the most frequently | context |
| "these like unk token" | these, like, UNK tokens | standard term |
| "which you can call ORD in Python" | which you can get by calling `ord` in Python | deck (`ord`) |
| "connected via you know NV link" | connected via NVLink | spoken narration of the DGX B200 topology figure; the interconnect name is not written in the source |

**Capitalization and formatting only.** The following were already correct in the
captions and were only cased, hyphenated or code-formatted — they are not
restorations, and a reader comparing the two files should expect them: Unicode,
UTF-8, BPE, FLOPs, petaFLOPs, BF16, HBM, MoE, RL, Triton, Modal, CGOE, `AGENTS.md`,
DPO, GRPO, PPO, MinHash, Muon, Qwen, DeepSeek, Olmo, Mistral, Llama, ByteDance,
Tencent, Claude Code, nanoGPT, InfiniBand, Karpathy, Mamba, Gated DeltaNet,
Common Crawl, Chinchilla, Kaplan, NVIDIA, Hugging Face, BigScience,
*How to Scale Your Model*, Statistical Learning Theory, CS224N.

**Spoken-only names.** Several restored or capitalized names appear nowhere in the
lecture source, because Percy says them without writing them: NVLink and
InfiniBand (narrating the DGX B200 topology figure), nanoGPT (an aside about the
speedruns), Claude Code, ByteDance and Tencent (an aside about Chinese labs he
has not listed on the slide), and Statistical Learning Theory (a class he
mentions teaching). None is a claim about course content.

**Numeral conventions.** Two normalizations run through the text and account for
almost every difference a number-diff will report against the original: assignment
numbers are written as digits ("assignment one" → "assignment 1"), matching how the
course names them; and small counts of years are spelled out ("2 years ago" → "two
years ago"). One token count was made a digit to match the code being shown ("the
number of tokens is eight" → "is 8"). No quantity, budget, model size or measured
value was altered.

**Marked as unclear.** Where the captions leave genuine ambiguity, the text carries
an inline `*[Ed: …]*` note rather than a guess. See those notes in place; they are
the only places where the reading is uncertain.

**Speakers.** Lecture 1 is taught by Percy Liang, with Tatsunori Hashimoto and the
teaching assistants introducing themselves at the start. Speaker changes in the
opening round of introductions are marked; after that, the voice is Percy's unless
noted.

---

**[0:04]** Welcome everyone to CS336, Language Models from Scratch. This is the teaching staff. I'm Percy; this is Tatsu, Marcel, Herman, and Steven, and we're bringing you the third edition of 336. So we'll just do a quick round of introductions. I'd like to say that I've been doing language models for 20 years, but most of that time was on small language models — actually, this is still small in the grand scheme of things. I think when Tatsu and I started teaching this class two years ago, we weren't really sure what we were going to expect, but we were very pleasantly surprised that so many people wanted to learn how to build language models from scratch, especially in these days when a coding agent could probably zero-shot a language model. I'm

**[0:50]** really glad to see all of you here actually wanting to learn how they work.

*[Tatsu]* Yeah, thanks. I'm Tatsu, one of the co-instructors. I'll be talking to you once we get to architectures and scaling and all these other very fun things. I'm really excited. This is the most fun class I've taught in my time here. Every year Percy makes fun of me, because he says you have to redo all your lectures — because you took on architectures, and everything changes every time. But it's actually pretty fun for me to do it, and it's kind of the first time that I've had that experience. So I'm looking forward to going through this experience again with you all.

*[Marcel]* Hello, I'm Marcel. I've seen this class once before and I'm returning, because it was so much fun last time. It's a lot of work, though. In my research I do architecture stuff.

**[1:35]** I do higher-order gradients and I do training. Yeah, looking forward to working with you guys.

*[Herman]* Hello everyone, I'm Herman. A year ago I didn't really know how LLMs worked. I think for me, tokens were the things you collect in video games, and attention was the thing you had — like the attention economy. Then, after spending a lot of time on this course last year, I'm now doing LLM research and I'm really excited to TA this year.

*[Steven]* Hi everyone, I'm Steven. I am a first-time CA for this course and I'm very excited about it. I think it'll be a lot of fun. Broadly, in my research I work on language models, theory, and some data efficiency stuff, and I'm excited to meet you all.

*[Percy]* All right, so let's go into

**[2:21]** things. So this is the third time we're offering it. Last year we decided to put all our lectures on YouTube, so some of you will have seen it. So what's new? Well, what hasn't changed is the from-scratch philosophy. We still believe strongly that by building everything from the ground up, you really learn how everything works. Of course, we don't actually build everything up from scratch, because that wouldn't fit in a quarter. So over the last two years we've been refining our recipe and figuring out what are the things that you build up from scratch that are most high-value. And then finally, as Tatsu alluded to, even in a year a lot has changed. I think this

**[3:08]** year we're going to spend maybe a bit more time on mixture of experts, and of course agents are very popular these days, so getting a handle on long context and what is needed for that are going to be important.

So, why did we make this course? I think the problem, two years ago, was that researchers were becoming disconnected from the underlying technology. It used to be the case — maybe 10 years ago — that all AI researchers would just implement and train their own models. And then, even 8 years ago, people would download pre-trained models such as BERT and fine-tune them. And I think a lot of today,

**[3:54]** you can get by by simply prompting a model. And of course there's nothing wrong with prompting a model — I think you can do really amazing things with it. I think moving up the abstraction in general is a great thing, but abstractions are leaky, and — I'm sure all of you have prompted models — you run into situations where you wanted to do something but it just can't do it, and there's no recourse. And I would argue that if you're really interested in fundamental research, by simply prompting a model you're sort of vastly constraining the set of options, the design space you're looking at. And for fundamental research, you really need to tear up the whole stack. So I would argue that full

**[4:40]** understanding of how language models work is really necessary for fundamental research. And the way that we're going to get our understanding is by building. That's the philosophy of the class.

But there's one small problem, which is that industrialization of language models has happened. Frontier models are really, really expensive. Even — this is three years ago — GPT-4 was supposedly costing $100 million to train, and now costs are probably on the order of, you know, a billion, although that's speculative. And the number of GPUs that all the big labs are building is just kind of immense. Furthermore, there are no details on how many of these models are built. So even back in 2023, the GPT-4 paper explicitly says

**[5:27]** that due to the competitive landscape and safety implications, we're not going to share anything about how the models are being built.

So these frontier models are in some sense out of reach for us. Now, we could build small language models — and we will build small language models — but I think it's important to remember that these might not be representative of the actual frontier models. And I'll give you two examples why this might be the case. So here's one. Of course, this is from actually quite a long time ago — think back in 2021. We're going to spend more time on FLOP counting and looking at where the compute is being spent, but if you look at small scales, the fraction of FLOPs spent in the MLP layers is around 44%,

**[6:14]** and if you scale up to 175B, then it goes to 80%. So what you optimize and what matters at large scale is going to be different from small scale. So if you did a bunch of small-scale stuff on the attention, you might not experience the same benefits at large scale.

The second example is that we know of emergence of behavior with scale. So small models — this is again from a while ago, but even back then, if you have zero-shot or few-shot learning of various tasks, it basically seemed like nothing was working. And only when you reach a critical scale do you suddenly see a lot of improvement. So again, if

**[7:01]** you're working at small scale, you might not see certain types of phenomena compared to if you were working at full scale.

Okay, so that might be a little bit disheartening, but fear not — we're going to learn something in this class. And the question is: what can we learn that actually transfers? I think it's important to break this down into three types of knowledge. First, there's the mechanics of how things work: what a Transformer is, how model parallelism works. Then there's mindset, which is how do you go about approaching building a language model. We're going to talk about how you want to squeeze the most out of your hardware, taking scaling seriously. And finally, intuitions: which data and modeling decisions are going to yield

**[7:47]** good performance. So in this class, I think we can do a pretty good job of teaching the mechanics — which is how things work — and the mindset. And we're going to really emphasize that you profile and benchmark everything and try to optimize for efficiency. These things do transfer to a larger scale. Now, the intuitions about what modeling decisions and what data decisions do work might not necessarily transfer across scales. For that you actually have to go somewhere where you can do things at scale.

So, on the note of intuitions, it is worth remarking that some design decisions are just not justifiable,

**[8:33]** and just purely come from experimentation. Whereas with mechanics, you can kind of by construction see how the parallelism and how kernels are going to speed things up and so on. But for intuitions about what modeling changes work, I think you just have to run experiments. There's this famous Noam Shazeer paper that introduces the SwiGLU activation, which we're going to look at. And in the conclusion section, he very honestly has this final sentence which says: "We offer no explanation as to why these architectures seem to work; we attribute their success, as all else, to divine benevolence." So that in some sense is something that you just have to gain from experience.

A final note about the bitter lesson, which I think has been circulating and people talk about it. I think there's sort of a common

**[9:19]** misconception here about what it means. I think the wrong interpretation is that scale is all that matters, algorithms don't matter. But that's not correct. The right interpretation is that algorithms that scale are what matters. Okay? And you can think about it very simply: that the accuracy of your model is basically your efficiency times the resources. So efficiency is output over input, and resources is the input. And efficiency is actually, in some sense, way more important at larger scale. If you're doing a small-scale experiment, if your run takes twice as long, maybe you just wait

**[10:05]** twice as long and then you come back later. But if you're doing things at scale, that could be hundreds of millions of dollars, and you definitely don't want to do that. Even a 5% improvement might be a big deal. So in fact efficiency is actually really, really critical, and I hope to bake that into your mindset as one of the consequences of this class.

And empirically, if you look — there's this paper from OpenAI in 2020 that showed there's a 44x algorithmic efficiency on ImageNet between 2012 and 2019. And it's surely the case that hardware did get a lot better, but that also comes with algorithmic improvements, and of course when you multiply them

**[10:50]** together, that's when you see a huge bump in efficiency and accuracy.

So the framing, with all of that said, is: what is the best model one can build with a certain data and compute budget? For pre-training, mostly we're going to talk about compute budget, because we're going to assume that we have a lot more data than we have compute. But if you're in a setting where you're data-limited, or you have, you know, stashed away actually tons of B200s, then you might be data-bound. So in other words: maximize efficiency. And we're going to see this theme come up throughout the class.

**[11:36]** Okay. So next I want to spend a bit of time talking about language models and a bit of history, and just contextualization, before we jump into the more technical details. Language models have been around for a while. Shannon, back in the '50s, was using language models to measure the entropy of English. And for a long time, n-gram models were used actually in machine translation and speech recognition systems. They weren't the whole system, but they were an important part of making sure that you generated fluent text. I would say that the lineage of the modern language models comes from neural architectures. And so there are a bunch of

**[12:23]** ideas, I think, which are important to this development. So in the '90s there were LSTMs. Yoshua Bengio actually wrote the first neural language model paper back in 2003. This was actually not an LSTM — this was just a feedforward network that looked at a small context. And then there was sequence-to-sequence modeling, which boldly said we can actually compress a whole sentence into a vector. The Adam optimizer; the attention mechanism, which was developed for machine translation; the Transformer architecture, which built on top of that, which was also developed for machine translation. And then, scaling up: mixture of experts, model parallelism. You see a lot of

**[13:09]** different architecture and also systems and optimizer ideas developing in the 2010s.

And then by the late 2010s, I think things were starting to get really interesting. So there was ELMo and BERT. These are language models that were trained on lots of text, and then you could fine-tune them on some downstream task like question answering and it would show a huge improvement on that. So the model there was: take one of these models and then fine-tune. And then Google had a paper that was, I think, really foreshadowing this view of "prompt in, response out." *[Ed: T5, per the lecture source; Percy does not name it aloud here.]*

So it was really I think OpenAI that

**[13:54]** really opened up the floodgates here by embracing scaling. So they had a GPT paper back in, I think, 2018 or so, that they scaled up to GPT-2, and then they really figured out — or embraced — the idea of scaling laws, which we're going to talk about in a second, which enabled them to train GPT-3, which was a much more massive, almost more than 10x larger model at the time. And it could show emergent behavior like in-context learning.

At that point, Google was saying, "Okay, we need to do something too." So they trained a massive model. It turned out to be kind of under-trained,

**[14:40]** and it turned out that DeepMind, which was not integrated with Google at the time, had figured out compute-optimal scaling laws. So all this was happening.

And after GPT-3 came out, I think for many folks this was sort of a wake-up call. And at that time there were a lot of early attempts to try to replicate this. So there was a grassroots organization called EleutherAI that created some open datasets and models. They weren't very large, because they didn't have much compute. Meta's first LLM — you could tell that it's a replication, because it was like 175 billion parameters. But it was not a very good model; they ran into a lot of hardware issues.

**[15:26]** And then there was another Hugging Face / BigScience project. So these models were, I would say, not very strong.

Then in the last three years, I think the open model ecosystem has changed quite a bit, with Meta kind of leading the way with the Llama series of models — Llama, Llama 2, Llama 3. Mistral got in the game. And then a whole set of Chinese models — I think I'm missing some; like, ByteDance has something, and I think Tencent probably has some other stuff. So it's hard to keep track of everything, but everyone's heard of DeepSeek and Qwen. So I think I got the main ones. But I think what's interesting and exciting about these is that now we have open-weight models that

**[16:12]** are approaching closed models. So depending on who you're asking and how you benchmark, they might be a little bit behind, or comparable — but they are definitely very, very credible models that are being widely used in industry.

Now there's another line of work, which is going beyond just releasing open-weight models. AI2, NVIDIA, and the Marin project, which I work on — we try to provide not just the weights but the paper and the code and the data, so that we can understand how these models are built in a more thorough way. Why do I emphasize this open ecosystem so much? Mostly because this course would not be possible, I think, without these models.

**[17:00]** The fact that there are still many papers being published about how these big MoEs and RL systems are working enables us to at least glimpse into how these frontier models are being built, and try to triangulate the pieces. Now, of course, a lot of the details even with these — you know, the Qwen papers — I think they're missing a lot of details. You can't reproduce them. Notably the data mixture is something that we don't know. But I think it's much better than nothing.

Okay. So in the last decade, I think the idea of what a language model is has changed. Right? It used to be something that you fine-tune. And then it was something you prompt.

**[17:45]** And now, in the ChatGPT era, it was something you talk to, that you can have a conversation with. And now — okay, I guess I don't have internet. That's fine. And now we're in the era of agents. If you click on this link, it basically shows you a giant agent trace. And I'm still kind of — it's mind-boggling how strong some of these models are. You give it like a page of text and it does some really complicated agentic coding task. So what we demand of our language models today is probably beyond the imagination of anyone 10 years ago.

That said, I think the fundamentals haven't changed that much. Largely, we still build on GPUs and

**[18:32]** kernels. We still optimize using gradient, or stochastic-gradient-like, approaches. We still have the Transformer and attention — and we'll talk a little bit more about architectures, but it hasn't changed that much. I think the specs are different: now we demand greater context lengths, which means that inference efficiency matters even more. So the good news for us is that we didn't have to completely change this class — only touch two sections on the latest Chinese architectures. But the fundamentals, I think, are here to stay, at least for now.

Okay. Maybe I'll pause there in case there are any questions or thoughts.

**[19:26]** Okay. So let's go on. So, brief interlude: what is this program? This is what I call an *executable lecture*. If you think it looks like a Python program, it is — because it actually is a Python program, but it has been rendered for your viewing pleasure. So when I step through it, it is actually executing the lecture, and it makes it possible to basically step through code, which will be hopefully interesting and important later. And you can also see the hierarchical structure of this lecture. For example, we're done with this function, now we go back to `main`.

Okay, so let's talk about the course logistics and the syllabus, which I think is going to take a

**[20:12]** good chunk of time. Okay, so all the information is online, on this website — cs336.stanford.edu. This is a five-unit class. I think this class probably has a certain reputation, so I don't need to belabor the point too much, but we have five assignments. They are pretty intense — even the first assignment, according to this one review, was actually equivalent to the five assignments for CS224N. I've been told also that this is exaggerated, but better to, I guess, be conservative in your estimates.

So why should you take

**[20:57]** this course? Okay, so first: you have an obsessive need, like me, to understand how things work. That should be your primary objective — I think just pure curiosity about how language models work. And then, in doing the class, you'll develop much stronger research engineering muscles, and have the confidence to go into a new setting and be equipped to deal with whatever comes up. So I think when I started at Stanford, I *[Ed: verb garbled in the captions, which read "screwed"; likely "taught" or "created"]* this class called Statistical Learning Theory, which was basically teaching the theoretical side of machine learning. And that was nice, because it equipped people — when you read a paper you can understand all the math. And now, since the field has kind of shifted a lot more towards this more systems and empirical side of things,

**[21:42]** this is sort of an analogous class, which gives people enough depth that they feel that everything else seems kind of easy.

So why should you *not* take this course? This is important, because there are reasons you shouldn't take this course. First: you actually want to get some research done this quarter. You should probably talk to your advisor. They should know you're taking this course, otherwise there might be some surprises. Second: you're interested in learning the hottest new techniques in AI. I think there are many other great courses — seminar courses, topics courses — that are good for that. We don't do many of the things; we don't do multimodality, we don't talk about agents in any depth. So

**[22:27]** if you want to learn about that stuff, this is not the right course for that. Third: if you come in and say, "I have an application domain, I want to get good results on it" — probably this is not the right course, at least to start. I always recommend: just prompt the model, fine-tune the model, and then as a last resort you pre-train your own model, because it is a pain and it's also expensive. But it's a lot of fun.

Okay, so if you're not taking the class, you can soft-follow along at home. All the lecture materials will be posted on the website — oops — and they are also recorded through CGOE, so thank you CGOE for doing that. And later

**[23:13]** they will be published to YouTube. Now, of course, following at home, watching the lectures is great, but you learn really by doing the assignments, so you'll have to figure out how to motivate yourself to do that.

Okay, so speaking of assignments: we have five assignments. The philosophy of the assignment is, how do we do the "from scratch" but not just say "build a language model" and that's the assignment. So we don't provide scaffolding code, but we do provide a bunch of unit tests to make sure that whatever you're building is actually correct — so that you don't get the sparse-rewards setting where you submit a homework and it's either correct or not.

**[23:58]** What I recommend — the assignments are structured so that much of the assignment can actually be done locally on your laptop. You can implement it and check for correctness, and then we are providing a cluster so that you can do an actual training run to see what the accuracy is, or get a bunch of GPUs to actually benchmark the performance of some kernel. And then, for fun, we will have some leaderboards for most of the assignments, at least, and they will look something like: well, now that you've learned about this topic, try to minimize the perplexity given some sort of budget.

And both Marcel and

**[24:44]** Herman were masters of leaderboarding back in the day — which was last year, or the year before. So if you want tips on that, I'm sure they would be happy. Actually, I don't know if they would tell you their secrets, but you can try.

Okay. So last year we were thinking about, well, what can AI do? It's like, okay, well, I mean, yes, everyone can use AI, but just try to do your best. I think now, coding agents have gotten so good that they can just also solve all the assignments, right? But, to state the obvious: obviously you're not going to learn anything if you just feed the assignment 1 PDF into Claude Code.

**[25:31]** At the same time, AI can be very useful for answering questions and tutoring. So we have to find a way to leverage AI. So what we've decided to do is, we provide you an `AGENTS.md` file — or equivalently a prompt — which asks the AI to be pedagogically minded. You can read more about it in our AI policy guide. And the requirement is that if you're going to use AI, use it with this prompt. And so it will answer questions about code, it will clarify any understanding, but it won't accidentally generate the Transformer for you when the homework is to implement the Transformer. Okay, and this is the first year we're trying to do this, so please try it out

**[26:18]** and give us feedback on whether it's working or not working.

Okay, so compute. So this year we have — thanks to Modal — they have provided us with compute credits on their platform. It's actually quite nice. Unlike last year — well, I guess you guys didn't take the class last year — last year we had a cluster you SSH into. This is using more of an API, which I was initially skeptical of, but looking at it, it's actually pretty pleasant to use. So again, try it out and give us feedback on how it is. We've written a guide on how to access and use the compute.

**[27:04]** Okay. Any questions about course logistics? Okay.

All right, let's talk about what we're going to cover in this class. So there are basically five parts, mirroring the five assignments that you'll have: basics, systems, scaling laws, data, and alignment. So I'm going to now go through each part and just give you a taste of what you will learn.

Okay, so in the basics — this is basically the first two weeks — the goal is to just be able to train a language model and

**[27:51]** build it from scratch. So the components here are: we're going to tokenize the data, we're going to define the architecture, and then we're going to implement the optimizer and trainer. Okay. So then you wonder, what is the rest of the class for? Well, we'll get there.

So let's start with tokenization. Tokenization is really about what are the atoms that the model operates on. And formally, a tokenizer converts between raw inputs, which are just bytes, and a sequence of integers which represent the tokens. Conceptually it's a segmentation of the text. We're going to talk about the byte-pair encoding (BPE) tokenizer, which intuitively

**[28:36]** breaks the input into frequently occurring chunks.

And then, remember, this class is about maximizing efficiency. So through an efficiency lens, tokenization is good because it takes a long sequence — if you just think about the raw byte stream — and reduces it into a smaller number of tokens. But more subtly, and maybe more importantly, it allows you to do adaptive computation. So maybe some places are actually a lot of bytes but should be compressed into one token, whereas some of the more rare or interesting parts of the input should be left as multiple tokens. Okay, we'll talk more about this.

I just want to mention that every year I'm hoping that I don't have to teach tokenization,

**[29:24]** because the dream is to really have an end-to-end way that directly operates on bytes. And there's been a number of works, including recently this H-Net work, that seem promising — but so far these have not been scaled to the frontier, and since the frontier models are still using tokenizers, it felt like it would be still wise to teach tokenizers.

Okay, so now, after you tokenize your input, you have a bunch of tokens. Now you define a model on top, and everyone, I think, has a familiarity with the Transformer. And if you took CS224N, the NLP class, then you've seen Transformers.

**[30:09]** Since then, I think there have been a lot of improvements or refinements to Transformers, which I think are important, and Tatsu is going to talk more about this in a bit. But just to run through a set of types of things that one might have to think about. So the activation functions have evolved. How you do positional encodings has evolved. How you normalize the different layers, to prevent blow-up, has evolved. Instead of doing full attention, there are many ways to basically reduce the attention computation, because attention is n-squared — where n is the sequence length — and that gets really expensive.

**[30:55]** So there's a bunch of ideas around that. If you're more ambitious, you can look at these state space models, or equivalently linear attention — like Mamba and Gated DeltaNet. These have become popular in the last few years, and usually some hybrid between these models and attention seems to work quite well. So we'll be exploring some of that.

And then, within the MLP layers of the Transformer: the original Transformer was just a dense MLP, and now mixture of experts has become sort of the dominant paradigm for building compute-efficient Transformers. So we're going to talk about that. And of course, with mixture of experts,

**[31:40]** it's not just defining your architecture — we'll see that we'll also need different techniques for training the model.

And then finally, perhaps somewhat boringly but an important question is: what is the shape of your Transformer? How many layers? How many heads? What is the hidden dimension? Number of experts? This might come up more as we talk about scaling laws, but setting these actually seems kind of trivial — it's a hyperparameter — but actually, in the context of scaling language models, has a huge implication.

Okay. So once you define your model architecture, how do you train the model? And here there's a bunch of design decisions around the loss function.

**[32:27]** There's next-word — next-token — prediction, which is the default, but people have found that looking at predicting more than one token seems to be helpful for improving the model. And there's optimizers: people used to use Adam, but increasingly Muon has been used, especially with some of the latest open models such as the Kimi K2 models. Initialization, which again sounds kind of boring, but turns out to have a huge impact on the training stability of larger models. Learning rate schedule, regularization, batch size.

**[33:15]** And then MoE-specific things. So you look at this list and you might think, well, these are just hyperparameters, I'm going to try a bunch of different options out. But it turns out that really being very careful about setting these hyperparameters in a principled way will make the difference between a run that just blows up and is useless, and a run that is achieving state of the art. Okay, I'll come back to that point when we talk about scaling laws.

Okay, so then in assignment 1, what you're going to do is you're going to implement the BPE tokenizer, implement the Transformer, the loss function, optimizer, the whole training stack. We're going to make you do a bunch of resource accounting, so you understand where your FLOPs are

**[34:00]** going. You're going to train some models on these datasets like TinyStories and OpenWebText. And then there's going to be a leaderboard where you're going to try to drive down perplexity as fast as you can. So for those of you familiar with the nanoGPT speedruns, it's kind of similar to that.

*[Interjection from the floor, inaudible]*

Okay. So by the end of assignment 1, you should be able to walk away and build a language model from scratch. So that's very exciting. If I have a high-level takeaway here, it's that while the tokenizer and modeling and training are sort of

**[34:46]** presented as distinct pieces, everything is actually about kind of balancing the following. So you want expressive models, because you want to represent the complexities of the data. But at the same time you want your training to be stable, and we're going to talk a lot about how do you keep the parameter and gradient norms in the sort of Goldilocks zone, so they don't blow up and they don't vanish. Turns out a lot of training language models is about just stability. And then finally efficiency, which is somewhat more straightforward — you just make it run fast on hardware. But you're going to see interesting things like: if we change the architecture, a

**[35:32]** lot of the architecture decisions are, well, we can make it faster by, let's say, projecting into a lower-dimensional space. But then the question is, does it work as well? And so making those tradeoffs is sort of the name of the game here.

Okay, so in assignment 2 we're going to dive more deeply into systems. And the goal here is just to get the most out of your hardware. So we're going to talk about kernels, how you parallelize across multiple GPUs, and how you do inference. So, the basics — which we're actually going to start on in our next lecture.

**[36:18]** I mentioned resource accounting, and you've all probably built models, but this is really about keeping track of where all the FLOPs go, and where all the memory is being spent. So we're going to spend some time basically doing the resource accounting. We're going to see this formula that comes up: how many FLOPs does training a 7B model *[Ed: the program on screen computes this for 70B — `6 * 70e9 * 1e12` = 4.2e23 FLOPs. The captions say "7B" here; one of the two is a slip and it is not recoverable from audio alone.]* on 1 trillion tokens take? Well, it's 6 × N × D, roughly. And where does that come from?

And then we're going to look at the hardware. And here's a very cartoon-ish picture of what to remark about hardware, which is that

**[37:04]** your memory is not where your compute is, and you have to move either your parameters or activations from the memory to the compute, do the compute, and move it back. Right? And that often is the bottleneck. So for example, the B200s, which we'll have the opportunity to play with, have 2.25 petaFLOPs per second in BF16, and have 8 terabytes a second of memory bandwidth. So what does that mean? I think I'll do this next lecture — we're going to break this down and use this information to do some calculations and see how long different types of algorithms will need to take. We're going to talk about roofline

**[37:50]** analysis, which allows us to understand whether a computation is bottlenecked by either compute or memory. In general, it is memory. And then talk a little bit about benchmarking and profiling.

Okay, so here's what a DGX B200 looks like. You have eight GPUs. They're connected via NVLink. And then if you have a thousand GPUs, then you would have multiple of these, and they're connected either by InfiniBand or Ethernet.

So the next two parts, when we talk about systems, is kernels. So a kernel is basically a function that runs on the GPU, and when you're just using plain

**[38:37]** PyTorch, all the PyTorch primitives actually correspond to launching particular kernels, which are built in. So you're already using kernels, whether you know it or not. But the point is that for certain types of computation, if you look at it, you can actually write custom kernels to make the GPUs go faster. And the main principle here is organizing the compute to minimize data movement. So, remember this picture: moving data from memory is expensive, so you want to try to minimize that.

So,

**[39:22]** just as a simple example, suppose you wanted to compute A and B. So often you would have to read from your high-bandwidth memory (HBM), compute it, write it back, and then read again, compute it, write it back. Right? So then you're basically sending the data back and forth twice. And there's this idea called fusion, where you read it once, do both of the computations, and you write it back. And that will save you a lot of time. So that's operator fusion. Tiling is kind of a more sophisticated variant around the same idea.

GPUs have also gotten a lot more complicated, and I'm not sure how many of these details we'll have time to get into, but at least we want to expose you to some

**[40:08]** of the peculiarities, I would say, of GPUs, and give you an appreciation for the types of things that one has to consider in order to squeeze as much juice out of them as possible. And we'll write some kernels and try.

So what happens if you have thousands of GPUs? So the principle of minimizing data movement is still the same. The only thing is that moving data between different GPUs is even more expensive. We're going to talk about how these very classic collective operations, like gather and reduce and all-reduce, are the way to think about

**[40:53]** basically distributed training. The general game is that we have these model parameters, we have activations, gradients and optimizer states, and they need to be sharded or split across multiple GPUs. And of course you need to bring the right data to the right nodes to do the compute and write it back. So there's a whole kind of orchestration, and how to do that efficiently is going to be the topic of this unit. And there's multiple ways to shard: you can shard by splitting up your data, splitting up your model, splitting up different layers in the model, splitting up the sequences, splitting up between experts — and we'll talk about the tradeoffs that come with each of these.

**[41:39]** Okay, and then finally we're going to talk about inference, which, as I mentioned, is growing in importance. So the goal of inference is to actually use the model — minor detail here. So of course you need to use inference when you're chatting with the model, but it also is useful for reinforcement learning; it's useful for doing the rollouts, test-time compute, generating synthetic data, evaluation. So inference is a very critical part of what it means to do language modeling work. So we're probably not going to spend as much time as I'd like on this, because the course is already kind of filled up, but we'll see what we can do.

**[42:25]** There was some discussion around whether we should make you write inference from scratch, but we'll see.

So, the way to think about inference is that there are two phases: a prefill and a decode. In the prefill, you take the prompt and then you feed all the tokens forward and build key-value pairs. This is very much like what happens in training. And then in the decoding part, tokens are generated one at a time. And this is the part that quickly becomes memory-bound, and this is why inference is hard.

So there are many things you can do to speed up inference. You can try to use a cheaper model, by pruning a larger model.

**[43:12]** You can quantize, you can distill. You can use this technique called speculative decoding, where you use a cheaper model to sort of run ahead and guess a bunch of tokens, and now you use the full model, which can operate on those tokens in parallel, to see if it's good. And if you got lucky, then you can accept all those tokens and you're much faster than if you were doing one token at a time. And of course you can do systems optimizations — there's a bunch of kernels that are designed specifically for inference. And then one of the interesting things about inference is that if you're running a service, queries are coming at you at potentially different times, and then you have to figure out how to batch them up. Whereas in training, you're basically already defining your

**[43:58]** batches and everything is much more predictable.

So in assignment 2, there's going to be implementing kernels in Triton, and doing some sort of parallel training. The details here might actually change, as the CAs have grand plans of revamping the systems assignment. So the assignment might look a bit different from last year's, but it will cover roughly the same material.

One thing I will mention is that there's this wonderful book out of some Google people called *How to Scale Your Model*. And I think it's really nice for providing a conceptual understanding of

**[44:44]** roofline analysis and Transformer math and doing LMs conceptually. So I highly recommend you take a look at that. Now, the only thing is that it's from Google, so it's about TPUs, but a lot of the high-level concepts are similar. And now they have a new chapter on how to think about GPUs.

Okay. So the third assignment is about scaling laws. So by now you've trained a language model, you can make it go really fast by optimizing kernels and parallelizing. Now you want to scale up. So how do you scale up?

**[45:30]** So imagine the following setting. If you had 1e25 FLOPs — so this is tens of millions of dollars of compute — what model would you train? Okay, so this is, I think, kind of a daunting task, because if you mess up, well, that's a lot of money down the drain. And you can't do your probably typical hyperparameter tuning at that scale, because you only get to train one model. And so this is the key problem that you have to deal with in large language model training, that you don't really have to deal with if you're just fine-tuning a model or doing small-scale stuff.

And so the key conceptual shift here is that we shouldn't think about a single

**[46:16]** model that we're training, but really think about a *scaling recipe*. And a scaling recipe is a mapping from a FLOP budget — let's say 1e25 or 1e24 — to a set of hyperparameters, basically a config file. And for a given scaling recipe, what we will do is run a bunch of experiments to compute the loss that you get at smaller scales, and then you fit a scaling law, and that enables you to predict the loss at a target scale. So maybe you run some small experiments, you fit the scaling law, and then you project out what you're going to get at larger scale.

Okay, so that's the primitive. Now, using this, what you can do is now you

**[47:02]** can optimize the scaling recipe targeting a larger scale using smaller-scale experiments, which is wonderful. And second of all, you can predict the loss that you're going to, in theory, achieve before actually running the experiment. Which allows you to go raise money, and you say, "Well, look, I ran the small-scale experiments and I think I can get a really — like a GPT-5-level model. Please give me a lot of money so I can train that model."

So, one thing I think is maybe another misconception is that scaling laws are not laws of nature. They don't just happen automatically. You kind of

**[47:47]** have to will them into existence. And this happens by careful construction of a scaling recipe. Right? And a scaling recipe, remember, has to extrapolate. So what this typically means is that you have a sequence of hyperparameters, which say: as the scale increases, maybe the learning rate is a constant, maybe it drops, maybe the batch size increases — by how much? And these are things that a scaling recipe has to figure out.

And so, in order to get these predictable scaling laws, one thing that you actually have to think about is how do you parameterize the model in a way to get what's called *hyperparameter transfer*. Right? Meaning that the

**[48:32]** hyperparameters you use at small scale are either the ones that you use at larger scale, or are predictable functions of that. Right? Because if at every scale your learning rate acts like sometimes 1e-5 and sometimes 1e-4, then you're not going to be able to magically guess the right learning rate at larger scale.

So one shift in thinking is that the predictability is actually at least as important as optimality. So you normally think of, oh, we're trying to optimize for efficiency here, and we want to hyperparameter tune and make things optimal. And yes, you do want to do that, but you also want this predictability so that you don't get [surprised] at larger scale. *[Ed: a word is dropped in the captions here; the sense is "so that you don't get surprised at larger scale."]*

Okay, so this is kind of stage-setting. The actual scaling laws we're going

**[49:18]** to look at are fairly classic. So some of you might have seen this idea of, well, if you're given a FLOPs budget, how should you balance training a larger model versus training on more tokens? And this is where the classic compute-optimal scaling laws from Kaplan et al., and the so-called Chinchilla scaling laws, come in.

The basic idea is that for each FLOPs budget — so let's say 6e18 all the way to 3e21 — you sweep across different model sizes and you choose the best

**[50:05]** one. So think about minimizing each of these. And then you fit a curve that allows you to basically predict the number of parameters given a FLOPs budget. And if you're lucky, it will lie roughly on a line. And if you're unlucky, it's going to be all over the place, which means that you should have no confidence that you're going to be able to predict reliably.

And so the upshot of this — this is quite crude, but a rule of thumb — is that 20 times the number of parameters is the number of data points you should train on. So a 70B parameter model should be trained on roughly 1.4 trillion tokens. Of course, depending

**[50:51]** on the dataset and architecture, this number will actually vary.

Okay. Also, this doesn't take into account the inference cost. A lot of models these days are small, but they're trained on way more tokens than is compute-optimal, because you want a smaller model for inference reasons.

Okay. So one fun thing that we've been doing in the Marin project is pre-registering our results. So we fit a bunch of scaling plots at different compute budgets, we fit a scaling law, and we basically made these predictions out to 1e22 FLOPs. This one is actually training — if you go to the Marin site, you can follow along. It should actually be done maybe as early as tonight. So maybe on

**[51:38]** Wednesday I'll report back on how we did, and see how we match the pre-registered loss. So the idea here is that we made a prediction: if we were to train this large model, which we've never trained before, can we predict how well it's going to do? Then that's really nice.

Okay. So in assignment 3 — I think this is kind of a fun assignment; let's just say it's a fun assignment. So we're going to define this training API, which basically, you give us hyperparameters and we're going to give you a loss back. So what we're going to try to do is simulate what happens if you could do a

**[52:24]** lot of training runs. And of course we don't have enough compute to actually allow everyone to train their own 8B model or anything. So basically, we train a bunch of models offline, and we provide this cache, so it looks like you're training. And what you're going to do is submit training jobs — you give us a config, we give you a loss back. And then you can do whatever you want. We would recommend you fit scaling laws to these points, and then you extrapolate, and then we give you a budget, and we basically evaluate how well your model landed.

So it is meant to — I was going to say it's meant to kind of replicate the high-stress scenario if you actually

**[53:11]** had a budget of, like, $100 million that you needed to spend, and you have to be very careful about how you spend this. Of course, this is low-stakes.

Okay. So at this point, you will have trained a model, you know how to make it fast, you know how to scale up. Now what's missing? What do you train the model on? And that's going to be the subject of the data section, which is arguably one of the most important things, because data quality basically specifies how good your model is going to be.

**[53:57]** One way to also frame it is: what do you want your model to do? Right? Data basically reflects what you want your model to be. So, do you want to speak multiple languages, be good at having a conversation? Do you want to run long agentic coding tasks?

And so part of that is — we're going to start by talking about evaluation, which basically defines the capabilities that you'd like your model to have. One thing we'll talk about is that evaluation is a fairly deep topic. It's not just about running on some benchmarks. There are internal evaluation metrics

**[54:45]** for model development, and what matters here is smoothness across scales — remember, we want things to be predictable — and relative performance matters. You don't necessarily care how well this does in an absolute sense, because, let's say, the perplexity number — what does a perplexity of, you know, 1.2 *[Ed: the captions read "a perplexity 1. you know, two", so the value is a reading of a garble, not a certainty.]* on some held-out data really mean?

And then there are external metrics. These are things that you report to your customers or your reviewers, or whoever you're presenting your thing to. And here, ecological validity really matters. And I think sometimes these two things get conflated, but I think they really serve

**[55:30]** two distinct purposes. And so you can think about perplexity as something that is really helpful for internal development. And still to this day, perplexity is a very good way of capturing the intrinsic quality of a model without worrying about benchmaxing.

Now, there's a separate question of what you run your evals on. And the recommended thing is, well, if you have some data that's not on the internet, that would be good, because you can avoid contamination. And then there are advanced use cases, which are more representative of external-facing use cases.

**[56:17]** So one thing to also note is that language models are purportedly very general purpose. Right? So it's only fitting that we actually need a very diverse set of evaluations. And I would always recommend having many evaluations — you can average them into a single number, but often that average conflates a lot of different things.

Okay. So now, after we set up the evals, we know what we're building. How do you get the data? Well, the first thing is that data does not just fall from the sky. It has to be actively curated. Often, especially in classes and also in research, sometimes you're

**[57:02]** just given a dataset, and then it's like, okay, well, now I do stuff on the dataset. But for a lot of language models, especially if you want to collect these large datasets, you have to go actively look at it. So: webpages crawled from the internet; there's books, I guess, which is controversial at this point; but arXiv papers, GitHub code, and so on. This is an old figure from The Pile, from 2021, and you can see language modeling datasets are fairly diverse.

There's, especially these days, I think, a lot of contention around: is it fair use to train on copyrighted data? Maybe sometimes you have to license data, and so on. So there are legal issues around data, which I think are quite

**[57:48]** important. For example, a lot of GitHub code doesn't have a license. So how do you interpret that? Do you assume it's permissive, or are you conservative and assume it's not permissive?

So there's also the fact that data is not even text, right? It's either HTML or PDFs, or code is directories, and this requires processing to turn it into actual text to be usable for training. So that's the topic of data processing. There are a few steps that have to happen here. Transformation: converting some non-text thing into text. Filtering: keeping only the good stuff — if

**[58:35]** the random internet document in Common Crawl is extremely bad, you don't want to train on it, most likely. You want to deduplicate. There are multiple sources, so how do you combine the different sources? And then finally, more recently, there's been a lot of work on generating synthetic data, which could mean taking the real data and just rewriting it into things that are more like the downstream task, or just more Wikipedia-like, or whatever you want. So this is an active area of research.

Also, data can be used both for pre-training; something called mid-training, which is usually the high-quality data that

**[59:20]** you put at the end of the pre-training step — and this includes long-context data, such as maybe big code repositories or books. And then finally there's post-training data, which is maybe conversations or agentic traces with tool calling.

So in assignment 4, we're going to make you start with a very raw corpus, like a raw web crawl, and do all the work to filter and to dedupe and make the data clean. So this is — I would say, I don't know if I would call it not fun,

**[1:00:07]** but it is certainly a lot of what people would call dirty work. But that is an important part of building a language model from scratch. So you have to get the full experience.

Okay. So finally, alignment. So far we've basically trained a model using full supervision: predict the next token, or the next few tokens. Now, at this point the model should already be reasonable, but we can improve it further by using weak supervision. And why weak supervision? It's because sometimes it's easier to critique than it is to generate. So you can't always have data that says this is the

**[1:00:53]** right response to this prompt, but maybe you can have a way of specifying what good looks like.

So then the basic template is that you generate responses from a model, you score them either with a human, or a verifier, or an LM judge, and then you update the model to prefer better responses. This can be instantiated either through various RL algorithms such as PPO or GRPO, or — in a simpler way, for preference data — DPO.

So the challenges around RL are that RL algorithms are unstable and hard to tune. Some of you probably know this from first-hand experience. Personally, I prefer to keep things as

**[1:01:38]** much in the fully supervised case as long as possible, and then finally, okay, fine, I have to do RL. But some people, for whatever reason, like doing RL.

Also, what we'll hopefully talk about this year is that if you do RL at scale and try to maximize your throughput, there are actually a lot of systems challenges. You have to have an inference server and a training server, and then the inference server has to generate these rollouts — especially if you do RL against environments that involve code execution. It's a whole kind of orchestration game. And then if your workers lag behind, then you get into off-policy issues, and then you're constantly kind of juggling

**[1:02:23]** this on-policyness with a desire to maximize throughput. It's a big wonderful mess, which hopefully we'll talk more about when we come to that.

So, assignment 5 — we're still deciding what exactly we want to do. Last year it was: implement DPO and GRPO and get it working for some math benchmark. But we'll see how much farther we can push it on the realistic dimension this year.

Okay. So again, remember: it's about efficiency, and efficiency can mean either data efficiency or compute efficiency. So the way to think about it: you have all these resources. You have data, you have the hardware, which has compute cores, you have memory,

**[1:03:10]** communication bandwidth — and you're just trying to figure out how do you build the best model according to some evaluation, given a fixed set of resources.

So through this lens, I think you can actually think about a lot of these design decisions as optimizing for this. So: systems, clearly that's about compute efficiency. Tokenization, as I mentioned before — you can't just work with raw bytes, that's going to be very compute-inefficient, at least with today's model architectures. And so a lot of tokenization is about improving compute efficiency. Model architecture: many of the changes that we'll see are motivated by reducing the memory or

**[1:03:57]** FLOPs — in fact, a lot of them are influenced by the need to have faster inference. Data filtering, you can also view through an efficiency lens: we don't want to waste time updating gradients on a bunch of redundant, bad data. Even if it might not hurt you, it hurts you in the sense that if you have a fixed compute budget, more time on bad data means less time on good data. And then finally, scaling laws is explicitly about how you can essentially do effective hyperparameter tuning on much smaller models.

And now, tomorrow we might become data-constrained, and the calculus of what design decisions you should take might change. But I think the overall mindset, which we're trying to teach you, is: think about the efficiency of your approach.

**[1:04:46]** Okay. Let me stop there. Are there any questions about any of these assignments or topics? Okay.

So now let's do tokenization. So this is — we're jumping into our first unit here. So, Andrej Karpathy has this really good video on tokenization; you should check it out.

So, the starting point: raw text — what is text? It's Unicode strings. And on the other hand, the language model places a distribution over

**[1:05:32]** sequences of tokens, usually represented as indices. So we need a procedure that encodes these strings into tokens, and also a procedure that decodes tokens back into strings. So a tokenizer is basically something that can do this round trip.

So here are some examples to give you a flavor for how tokenizers work. Actually, I should have tried to get internet earlier, so this is not going to work. Okay — if you go to the site, you can play around with different tokenizers.

So, some observations here. And you'll appreciate why tokenizers are kind of annoying, why people want to get rid of them. So a word, and a word conglomerated with its

**[1:06:19]** preceding space, are different tokens. So many of the tokens you'll actually see are space-word, which is fine, but kind of strange. So this `hello` and this ` hello` are actually two completely different indices that have nothing to do with each other.

And sometimes, depending on the tokenizer you use, numbers are represented with — every few digits is a token. Sometimes it's predictable, and sometimes it's not. Some tokenizers try to make every digit a token, but then you're blowing up the number of tokens you have, so there's some trade-off there.

So, here's the GPT-5 tokenizer.

**[1:07:04]** So it can take the string and convert it into these indices. And then you can decode it back into the string. And a tokenizer should round-trip. If you implement a tokenizer that doesn't round-trip, you have a problem.

So, the compression ratio here is the number of bytes per token. So in this case, the number of bytes of this string is 20. The number of tokens is 8, and you do 20 / 8, so the compression ratio is 2.5. Okay, so 2.5 bytes per token. The larger the compression ratio, that means the shorter the sequence, which is good

**[1:07:50]** because attention is quadratic, and you want to make sure the sequence is shorter.

Now, you could obviously increase the compression ratio by increasing the vocab size, but then you get into sparsity, because every element of the vocab is treated as a distinct element. So these days tokenizers, especially multilingual tokenizers, have 100k or 200k distinct tokens. So you can look at the GPT token vocabulary — I think we'll skip this in the interest of time.

Okay, so how do you build a tokenizer? So I'm going to go through this fast. So the first thing you might do is

**[1:08:35]** like, well, a Unicode string, that's already a sequence of Unicode characters, and each character is an integer, which you can get by calling `ord` in Python, and you get some number out. And this can be converted back into characters. So let's just build a character-level tokenizer, which basically breaks up each character and encodes it as a token, and then this can decode back. So this is good.

So now, there are 150k Unicode characters, so your vocab size could be 150k — which is a lot. I mean, it's not crazy, but I think the bigger problem is that many characters are actually rare. Which means that it's really an

**[1:09:20]** inefficient use of a vocabulary. And also the compression ratio, which kind of reflects this, is not that great. So most of the time you're actually using a lot of tokens to represent your sequence, and many of the indices are actually not being used very much. So this is not a very good tokenizer.

So here's another attempt. So you can turn strings into bytes. Unicode has a UTF-8 encoding, which means that sometimes a string like `a` is just one byte, and sometimes a string is multiple bytes. So

**[1:10:06]** let's build a tokenizer around that. So we can take this string and convert it into a sequence of bytes, and notice that this is a longer sequence now — but all the numbers are between 0 and 255, because that's what a byte means. And the compression ratio is one, which is not great. Right? So byte sequences can be very long, but the vocab size is small.

Okay, so both of these are really bad. So let's try to make some progress. So this is what

**[1:10:51]** people actually used to do in NLP, if people remember. So if you take a string, I can just chunk it up — break it up by spaces or some regular expression — and let's just call each of these chunks a token. Okay, so what is good about this one is that each token is meaningful, because humans invented words and words tend to have a stable semantic meaning. But your vocab size is the number of distinct chunks in the training data, which could be a lot. And also your compression ratio — I mean, it's quite good, but the vocabulary can be huge.

Actually, it's worse than that, because the vocabulary could be

**[1:11:39]** actually unbounded, right? Because at test time you might get some sequence, and you tokenize, and then you have a token you've never seen before. And people used to assign these, like, UNK tokens, but that's really ugly and can mess up your perplexity calculations. So this is also not great.

Okay, so what we're actually going to do is called byte-pair encoding. And this was introduced a long time ago for data compression, way before language models were really on the scene. It was first introduced to NLP for doing neural machine translation, and the first paper that used BPE for LMs was GPT-2.

**[1:12:27]** So the basic idea is you're going to train the tokenizer on raw text to construct a vocabulary that's tailored to the data. And you're also going to have this property that everything can be tokenized: if it's rare, then it just breaks up into smaller units, rather than having this UNK token. So common sequences are going to be represented as one token, rare sequences are going to be split into multiple tokens. That's the idea.

Okay, so the algorithm is fairly simple conceptually. So you start basically with your corpus — let's assume it's one long sequence. You get a byte sequence, each byte starts as a token, and then we're going to merge successive pairs of adjacent tokens

**[1:13:13]** that occur the most frequently.

Okay, so let's step through how this is going to work in code. So here's a simple string, `the cat in the hat`, and here's the implementation of the BPE algorithm. So we're going to turn that into a sequence of bytes, and then first we're going to basically count the number of times successive tokens appear. So `116 104` shows up twice. So we get this, and then we're going to find the pair that happens the most number of times. So that's `116 104`. I guess there are a few ties, but

**[1:13:59]** we'll just take the first one. And then we're going to merge that pair. And by merging that pair, what we do is we create a new token. In this case this is going to be called token 256. It's going to represent this pair, and we're going to add it to our vocabulary. So 256 is going to represent the sequence `th`. So `t` and `h` have been merged, and every time we see `th`, we're going to use 256 to represent that. And then we go through `indices` and we replace every occurrence of `116 104` with 256. So those two places have been replaced.

And then we iterate. So the next time we do this, we're going to find 256

**[1:14:47]** and 101, we're going to merge that, and now we have 257. And then we're going to merge one more time and we're going to get 258. So over time the sequence is shrinking, and the vocabulary size is growing.

Okay, so — and the compression ratio here that we get for this, I mean, this toy example, is 1.5.

Okay, so now that you have a tokenizer, how do you tokenize new text? Well, you take a new string and you encode it, and conceptually what happens is

**[1:15:34]** that you basically go through the set of merges that you've made, and then you just apply the merges to your string. Okay, let me not actually step through that code. So that will give you a sequence. So this is the sequence encoding of `the quick brown fox`, and then when you decode it, you get the same thing back.

So I went through this a bit fast, just in the interest of time. I will say that this implementation works — this is a full-blown BPE implementation. It's extremely slow. So in assignment 1, we're going to ask you to basically

**[1:16:20]** make it faster. So currently `encode` loops over all the merges, which is very slow, because the number of merges you have is essentially the vocab size minus 256. So you only want to loop over the merges that matter, and you have to build some indices to make that happen.

There are some details around special tokens — conceptually not deep, but important to building a modern tokenizer. Another thing is that I've presented the tokenizer, just for simplicity, as: you take an entire string and then you try to tokenize it. Really, what happens is that you break up your text into chunks and then you apply the

**[1:17:06]** tokenizer on each chunk. So that's going to be much faster. And then try to make it as fast as possible. At some point you might realize that Python is just not very fast, and if you want to implement it in, say, your favorite language — Rust or C or something — then go for it.

Okay, so quick summary. Tokenizers convert between strings and tokens, or indices. The previous character-based, byte-based, word-based approaches are highly suboptimal in their own way. BPE is an effective heuristic that is data-driven, so it seems to be pretty effective.

Now, like I said

**[1:17:54]** before, maybe next year I don't have to teach this, but for this year we're stuck with tokenization. Even if we get rid of tokenization, though, I think whatever solution replaces it has to satisfy the following properties, right? If you have the model, the Transformer needs to operate on some sort of abstractions of the sequence. And this is most evident if you think about not just text but video, or DNA sequences, where the individual bytes or units are actually quite low signal-to-noise, and you have to do some sort of abstraction to lift it into a place where you can do modeling on that.

**[1:18:41]** And then finally, as I mentioned, chunks should be kind of variable. You want adaptive computation. Not all bytes are treated the same, and if you don't do that, I think you're going to be suboptimal. So any end-to-end solution also, I think, has to have these properties.

Okay, so with that, I will end. Next time, on Wednesday, we're going to start the unit on resource accounting, which is sort of a baby systems, I would say. And then after that we're going to go back into architectures and go from there. All right.
