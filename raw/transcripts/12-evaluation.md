---
title: "Lecture 12: Evaluation"
lecture: 12
video: https://www.youtube.com/watch?v=JpAxdTWQJxM
source: copy-edited from the YouTube auto-captions
verbatim_original: not committed - regenerate from the video (see below)
material: ../slides/12-evaluation.md
---

# Lecture 12: Evaluation — transcript

**This is the edited transcript.** The auto-captions have been repunctuated,
segmented into sentences, stripped of filler ("um," "uh," "you know," "sort
of," "kind of," "right?") and false starts, and had mis-heard technical terms
restored. No content was added, removed, or reordered, and every `[MM:SS]`
marker is preserved in its original position — content that was under a
marker is still under that marker.

The verbatim captions are **not committed to this repo** — they are a complete
reproduction of the lecture and none of that text is this KB's own work. They stay
reproducible on demand: the `cairn-kb` skill's `fetch_transcript.py` run on the
video id in this file's front matter, piped through `transcript_to_md.py`,
reproduces them exactly, so the checks below can be re-run by anyone who wants to
verify this edit.

Terminology was cross-checked against
[the lecture's source program](../slides/12-evaluation.md) (`lecture_12.py`),
which is the authoritative written transcription of this lecture's content —
benchmark names, paper citations, author/organization attributions, model
names, and numbers.

**Verification.** All three checks pass, after two corrections made here.

- **Timestamps** — 102 markers, identical sequence, in order, diffed against the
  verbatim captions.
- **Numbers** — nothing lost. Every difference is accounted for: three "1 billion
  word benchmark" and one "1 hour" spelled out to match the deck ("One Billion Word
  Benchmark", "one hour"), "seven out of 10" spelled out to "out of ten", a
  "20 20 ... 2024 or 23" stutter resolved to "beginning of 2024, or maybe 2023",
  "01 and 03" restored to "o1 and o3", "ARC-AGI 1" hyphenated, and a "-4" restored to
  "GPT-4 preview" where the captions had dropped it.
- **Word ratios** — 101 of 102 paragraphs inside the 0.72–1.10 band, at 88.3%
  retention overall. The one outlier is [48:32] at 0.70, read and confirmed as pure
  filler removal ("um" ×6, "you know" ×5, "uh" ×4 in a single paragraph).

**Two corrections made in the parent, both of which the drafting pass got wrong:**

1. **A sentence had moved across a timestamp boundary.** The draft completed a
   sentence split across the [55:30]/[56:16] marker by pulling its second half back
   into [55:30], and carried the *following* whole sentence ("And then something
   happened in 2024") back with it. Both were returned to [56:16]. This is the
   failure the word-ratio check exists to catch — it showed as 1.10/0.61 on that
   pair, and the numbers check caught the stray "2024" independently. The split now
   sits where the captions put it, which is this KB's convention: 68 of this file's
   102 paragraphs begin mid-sentence, in line with lectures 9, 10 and 11 (80/102,
   78/111 and 82/101).
2. **A possible benchmark name had been deleted rather than flagged.** At [1:13:22]
   the captions read "like like Gia GSM8K or MMLU"; the draft dropped "Gia" as
   noise. It sits inside a list of benchmarks and is more likely a mangled name, so
   it is now marked `*[Ed: unclear]*`. Deleting is not an available option for
   something that might be content.

**Restorations made.** These are places where the captions produced a
*wrong word* or dropped/mangled a word, and the text now reads differently.
Each is confirmed against [the slide file](../slides/12-evaluation.md) unless
the note says otherwise. Grouped by kind:

*Benchmark/paper names (evidence: the slide's "Benchmarks named in this
lecture" table and the matching per-benchmark sections):*

| Caption | Restored | Confirmed by |
| --- | --- | --- |
| "artificialanalysis" (as a name, 2:23) | Artificial Analysis | slide: "**1. Maybe a model is good if it does well on benchmarks.** [Artificial Analysis](https://artificialanalysis.ai/)" — the bare URL `artificialanalysis.ai` is kept as heard |
| "LLM Arena" (35:26) | LMArena | slide's own image filename for this exact benchmark is `lmarena-leaderboard.png` (used twice, for "Arena AI (formerly Chatbot Arena)") |
| "chat about arena" (1:05:37) | Chatbot Arena | slide's Realism section, near-verbatim: "Chatbot Arena prompts are from real people, but distribution is uncontrolled" |
| "Wiki, uh, Text 103" (6:14) | WikiText-103 | slide: "WikiText-103 (Wikipedia)" |
| "1 billion word benchmark" (6:14, 7:00) | One Billion Word Benchmark | slide: "One Billion Word Benchmark [arXiv 1602.02410]"; the 2016 CNN/LSTM paper described right after is the same arXiv id |
| "Lambada" (13:11) | LAMBADA | slide: "**LAMBADA** — [arXiv 1606.06031]", a 2016 paper, matching "from actually a while ago, 2016" |
| "Google Proof uh QA" (23:11) | Google-Proof QA | slide: "Graduate-Level Google-Proof Q&A (GPQA)" — GPQA's own four letters spell exactly "Google-Proof-Q-A" |
| "GBQA" (25:27) | GPQA | the whole paragraph is about GPQA, spelled correctly everywhere else in this transcript; plain letter-order slip |
| "Humanities Last Exam" (27:00) | Humanity's Last Exam | slide: "Humanity's Last Exam (HLE)" |
| "SWE bench" (45:29, 46:17, 47:02 x2) / "SweetBench" (1:13:22) | SWEBench | slide's own section header: "### SWEBench" |
| "SWE bench verified" (46:17/47:02, split across the marker) | SWE-Bench Verified | slide's Validity section: "Fixed up SWE-Bench to produce SWE-Bench Verified" |
| "Terminal Bench" (47:47) | TerminalBench | slide: "### TerminalBench" |
| "Terminal Bench 2.0" (48:32) | Terminal-Bench 2.0 | slide: "89 tasks constitute Terminal-Bench 2.0" |
| "that's uh that's I bench" (50:54) | that's CyBench | slide: "### CyBench"; this is the sentence that closes the CTF/cybersecurity subsection, matching CyBench's place in the lecture |
| "MLE engineering" (50:54) | MLEBench | slide: "### MLEBench ... 75 Kaggle competitions", matching "a bunch of Kaggle competitions" in the very same sentence |
| "harm bench" (1:01:00) | HarmBench | slide: "### HarmBench" |
| "Airbench" (1:01:47) | AIR-Bench | slide: "### AIR-Bench", matching "regulatory frameworks... EU, China... US... taxonomy" |
| "GDP Val" (1:05:37) | GDPVal | slide: "### GDPVal (OpenAI)" |
| "Code Bench, Untreated Eval" (1:11:04) | LiveCodeBench, UncheatableEval | slide: "LiveCodeBench, UncheatableEval: scrape new webpages", matching "scrape new webpages" in the same sentence |
| "genetic benchmarks" (1:14:07) | agentic benchmarks | slide's Validity section: "Problems with agentic benchmarks: insufficient test cases, trivial agent can solve task" |
| "Gia GSM8K" (1:13:22) | GSM8K, with "Gia" flagged inline | "GSM8K" itself is already transcribed correctly. "Gia" sits inside a list of benchmark names and is most likely a mangled one, so it is marked `*[Ed: unclear]*` rather than deleted. The drafting pass had dropped it as noise; the parent reversed that, because a possible benchmark name is exactly the kind of thing to flag rather than remove |
| "nano GPT speedrun" (1:16:27) | nanogpt speedrun | slide's own section/table name: "nanogpt speedrun" |

*Model/person names and other terms:*

| Caption | Restored | Confirmed by |
| --- | --- | --- |
| "Even Methos is only at 64.7" (28:32) | Even Mythos is only at 64.7 | slide names this model "Mythos" explicitly (dual-use cybersecurity-agent example); this same transcript spells it correctly, "Mythos," at 47:02 |
| "3.5 turbo" (21:38) | GPT-3.5 Turbo | standard industry shorthand; the same sentence contrasts it, unprefixed, with "GPT-4" |
| "judge using GPT preview" (39:17) | judge using GPT-4 preview | slide: "as judged by GPT-4 preview"; the baseline model named one clause earlier in this transcript is already "GPT-4 preview" |
| "OpenAI released their 01 and 03 models" (56:16) | OpenAI released their o1 and o3 models | slide: "Reasoning models (o1, o3) started making things take off" |
| "Sickle fancy is is a problem" (1:04:05) | Sycophancy is a problem | slide: "Many risks are quite varied (hallucinations, **sycophancy**, abetting crimes, inequality, losing critical thinking)" |
| "betting crimes" (1:04:05) | abetting crimes | same slide line as above |
| "Open AI's model will comply" (1:02:34) | OpenAI's model will comply | casing/spacing only — spelled correctly ("OpenAI") everywhere else in this transcript and the slide |
| "train task overlap" (1:08:43) | train-test overlap | slide's own section header: "### Train-test overlap" |
| "Squad" (1:08:43) | SQuAD | slide: "Pre-foundation models (ImageNet, SQuAD): well-defined train-test splits" |
| "Tatsuo's group" (1:09:30) | Tatsu's group | slide's own AlpacaEval links point to `tatsu-lab.github.io` (Tatsu Lab); "Tatsu" is this course's co-instructor, named the same way in lecture 10's transcript |
| "timestamps always safe either" (1:11:50) | timestamps aren't always safe either | slide, near-verbatim: "Timestamps aren't always safe due to copying either" — restores a dropped negation |
| "log prop" (15:27 x2, 16:14) | log prob | slide's own Warning subsection: "you compute `log_prob = LM(test_data)`" |
| "Hendricks at all 2020" (18:32) | Hendrycks et al., 2020 | the slide names no author, but the lecture cites the paper by arXiv id (2009.03300) through `references.py`, and that record's author list begins "Hendrycks, Dan" — so the spelling is verifiable **from the source the lecture itself cites**, not from outside knowledge. Adjudicated in the parent; the drafting pass had kept "Hendricks" as heard and flagged it |

**Three of the 35 restorations are NOT attested anywhere in this lecture's own
material**, and are called out here so a reader can weigh them separately. The
other 32 appear verbatim in [the slide file](../slides/12-evaluation.md).

- **Hendrycks** (18:32) — not in the deck, which names no author for MMLU. But
  the lecture cites the paper by arXiv id through `references.py`, and that
  record's authors begin "Hendrycks, Dan". Verified against the cited source.
- **Tatsu's group** (1:09:30) — not in the deck as a person's name. The deck does
  link `tatsu-lab.github.io` twice for AlpacaEval, and Tatsunori Hashimoto is this
  course's co-instructor, named the same way in earlier transcripts in this KB.
- **GPT-3.5 Turbo** (21:38) — the weakest of the three, and the only one that
  *adds* a token rather than fixing a spelling. He says "about 3.5 turbo"; the
  same sentence contrasts it, unprefixed, with "GPT-4", and the passage is
  narrating an MMLU leaderboard. The expansion is standard shorthand rather than
  an attested string, so treat it as an editorial reading.

*Casing/formatting normalized, not counted as restorations of wrong words:*
"Elo"/"ELO" appears both ways in the captions (33:53–37:44); standardized to
"ELO" throughout to match the slide's own consistent usage in the "Compute
ELO rankings" subsection. "ARC AGI" → "ARC-AGI" and "SWE bench" hyphenation
generally follow the slide's own section headers.

*Stray tokens/false starts dropped as noise, not restored (same treatment as
lecture 10's precedent for an unresolvable dropped token):*

- 3:56, "sta- statistics" — abandoned false start before "statistics."
- 1:05:37, "top sec- nine sectors" — abandoned false start before "nine sectors."
- 1:08:43, "train task overlap or for So, in a machine learning 101" — "or
  for" does not parse and is dropped as noise; "train task overlap" itself is
  restored per the table above.
- 1:11:50, "because I might post a or, you know, um you know, if with code"
  — "because I might post a" is an abandoned clause; the completed thought
  ("if, with code, for example, I might put something on GitHub...")
  is kept.
- 56:16, "now it our AGI ARC-AGI 1 is basically solved" — "it our AGI" is a
  stutter/false start on the same phrase that follows; dropped, leaving "now
  ARC-AGI 1 is basically solved."

**Restorations considered and *not* made, because the slide does not name
the term and there is no phonetic residue in the captions to restore from —
inserting the name would be adding content, not restoring it:**

- **MMLU-Pro** (21:38–23:11): the captions describe this benchmark in full
  (expanding 4 choices to 10, using chain-of-thought, getting harder) but
  never once utter its name — the passage just says "this new benchmark."
  Left unnamed rather than inserted.
- **MedHELM** (1:06:24–1:07:10): the medical-benchmark passage ("121 tasks
  ... 29 clinicians") never names the benchmark. Left as "a project ... in
  the medical domain."
- **Clio** (1:07:10–1:07:58): the passage about analyzing real usage data
  (which does correctly name "Claude" as the product being analyzed) never
  names the project "Clio." Left as "this project."
- **Greedy Coordinate Gradient** (1:02:34): the captions say "GCG" and
  describe it as "a coordinate-wise optimization algorithm," but never say
  the words "Greedy Coordinate Gradient." Left unexpanded.

**`[Ed:]` notes inserted, and why:**

- *At 28:32*, "So, this is a a kind of plum and powder" does not parse as
  English and is not resolved by the slide (which does not narrate this
  aside). Left close to verbatim with an inline `*[Ed: unclear]*` flag rather
  than guessed at.
- *At 1:13:22*, the captions read "like like Gia GSM8K or MMLU" inside a list
  of benchmarks that have been re-audited for quality. "Gia" is very likely a
  mangled benchmark name, but the slide file does not name it, so it is
  flagged `*[Ed: unclear]*` rather than guessed at or removed.

The `[Ed:]` note that a drafting pass had put on "Hendricks et al., 2020" at
18:32 has been **removed**, because the name turned out to be recoverable
after all: the lecture cites the MMLU paper by arXiv id through its own
`references.py`, and that record's author list begins "Hendrycks, Dan". The
spelling is therefore fixed from the source the lecture itself cites, and the
passage now reads "Hendrycks et al., 2020" with no flag.

**Student questions.** Following the KB's convention (see lecture 10's
transcript), wherever the recording makes clear someone from the floor is
speaking, the transcript marks it `*[Question from the floor: ...]*`, using
the captions' own words with only light punctuation. There are 5 such
exchanges, marked by `>>` speaker-turn markers in the original captions:
one at 26:14–27:00 (train/test contamination), one at 30:06–30:51 (how HLE
accuracy is graded — messy multi-turn crosstalk, consolidated the way
lecture 10 consolidated its garbled exchange), and one at 59:20 (whether
ARC-AGI-3 is multimodal).

---

**[0:05]** So, today we're going to talk about evaluation. So far in this
class, we've done everything we need to train a language model: we defined
the architecture, the optimizer, the main training loop. We showed how to
make the training fast with kernels and parallelism, and then we saw
scaling laws. We saw a little bit of how to make inference fast as well. So
the only missing piece now is talking about the data that you train on — we're
going to spend next week talking about that. And data is going to shape the
model behavior: if you train on code, then presumably your model is going to
be good at code.

**[0:51]** If you train only on DNA sequences, it probably can't speak
English. But before we even get to data, we need to talk about what behavior
we want from a model, and that's going to be the topic of evaluation. So,
evaluation asks a very simple question: given a model that we've trained,
how good is it? What is good? Evaluation might seem like a fairly mechanical
process — you define some prompts, you send these prompts to your model, you
get back some responses, and then you compute the accuracy. So why do we
have a whole lecture on evaluation? Actually, evaluation is a very deep
topic, and it's also very important, because evaluation is what shapes the
development of AI. Evaluation sets north

**[1:37]** stars, and model developers — open, closed, everyone — look at
evaluation as a sort of measure of progress. So by thinking carefully about
your evaluation, you implicitly shape the development of what your model is
going to be able to do. So what makes evaluation hard is the following.
Often you start with an abstract construct of what you want your model to
do — maybe you want your model to be good at conversation, or good at
reasoning. Those are abstract concepts. And evaluation is really the process
of turning that into a concrete metric, powered by presumably some

**[2:23]** concrete prompts or environments. So this is going to be the core
challenge, and we'll see many examples of how this process — abstract
construct to concrete metric — shows up. So let's go back to the question of
what makes a model good. Just kind of superficially, maybe a model is good
if it does well on benchmarks. So there's this website, Artificial Analysis,
which has become a bit of a standard for thinking about the — quote unquote —
intelligence of models. If you go to artificialanalysis.ai, you can see a
ranking of different models, from GPT-5.5 all the way down. There are many
more models after this point, but —

**[3:09]** and so this is a measure that includes a bunch of different
datasets, some of which we'll talk about. Or maybe a model is good if it
does well on benchmarks and is also cheap to run, because cost should
matter. And this plot shows both the intelligence index from before, but
against how much money it costs to run a particular model, based on
inference cost. And you can see, of course, as you might expect, that the
two are correlated — the more you pay to run, the higher the intelligence
index — but it's not exactly aligned. So, but maybe a model is good if
people just

**[3:56]** prefer it — later we'll also go a little bit more into depth.
This is the take that Arena AI, formerly known as Chatbot Arena, takes: you
can form a ranking of models based on whether people like them or not. Or
maybe a model is good if people simply choose to use it, and therefore pay
for it. So, OpenRouter is this service that allows one to use multiple
different models through a central endpoint, and as a result they collect a
lot of statistics on what models people are using. They publish statistics
which are really interesting — you can look at the top models that are used
through

**[4:42]** OpenRouter. Now, this is not representative of all model usage,
but it does give you a sense of what people on OpenRouter are doing. So this
is kind of more of an economic lens — I don't know what "good" is, but if
people are paying for it, it must be good. So, okay — none of those is
necessarily the correct answer. I don't know if there is a correct answer,
but I'm just trying to get you to think about different ways of thinking
about what "good" is. So let's start now by looking at different, concrete
ways of evaluating language models. And there's no better place to start
than going back to the core idea of

**[5:29]** a language model, which is that it's a distribution over
sequences of tokens — P of X. And through that lens, there's a very natural
way to evaluate a distribution, which is perplexity, or, you know,
likelihood, or log loss — those are all kind of related concepts. It
basically says: I have a test dataset, D, and I'm going to see how much
probability mass my language model assigns to that D. And I normalize and
divide so that the numbers are more interpretable. But that's basically it —
how much mass does my probability language model assign to a dataset?

**[6:14]** Okay, so this is fairly natural. In fact, when we were doing
training, we minimized the perplexity on the training set — so the obvious
thing to do is measure the perplexity on the test set. And this is what
people traditionally did in language modeling research for many years.
Throughout the 2010s, if you pick up a language modeling paper, this is what
they did. And back then, standard datasets included the Penn Treebank,
WikiText-103; there was also this One Billion Word Benchmark, which was
taken from machine translation sources. And this is all in the classic
paradigm, which is in-distribution evaluation: you train on some train

**[7:00]** split of that dataset, and then you evaluate on some test split
of that dataset. And people actually made a lot of progress, which was just
measured in perplexity reduction. For example, there's this famous paper
from 2016 which showed that you could apply CNNs and LSTMs to this One
Billion Word Benchmark and get a huge perplexity reduction. And bear in mind
that around that time people were still talking about n-gram models, and
maybe hybrid models — and this was the first definitive kind of result that
showed pure neural is clearly the way to go.

**[7:46]** Okay, so life was very simple back then — you train on the train
split, you test on the test split, for perplexity. So then when GPT-2 came
around in 2019 from OpenAI, they trained on the dataset called WebText,
which was 40 GB of text from websites that were linked from Reddit. And the
way they evaluated was actually different — they evaluated zero-shot on
standard datasets that everyone else was evaluating on. So this is
out-of-distribution evaluation. This is a table from their paper which
showed that as they trained different models — the biggest model is 1.5
billion parameters — they were able to show quite a

**[8:33]** bit of improvement, especially on the small datasets. For
example, PTB, which is a tiny dataset — they were getting 35 perplexity
compared to the state of the art, which was 46. Now, if you have enough
in-distribution data, you don't quite do as well as the SOTA, but
nonetheless this is kind of impressive, given that this model was not
trained on the One Billion Word dataset at all. There might have — I don't
know, maybe there was some overlap, because you never — I don't know if
they did a careful train-and-test decontamination. And so this ushered in a
new

**[9:19]** paradigm of thinking about language models — not just in the
in-distribution setting, but training on a large dataset and evaluating on
standard benchmarks, which nowadays I think has become very common and
standard, but back then was seen as a bit of a novelty. So, I will also
argue, since we're talking about perplexity, that perplexity is all you
need. I wouldn't take this too seriously, but I think it is a mindset that
drives a lot of language modeling research. And the basic argument goes
like this. There's some true distribution out there — let's

**[10:05]** call it T — and you're training this model, P. So what is the
best perplexity you could obtain? Well, the best perplexity is the entropy
of that true distribution, and that is realized when P equals T. So by
minimizing perplexity, you're basically pushing down the perplexity, and the
only unique minimizer is when you actually get the true distribution. And
when you have the true distribution, you're done — you just model
everything, you can solve all the problems, and you can do things like
condition on the problem, generate a

**[10:51]** solution; condition on a question, generate the answer. So by
pushing down on perplexity, we'll eventually reach AGI — that's the
argument. I mean, this might sound, I don't know, either a bit fantastical,
but in some sense this is really, I think, a strong driver for many people
who have been scaling and scaling language models, at a time when maybe the
gains weren't as obvious. Before GPT-3, I think it wasn't clear that
language models were just going to have such a huge impact, and it was
really this belief that if you drove perplexity down, good things would
happen.

**[11:36]** So, of course, perplexity is maybe not all — one way to frame it
is that perplexity is maybe more than you need. For example, if you have a
sentence like "Stanford was founded in 1885," perplexity will penalize the
prediction on all tokens. In particular, it'll ask: what is the probability
of "1885" given the rest? And that token is probably pretty useful — it's
basically a Q&A example, a sentence-completion phrase. So it captures
knowledge about the world. But other tokens — for example, the first word of
a sentence, or even

**[12:23]** "founded" — maybe not that interesting, I think. And perplexity
doesn't care — it's going to charge you bits for every bit of deviation from
the true distribution. So there is a bit of a solution here, which is you
can always measure the conditional perplexity: you can condition on some
prompt, and then measure the perplexity of the remaining response. That
allows you, in general, to focus on certain tokens that you believe to be
relevant, and less on others that are sort of incidental. And also it's
important to

**[13:11]** remember that some benchmarks, even though they are not
explicitly a perplexity measure, are actually just perplexity in disguise.
For example, if you think about cloze tasks — there's this LAMBADA paper
from actually a while ago, 2016 — which is basically fill-in-the-blank: you
have a spurious context, you have a sentence, "Do you honestly think I will
want you to have a ___," and the goal is to figure out that word. And even
though this is measured in terms of accuracy, it is really a next-token
prediction problem. And here it's not the perplexity of all the words — it's
carefully chosen; if you read this paper, it's really carefully

**[13:56]** chosen to be words such that you actually need a bunch of long
context to resolve. Right? So this is actually something that the early
GPT papers really latched onto, because they were interested in long-context
modeling, thinking that that was a key to unlocking reasoning and all these
things. And really, by focusing on positions where, to resolve them, you
require long-distance dependencies, that's a way to kind of sharpen the
perplexity to some phenomena that you care about.

**[14:41]** Here's another popular, classic dataset that you see in some
language-modeling evals: HellaSwag. So what is HellaSwag? They went to
various different datasets and sources and came up with it. It's multiple
choice, but it's really sentence completion, and at some level it's
perplexity. So here's a sentence: "A woman is outside with a pug and a dog.
The dog is running around, she ___." And then you're trying to figure out
which sentence completes it best. So it's not exactly perplexity as multiple
choice, but in some ways it is perplexity. So, one word of warning: if you
ever decide that you're sold on perplexity as really the

**[15:27]** measure, and you're going to launch a leaderboard and have
people compete on perplexity — just a quick word of caution: what does a
perplexity leaderboard look like? People are going to submit a language
model to you, and you have to compute the log prob. You send them the test
data, and they give you back a log prob. Well, you look at this and say,
"Hmm, that might be a little bit fishy," because you need to trust that the
probabilities they hand back to you are valid, and that they sum to one.
Otherwise, I could just implement an LM that always returns one — or, I
guess, log prob zero —

**[16:14]** and I would get a very good perplexity, but clearly that's not a
valid distribution. And with this contract, it's not really easy to check
that someone is actually internally computing a log probability — so
there's some trust involved; maybe you have to look at the code or
something. For downstream tasks, this is much more straightforward, because
you can have a black-box LM — I give you a prompt, you give me a response,
and I can just use that response and calculate some accuracy. So perplexity
is fundamentally different, because it deals with distributions, and you
have to be a bit careful. And this is — not to mention, this is less of a
problem with autoregressive models — but if you have something like VAEs,
where

**[16:59]** there are some models where you can't even compute the
distribution, you can only compute a bound, then you have to really trust
the math as well — that this is actually a valid bound. Okay, so the
summary of perplexity is that it is actually still used quite heavily in
language model development. It has some nice properties — we saw that it's
generally used for scaling laws, because it's also smoothly varying with
scale, which is important for getting scaling laws. But, at the end of the
day, especially if you're not a believer, then you need some benchmarks to
really capture real-world situations, to

**[17:46]** convince people that your language model is good. For the
believers, if I show you a remarkably low perplexity, they'll be convinced.
Okay, any questions about perplexity? Okay, hopefully everyone's a believer
now. So let's talk about exam benchmarks. Exams, as all you students know,
are a way that we use to test humans, and so you can adopt the same
mentality to test language models. The nice thing about exams is that

**[18:32]** you have very careful control over the subject, over the
difficulty. You can design them to have unambiguous correct answers, which
makes them easy to grade. So there's a lot of nice properties about exams.
And for this reason, a lot of LM benchmarking culture was based on just this
idea of exams. So, obviously there's been a lot of benchmarks, but I'll just
highlight some of the ones that I think have been influential in the
development of language models. So this one is MMLU, or Massive Multitask
Language Understanding, from Hendrycks et al., 2020. And at
that time,

**[19:18]** it wasn't clear that language models were really like
general-purpose task solvers. This was around the time when GPT-3 came out.
Many people, I think, still thought of language models as — well — language
models: you generate fluent English. But they had this idea that, well,
let's just try to be ahead of the curve here. We're going to go through 57
subjects. We're going to have a team of students scour the internet for
different sources and really put together this very comprehensive dataset.
And despite the name MMLU, which has the word "language understanding," it's

**[20:04]** really about testing knowledge and reasoning — it's less so just
pure language understanding. And the way they evaluated it was with GPT-3,
using few-shot prompting. So they essentially said: I have formed this
prompt — "The following are questions about high school mathematics" —
some in-context examples of question, answer, question, answer, and then
the final question, and the model has to predict the answer. So this seems
very mundane, but at that time it was, I think, pretty radical that you
would construct this fairly complicated prompt, with in-context examples
and everything, and expect a language model to actually do something

**[20:50]** reasonable. And sure enough, at the outset, the models were not
great, I would say — some of the small models were barely above chance, and
the larger model, GPT-3, was actually well above chance. So this was kind of
encouraging. And then, over the years, as we noted, this benchmark has
essentially saturated. So, this is a site — I don't think it has all the
data in it, but — about GPT-3.5 Turbo. This was, I guess, beginning of
2024, or maybe 2023 actually,

**[21:38]** to GPT-4, and then now it's in the 90s. Okay. So there's also a
link you can follow up on later, where you can actually browse some of the
predictions of these models, which is always fun — to see what models
actually goof up on and what they get right. Okay, so — around 2024, it
seemed like a lot of progress had been made on MMLU. It's like, okay, you're
going to see this thing again: this benchmark is getting too easy, let's
try to make it harder. So, well, first of all, they actually

**[22:23]** noticed that there were some noisy and trivial problems, so
let's get rid of them. Four choices might be a little bit too easy, so
let's expand that to 10. At that point, it was much more in vogue to use
chain of thought to solve these benchmarks, whereas when GPT-3 came out that
wasn't really a thing. And it's clear that some of these questions could
benefit from a few reasoning steps. So this new benchmark was harder — now
the accuracy of the models is back at 33. But, as you can maybe see from the
progress so far, it started low and now it's back to, I guess, like 88,

**[23:11]** almost 90. Okay, so this benchmark might be a little bit too
easy as well. So there's another benchmark called GPQA, which stands for
Google-Proof QA. And the idea here is: okay, how can we get a really good
benchmark? And the idea is that, well, if you can solve a problem by
looking at Google — Google has kind of the internet, and the language model
is trained on the internet — so if you can just solve it that way, then
that's probably too easy. So they got some — now, PhD level; I think MMLU
was more like — actually, some of these maybe — some of these were like

**[23:57]** undergrad, but maybe grad level — but now it's more explicitly
about questions that are meant to be pretty hard. And at this point, now
you say, okay, how do I come up with really hard questions? You actually
have to go through quite a bit of human labor to get really hard questions.
So, this was 61 PhD contractors from Upwork. Someone writes a question, it
goes through expert validation, which then provides feedback on the
question, and then the question writer goes and revises the question, and
then another expert looks at the question, and then — you have the
question, and then you send it to some non-experts who are given Google
access, and you see if

**[24:42]** they can solve the question. So there's a subset of these
questions called the diamond set, which you might see on people's
leaderboards, where if the two experts agree, and at most one of the
non-experts is able to answer the question, then that's deemed a good, or
diamond, question. So this is a pretty rigorous review process. And PhD
experts were only able to get 65% — I think this is mostly four-way
multiple choice — so this is above random, but this is not a good grade if
you're getting 65% on an exam. And non-experts, even if they

**[25:27]** had 30 minutes with access to Google, were getting about a bit
over chance, but not much. And at that point, GPT-4 was at 39%, and let's
see how it's doing now. So GPQA is now, again, at 94 — okay, 94. So, yeah,
these benchmarks have a shelf life that's not too high, I guess. I would
say that some of these benchmarks, even though they saturate for the large
models and small ones, are still useful for developing smaller models, or
fitting scaling laws, and so on. Okay, so then —

**[26:14]** *[Question from the floor: How do you know that these questions
aren't going to start being used for training, and that your models
actually have good knowledge?]*

Yeah, so this is a very important question. How do you know that these
questions aren't just in the training set? So the short answer is: we
don't know, because I don't know what's in the training set. I'll come back
to this contamination point a little bit later, and to ways of addressing
it — but you should always — I'm glad you asked this — you guys should
always take these numbers with a little bit of grain of salt, because it's
only trust. Let me just say one more thing. I think training contamination
is a little bit subtle, right? Because it's

**[27:00]** probably not the case that they literally train on the test
set, but questions can be derived from other sources, and those sources
could be trained on. So, often, contamination is a much more subtle process
than literally accidentally training on the test. Was there another
question?

*[Question from the floor: The same question.]*

Okay — great, good, it's always good to be skeptical. Okay, so the final
exam benchmark I'll talk about is something ominously called Humanity's
Last Exam. This is from last year. And they were saying, "Okay, well, these
models are getting so good — let's just give it all we can to create
something that's really tough." So, this is multimodal, many

**[27:45]** subjects. It's still multiple choice, and I guess some short
answer. And this was crowd-sourced — they had some money to incentivize
people who were into money to create questions. They also incentivized with
authorship, for those who were into that. And they went through multiple
stages of review, filtered by frontier models. There's a whole process.
Another answer to your question is that they also held out a private set
that was not released publicly, so that they could make sure that this did
not enter training. And then you just hope that the APIs — because you
still have to send this to the model — that those prompts don't end up in
training.

**[28:32]** But let's hope that's the case. So, this — every dataset paper
kind of looks like this: previous benchmarks, look, the models are doing so
well; my dataset, look, all the models are doing really poorly. So, this is
a kind of *[Ed: unclear — captions read "plum and powder"; not confirmable
against the slide]*, and this was especially — like, single digits, it's
pretty abysmal here. And let's see how we're doing in 2026. So, this one —
it still has some room. It's only at — even Mythos is only at 64.7. Okay.
So, this is indeed a pretty — it seems like a pretty hard exam.

**[29:20]** Okay. So, to summarize the section — as you can see, we've
trended towards harder and harder questions as the models improve and we
saturate existing benchmarks. It's interesting that the multiple-choice
format still survives, because — with multiple choice, I mean, you can make
it as difficult as you want. Right? Some people think that, well, multiple
choice is too easy — well, I mean, I can give you a very hard
multiple-choice question, that's not really the point. I think the point,
more, is that multiple choice restricts the set of questions you can ask,
but in terms of difficulty, no problem with multiple choice. The main
problem with exam-based

**[30:06]** questions is that this does not capture real-world usage. No
one asks HLE questions to a language model, except for when you're
evaluating HLE. Most of the time, people are asking open-ended questions —
they don't even necessarily have a correct answer, they might not be
well-formed, and these benchmarks do not capture that. Okay. So, question?

*[Question from the floor, crosstalk: For accuracy, how is the model's
output compared with the ground truth answer?]*

So, for HLE —

*[continued from the floor: Like, for accuracy, how is the model's output
compared with the ground truth answer?]*

For accuracy, to calculate the accuracy — of calculate the accuracy, for how
is the

**[30:51]** ground truth compared? So, for multiple choice, it's just like:
did you get the right answer?

*[Question from the floor: So, would we check the probability of the
solution for each option, or how is —]*

The language model generates, say, a letter — C. And my answer is either B,
C, A, or D, and I just check if they're the same or not.

*[Question from the floor: Okay, so you're only generating one token? The
language —]*

Yes. Well, okay — so then the question is less about evaluation, but about
how the model generates an answer. Right? So there are a few options. One is
that you literally sample and it generates a letter. Or, more commonly now,
you generate a chain of thought, you generate the answer, maybe some

**[31:37]** explanation. But there is a way of extracting an answer, which
does matter, and language model evaluations can be very sensitive to that —
but I'm not going to talk about that here. Okay, so let's talk about a very
different class of benchmarks, called chat benchmarks. So far we've talked
about multiple-choice tasks — they're still pretty good at capturing some
notion of intelligence and difficulty. But most people don't ask a
multiple-choice exam question to their AI assistant, unless they're trying
to use their AI assistant to do their exam for them — which you would never
— no one does that.

**[32:22]** Okay. So, here's an example, semi-inspired by a real-world use
case. So, you know, I might say: I want to make a beet salad — which herbs
will work well, and which won't? Okay, so this is a very open-ended
question, and then you get some response, which is again an open-ended
response. And then the question is: how do you evaluate this thing? Right,
you can't just check exact match with equality — there's no ground truth
even. So there are a few ideas here — we should go through them. One of the
ideas was pioneered by Chatbot Arena. And the idea is: well, you ask
humans.

**[33:07]** Okay. But the way they ask humans, I think, is a kind of clever
idea. So they basically had this website where any random person from the
internet can go in and chat. And then, instead of, like, a normal assistant
getting one response, you actually get two responses from two different
models, which are anonymized. And then you rate which one is better. So,
for example, you put in this prompt, you get these two responses, Assistant
A, Assistant B, and then you say A is better, both are good, both are bad,
or B is better. Okay? So, using this process

**[33:53]** you can get a bunch of pairwise-comparison data — model A,
model B, which one is better. And then you can use this to compute ELO
rankings based on this. So — I don't know how many of you are familiar with
this — but just to get everyone on the same page: in these ELO rankings you
define your rating model as the probability of a particular model beating
another model, as some function of the ELO scores of model B and model A.
So, the larger the ELO score of A is, the higher the probability. So it's a
smooth mapping.

**[34:39]** And then you fit this model with the parameters, which are the
ELO ratings, to maximize the probability of the pairwise comparisons. Okay?
And then, as a result, you basically get a rating for each of your models.
Okay? So, this is what Chatbot Arena, nowadays known as Arena AI, did. And
you can get this kind of nice ranking. I guess Claude Opus is doing pretty
well here, but, you know, some other models are on the list as well. Okay,
so there are a few things that are nice about the setup. One is that you
have real-world prompts. Right? The idea is that anyone can

**[35:26]** come to the site, and the incentive for someone to come to the
site is that they get free access to a language model. And so there's sort
of this implicit assumption that they're actually trying to use it to do
something useful, and therefore you get sort of like a real-world prompt.
But one thing you should keep in mind is: who are these people? Random
person on the internet — I don't know what that distribution, random person
on the internet who comes to LMArena, I don't know what that distribution
looks like. I think in their paper they have some demographics, but
demographics don't tell the whole story. You could worry about different
biases coming from the different

**[36:12]** models — maybe spammers, people who are trying to game — you
know, who maybe even submitted a model, and they want their model to look
good. You just don't know, it's a little bit of a wild west. There's
another issue, which is that the binary preference — which is better, A or
B — is very nice and simple, and feeds into this rating, but it does
conflate style and correctness. So, ELO ratings for chess make a lot of
sense, because the only thing that matters is: did you win? But, you know,
which one is better is much less clear-cut. And there are questions of, you

**[36:58]** know, the human is the judge here. The person who puts in the
prompt is answering the question, and this is good, because to some level
the person has some intent, and they can sort of declare whether their
intent was met. To some extent — except that they're presumably asking the
question because they don't know the answer. So, if I give you two answers,
how are they to judge which one's correct? And then there's also the
problem of sycophancy, where maybe more pleasing answers are going to be
upweighted, compared to correct but honest answers. So there's a lot of
kind of potential issues here. But the nice thing about this method,

**[37:44]** is that you don't need to feed the same prompts to all the
models. And this is the beauty of ELO, right — in chess, you don't have to
have everyone play everyone else. It can just be kind of some sparse sample
of that, and you can still derive rankings, as long as the graph is kind of
connected. And this is important, because a human is rating, and you can't
expect a human to rate all the models, or even more than two models. And
the nice thing as well is that, as new prompts and new models come in,
there's a natural story for updating this over time. Okay, so let me talk
about a different approach. This is AlpacaEval, from 2023.

**[38:30]** There were some instructions, derived from various sources. And
the metric here is the win rate against a baseline model. So, you have a
model — AlpacaEval says: I'm going to generate a response for a prompt,
generate using, let's say, a baseline model, GPT-4 preview at that time, and
then judge, using GPT-4 preview, which one is better. So immediately this
raises a question of potential bias — but let's put that aside for now. So
this is basically an instantiation of LLM-as-a-judge, which is very popular
these days for evaluating language models. And you can mitigate this

**[39:17]** bias by having multiple judges and ensembling, and so on and so
forth. So, one problem, initially, with AlpacaEval, is that LLM judges favor
long responses. And this led to a bunch of leaderboard gaming, where a bunch
of fine-tuned models were submitted and got really high AlpacaEval
performance just because the responses were longer. So this got fixed in a
subsequent paper, where they use a very simple regression method to debias
the metric. Okay, but this raises a kind of more general question here,
which is not actually specific to AlpacaEval, which is: how do you evaluate
a metric?

**[40:02]** Okay, so we've been talking about evaluation of models — and if
I have a metric, I can evaluate a model. But if I'm coming up with a new
metric, how do I know this metric is any good? This is a hard problem, and
there's no real answer here. One thing you can do, more of a sanity check,
is look at correlation with other metrics, with the idea that "higher isn't
necessarily better" in all cases, unless you're trying to mimic this other
metric. So, AlpacaEval's correlation with Chatbot Arena is 0.98, which is
quite high. So, which suggests that

**[40:49]** you could — let's say you wanted to evaluate on Chatbot Arena,
but you're, I don't know, too shy to put your model up, or you don't want
to wait for humans, or whatever — you can use AlpacaEval to evaluate
instead. Now, this correlation, of course, is with respect to a certain set
of models, and so this might not hold for models that, let's say, are
stronger than GPT-4 preview. Okay, so, at this point, this leaderboard — I
don't think it's really been maintained in over a year, but this is kind of
what it looked like at the time.

**[41:34]** Okay, so there's another benchmark I'll talk about, which is
WildBench, and this one sourced a bunch of examples from human chatbot
conversations. So, like Chatbot Arena, they also put up a free service
where people could chat with this bot, and they collected the queries. And
then, like AlpacaEval, they used an LLM as a judge to evaluate the model.
The main innovation here is the use of a checklist, which is generated,
specific to a

**[42:23]** particular prompt or task. And the idea here is that if you
just ask a language model to judge whether this response is good or not,
this is, in some sense, a very ill-defined task — it depends on what you
care about. And so the idea here is that, by having a checklist, or some
sort of rubric, this greatly scopes and makes the evaluation task more
well-defined. So they did this too — they showed correlation with Chatbot
Arena, which admittedly is a little bit circular, because then you can ask,
well, is Chatbot Arena really the ground truth? But at least we're all in
the same boat together.

**[43:11]** So, okay — to conclude this section: how do you evaluate
open-ended responses? There's no clear solution here, but there are some
ideas. Pairwise comparisons between a reference and a model response are
generally a good idea, because, especially for very similar responses, you
can get directional information — well, this one is slightly better than
this one — but in terms of absolute ranking, like, is this a seven out of
ten or an eight out of ten, that tends to be a lot less high-signal. One
should always be aware of biases, both in terms of humans or LLM judges —
they have different biases, but

**[43:57]** nonetheless that's something important to pay attention to.
Perhaps the only thing you can do is evaluate over multiple judges — humans
and judges — and if everything is saying your model is better, then maybe
your model is better. And finally, the evaluation problem here, for
open-ended responses, I would emphasize, is not really well-defined. And so
increasingly, I think, it's important to define rubrics or checklists to
improve the reliability, or well-definedness, of an evaluation. And this is
true regardless

**[44:43]** of whether it's a human or a judge. Anyone who's done
crowdsourcing knows that if you just ask a human to rate something without
giving some sort of rubric, you're probably going to get very nonsensical
results. Okay. So let's move on to the agentic world. So, you can think
about what we've been doing so far as evaluating what a language model
says — that's chat. And now we're going to evaluate what LLMs do — which is,
you know, agents. So, for anyone who's following AI hype, agents are
everywhere, and, you know, these are definitely,

**[45:29]** I think, going to transform the way that we think about
language models. So, an agent is basically a language model plus some sort
of scaffold, which is the logic for deciding how the language model is
going to be called, and what tools it has access to, and so on and so
forth. So there are a few benchmarks I want to point out that are agentic.
One is very popular — it's just called SWEBench. Basically, the task is:
you're given a codebase and a description of a GitHub issue, submit a PR.
And the way you evaluate a PR is by whether it passes

**[46:17]** the unit tests. So, these tests are designed so that, before you
submit a PR, certain unit tests don't pass, and then your goal is to fix
it — so certain unit tests pass and you don't break anything else. So,
here's an example of some instructions: here's the issue, and then some
code that is provided in the context. And your goal is to generate a patch,
and given a patch, you are evaluated on whether you pass or not. So the
good thing about this is that the evaluation is very straightforward. And
let's see how we're doing on SWEBench. Actually, this is going to be
SWEBench

**[47:02]** Verified, which I'll show you later — I'll tell you why this
happened. So, SWEBench, in this — was only, I guess, 2024 — was still around
16%, and now it's up to like 93. Good job, Mythos. Okay. So, this is sort
of a very canonical, generic benchmark, as a lot of attention on agents has
been invested in coding abilities. So, SWEBench really pioneered this way
of thinking about how to assess

**[47:47]** an agent's ability to write and code in a realistic
environment. So, here's another project called TerminalBench. And here the
idea is that this is going to be more general-purpose, right — we're going
to basically have the environment be a computer terminal, and we're going
to define a bunch of tasks that can be done in a computer terminal by
typing in a bunch of commands. And the nice thing about this environment is
it's very simple and universal. And these tasks were crowdsourced from 93
people from across the

**[48:32]** world. 89 of these tasks constitute Terminal-Bench 2.0. These
tasks generally take — ranging from one hour to over a week, depending on
whether you're expert or junior. And here's what the leaderboard looks like
as of today. So, top models are the frontier models. One thing that's
interesting is that the agent is also important here, because you can have
two agents on the same model, and they have different accuracies. Okay, so
another benchmark I'll talk

**[49:19]** about is cybersecurity. So, this is 40 capture-the-flag tasks.
And the setting here is that you have an agent that is given an
environment, and it can run various commands — like, you know, look at the
source code, it can access the web server — and the goal is essentially to
hack into this server to extract some flag, which is a unique string that
proves that you've hacked into the server. So these are competitions,
cybersecurity exercises, that are normally done by humans. And the

**[50:05]** — this is an example of what the agent scaffold kind of looks
like here. This is a very simplistic version, where basically you have one
continuous memory: you produce some sort of action, you get environment
feedback, and then you put the results into this buffer, and this is
concatenated. As I'll show you later, this history obviously grows quite a
bit, and you'll need to have better ways of managing context. Okay, so this
is what the leaderboard looks like when it came out — the best models were
like about 10% or so, and over time now it's completely

**[50:54]** solved, actually. So, that's CyBench. There's another class of
problems, which is MLEBench. So, this benchmark has a bunch of Kaggle
competitions, which involve processing data, training models, and so on
and so forth. And the agent can look at the dataset, read the description,
write code, train models, and eventually submit something, and you get a
grade. So this is what the leaderboard looks like right now, and again you
can see that it's really the same usual suspects on the LM side. But

**[51:39]** there's quite a bit of variation in terms of the different
agent scaffolds. So one thing I'll point out about agent scaffolds is that
they matter a lot. So, when I talk about — when I said language model
evaluation — this is broader than language model evaluation. And the reason
is that, unlike what I mentioned before with this early CyBench agent
scaffold, now you have to be a lot more sophisticated to solve really
complicated tasks. One thing people realized is that explicit planning is
helpful — you can't just kind of stream-of-consciousness chain-of-thought
this, and

**[52:25]** then you build up this context, and the agent can easily lose
track of where it is. So instead, keep a to-do list, get it checked off.
Hierarchical delegation: agents should call other sub-agents with some
cleaned context — that agent does some stuff and returns only the result;
the master agent doesn't need to see all the gory details, and this can
help provide some encapsulation. Memory: people have been experimenting
with explicitly reading and writing files, especially as the context grows —
you can't just store it in your context window. And then, finally,

**[53:10]** there is more context engineering, in the style of managing the
process: when should you delegate to sub-agents, when to try this strategy
versus this other strategy, what to write to persistent memory. And these
things are generally optimized for the particular language model. Okay, so
agents are exciting because they enhance the capability surface of language
models. Scaffolds are very important. So, really, when we talk about
evaluating agents, it's evaluating both the language model and the agent
scaffold.

**[53:56]** Okay, now we're going to switch gears a little bit and talk
about this other, very different style of benchmark. I call them "pure
reasoning," for lack of, I think, a better term. And the motivation is that
all the tasks so far require some linguistic and world knowledge. And so
the question is: can we just isolate reasoning, or pure fluid intelligence,
from knowledge and knowing facts? And you can argue that this gives you a
form of more pure intelligence. So, there was this project, ARC-AGI, which
started back in 2019, where the goal was exactly this.

**[54:43]** So, this — the task was meant to be 100% solvable by humans, but
challenging for AIs. And every task is sort of a special snowflake — or
that's the intent. So memorizing a bunch of facts, or solving previous
problems, shouldn't really help you that much. So, if you're remembering
your LM history, 2019 was pre-LM — I mean, this was GPT-2 era. So this was
kind of prescient, in that regard. So here's an example of the first
iteration of the task: you're given these pictures, and you're trying to
guess what comes next. And if you take, like, 10 seconds, you can probably
solve this problem.

**[55:30]** Okay. So, you probably guess it's to just fill in the yellow to
form the rectangle. Okay, so — and then, last year, there's an updated
version of this task, which was a bit more complicated. You can take a look
at this. So, here's the trajectory: so, when ARC-AGI-1 came out — I think
this is actually zero, so basically models didn't do anything. This is like
GPT-3, pre-trained models didn't move the needle at all. Right? And this
was exactly as the creators had intended — pre-trained models just

**[56:16]** learn what the facts are on the internet, and maybe learn linguistic
patterns, but none of that is actually helpful, directly, at least, for
these ARC-AGI tasks. And then something happened in 2024. This is when
OpenAI released their o1 and o3 models, and then
things started taking off pretty abruptly. And now ARC-AGI-1 is basically
solved. And even around 2025, when ARC-AGI-2 came out — this has also, you
know, it's not quite solved, but it looks like it's on its way to being
solved. So it was really the reasoning

**[57:04]** capability that kind of unlocked this. And now one could wonder
whether all the knowledge of actual pre-training was necessary for these
tasks. And certainly, without pre-training, we wouldn't have this explosion
of reasoning models. So, arguably, it was still important, but not maybe
directly visible, for much of the history of ARC-AGI. So, now, just last
month, ARC-AGI-3 comes out. So you see this familiar pattern — all the
creators look at this and think, "Oh, time for a new benchmark." So this is
now an interactive environment, where you can go online and play this
game. So again, no

**[57:49]** language — you're just trying to figure out patterns. It's kind
of fun. And now the scores are extremely low. So, next year, when I teach
this class, I'm sure I'll have to update this slide, but we'll see. So, I
think this ARC-AGI line is really interesting to think about — the goal is
to disentangle reasoning from knowledge. This is really hard to do, and I
think this is probably the best attempt at it. It's not clear whether you
can really fully decouple things. And certainly, if people really cared
about this benchmark, I think people could probably

**[58:35]** benchmark anything — so you could design questions like these.
These still come from some prior — I'm not sure there's anything as
quote-unquote "pure" as reasoning. This is also constrained to human
reasoning; the explicit goal is to be 100% human-solvable within some
reasonable time, so this doesn't really extend to superhuman reasoning — for
example, winning IMO gold medals, or solving open math problems, which
arguably are still very useful and important. But clearly this exposes some
gaps in the current models — so there's kind of work to do, if you think
the goal is basically being able to solve all the tasks.

**[59:20]** Yeah?

*[Question from the floor: This ARC-AGI-3 especially looks quite
graphical — is there some multi-modal content involved, or is it
represented as text as well?]*

Yeah, so the question is: this looks very graphical, so what is the input
to our language models? So this is essentially — I don't know whether it's
64 by 64, I think — so you can provide this as an image, or you can provide
it as ASCII art, or some sort of textual representation. Either way, there
is a sort of spatial element here that the model has to reason about, which
is really not English or natural language.

**[1:00:12]** Okay, so let's shift gears and talk about another class of
benchmarks, which is related to safety. People have been talking about
safety in language models for a while, and if you think about other areas,
such as cars, it's pretty clear what safety means there, right? There are
these safety ratings for cars, which involve testing based on smashing
cars into a wall with an airbag, and seeing how well it protects the
dummies in the car — and this is with decades of lobbying and figuring out
what safety means for vehicles. So, what does it mean for AI? I would say
there is not a great

**[1:01:00]** answer here, but I'll give you some examples of what people
have been thinking about. So, there's this benchmark called HarmBench, and
this is really about prompting a model with things that are meant to be
harmful, and expecting the model to essentially refuse. So this is one
type of safety, and I think probably the dominant thing when people think
about safety, is that I want to prevent bad actors from doing things with
language models. But it turns out that's not the only thing that matters.
So, there's this other work, AIR-Bench, which tries to think about safety
holistically, by looking at all the regulatory

**[1:01:47]** frameworks in the EU, China, and the US, company policies, and
builds a taxonomy of all the different things that could go wrong. And then,
using this, constructing a particular set of prompts, and evaluating, and
so on and so forth. But one problem with safety is jailbreaking, which is
its own kind of sub-problem — which is that language models are generally
trained to refuse harmful instructions, but, as we know, you can get around
this if you're clever. And so there's some early work that uses automatic
ways to

**[1:02:34]** optimize prompts to bypass safety, using GCG, which is
basically this kind of coordinate-wise optimization algorithm. And
remarkably, optimizing on open models allows this to transfer to other
models. So — I hope these attacks don't work anymore, but at the time, you
can generate some basically gibberish text, and — step by step — a plan to
destroy humanity, and OpenAI's model will comply. Obviously, you could
argue, "Well, is this actually a harmful plan to destroy humanity?" But
that's — at least it did not — the expected behavior is that it

**[1:03:19]** should probably refuse that. Okay, so this raises a kind of a
bigger problem, which is: what is safety? And one thing that makes this
tricky is that many aspects of safety are very contextual — it involves
politics, laws, social norms, and this stuff varies across different
countries. And then, also, if you think about safety, you have to think
about what the risks are. And the risks are actually quite varied as well,
and demand probably different attention. So, hallucination is a risk,
especially, let's say, in medical, or legal, or financial settings. But
this is also correlated with just

**[1:04:05]** capabilities. Like, if you make a model more accurate, it's
correlated with not hallucinating as much. Sycophancy is a problem.
Abetting crimes is a problem, but this might be counter to capabilities.
Inequality — losing critical thinking might be a risk as well, that's
correlated with capabilities. So, if you think about the societal impact of
language models on the world, and think about safety as ensuring that AI
goes well for people, this is actually a much more complex topic than I'll
have time to talk about. And finally, there is a dual-use

**[1:04:51]** aspect of language models. So, cybersecurity agents can be
used to hack into a system, or to do penetration testing and make systems
more secure, which makes talking about whether these are safety risks, or
actually beneficial, a double-edged sword. Okay, so let's move on now to
talk about some broader considerations of evaluation. So, one thing that I
think is an important aspect of evaluation is realism, or ecological
validity — how well does the evaluation capture real-world use? So, in
particular, exams are very far away from real-world use.

**[1:05:37]** You know, Chatbot Arena are from real people, but one could
wonder whether this distribution is the right distribution of people, or
use cases, or not. There are a few benchmarks I'll mention that try to get
at ecological validity, in particular at the use-case level, at least, not
the individual query level. So, there's this OpenAI-created benchmark
called GDPVal. They looked at the top nine sectors according to US GDP, got
a bunch of professionals to create tasks. These are professionals with
about 14 years of experience. And you can see people who are nurses, who
work, you

**[1:06:24]** know, concierge, real estate agents, film and video editors,
and so on and so forth. And they created a bunch of tasks. Here's another
example, in the medical domain, where, up until this point, a lot of
medical benchmarks were based on standardized exams. And in fact, actually,
for humans, this is also what happens — you take these standardized exams,
and you graduate from med school, and then you go operate on — well,
actually there's a longer process, you don't operate on patients directly —
but the point is that, for a language model, you should also not just pass
medical exams and

**[1:07:10]** deploy them directly. So, this was a project where 121 tasks
were sourced from 29 clinicians. And this was meant to represent the type
of things that clinicians would actually ask a language model, which are
quite different from the multiple-choice type of exam questions. Here's
another project which gets at real data. So, you know, if you ask who
actually has the data on what people are doing — it's the model developers.
And so this project is interesting, because it uses language models
themselves to analyze real data. So, in particular, due to privacy,

**[1:07:58]** you can't literally go and look at people's data, but you can
have language models look at the data and analyze and summarize general
patterns. So then the language models could tell you about what types of
people, what types of things, people use Claude for. And, you know,
sometimes realism and privacy are at odds with each other, because,
ideally, you would want to actually sample from the actual query stream,
and that would be very valuable, to assess how well a model is working and
understand deeply what the errors are — but then that runs into privacy
considerations.

**[1:08:43]** Okay, so another consideration, which is another type of
validity, is: how do we know that the evaluations are scientifically
valid? And this was brought up fairly early — train-test overlap. So, in
machine learning 101, you learn: don't train on your test set. Hopefully.
And before foundation models, this was actually pretty clear, right? With
things like ImageNet and SQuAD, and many other benchmarks and datasets,
there was always a train split and a test split, and everyone played the
same game — you train on your training data, you test on your test data.
Life was simple.

**[1:09:30]** And now you have these models that are trained on the
internet, and much more than that, and furthermore you don't know what's
in the data. So this makes evaluation quite different, I think. So, what
can you do here? The first thing you can do is try to infer whether a
model has actually seen the test data. So, this is a neat idea from Tatsu's
group, where you leverage the fact that the order in which the questions
in a benchmark occur should be random. But, if the model prefers some sort
of

**[1:10:17]** particular order that's consistent with a benchmark, that is
probably — it just, like, trained on that benchmark. Another route is that
you can encourage people to report actual train-test overlap. So, this has
to do with norms, right — I think, in general, especially maybe less in
machine learning but more in statistics, you always report confidence
intervals, right? You have an estimate, and you always want to know: how
reliable is that estimate? This is just standard practice. And so the idea
of this position paper is that model providers, if you're going to claim
some

**[1:11:04]** GPQA score, should always provide some justification that
you didn't train on the test set. So, another route is that you can just
give up — it's like, okay, all these evaluations, let's just assume that
people trained on them, assume the worst case. And so the thing that we
have on our side is that you can always define fresh evals. So, there's a
bunch of ideas, like LiveCodeBench, UncheatableEval, which basically scrape
new webpages, or archive papers, or GitHub, and define evaluations that are
past the cutoff date of the language models.

**[1:11:50]** So, this seems pretty robust and good. But one thing to keep
in mind is that timestamps aren't always safe either — if, with code, for
example, I might put something on GitHub that's actually derived from some
other repo. So, how do I know that that's actually generally fresh? Route
four is to use private evaluations — companies, model developers, do this,
right? So, if you're Google or OpenAI, you have your internal codebase,
which you assume is not on the internet, and you obviously wouldn't train
on this data. So you can use it for evaluation.

**[1:12:37]** So, this generally is pretty reliable. And if you're not a
company, you can use your personal writings. I have a bunch of rejected
papers from when I was in grad school that I never put online, so I can use
those. And in the — this type of evaluation, I think, is very favorable
for perplexity evals, because all you need, in perplexity, is a good
dataset, and you can evaluate log probabilities. So, another concern I'll
mention is dataset quality. Remember I showed the leaderboard for SWEBench,
and, you know, Verified —

**[1:13:22]** this is because SWEBench had a bunch of tasks whose unit
tests weren't quite rigorous enough, and other things that got fixed. And
there are many other benchmarks that have gone through this process, like *[Ed:
unclear — captions read "Gia" here; a benchmark name, not recoverable from
the slide file]* GSM8K or MMLU, where people have gone and done a more careful audit and
realized that, "Wait a minute, something's broken around this question."
Okay? So, like this one — there's no curve given in this text, so you

**[1:14:07]** can't answer this question. You know, does the baby have
socks on? There's no way to tell. Okay? And there's also this paper that
showed that there's a lot of problems with agentic benchmarks, which
arguably are even harder to assess, because it's not just — you can look at
the question and the possible answers and look at it — but it's a whole
environment. And there are cases where, with coding, it's great to have
test cases that either pass or don't pass, but, as we all know, test cases
can be incomplete — so you might pass all the test cases but still not have
a working solution. There's another case of this,

**[1:14:53]** where they show that if an agent just outputs the empty
response, it can get like 38%. So there are some trivialities with these
benchmarks that one has to be very careful to avoid. One thing I'll call
out is that there's this tool called Docent, that uses language models to
inspect agent traces to detect problems. So this is kind of a qualitative
response to our very quantitative-heavy way of benchmarking. And I would
always recommend that, whenever you develop a benchmark, or run a model on
a benchmark, you always look at the output and try to audit it, to make
sure you actually think you're measuring what you think you're measuring.

**[1:15:41]** Okay, so, maybe getting a bit more philosophical: what's the
point of evaluation? So, I've talked a lot about different types of
evaluation, and really there's no one evaluation to rule them all. I think
when you think about evaluation, you have to be very clear about what the
purpose is, and I think often this is not really stated clearly. So, for
example, you could be a user or a company that's trying to make a purchase
decision — are you going to go with A or B for a particular use case? Or
you're just a researcher, and you have this intuitive notion of
intelligence, and you want to measure it somehow. Or you want to understand
the benefits and harms, for business or policy reasons. Or you're a model

**[1:16:27]** developer, and you want to get feedback to improve the model.
So, each of these goals will lead you to potentially different benchmarks,
or some combination of evaluation strategies. And finally, one thing to
address, which I hinted at briefly, is: what are we even evaluating? So,
before foundation models, researchers evaluated methods, because we had
these train-test splits, and the only thing that was varying was the actual
algorithm. And today, we're mostly evaluating models and systems, where
anything goes. There are some exceptions — the nanogpt speedrun is
deliberately a way to evaluate an algorithm, in particular how fast you can
train a model — but most of

**[1:17:15]** language model evaluation is about evaluating the actual end
model, which is useful, because that's the thing that's actually going to
get shipped, and that people are going to use. But I think the punchline
is that you should always be very deliberate about declaring what your
goals are. Okay, so there's no one true evaluation — choose what you're
trying to measure. And we talked about a bunch of different benchmarks,
starting with perplexity, exam-based, chat, agentic, reasoning, safety.
They vary in terms of difficulty, realism, how valid they are, and there
are

**[1:18:01]** often trade-offs. It's hard to have something that's really
real, and difficult, and ecologically valid, and has no train-test
contamination. So often you have to compromise on one of these factors,
and, depending on your goals, you have to choose which one you're willing
to compromise on. Okay, so that is the end of this lecture. Next week we're
going to start talking about training data. Okay, see you next time.
