---
title: Data II — Filtering, Deduplication, Mixing, Post-Training Data
lecture: 14
video: https://www.youtube.com/watch?v=5sxHosTLPF8
source: YouTube auto-captions, copy-edited
original: original/14-data-filtering-dedup-mixing.md
---

# Lecture 14 — Data II (transcript)

This is the copy-edited transcript of Lecture 14, the second of two lectures on
data — filtering, deduplication, mixing, and post-training data. The
auto-generated YouTube captions have been repunctuated, segmented into
sentences, stripped of filler ("um," "uh," "you know," "sort of" when vacuous)
and false starts, and had mis-heard technical terms and names restored, checked
against the lecture's source program. The verbatim captions are kept alongside,
in [`original/14-data-filtering-dedup-mixing.md`](original/14-data-filtering-dedup-mixing.md).

## Editorial notes

The captions were repunctuated into sentences, filler ("um," "uh," "you know,"
"like" as filler, "sort of" when vacuous) and false starts were removed, and
speaker turns were disentangled. Two spoken self-corrections are kept only in
their corrected form, following the precedent set in lecture 13's edit: at
13:20, the lecturer says "if it has LaTeX in it, then it's a higher bar — if
it's, uh, sorry, if it's — it's a lower bar," and only "if it has LaTeX in it,
it's a lower bar" is kept (confirmed against the material's own thresholds,
0.17 with LaTeX / 0.8 without); and at 56:14, "worst case it's wasting compute,
sorry, best case is wasting compute" is kept only as "best case, it's wasting
compute; worst case, you're overfitting." One apparent caption error was
corrected against the material rather than left as spoken: at 14:52, the
captions have "the raw data is **not** already a Python subset of The Stack,"
which contradicts phi-1's own setup (`R = "Python subset of the Stack"`); the
stray "not" is dropped. One stray word with no discernible meaning ("security")
was dropped from "this more targeted security data collection" at 13:20 as
caption noise, not content. Nothing else was deleted, added, or reordered, and
every timestamp marker — 109 of them, 32 in `[H:MM:SS]` form — is preserved
in its original position, split exactly where the captions split each
sentence, with nothing moved across a marker boundary.

Eight student questions were audible enough in the captions' own words to mark;
they are set off as `*[Question from the floor: ...]*`, with
`*[continued from the floor: ...]*` where a question's own words span a
timestamp boundary (19:35/20:21), following the convention lecture 13's edit
established. Where the lecturer's own paraphrase ("so the question is...")
carries the content, that paraphrase is left as ordinary prose, per the same
convention.

**Restored terminology.** Grouped by kind; each is confirmed against
[the material file](../slides/14-data-filtering-dedup-mixing.md) unless noted
as a context-only restoration.

*Tool, dataset, and paper names:*

| Caption | Restored | Confirmed by |
| --- | --- | --- |
| "Chromic Crawl" / "common call" / "comic crawl" (throughout) | Common Crawl | material's own spelling throughout |
| "Brazil parse" / "Brazilia parse" (3:59) | resiliparse | material: "Tools ... trafilatura, resiliparse" |
| "Traflera" (3:59) | trafilatura | material: same list, named alongside resiliparse |
| "resilient pars" (18:47) | resiliparse | same tool, second mention |
| "find PDFs" (3:59) | FinePDFs | material's own section header "FinePDFs" |
| "klm" (9:27), "kenlm" (12:34) | KenLM | material: "KenLM trained on ProofPile" |
| "fast text" / "fastax" (10:13, 11:47, 13:20, 15:37) | fastText | material's own spelling throughout |
| "open math text" (12:34) | OpenMathText | material's own section header "OpenMathText" |
| "latte" (12:34, 13:20 ×2) | LaTeX | material: "contains latex commands" |
| "proof pile" (13:20) | ProofPile | material: "KenLM trained on ProofPile" |
| "GPT3" (14:06) | GPT-3 | material's own spelling |
| "web text" (14:06) | WebText2 | material: "Positives: samples from {Wikipedia, WebText2, Books1, Books2}" |
| "star Reddit posts" (14:06) | high-**karma** Reddit posts | Reddit ranks posts by karma, not stars; corroborated by GPT-3's own Appendix A description of its web-text positives |
| "some books" (14:06) | Books1 and Books2 | material's own list of GPT-3's four positive sources |
| "llama" (14:52) | LLaMA | material's own spelling |
| "the stack" (14:52) | The Stack | material's own spelling |
| "51 from Microsoft" (14:52) | phi-1, from Microsoft | material's own section header "phi-1" |
| "GPD4" (15:37) | GPT-4 | material: phi-1 "later: GPT-4" |
| "Jigsaw toxic uh comments" (16:24) | Jigsaw Toxic Comments | material's own dataset name |
| "LM1B ... the one billion or benchmark" (24:58) | the One Billion Word Benchmark | standard name of the LM1B dataset; "or" is a caption mishearing of "Word" |
| "dduplication" (throughout) | deduplication | consistent typo in the auto-captions |
| "jakard" / "jacard" / "chakard" (throughout) | Jaccard | material's own spelling and definition |
| "minash" / "minhash" (throughout) | MinHash | material's own spelling |
| "sarcastic" (37:29) | stochastic | fits the sentence ("it's stochastic, and I can't really get anything reliable") where "sarcastic" does not |
| "loaded your card" (45:21) | a low Jaccard | fits the LSH discussion of similarity thresholds |
| "ragmix" (1:00:07) | RegMix | material's own section header "Regression-based mixing"; RegMix, arXiv 2407.01492 |
| "durlay" / "dishlay" (1:02:25, 1:03:58) | Dirichlet | material: "often people use some Dirichlet distribution" |
| "Omix" / "OMIX" (1:03:11, 1:07:05) | OLMix | material's data-mixing-methods table lists "OlmixBase (Algorithm 1)" as one of the compared methods, from the second regression-mixing paper the material cites (arXiv 2602.12237) |
| "unimax" (58:34, 59:20) | UniMax | material's own section header "UniMax" |
| "10 code 10 to see" (57:00) | 10 code, 10 CC | material's own mixture-source shorthand, `p = {"Wikipedia": ..., "CC": ..., "GitHub": ...}`, where CC = Common Crawl |
| "unexture assumptions" (57:47) | mixture assumptions | context — the discussion is about mixture, not any "unexture" |
| "downstream your sources" (1:07:51) | downsample your sources | material: "downsample all sources proportionally" |
| "Neotron" (1:11:44) | Nemotron | material's own section header "Nemotron-CC"; here referring to the Nemotron paper's practice of mixing within a source |
| "OMO stuff" (1:11:44) | OLMo | context — "the AI2 folks" mentioned in the same breath is AI2's own model family |
| "open thoughts" (1:14:48) | OpenThoughts | material's own section header "OpenThoughts" |
| "01 came out" (1:14:48) | o1 came out | OpenAI's o1 model, the reasoning-model release the material's OpenThoughts section says motivated the dataset |
| "1.2 two million" (1:14:48, 1:17:06) | 1.2 million | material: "1.2M examples"; and 75k questions × 16 answers = 1.2M |
| "100 uh you know works" (18:00) | 100 WARCs | the chart's own printed title: "N=100 WARCs vs tokens" (promoted in review) |
| "sack exchange" (1:15:35) | StackExchange | material: "e.g., StackExchange, NuminaMath, Chemistry" |
| "num math" (1:15:35) | NuminaMath | same list |
| "QWQ 32B" (1:16:20) | QwQ-32B | material's own spelling |
| "deepseeek R1" (1:16:20) | DeepSeek-R1 | material's own spelling |
| "you you duplicate" (1:17:06) | you deduplicate | context — the pipeline step is deduplication, not duplication |
| "a gentic coding" (1:17:54) | agentic coding | context |
| "Swissmith" (1:17:54) | SWE-smith | material's own section header "SWE-smith" |
| "sweet zero" (1:18:41, 1:21:01, 1:22:34) | SWE-Zero | material's own section header "SWE-Zero" |
| "su task" (1:18:41) | SWE tasks | material: "SWE tasks have heavy dependencies" |
| "docker image" (1:19:27) | Docker image | standard capitalization of the product name |
| "educ execution" / "educa not education execution" (1:19:27) | execution / execution | context — both clauses are about running code, cleaned of stray fragments |
| "Sweenith Smith" (1:20:15) | SWE-smith | second mention, contrasting SWE-Zero's real PRs with SWE-smith's synthetic tasks |
| "open hands scaffold" (1:20:15) | OpenHands scaffold | material: "OpenHands scaffold" |
| "agent hacking" (1:20:15) | git hacking | material: "remove future git commits to prevent 'git hacking' by agent" |
| "30 version" (1:20:15) | SWE-Zero version | context — the passage is describing SWE-Zero's own no-execution instruction variant |
| "set and grap" (1:20:15) | cat and grep | standard Unix read-only commands, fitting the "cannot run Python code" restriction being described |
| "sweet hero" (1:21:01) | SWE-Hero | material's own section header "SWE-Hero" |
| "rebench" (1:21:47, 1:22:34) / "sweet ben rebench" (1:22:34) | SWE-rebench | material's own section header "SWE-rebench" |
| "three zero" (1:22:34) | SWE-Zero | context — same paper discussed two sentences earlier |
| "but zero" (1:22:34) | SWE-Zero | context — "SWE-Zero doesn't care [about execution]" continues the same point |
| "120 of them didn't execute" (1:22:34) | 120,000 of them didn't execute | material: "32K executable tasks + 120K nonexecutable tasks"; the scale mismatch against "32,000 executed" in the same sentence is the caption dropping "thousand" |

*General word restorations from context* (not proper names, but clear
mishearings of ordinary words): "tension" → attention (1:40); "heristic" →
heuristic (throughout); "rec crawling" → re-crawling (4:44); "distribution of
resources" → distribution over sources (50:53); "intention on reasoning"
(1:14:48) → attention on reasoning.

**Unresolved.** None. One passage was left marked by the drafting pass and has
since been resolved in review — see the parent corrections below.

## Restored-name sweep

Every restored proper noun above was grepped against the finished material files.
**All but three appear verbatim in a course material file** — most in
[lecture 14's](../slides/14-data-filtering-dedup-mixing.md), and four in
[lecture 13's](../slides/13-data-sources-datasets.md) because the lecturer refers
back to material the earlier program carries: **One Billion Word Benchmark**,
**Nemotron**, **OLMo**, and Reddit **karma** (lecture 13's WebText entry is "Reddit
outlinks with ≥ 3 karma", which is what confirms "star Reddit posts" → high-karma).

**Three are spoken-only and have no printed counterpart anywhere in the course
material.** They are recorded here so a reader knows they rest on the captions plus
context rather than on a document:

- **Michael Ryan** (18:00), credited with the preliminary filtering-scale
  experiment. The captions are clean here — this is a name that was said, not a
  garble that was resolved.
- **WebOrganizer** (1:11:44), AI2's tool for grouping a web corpus by topic. Spoken
  as "web organizer" in an answer to a student question; the whole passage is
  material the program does not contain.
- **o1** (1:14:48), from the captions' "01 came out". Unambiguous in context — the
  reasoning-model release that motivated OpenThoughts — but not named in the
  program.

## Corrections made in review

Three changes were made after the draft, by the same process that runs the
timestamp, number-inventory and word-ratio checks against the verbatim captions.

1. **A promoted restoration.** At 18:00 the captions read "we're taking a very
   small uh pool so uh 100 uh you know **works**", which the drafting pass left
   marked as unresolved rather than guess a unit. It resolves exactly: the chart
   being described prints its own title, "Method comparison at d512 (157M),
   **N=100 WARCs** vs tokens (all-epoch markers)", recorded in
   [the material file's figure description](../slides/14-data-filtering-dedup-mixing.md#there-is-no-optimal-threshold).
   A WARC is Common Crawl's raw archive file format, which also makes "a tiny
   fraction of Common Crawl" in the same sentence exact rather than vague.
   Restored to "100 WARCs".
2. **A digit join reverted.** The draft's own restoration table correctly recorded
   "1.2 two million" → **1.2 million**, but the body wrote "1.22 million" in three
   places. 1.2M is the figure the material states, and it is confirmed by the
   pipeline's own arithmetic — 75k questions × 16 answers = 1.2M exactly.
   Corrected to 1.2 million throughout.
3. **A corroborated digit restoration, recorded rather than reverted.** At 1:22:34
   the captions read "120 of them didn't execute" against "32,000 of them to
   execute" in the same sentence. The material states the pool as "32K executable
   tasks + 120K nonexecutable tasks", so the captions dropped a "thousand" and the
   draft's "120,000" is right. Noted here because it is a change to a spoken
   number, which normally requires this kind of external corroboration to stand.

---

**[0:05]** Okay, let's get started. Today is the second day on data. Last time we talked about how data doesn't really just fall from the sky — you actually have to think about where it comes from. So in general, the internet consists of live services, and the data on those services have to be either dumped or crawled, and then there's an additional step where you have to process the data. We also talked about various social and societal considerations — terms of service, copyright — you have to either get a license or appeal to fair use. So there's a lot of complexity that goes into data. So today we're

**[0:51]** going to look more at the data pipeline: transforming the data, filtering the data, deduplication, and mixing. And then we're going to top it off by talking a bit about post-training data, in particular how folks are using synthetic data these days. So the first part is going to be mostly about pre-training — that's what you should have in mind. So we talked a little bit about data transformation already, but just as a kind of reminder: raw data doesn't come as text, even if you've scraped something. If you ever look inside Common Crawl, it's not text — it's either HTML, sometimes it could be PDFs, or directories in the case

**[1:40]** of GitHub. So most of the attention on transforming data is dealing with HTML, because most of the web is in HTML, and for this processing there's a lot — this is fairly heuristic. There's removing of boilerplate, like navigation and ads, and extracting content, which is like the main part of the page, and there's some subtleties around what constitutes content. Usually you get rid of the footers and headers, and maybe menus and those things, and you try to extract the content. But you could imagine cases where some of the navigation elements might be helpful to

**[2:26]** learn what web pages look like. So what is content and what is not content is not always clear, and then what do you do about images and tables that are in web pages? So inherently this is a lossy process, because you need to linearize HTML — which is either hierarchical, or visual if you think about the rendered output — and turn it into a sequence of tokens. In particular, tables are a bit tricky to deal with. Simple tables you can render using markdown, but if you have nested tables then that becomes quite challenging — you have to give up at some point, or approximate at some point. So typically HTML to

**[3:12]** text processing is rule-based, and the reason for this is that rule-based processors are very fast, and also — you're not trying to do too much here, you don't need too much intelligence. So rule-based generally works. Now, I think there could be a case for model-based interventions at this point — they have to be very fast, and they have to do something more intelligent. But if you ever look at data, you'll notice there are imperfections in the data, just because any rule-based processing is going to have some failure rate. As we showed last time, the

**[3:59]** accuracy does matter depending on which tool you choose. If you use resiliparse or trafilatura — I guess resiliparse, on these extended DCLM evals, works better than the others. I'll talk also briefly about PDFs. So there's this Hugging Face work which released a dataset called FinePDFs. So PDF files, if you open them up not in a PDF reader, look kind of like this, and this needs to get kind of rendered. So PDFs can be found just like web pages on the

**[4:44]** internet, and Common Crawl does have some PDFs. Generally, Common Crawl focuses on text, but sometimes if you're just given a URL, you might not even know before you fetch it whether it's a PDF or not, if it doesn't have the extension. There's a lot — if you read this blog post, there's a lot of details which I'll spare you. For example, one thing is that many of the PDFs that are in Common Crawl are truncated, because PDFs are big. So that demands that re-crawling is necessary. And then there's a question, once you have the PDF file, how do you actually convert it to text — and some PDFs might be just scans as well, right, so they're essentially images. So there's a bunch of

**[5:32]** tools that this paper tried out, but mostly this involves running OCR using a VLM, and this obviously can be much more expensive than what we were doing before with text. Fortunately, or unfortunately, the PDFs are a very small fraction of the whole internet, but they are very valuable, because generally if you bother to make a PDF that means you probably have something interesting to say, as opposed to a web page. So the quality of a PDF — an average PDF — is generally higher than for an HTML file. There's a lot of cleanup and filtering on PDFs, more so even than web

**[6:20]** pages. A lot of layout information is missing, because in HTML you have various tags, like H1 and P, that give you some semantic information. PDFs are by design all about layout, so they don't necessarily preserve the kind of semantic structure. Okay, so there's a bunch of transformation information that happens at this point. You have text, but you're not done — you're far from done. So the next step is filtering. I'm going to talk somewhat abstractly about what filtering is. Here's a sort of the building block. So suppose you have some target data that

**[7:07]** you want to get. This is usually a small amount of high-quality data, and lots of raw data — this is the fresh shipment of the tokens that you transform from the previous step — and the goal is to find a subset of this raw data that is similar to the target. So this is the general skeleton for filtering, and almost all kinds of filtering fall into this schema. There are many reasons you might want to filter. If you're training an English language model or a German language model, you might want to identify the language and filter out things that don't match that language. The main reason for

**[7:54]** filtering is quality filtering. You want to find things that are high quality, as opposed to low quality. You don't want just spam — you want encyclopedia-type information. And then another application is toxicity filtering. Of course, the internet has plenty of nasty content, and maybe you don't want to train your language model on that. Okay, so for a filtering algorithm, you want some generalization from the target data, right? Because you already have the target data — you don't want to just get the target data back — but you also want it to be extremely fast, because you have to run it on the whole internet. So this could be 100

**[8:40]** trillion tokens worth of data. So generally with filtering you end up with a very small fraction — a single-digits fraction — of your entire data. Okay, so remember the general framework: given the target and raw, you're trying to find a subset. So the general scheme is that you estimate some sort of model based on R and T and derive a scoring function, and then you keep the examples in R based on their score. So typically there are two types of classifiers. One is generative models — so this is basically, you have your target data, you can just

**[9:27]** estimate a model of that data, and remember this has to be cheap, so probably you're not training a big language model. Generally, KenLM basically says, I'm going to train a five-gram model. A more common thing, which I think we're mostly seeing these days, is just to train a classifier. The classifier says, I'm going to predict a positive label for examples that are in T, my target, and negative labels for things that are in raw but not in T. So basically, T are your positive examples, some random subset of R are your negative examples. You maybe balance it, and then you train a

**[10:13]** classifier. And the tool that people generally use is fastText, because it's fast, and it's generally just a linear classifier — bag of words. And then, once you have this model, for every new document you can score it, and you set some appropriate threshold depending on your quality bar, and you keep the examples — sometimes stochastically, sometimes not. Okay, so this is very much a model-based filtering approach. Remember from the last lecture, there are many datasets, generally from a few years ago, that did not use model-based filtering, because they wanted to not bias things too much. These days I think

**[11:01]** basically everyone does some amount of model-based filtering, because unless you are compute-plentiful — in which case you probably don't need to do as much filtering, you can just train on everything — most people are compute-poor, and you have to be very smart about how you're filtering, otherwise you're just wasting flops on low-quality content. Okay, so let me talk through some instantiations of filtering. So remember I mentioned language identification — the goal is just, given a piece of text, detect whether it's of a particular language. So Meta has trained this set of fastText language identification models

**[11:47]** which often you can just use off the shelf. It supports 176 different languages. It's been trained on a bunch of multilingual sites — Wikipedia, remember, has a lot of languages — there's also translation sites, and sites for different languages. So language ID is generally a fairly easy problem, compared to many other tasks that we're dealing with. A simple classifier — if you look at a few words, you can tell that it's Spanish or Japanese. Now, there are subtleties because of code switching in text, and there are some dialects, so I wouldn't say it's an absolutely solved problem, but this is not really the bottleneck for training a good language model.

**[12:34]** Okay, and then once you have your classifier, you just choose some threshold — this is generally fairly heuristic. So another thing you can do with filtering is, suppose you're looking for data of a certain type — let's say you want to get really good at math, so you want to go and find a bunch of math data. So OpenMathText is this paper from 2023 where the goal is to create a large corpus of math text. This pipeline consists of a few steps — it's not training a single classifier. So first you use some rules to filter — does it contain LaTeX commands. They also use KenLM — so this is a generative approach — and

**[13:20]** trained it on ProofPile, which is a known dataset of math, and keep it if the perplexity is below some threshold. And then you also train a fastText classifier to predict whether it's mathematical writing or not, and there are two different thresholds: if it has LaTeX in it, it's a lower bar, but if it doesn't have LaTeX in it, it's a higher bar. And then, as a result, they got 15 billion tokens, which they used to train models. And in the paper they show that this more targeted data collection results in models that are better at math than models

**[14:06]** that were trained on 20 times as much data, which was not filtered in this way. Okay, so quality filtering, again, think about it as a tool — you can define quality however you want, there's no universal notion of quality. If you define quality to be math, then you can go and get math, and you get better at it. So GPT-3 — I think I mentioned this last time — but just to put it in this framework: positive examples are Wikipedia, WebText2 — these are pages that link out from high-karma Reddit posts — and Books1 and Books2, and negatives are generally sampled from the web. They train a linear classifier, and keep documents if the linear classifier scores highly

**[14:52]** enough. In the first LLaMA paper, the positives were pages referenced by Wikipedia, not Wikipedia articles themselves — it's just kind of the same idea. So phi-1, from Microsoft, is interesting — it also falls into this framework. They started with — the raw data is already a Python subset of The Stack, remember, which is all code — and they define a prompt, which is: determine educational value, and

**[15:37]** then they use GPT-4 to classify a 100K subset of R with this prompt, and see if it's — and the ones that are positive are kept and called the target, right? So the target here is actually the output of an expensive classifier, and then you train a cheaper classifier — in their case a random forest, but you could probably have used fastText as well — and then you select data that is classified positive by this classifier. And they also show that, using this dataset, you do much better than if you're just using the

**[16:24]** raw data — the performance goes up, in fewer steps, to a higher value. Okay, so toxicity filtering kind of works the same way. There's this dataset, Jigsaw Toxic Comments, which came from this project where the goal is to help people have better discussions online. So here the data is the Wikipedia talk pages, which — for controversial sites — can get quite heated, and this has been annotated with whether there are any toxic comments on there, and you can similarly define positive and negative examples and train a classifier on that. Okay, so now I think you have the

**[17:12]** tools, or some examples and inspiration. You can identify a particular type of data that you want, you build a classifier for it, and then you can go filter Common Crawl for that. I want to talk about one subtlety here, which is that the notion of what you want for data actually depends on what model you want to train. In particular, it depends on the number of tokens you're training on, and so there's no optimal threshold. So before, when I said the classifier gives you a score — you can't say 0.9 and say that's the best, because it depends on what you want to do. Intuitively, if you are going to train for a longer period of time, then you can tolerate

**[18:00]** lower-quality data. If you're training for shorter, then you want higher-quality data, in general. And, of course, if you could wave a magic wand, if you're training for longer you'd want more high-quality data — but that's not an option you're given, the data pool is what it is. So here's kind of a preliminary experiment that Michael Ryan did, and this plot shows, for a 157-million-parameter model, we're taking a very small pool — 100 WARCs — which is a tiny fraction of Common Crawl, and we're look

**[18:47]** training, for more, over time. So let's take a look at the blue curve — the blue curve is DCLM. The loss starts here and then it comes down, and each of these lines is when you epoch over the data. So this is one epoch, and then the next blue is the second epoch, and so on and so forth. So there's not that much data, so eventually you have to repeat your data, and the loss continues going down, because the second time you look at the data you're still learning things, and at some point you start to overfit. Whereas if you look at resiliparse — this is basically no filtering — here it's much

**[19:35]** worse in the beginning. But as you continue, at some point you also epoch, and it starts to go down more slowly. So you kind of see this trend, where high-quality data is better in this regime where you're not epoching, but once you get to lots and lots of tokens, high-quality data is no longer that great. And, of course, with high-quality data, you wouldn't even want to go into this regime, because you're overfitting — you'll probably stop here. But even at this point, this is worse than if you had trained for longer using low-quality data.

Yeah?

*[Question from the floor: Just a question about computing the metrics — this is not totally about the graph, but]*

**[20:21]** *[continued from the floor: when each of those dots correspond with one training run, right?]*

Yes.

*[Question from the floor: Do you ever need to do the confidence interval when you're doing these kinds of pre-training experiments, or is it kind of enough just to do it one time?]*

So the question is: each of these points is a single training run, and should you do it multiple times and get confidence intervals? Ideally that would be good practice. Often you'll see in these papers that these are kind of scarce, because each training run is fairly expensive. But in reality, when we have done these experiments, it generally tends to be stable, I would say, at least for pre-training.

**[21:07]** Yeah.

*[Question from the floor: Yeah — so you mentioned training for longer and high-quality data, that's not like a combination that we're considering, but if we are training for longer on this high-quality data, would it have diminishing returns compared to training for longer with quality data?]*

Yeah. So the question is: if you were able to get higher-quality data and you train for longer, would it still have diminishing returns? Every dataset is going to have diminishing returns eventually — that's finite — but it would probably be here, it would just be down here, and just keep on going down.

**[21:53]** Okay, great, let's move on. So, summary: filtering is pretty critical for building a good model, especially when you're compute-limited, like most of us. Technically, if you have infinite compute you don't need to filter, and you can train on everything, and it'll be a giant model, but realistically everyone has to filter. So the recipe here is: figure out what good data looks like, and then you can train a classifier, and that will extrapolate to the rest of your data — what the good data looks like. You can either find it by saying, "Aha, there's this dataset out there that I really like, and I just want more of it," or you can craft a prompt to a

**[22:39]** language model, and then use that to construct — to do a sort of a preliminary filter of a large pool — and then use that good-quality data to train a smaller classifier and extrapolate to everyone else. Okay, next I'm going to talk about deduplication. So at this point we have filtered our dataset, so we only have what we deem to be high-quality data. But often data still has duplicates in it, and there's two types of duplicates. There's exact duplicates — this happens when, for example, if you look at these mirror sites, the whole

**[23:26]** point of a mirror is that it's a duplicate, and sometimes the web crawler isn't smart enough to know that this mirror is exactly this mirror, so it just crawls many sites and you'll get this exact same content. Also, when you fork a repo, that is also a duplicate — even if you make changes, probably you're making changes to a few files, so 99% of that repo might be the same. So duplication is abundant, and, yeah, sometimes there's actually near duplicates, which are not mirrors or derivatives but they just happen to be the same text, differing by a few tokens. Usually, probably

**[24:11]** this came from — it could be copying, or it came from a different common source. So here's some examples of near duplicates. So terms of service and licenses — I guess the MIT license probably shows up on a lot of places, and in many cases it is an exact copy, but only of that license, unless someone made a typo when they copied it. But the rest of the page that the license might be part of might be different. And we talked about how a lot of websites have the same headers and footers — those are also duplicates. There's

**[24:58]** also cases where, for whatever reason, — this is from LM1B, the One Billion Word Benchmark — there's like these articles where you just have typographic differences, like there's one version with a comma and one version without a comma. I don't really know why, but that happens. And sometimes you see these templates where — this is like some, probably, low-quality content — it's essentially an ad of some sort, and someone just templatized it, replacing "Canada" with "USA." So if you train on this data, but you are just training on different variations of this with different entities, this is going to be wasting

**[25:43]** your GPUs. There's more extreme cases — this is why it's good to look at your data. So this audit of the C4 dataset found this product description 61,000 times in the dataset. And if you trace back — this is really bizarre, I don't know why — this is like some description of this gas mask, and it just showed up in this dataset, in Common Crawl, 61,000 times. So, yeah, the web is weird. So why deduplicate? The first clear idea is: you want to train more efficiently. Deduplication reduces your dataset size without really losing information, because you're

**[26:30]** just removing duplicates. And it also has this benefit of avoiding memorization, which this paper talks about — for example, if you have some copyrighted content that's duplicated a lot, then if you train on it you memorize it, and also there are privacy concerns. But mostly, I think, it's just to make sure that you're not wasting flops. Related to deduplication, that's also decontamination, which is arguably even more important, and it's the same sort of deal — you want to make sure that your test set is not in your training set. Okay, so how do we dedup? So here's

**[27:18]** the design space to think about. So first of all, what are the items that you're deduping? Do you do it at the sentence level, the paragraph level, or the document level? How do you determine a match — is it an exact match, is it the existence of a common subitem, is it the fraction of common subitems, for near deduplication? And then, once you've found a duplicate between two pages, what do you do — do you remove all the instances, or do you remove all but one? Okay, and the key algorithmic challenge is that deduplication is fundamentally about comparing items to other items. And normally, if you're doing filtering, this

**[28:03]** is about an individual item — is this item good or not — and this can be parallelized, it's sort of linear time, which is good. And even in linear time, we're trying to make it fast by having rule-based or very small models. But deduplication, clearly, you can't do the n-squared thing where you compare everything to everything. You need linear-time algorithms to scale, especially at this web scale. So typically the deduplication literature uses hash functions to get around this. So we'll develop some of these ideas — these are quite nice ideas, thanks to the algorithms community. I think everyone knows what a hash function is.

**[28:49]** It takes some sort of value, like a string, and maps it into something like a string or integer. And a hash value is much smaller than the item. Hash collisions are when two distinct items map to the same hash. Okay, so when you look at hash functions, there are really fancy hash functions which are cryptographic in nature — these are collision-resistant, used in cryptography, and Bitcoin, and things like that. And then there are these faster ones, which are used for hash tables, where hash collisions aren't the end of the world, and so we'll be using these. Okay, so take a string and map it to some value — that's what

**[29:36]** hash functions do. Okay, so exact deduplication is conceptually very simple — you take a string, you see if there's an exact match, and you remove all but one. So here you have a bunch of elements, and you hash them, and you dedup. So exact deduplication is very nice, it's very clear what's happening, but this isn't really good enough for the messy web data, because often you have these near duplicates. And one note is that there are many ways to have written this — this is written in a sort of MapReduce

**[30:24]** style way, which makes it more easily parallelizable and scalable. So C4 — this paper from the T5 paper, which processed Common Crawl — did exact deduplication. The items they operate on were three-sentence spans, and they do an exact match, and remove all but one. So, if you're paying attention, you realize that there's something a bit strange about this, because you're looking at three-sentence spans, and if you find two documents with a three-sentence span, you remove all but one. That means you're

**[31:10]** just going to rip out three sentences from that document, which is a little bit strange, because it breaks the coherence, but that's what they did. Okay, so that's exact deduplication. So how do we do near deduplication? First of all, we have to define what "approximate match" means here. So to do that, we're going to define this thing called Jaccard similarity, which is a fairly standard notion. Jaccard of two sets is basically the size of the intersection over the size of the union. So, given these two sets — 1, 2, 3, 4 and 1, 2, 3, 5 — when you compute Jaccard, you take the

**[31:58]** intersection — that's 1, 2, 3 — you take the union — that's 1, 2, 3, 4, 5 — and you divide, and the Jaccard is 0.6. So the Jaccard is a number between 0 and 1 — zero means they're disjoint, one means they're identical. So, a fairly natural notion, and we're going to say that two documents are near duplicates if their Jaccard similarity is above some threshold, let's say 0.99. So now the question is: how do you find near duplicates in linear time? Fortunately, this is a solved algorithms question, and the answer is to use MinHash. So

**[32:45]** well, okay, so the first step is to use MinHash — that's not the final answer. MinHash is a random hash function, so that the probability of a hash collision is exactly the Jaccard of A and B. So this is a very nice property, because hashing is good for making things linear time, Jaccard is the metric that we want, and we're sort of connecting these two — right now, in expectation. So what's interesting is that, normally, you want hash functions defined so that distinct elements are hashing to different things — you don't want collisions. But here, you actually want collisions — not arbitrary

**[33:33]** collisions, but you want to control the collisions in a certain way, to align with a similarity — similar things you want to collide more than disparate things. So the MinHash is essentially — okay, here's the MinHash, it's fairly simple. You take the set, and you hash every element in that set, and you take the minimum element. If you're seeing this for the first time, it may seem a little bit strange — why are you taking the min? You can take a max too, it doesn't really matter, it's just a way to break ties.

**[34:18]** So here's the picture you should have in your mind. If you have — remember, A contains 1, 2, 3, 4, and B contains 1, 2, 3, 5 — this is a characteristic matrix representation of these two sets, right? And what the random hash function is doing is inducing a permutation over the items — so it might be 4, 3, 1, 5, 2, or something else. And then you look at which item is first, according to this permutation, in A, and which item is first in B. So each item has the same probability of

**[35:04]** being first. So, if the random hash function puts one first, then first in A will be equal to first in B. And if it's two first, then it's the same — if it's three first, it's also the same. But if the random permutation puts four first, then the first element in A is going to be different from the first element in B, and same with five. So look at this representation — basically the random hash function elects one of these rows to be first, and then the min is just

**[35:50]** telling me — the min of A equal to the min of B is basically telling me whether that row is the same. Okay, so that is, I guess, the proof of why MinHash — why this property holds — which is that, in expectation over your random choice of hash functions, the probability of collision is the similarity metric you want. Okay, so let's just check this in code — I'm going to generate 100 different

**[36:36]** hash functions — each hash function is given by a seed — and I just check whether the MinHash of A is equal to the MinHash of B, and get the estimated Jaccard. So I'm going to just look at the fraction of matches, and you get 0.6. And the key thing with a MinHash is that I don't have to do the n-squared thing — I can compute the MinHash of A, and the MinHash of B, and the MinHash of a different set, and I just look for collisions. Okay, any questions so far about MinHash?

**[37:29]** So now we can hash our items, but we're not done yet, because a collision doesn't tell us that Jaccard is above some threshold, which is what we want. We want to find A and B such that Jaccard is greater than 0.99. All we've done is said: okay, well, if we got a collision, the probability of two things colliding is the Jaccard. But that's not really that useful by itself — it's stochastic, and I can't really get anything reliable here. So the next idea is this thing called locality-sensitive hashing, which essentially solves this problem. This is a very classic idea in theoretical computer science, and it's quite nice. So the idea here is that

**[38:20]** we have A and B colliding with probability equal to Jaccard. So it is true that more similar items will collide more often, but it's very stochastic, right? The variance is quite large here. So our goal is to have A and B collide if the Jaccard is greater than some threshold. So, in some sense, we have to sharpen these probabilities somehow — the probability can't just be literally equal to the Jaccard. So the solution is to use more hash functions, and these hash functions are going to be independent. This part gets a bit technical, but we'll walk through it.

**[39:08]** So you break the n hash functions into b bands of r hash functions. So if you have 12 hash functions, you have three bands, each band has r — four — hash functions. So there's a band here, H1 through H4; here's a second band, H5 through H8; third band, H9 through H12. And so what we're going to try to do — so each hash function gives us either collision or not collision — and for every hash function there's a probability of colliding. So the key here is that we want to say

**[39:53]** compute the probability that A and B collide. We say that A and B collide if, for some band, all of its hash functions return the same value. So if H5 and H6 and H7 and H8 all return the same value — so, sorry, if H5 of A equals H5 of B, and H6 of A equals H6 of B, and so on — that means this band is triggered, and then I would say they collide. Or if this band — if all of these agree

**[40:38]** then I say it's collision — for these two, but you don't have to have all the hash functions return the same value. So there's sort of this and-or structure here that's doing the lifting, and we'll see why this works. So now let's say you have a particular Jaccard of A and B — what is the probability that A and B collide, according to this definition? We can calculate this. So let's say that this Jaccard similarity is 0.8.

**[41:26]** And we have five bands, and each band has 10 hash functions. So then we can say: what is the probability of a fixed band matching? That's 0.8 to the r, because each band has r hash functions, so the probability that they all have to match is 0.8 to the r. So the probability of a fixed band matching is generally quite low — it's exponential in r. And then the probability of collision is the probability that some band matches,

**[42:11]** and that's going to be one minus the probability of — so this is the probability of a band not matching, and if you raise it to the B — there are B bands — this is the probability that all of them don't match, and then one minus that is the probability that some match. So the probability of collision — you'd expect it to be higher, because each try gets a chance to match. So if you plot this, this is what it looks like. So you have, on the x-axis, the similarity here,

**[42:59]** and we were looking at 0.8, and you look at the probability of a collision here. So we would expect that if the similarity is zero, then it should be zero — if it's one, it's one. But notice that this is kind of an interesting S-shape, which is nice, because this is what we wanted to sharpen. We wanted to basically say: if the similarity is below some threshold, then we want the probability of a match to be as low as possible, and if the similarity is above some threshold, we want it to be as high as possible. So we're trying to get this to be like a phase transition. And if you plot this function as a function of similarity,

**[43:45]** that's what you get. Okay, so let's just look at some examples. So, concretely, we have similarities 0.7 to 0.98, and we're going to look at how — b = 10, r = 10. So we look at the collision probability according to our definition, and we see that this gives us a range of 0.25 to 1.

**[44:34]** So if you were to set a threshold, this is not bad — we can set it here, and, with some probability, there are still false positives, because even if these are below a threshold, let's say 0.9, these might still collide, but we can filter them out if we want. But the ones above are mostly kept. So what happens when you move — increasing r, so remember r is the number of hash functions within a bucket — when you increase r, the threshold sharpens and moves the

**[45:21]** curve to the right — everything becomes harder to match, right, because you have more hash functions inside a bucket, so it sharpens that exponent. So the probabilities you see here are sharper. And now everything moves to the right, and now it's fairly unlikely that if something has a low Jaccard, you're going to match it. So before it was like 0.25, and now it's 0.08. So we're sharpening the probabilities, and then increasing B has the effect of moving the curve to the left — it makes it easier to match. B is the number of bands, and some band has to match — if you have more bands, then there are more chances of matching. So if you look at this, then, before

**[46:09]** and after — so now, before, this 0.9 was only 0.72, and now it's 0.92. And, of course, this also increases, but not by that much. So you can drive this phase transition to be as sharp as you want by increasing B and R — but, of course, if you increase B and R too much, then it can be more expensive. Okay, so let's look at a more real-world setting. In this paper on deduplication, they had 20 bands, and r is 450. So, to give you an idea of the magnitude of these numbers,

**[46:58]** and, in general, this phase transition happens at a threshold which is one over b, raised to the power of one over r. So if you want to filter based on Jaccard greater than 0.9, then you basically have to set b and r such that this is 0.9, and then, by changing b and r, you can make that phase transition sharper. So, remember, the probability that a fixed band matches is one over b. If you're at this threshold, then the probability that A and B collide is approximately

**[47:45]** a constant. So the probability of collision is 0.64. And this makes sense — basically, if you have a phase transition, the center of that phase transition is like 0.64, and everything below it should go to zero as b and r increase, and everything above it should go to one as b and r increase. Any questions about LSH? So this method is called MinHash LSH, because LSH really works for any hash functions. For deduplication of language model processing, we're using the MinHash, which approximates the Jaccard. And so that basically is the

**[48:34]** one you should use here. Okay, questions about dedup? So one note about dedup is that, often, as we'll see, datasets are coming in, and sometimes the deduplication will happen within a dataset, but you actually have to do deduplication across your entire dataset, because often datasets can be redundant with each other. Sometimes that's not done, but it should be.

**[49:19]** Okay, let's go on to the next topic, which is data mixing. So far we've transformed our data from raw HTML or PDFs into text, we've filtered for high quality, we've deduped. So we have a smaller set of high-quality documents, and this generally happens for a given data source, but language models are trained on multiple data sources. So, in Marin — currently this website tracks the different data sources that the next model

**[50:05]** will be trained on — and you can see that there's a bunch of things, there's some Nemotron, there's FinePDFs, which we talked about, there's some institutional books, and some code, and all these things. So you have all these different sources, and the question is: how do you combine these? If you look at an older paper, The Pile, this is the set of sources that they had at that time, and they essentially assign a particular weight to each component. So basically

**[50:53]** you have a distribution over sources. And so where does that weight come from? More cut and dried: let's say you have three sources, you want to find a data mixture, and a data mixture is just a distribution over your sources. So if you're just thinking about this problem from first principles — well, not from first principles, but from what you might do — vibes, which is, I guess, the opposite of first principles — is you just manually set it based on some intuition, which is more often than you might think what people do. So I think this is

**[51:40]** definitely fairly vibes-based. And even more recent papers — you just look at it, maybe you use some method, and then you just tweak things. You can also do uniform sampling, which is you just put a uniform distribution over your sources, and every chunk you get, you just sample a source. You can also do proportional mixing, which is that you sample proportional to the number of tokens in a source — so if you have a dataset with more tokens, then you put a higher weight. So this is also generally a rational thing to do, but you might worry

**[52:26]** that if you have a huge low-quality dataset, that's going to eat up a lot of your tokens, so this doesn't seem quite optimal either. Intuitively, you should upweight your higher-quality sources. But there are two things that are important to keep in mind. One is that you do want to ensure some diversity, right — so often sources are incomparable, for literature, code, and papers. You can't really say that this paper is higher quality than this code, because they're just incomparable objects, maybe, and if you want your language model to do well, you don't want to just put all your mass on

**[53:11]** just papers. The second thing, which we'll talk a bit more about, is that each source is actually finite, so if you put too much weight on a small source, then you essentially run out of that source, and you need to epoch over it — meaning that you keep on training on literally the same tokens — and this is going to be bad, for reasons we'll see. Okay, so let's try to unpack this a little bit more concretely, this last point. So imagine you just have two sources — a low-quality source, you have 10 trillion tokens, and a high-quality source that has 10 billion tokens. Generally, high-quality sources are smaller.

**[53:57]** So if you just do a naive data mixture, let's just do uniform — half on high, half on low — and I'm going to train for one trillion tokens. So when I say train for one trillion tokens, that doesn't mean unique tokens, I just mean train for that many steps, divided by batch size, essentially, or times batch size. Okay, so then let's look at this, which is the number of times a particular element in the low-quality dataset was trained on — that's the number of epochs a particular data point was trained on. So p(low) times train tokens is the

**[54:43]** number of times I want to request tokens from this low-quality source, divided by the total number of low-quality tokens. So this is less than one, so basically that means 5% of the tokens in this low-quality dataset I'm even going to touch and train on once. And then, if you look at high quality, you'll see that the number of epochs is 50, right, because I only have 10 billion tokens here, and if I'm half-half, that means 500 billion tokens — I need to train up 500

**[55:28]** billion tokens of high-quality data. But I don't have 500 billion tokens of data, so I have to repeat each data point 50 times. Okay, so this is actually really important, and some big model training runs have kind of messed this up — you can't just naively look at the datasets, define a distribution, and then sample, because if you just look at the quality, if you look at the data, you're sort of missing how many data points there are. Does this make sense? Maybe pause for questions — this is a kind of important point.

**[56:14]** Okay, yeah?

*[Question from the floor: I guess, why do we need it in the first place? Doesn't it achieve a comparable performance level with fewer, I don't know?]*

So the question is: why do you need 50 epochs? And I guess the point is that you don't need 50 epochs — best case, it's wasting compute; worst case, you're overfitting. But the reason it's 50 is that if you just go in and define this data mixture, then you're going to end up doing 50 epochs without realizing it, unless

**[57:00]** you're paying close attention. So this is basically the lesson: look at how many epochs you're actually doing on your data.

Yeah?

*[Question from the floor: How do the mixtures get expressed during training? So, when you switch every step, do you train, like, steps of 10, saying, oh, I want to do 10 code, 10 CC, because I feel like — is there also any intuition behind what decisions are made there?]*

Yeah, good question. So the question is: how, when you train, do the mixtures get kind of realized? And so, in general, you sample — so you want to fill a batch, right — so you can sample, essentially, for each

**[57:47]** element which mixture component it comes from, and you fill up a batch with sequences from that mixture component. Usually every sequence comes from one mixture component — you're not sampling per token.

*[Question from the floor: I guess, so each batch should be mixed — so I guess the mixture assumptions are at the batch level, and then you just train for that? It's not like one step is—]*

Yeah, in general you want to reduce variance, you want to train from multiple — a batch should have some mixture in it. Yeah. Okay, so this idea — I mean, this problem has been noticed quite a long time ago — and so, in this

**[58:34]** paper they introduced UniMax, and in that case it was — they were training multilingual models, and it was very clear that some languages were very low-resource, so you had to do something about this. So in this paper they noticed that, before, some works would essentially take the proportional mixing and raise it to some power, to kind of flatten out the distribution. But the idea in this paper is: let's be a bit more explicit about that — let's sample the sources uniformly, but with a hard cap on the number of epochs. Okay,

**[59:20]** so this is the key idea here: you say, I'm only going to take 20 epochs over a particular data source. If you've done that, then, too bad, you don't get any more tokens — you move on to the next thing. So this is sort of like a safety net, in some sense. In particular, if you look at the probability assigned to a source, times the number of training tokens, that should be less than or equal to the cap. And there's a simple procedure for actually determining the mixture, subject to this constraint. Okay, so let's switch gears a little bit and talk about how we define the

**[1:00:07]** mixture in the first place, right? Because if you have 50 sources, there are 50 numbers you have to fill. And we noticed that proportional mixing isn't really enough — maybe you can try to estimate some quality metric, and then sample proportional to that, but that's also heuristic. So the most principled methods, and the easiest to understand what's happening, are these regression-based mixing methods — RegMix and other papers like this — and the idea here is relatively simple. So first, let's say you're trying to train a large-scale model, so you go to a small scale,

**[1:00:52]** let's say — I guess let's say 300 million parameters — and you try different data mixtures. Let's say you have three components, you try different mixtures, and then you train these small models — you train a swarm of small models. Each model gives you a loss on some target metric — it could be some downstream evals, it could be perplexity, whatever you want. So you use these data points to basically fit a regression, where the regression model maps these inputs — which is the data mixture weights — to the loss. So now you have basically a very cheap

**[1:01:38]** model that tells you, if I were to train on this mixture, what loss would I get. Then you can essentially optimize — sorry, optimize that function for the optimal data mixture, and then that is the data mixture you use to train the large-scale model. So that's the kind of simple procedure, and you'll notice some similarities with scaling laws, where you're trying to do some cheap computations to figure out some answers, and then scale up. So there are a few design decisions here. One is: what are the mixtures that

**[1:02:25]** you're trying out here — you need a distribution over distributions here, so often people use some Dirichlet distribution. You also have to define the regression method — people have tried linear models, or boosted decision trees. You have to figure out the target here — often this is based on downstream evals. So we have to be very careful not to overfit, right, because we're doing pre-training, and supposedly pre-training is supposed to be training this general-purpose model, and we're not trying to fit some downstream evals. And if you're not careful — for example, if you have a bunch of code evals, then, well, guess what, you're going to upweight all the code data. That's not rocket

**[1:03:11]** science, and if you then go and say, I want to generate some poetry, you might realize that you've overfit. So you have to be very careful about this. Proportional mixing and uniform mixing don't have this problem, because there are no downstream evals involved. And the final thing is: how much of a difference is there between small and large scale — this is a cost-and-accuracy trade-off. Of course, if you train really, really small models, this might not be representative, but if you train large models to do this — well, there's no point in doing any of this, you're basically doing hyperparameter tuning at the largest scale, which is too expensive. So this OLMix paper actually has this nice table which shows

**[1:03:58]** you, for a number of different methods that all fall into this kind of framework. The size of the proxy models — they're generally fairly tens, or tens of millions of parameters. How many points, or mixtures, do you sample — m here being the number of parameters in your mixture, sorry, the number of domains, rather — and you can use Dirichlet, you can use exponential. And then the regression model — you can fit a log-linear model, which tends to work pretty well. And then you

**[1:04:44]** can use different ways to solve this optimization problem. Okay, so there are sort of two leaps of faith here that you have to be very careful of. One is that the regression model was trained on a bunch of these small proxy runs, and you're optimizing this — so the hope is that at your minimizer, your optimal data mixture, that regression model is still accurate. And this is a little bit — you have to be very careful, because if you, let's say, sample a random mixture, you'll probably be fine, because of classic generalization — you sample a mixture, and you fit a function, it

**[1:05:32]** should be able to predict in distribution. But when you're optimizing, you're essentially trying to go to, potentially, the extremes, where you might not have as much coverage — so that's one thing to be careful of. And the other thing is that optimal data mixtures — you just hope that they transfer from small scale to large scale, and this, in general, at least at the scales that the open community works with, seems to tend to be true, or not blatantly false. But clearly there are scale-dependent effects. If you just think about the previous slide, when we were talking about filtering — if you're training for many more tokens, then probably low-quality data is okay. So clearly the optimum

**[1:06:19]** is not the same, but you just kind of hope for the best here. Okay, so hold on — there's one scale-dependent effect that we already talked about, but that we need to address here, which is that — imagine, so we have a lot of low-quality data, 10 trillion tokens, 10 billion tokens of high-quality data — and if we train a small model on low token counts, remember, what happens is that you might — this is a made-up example — you might put a lot of mass on your high-quality data, because at low token counts, you're not epoching. So it's like, oh, wow,

**[1:07:05]** Wikipedia is so great, let's just train on Wikipedia. But if you then go and train your large model on this mixture, then you're going to end up epoching a lot on this high-quality data, and you'll overfit, and that's definitely bad. So one way to fix this, which is done in the OLMix paper, is you can just cap the number of epochs — that's one solution. There's another solution here, which goes by probably multiple names, but one name for it is simulated epoching. And the principle here is: make your small scale look like

**[1:07:51]** your large scale. This is kind of a general theme for the course — we saw it when Tatsu talked about muP, you parameterize your model so that your hyperparameters transfer. And so the same principle applies here. And the problem with doing this kind of naive scaling is that you can't be not-epoching at small scale, and then epoching at large scale, because those are qualitatively different operations on your dataset. So the way you would handle this is you downsample your sources proportionally. So, if your small runs were

**[1:08:37]** 10 billion tokens, and your big run were one trillion tokens, then you have a one-in-100 kind of downsampling, and then you basically use the downsampled mixture. And so the idea is that, if you have the downsampled data, then this solution is not going to look good, because you don't get to just train on all of Wikipedia — you're going to train on some minuscule fraction of Wikipedia, and you'll realize, oh, actually, I'm going to be epoching a lot, and that's going to have really bad loss — which means that, in the optimization, you're going to look for more balanced

**[1:09:24]** approaches. So you're kind of simulating the data scarcity that you would get in your high-token regime, at a low scale. Okay, so to summarize this section: the problem is, how do you weigh different sources? You have Wikipedia, you have your Common Crawl, you have your code, you have some math data that you scrape from somewhere — how do you balance them? Regression-based mixing is a nice framework, I think, to think about things — you estimate a function form that maps your mixture weights to a loss, at small scale, optimize, and then generalize to large scale. And you just have to be

**[1:10:11]** very careful with this, because of epoching and overfitting kind of issues, and you have to either use capped epoching or simulated epoching. So one lesson here is that, if you're trying to optimize anything, you have to be very careful, because you are in danger of optimizing the wrong thing. Okay, any questions about data mixing, before I move on?

*[Question from the floor: When you downsample — assume something is too small — you can't really generalize as well?]*

So the question is: if you're downsampling, is there a risk maybe you

**[1:10:57]** get too small a dataset? Yeah, absolutely. So, I think — right — you might get into a case where you just have so few tokens. I think then what will happen is that your optimum will just put very small mass on that, and when you scale up, maybe you get the right set of tokens. So, in principle, it should work out, but there might be rounding errors, where you end up, instead of training once on this dataset, training zero times, by accident. But you can sort of work around these by just — think, you always train once on this data. Yeah.

**[1:11:44]** Okay, so, yeah?

*[Question from the floor: Within data mixing, do you ever — because I guess these sources just happen to align with different topics — do you ever take a diverse dataset and then try to do data mixing within that?]*

Yeah. So the question is: do you apply data mixing to different domains, or even within a dataset? That's actually — I forgot to mention that, so I'm glad you brought it up. So, in the Nemotron paper, and actually in some of the OLMo stuff as well, you basically think about — let's say I give you just Common Crawl, right — you can actually break this up, so you can group by domain. The AI2 folks have this thing called the web organizer, so

**[1:12:29]** you can group into topics, but you can also quality-filter. So then you have this two-dimensional grid, where you have domains and you have quality, and each of these cells is something that you would data-mix. So that's one automatic way to determine the domains, and then, on top of that, you add extra sources that people hand you. Okay, so I'm going to quickly go through post-training data here. Up until now, this is basically pre-training, or maybe mid-training, data — it's generally fairly task-agnostic, except for the RegMix stuff, where you're

**[1:13:15]** trying to minimize the loss, but still the data itself is relatively task-agnostic, and you're trying to develop basic skills. So when you look at post-training, a lot of the data becomes very task-dependent, and I'm not going to do a comprehensive review — I just want to point out some interesting post-training datasets that have been recently released, in particular for coding, since that's of great interest these days. So the general recipe is: you define a set of environments — in the case of code, this might be GitHub repos — you define a set of tasks, or

**[1:14:03]** prompts, and then you collect responses from a strong model, or teacher. So, implicitly, at least in the open community, almost all the post-training data — most of it — is synthetically generated. You can also replace the strong model with a human, but that is slow and costs a lot of money. But if you're at the frontier — I guess a few years ago you kind of had to go and pay a lot of people a lot of money to give you responses — and nowadays, even at the frontier, you can do sort of hybrid human-AI things. But anyway, the point is that

**[1:14:48]** there's some teacher out there that's giving you responses. Okay, so I'm just going to talk through a few works here. One is this work called OpenThoughts. This idea came around — it was motivated by, when o1 came out, there was a lot of attention on reasoning, mostly for math and science — how do we get really good post-training datasets for this? And they eventually came out with 1.2 million examples, using this teacher model. So the steps are: first of all, there are a bunch of — so what

**[1:15:35]** are the environments and the tasks and questions. There are a lot of different sources that they drew from — there are human sources, like StackExchange or NuminaMath, and then there are some synthetic sources as well. These are just a subset of the coding ones, but there's also more for math and chemistry. This is a big project that involved a lot of different contributions from many people. You can see that some of these are like coding exercises, some of these are more realistic things that show up on — I guess code golf is not realistic, but there are some more realistic

**[1:16:20]** examples here — so code review, this is probably more realistic. And they did a fairly comprehensive analysis of, given this dataset, how do you generate responses. They show that having a few sources was actually good, but rather than trying to use all sources, sampling multiple generations is helpful — like 16. One thing that was interesting is that having better models aren't necessarily better teachers — so, for example, QwQ-32B, which is now a very old and small model, was a better teacher than DeepSeek-R1, which was, at the time, probably one of the strongest open

**[1:17:06]** models. They also found that basic answer filtering wasn't helpful. And so the entire pipeline looks something like this: you have a bunch of these sources going in, and then you deduplicate, you randomly sample — I guess, downsample — the questions, and you generate multiple answers, and that gets into the final dataset. So the 1.2 million is examples, but divided by 16 gives you the number of actual questions. Okay, so there's another — that was for math and science, and some code. I think, more recently,

**[1:17:54]** there's been a lot of interest in developing, in particular, agentic coding models — not just a model that can generate some code, but actually a model that can do software development. So this SWE-smith paper had the idea that, given a repository, you use a language model to automatically generate tasks. So they have an agent that takes a repository and actually makes it usable — like, installing dependencies, and so on — and then you generate tasks, which generally means you modify the code in some way, maybe introduce some bugs, and those get verified, and you

**[1:18:41]** get some task instances. So these are synthetic tasks, but you get 50K of them, which, at the time — which was last year — was quite large. So, since then, there's been a bunch of other work, which I'll mention. So there's this SWE-Zero paper, from Nvidia, which had this kind of — actually, okay, I'll talk about that. So this is an interesting idea, where the observation was that, unlike math, SWE tasks have a lot of heavy dependencies. Most GitHub repos

**[1:19:27]** don't even run, and you have to install all these dependencies, and they're out of date, especially if you roll back to, like, when a PR was — it's just kind of a mess, and this is just a nightmare. And so they were thinking about how can we actually get a dataset across all the repos, and rather than having a repo-specific Docker image, they noticed that the models are actually good enough that they can solve a lot of these tasks without execution feedback. So, if you look at some of these models — if you were allowing execution, you get like 80; if you don't allow execution, you get almost 70. So that's

**[1:20:15]** not bad, for not being able to execute code. So, somehow these models have some internal semantics of code. So they were able to generate 300,000 agent trajectories — all of these are real GitHub PRs, so they're realistic, unlike the SWE-smith examples. They use the OpenHands scaffold, and did some — there are a lot of details, like how to prevent git hacking. So, normally, you hand an agent this instruction: you explore, you test, you implement — and their SWE-Zero version was basically saying, you cannot run Python code, you can only do things like cat and grep, and all these

**[1:21:01]** basic operations. So then they distilled from a big coding model, and filtered, because sometimes a coding model will ignore these instructions and still try to execute anyway. And then — oh, they also had 13K agent trajectories that do require execution feedback. So they train models where they first fine-tune on these SWE-Zero examples, and then fine-tune again on these SWE-Hero examples, and they were able to — I guess the frontier is still pretty high up, but they were able to make some progress here. I'll quickly go through SWE-rebench,

**[1:21:47]** which was essentially another attempt to grab tons and tons of PRs. So, all of these kind of look like: get a bunch of GitHub repos, try to install the repo — most of them probably fail — try harder, and then you use a language model to give you the responses. So, this one actually just came out today: you can take the SWE-Zero idea, and now scale up to 12 million agent trajectories. The nice benefit of SWE-Zero is that it's almost — it's very lightweight. Here they use the SWE-

**[1:22:34]** rebench tasks. So, in this one, remember, they were trying to get things to execute — they only got 32,000 of them to execute, and 120,000 of them didn't execute. So, SWE-Zero doesn't care — you can use all of them. This is a very small model, because now this dataset is quite large. And so I think, as you can see, the datasets are getting more and more sophisticated — you go from environment-free things like math, to now coding, and now the coding datasets are growing quite a bit. But the general idea is that you have these prompts, which

**[1:23:20]** there are kind of trade-offs — you can have fully synthetic, you can have semi-synthetic, where you have a real environment and synthetic tasks, or you can have real — and the responses generally come from capable models, but they also need to be good teachers. Code environments are a pain, and there's a lot of filtering and other details which we don't have time for. Okay, to summarize this lecture: we talked about filtering — define what good looks like, and then train a lightweight classifier, go over your web crawl, and you can get a small subset that matches what you're looking for. Deduplication is important, to avoid overfitting and to save flops. Mixing —

**[1:24:05]** try mixtures at small scale, extrapolate to large scale. And then we looked at some post-training data. I will say, though, that a lot of the data work can be very grungy — it's very domain-specific, and requires looking at concrete examples to make these high-quality datasets. So this lecture is not really representative of what data work is like, but hopefully I've given you an idea of the data landscape out there. Okay, that'll be it for today.
