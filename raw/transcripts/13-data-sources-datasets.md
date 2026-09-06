---
title: "Lecture 13: Data I — Sources and Datasets"
lecture: 13
video: https://www.youtube.com/watch?v=-qm0ln33G24
source: copy-edited from the YouTube auto-captions
verbatim_original: not committed - regenerate from the video (see below)
material: ../slides/13-data-sources-datasets.md
---

# Lecture 13: Data I — Sources and Datasets — transcript

**This is the edited transcript.** The auto-captions have been repunctuated,
segmented into sentences, stripped of filler ("um," "uh," "you know," "kind
of," "right?") and false starts, and had mis-heard technical terms restored.
No content was added, removed, or reordered, and every `[MM:SS]` marker is
preserved in its original position — content that was under a marker is
still under that marker.

The verbatim captions are **not committed to this repo** — they are a complete
reproduction of the lecture and none of that text is this KB's own work. They stay
reproducible on demand: the `cairn-kb` skill's `fetch_transcript.py` run on the
video id in this file's front matter, piped through `transcript_to_md.py`,
reproduces them exactly, so the checks below can be re-run by anyone who wants to
verify this edit.

Terminology was cross-checked against
[the lecture's source program](../slides/13-data-sources-datasets.md)
(`lecture_13.py`), which is the authoritative written transcription of this
lecture's content — dataset names, paper citations, author/organization
attributions, and numbers.

**Verification.** All three checks were run in the parent against the verbatim
captions, and all three pass. The results below are the parent's, not the
drafting agent's — an agent's self-report about its own edit is not evidence.

- **Timestamps** — 107 markers, identical sequence, in order. **29 of the 107 are
  `[H:MM:SS]` form**, this lecture running to 1:21:47; a marker regex of
  `\d+:\d+` would silently skip all 29, which is a failure this build has hit
  before.
- **Numbers** — nothing substantive lost. Ten paragraphs differ and every
  difference was adjudicated individually against both texts:
  - four are **year joins** the captions had split — "20 um 18" → 2018, "20 20
    three" → 2023, and two bare "20" false starts before a year (10:05, 10:51,
    54:41);
  - one is a **digit join corroborated by the lecture's own material**: "400 uh
    20 million repositories" → **420 million**, which `lecture_13.py` prints as
    "420M+ repositories";
  - "11 labs" → **ElevenLabs** (30:51);
  - one is a **spoken self-correction removed as a false start**: at 50:49 the
    lecturer says "40 — sorry, 800 gigabytes of text", and only the corrected
    figure is kept;
  - the rest are **name restorations that happen to contain digits** — Books1 and
    Books2 (the material writes them that way), and WebText2.
- **Word ratios** — 84.6% retention, with 3 of 107 paragraphs outside the
  0.72–1.10 band, each read in full and confirmed as filler and false-start
  removal: 25:29 (0.65, an abandoned "let's see how I would have to take that"),
  45:26 (0.71) and 26:16 (0.72). This retention is in line with the same course's
  lectures 8 (84.0%) and 9 (84.6%).

**Two parent corrections were applied after the draft came back**, both found by
the ratio check:

1. **A substantive sentence had been dropped** at 44:40 (ratio 0.56, well outside
   the band). The draft opened the paragraph "website, but still", deleting the
   lecturer's answer to the student question that precedes it — that both a book
   and a website are copyrighted, but *"probably a book author has more of a — if
   they want to go into court, they'll probably be able to protect that better,
   because it's published, than your website."* That is the substance of the
   answer, not a disfluency. Restored; the paragraph is now inside the band and
   overall retention rose from 84.3% to 84.6%.
2. **A word had been inserted** at 25:29. The draft read "this is not necessarily
   a *settled* fact"; the lecturer said "not necessarily a fact". Reverted to what
   was said — the edit is not licensed to sharpen a claim, even a legal-sounding
   one that reads better.

**Restorations made.** These are places where the captions produced a *wrong
word* or dropped/mangled a word, and the text now reads differently. Grouped by
kind; unless noted otherwise, each is confirmed against
[the material file](../slides/13-data-sources-datasets.md).

*Dataset/format/tool names:*

| Caption | Restored | Confirmed by |
| --- | --- | --- |
| "WAT file" (34:42, 35:29) | WET file | material: "Two formats: WARC ... WET: converted to text (lossy process)" — the paragraph describes exactly this lossy HTML→text conversion, not WAT (which is metadata, not text) |
| "Brazilia Parser" (35:29) | resiliparse | material: "Tools to convert HTML to text: trafilatura, resiliparse" — named as a pair with trafilatura, exactly as heard |
| "Data Comp at WLM paper" (35:29) | DataComp-LM (DCLM) paper | material's own section "DataComp-LM / DCLM (2024)" cites the same WET-vs-tools ablation, arXiv 2406.11794 |
| "archive" as the preprint server, throughout (36:14, 41:33 ×2, 42:20 ×2, 43:06, 1:00:54, 1:01:40) | arXiv | material's "arXiv" section; distinguished from the legitimate "GitHub archive" (→ GitHub Archive) which is a different thing |
| "Archive GitHub" (54:41) | arXiv, GitHub | material lists arXiv and GitHub as separate Pile components; the caption ran the two list items together |
| "capture" (8:32) | CAPTCHA | material: "present CAPTCHAs" |
| "GitHub archive" (41:33) | GitHub Archive | material: "GitHub Archive" (hourly event-stream snapshots) |
| "software heritage" (41:33, 1:11:44) | Software Heritage | material: "Software Heritage" |
| "Olmo" (3:10) | OLMo | material: "OLMo from AI2" |
| "Qwen 3.5 397B" (3:10) | Qwen3.5-397B | material: "Qwen3.5-397B-A17B is an instruct model" (the "-A17B" suffix is not spoken, so not restored) |
| "Gwen 3" (1:09:20) | Qwen3 | material: "Qwen3 trained on 36T" |
| "Refined Web" (1:01:40, 1:06:18) | RefinedWeb | material's own section header "RefinedWeb" |
| "fine-web" (1:02:28) | FineWeb | material's own section header "FineWeb" |
| "Doma" (1:17:08 ×2) | Dolma | material's own section header "Dolma"; spelled correctly elsewhere in this same transcript (1:03:13) |
| "Bibliotek" (56:16) | Bibliotik | material: "the shadow library Bibliotik" |
| "Books 3" (throughout) | Books3 | material's own spelling, "Books3" |
| "Red Pajama V1" (1:00:54 ×2) | RedPajama v1 | material's own spelling, "RedPajama v1" |
| "web text" for the GPT-2 dataset (47:43, 51:37, 52:22) | WebText | material's own section header "WebText and GPT-2 (2019)" |
| "web text" for the GPT-3-era expanded version (53:09, 53:54) | WebText2 | material: "WebText2 (WebText expanded with more links)" |
| "books one and books two" (53:09) | Books1 and Books2 | material: "(Mysterious) Internet-based books corpora (Books1, Books2)" |
| "fast text model" (1:07:50) / "fast class classifier" (1:06:18) | fastText model / fastText classifier | material: "Trained a fastText classifier" (DCLM section) |
| "DCLAM" (1:04:01) / "DCL M" (1:07:04 ×2, 1:07:50) | DCLM | material's own section header "DataComp-LM / DCLM (2024)" |
| "data on pool" (1:04:47) | DCLM-pool | material: "Processed CommonCrawl to produce DCLM-pool (240T tokens)" |
| "Nematron" (1:07:04 ×2) | Nemotron | material's own section header "Nemotron-CC (2024)" |
| "Open or Hermes" (1:06:18) | OpenHermes | material: "OpenHermes-2.5" |
| "Eli5" (1:06:18 ×2) | ELI5 | material: "ELI5" |
| "score find" (1:07:50) | score FineWeb | material: "Prompt Nemotron-340B-instruct to score FineWeb documents based on educational value" |
| "PI removal" (1:02:28) / "PI reduction" (1:11:44) | PII removal / PII reduction | material: "Anonymize email and public IP addresses (PII)" |
| "fear it" (43:54, in "you appeal to ... fear it") | fair use | matches "appeal to fair use," used correctly elsewhere in this same transcript (e.g. 19:18, 44:40) |
| "PushShift" (1:03:13) | Pushshift | material: "the Pushshift project" |
| "137 new repos" (1:10:54) | 137 million repos | material: "git clone'd 137M repositories" |
| "open way models" (1:17:55) | open-weight models | standard term, matches "open weight model" used correctly earlier in this same transcript (0:04) and material's "Open-weight models" |
| "the pile" (53:54, 1:00:54) | The Pile | material's own section header "The Pile (2021)" |
| "Stack V2" (1:13:19) | Stack v2 | material's own spelling, "Stack v2" |
| "clone... 137" / "since it was cleared by 2022" (1:10:54) | "since it was clear by 2022" | grammar smoothing only, not a name — "cleared" doesn't parse; "clear" does |
| "paralyze that effort" (1:36) | parallelize that effort | homophone correction — sense of the sentence (spreading work across many people) requires "parallelize," not "paralyze" |

*Person/organization names not tied to a single named dataset:*

| Caption | Restored | Confirmed by |
| --- | --- | --- |
| "Shane Lampray" (9:18) | Shayne Longpre | the paper title itself, "Consent in Crisis," is already correct in the captions; Shayne Longpre (MIT) is the lead author of "Consent in Crisis: The Rapid Decline of the AI Data Commons" (arXiv 2407.14933), the paper material cites for this exact section |

**Restorations made that are NOT attested in this lecture's own material file**,
listed separately with evidence so a reader can weigh them:

- **Read the Docs** (11:38), for "read the docs was also getting hammered" — a
  real, well-known hosted documentation platform (readthedocs.org) that publicly
  reported being hit hard by AI-crawler traffic in 2024. The material file doesn't
  cover this specific anecdote.
- **OAI-SearchBot**, **PerplexityBot**, **ChatGPT-User**, **ClaudeBot** (7:46) —
  the captions read "OAI search bot," "perplexity bot," "chat GPT user," "Claude
  bot." These are the actual, publicly documented `robots.txt` user-agent strings
  for OpenAI's search crawler, Perplexity's crawler, OpenAI's browsing agent, and
  Anthropic's crawler, respectively — phonetically near-exact matches, and the
  kind of list a robots.txt discussion would name. Not in the material file, which
  doesn't enumerate specific bot names.
- **EleutherAI** (53:54), for "a grassroots effort uh from Eleuther AI" — the
  well-known organization behind The Pile (arXiv 2101.00027, cited in material),
  standard one-word branding.
- **ElevenLabs** (30:51), for "11 labs" — a well-known AI voice-synthesis company;
  "11 labs" is a direct phonetic rendering. Not covered in the material file (this
  is a student question, off the lecture's planned content).
- **MPT** (1:18:42) — the captions read "the first LLaMA, uh, MBT, um, and Qwen"
  in a list of comparison baselines, and "MBT"/"MPT" is a one-consonant slip.
  *[Parent note: the drafting agent listed this as its weakest restoration and
  "an editorial reading, not a certain identification". IT IS BETTER ATTESTED
  THAN THAT, and has been promoted to corroborated. MPT is one of the five series
  in `comma-results.png`, the very figure this passage is describing — the legend
  reads "Comma v0.1-1T, LLaMA, MPT, RPJ-INCITE, Qwen3", which was read directly
  off the image during the figure audit, and the name appears 18 times in the
  material file's description of it. The lecturer is reading the chart's legend
  aloud, so all four names he lists are printed in front of him.]*

**`[Ed: unclear]` markers left in place, because guessing felt riskier than
flagging:**

- **7:46**, "Cora bot" — sits in the same robots.txt list as OAI-SearchBot,
  PerplexityBot, ChatGPT-User and ClaudeBot (all restored above), so it is very
  likely a mangled crawler name, but no candidate name matched with confidence.
  Left verbatim with a flag rather than guessed.
- **30:51**, "voice dating at 11 labs" — "11 labs" is confidently ElevenLabs (see
  above), but "voice dating" does not parse as a coherent ElevenLabs product or
  feature. Left verbatim with a flag.
- **43:06**, "Are you using crawlers like Open or crawl and stuff like that?" —
  part of a student's rambling question about crawler tooling; "Open or crawl"
  does not resolve to any tool name with confidence (Apache Nutch, the crawler
  material names for Common Crawl, was considered but doesn't fit phonetically).
  Left verbatim with a flag.

**Student questions.** Following the KB's convention (see lecture 12's
transcript), wherever the recording makes clear someone from the floor is
speaking — as distinct from Percy paraphrasing a question in his own words —
the transcript marks it `*[Question from the floor: ...]*`, using the captions'
own words with only light punctuation. Unlike lecture 12, this lecture's captions
mostly capture only Percy's own paraphrase of what he was asked ("the question
is...") rather than the student's actual words, so most of the "any questions?"
exchanges in this lecture are left as ordinary running prose — there is nothing
of the student's own speech to bracket off. Two exceptions:

- **30:51–31:37**: a `>> [clears throat] >>` speaker-turn marker in the raw
  captions brackets an inaudible exchange with no recoverable words; marked
  `*[Question from the floor, inaudible]*` rather than invented.
- **43:06–43:54**: two questions where the captions do run the student's actual
  words together with Percy's, without a speaker break — "Are there restrictions
  on data generated by models?" / "Yeah, by using that data" (about synthetic
  data), and "Are you using crawlers like [unclear] ... how do you ensure that
  those types of websites [with pirated content] don't show up in your crawl?"
  (about filtering shadow libraries out of a crawl). The second question's own
  words span the 43:06/43:54 marker boundary and are marked
  `*[Question from the floor: ...]*` / `*[continued from the floor: ...]*`
  accordingly, split exactly where the captions split it.

---

**[0:04]** So, today we're going to talk about data. Where we are is that you
know how to train a model given data. Last time we saw evaluation, so we know
what a good model is. For the next two lectures this week we're going to focus
on what data we should train on. I want to argue that data is the most
important thing to get right in language models, and one justification for
this is if you look at what companies actually disclose. Here is the Llama 3
paper — they have full transparency into architecture, well, of course,
because it's an open-weight model. They even tell you about the training
procedures. But they don't say anything about their data.

**[0:51]** They just say, we train from a variety of data sources and do some
stuff. There are good reasons for this secrecy. One is that data is kind of
your competitive secret sauce — competitive dynamics. You don't want to let
your competitors know what you're doing. The second thing, which we'll talk a
little bit more about, is copyright liability — you don't want to get sued if
you tell people you're training on certain types of data. Data is a long
topic in machine learning. Before foundation models, data work meant you
annotate labeled data for supervised learning. Nowadays there's less manual

**[1:36]** annotation, at least in pre-training, but there's still a lot of
curation and cleaning work that needs to be done. Fundamentally this hasn't
really changed. Data has always been a bottleneck and always will be a
bottleneck, because it's in some sense a long-tail problem, and that scales
with human effort. If you have a lot of people trying to work on a problem —
well, there's only so many people who can work on architectures and systems,
but data, especially if you're training a foundation model that's trying to
do all these things, you can parallelize that effort easily. That's why these
data teams and these model developers are actually quite big. So data comes
in at different stages of

**[2:22]** the pipeline. Today we're focusing mostly on pre-training. In
pre-training, you take raw data — these are documents from the web. Then you
move into mid-training, where you train on more high-quality data to enhance
certain capabilities, give long context, things like that. And then finally
post-training, where you're training on things like chat transcripts, or if
you're doing reinforcement learning, you get some environments that you're
training on — this becomes more task-specific. In practice the lines are a
bit blurry, and there could be more than three stages, but this is sort of a
basic template. The trend is that we go from training on large amounts of
low-quality data to smaller amounts of high-quality data.

**[3:10]** As a result of this process, you'll see in the literature some
people talk about a base model. This usually means after pre-training and
mid-training. Instruct models or chat models are after post-training. But of
course the lines are becoming blurry enough that what a base model even is
anymore is kind of unclear, and more recently, for the largest models there's
no base model at all — there's just Qwen3.5-397B and that's it. You don't see
the intermediate checkpoints. For models that are open source, like OLMo from
AI2, you can see everything that's happening, and so we'll study some of
these models. So for example, in pre-training, there's a

**[3:57]** bunch of sources — we'll talk about what these mean, but things
like web pages, academic papers, math pages, proofs, and so on. Then there's
mid-training — higher-quality web data, instruction data, a bunch of
synthetic data often goes here. And then finally post-training, there's a
bunch of chat logs, more math and reasoning and coding, and you do safety at
this point as well. So what are all these datasets? How do you choose them,
and how do you process them? That's the main question. In the spirit of the
class, we're going to talk about data — I'm going to start from scratch. So,

**[4:42]** where does data come from? You might hear in the hallway, "Oh,
language models are trained on the entire internet." First of all, this
doesn't really quite type-check, because for that to be true, it would have
to be an agent — like an RL agent that goes on the internet and does stuff —
but that's not actually how pre-training works. Slightly more accurately,
it's trained on the public World Wide Web. This is not quite accurate either,
for reasons I'll explain. First of all, the web is actually a bunch of live
servers that just exist in the world, and you can connect to them. Take any
website — you can go and download a page.

**[5:27]** Or you can send a request and get back a response. You can't
actually train on these live servers, unless you're training an RL agent. So
typically what you do is you have a crawler — or someone builds a crawler,
not necessarily you. The crawler needs to discover what web pages are out
there, starting with a seed set, and it downloads the discovered web pages as
it's crawling. But still, you can't just run a crawl and download all the web
pages on the internet, and there are a few reasons for this. One is that a
lot of the web content is actually dynamic — especially these days, a lot of
sites are apps. The URL isn't even as full a

**[6:14]** specification of the content — it's just an app that you interact
with, and you often need to click buttons or submit forms to access content.
For example, if you're on Discord or something, it's not like you can just
crawl Discord and get all the content out of it. So that's one kind of thing,
and a lot of content is actually known as the deep web, which isn't just web
page crawling and following hyperlinks — the very traditional model of web
crawling. The second thing is authentication. Some pages need a login and an
account, and generally

**[7:00]** you have to pay. For example, there's Facebook, X, LinkedIn, New
York Times — there are huge amounts of content locked up behind these walled
gardens. These are technically on the internet, on the web, but you can't
just build a crawler and go crawl Facebook. I guess if you're Facebook —
well, you don't need to crawl Facebook, you have the data and you can train a
model. Or if you're xAI, you can train on X. But if you're anyone else, you
can't actually access this content. Okay, so suppose you don't have the
authentication issue — there are still potential restrictions here. There's
something called robots.txt.

**[7:46]** This is a file that's usually placed at the root level of a
directory, such as nytimes.com, and it basically tells you what you're
allowed to crawl and what not. A lot of things are disallowed — things like
OAI-SearchBot or PerplexityBot, *[Ed: unclear — heard as "Cora bot"]*,
basically ChatGPT-User, ClaudeBot, and all these things. This is not a legal
restriction — this is just, you're supposed to be a good citizen and look at
robots.txt, and if your name is on that list, then you should not crawl.
That's basically the contract — not even a contract. That is the

**[8:32]** thing you're supposed to do. A lot of websites now use something
like Cloudflare to detect and block bot activity. Sometimes you go to a
website and it gives you a CAPTCHA — that's because some algorithm has
detected that you might be a bot, and it's making sure you're human. If
you're actually a bot, well, you could try to get around this, but generally
this is an obstacle in your way. A website might block certain IP addresses
or countries, and they might have rate limits. So there are technical
restrictions on what you can actually crawl. And then there are also legal
restrictions. So,

**[9:18]** websites have terms of service that say, if you go to a website,
you have to obey this contract to use the website. The terms of service will
often say, if you are a bot, go away — you cannot use the content of the site
for AI training, or whatever it might be. And also, even if the terms of
service don't say anything, the content of the website might have a license,
or you might not have a license to train on it — we'll talk more about this
in a bit. So there are legal restrictions on what you're allowed to train on,
and these things are also evolving over time. There's a nice paper by Shayne
Longpre called "Consent

**[10:05]** in Crisis." What they did is they examined the restrictions —
both the technical restrictions, robots.txt, and the legal restrictions, the
terms of service — for various URLs in common datasets, which we'll talk
about later. And the conclusion was that the restrictions have increased over
time. Here's a graphic that shows this — the top one shows robots.txt, and if
you just look at the red, that's the full restrictions. Up until 2023,
everything was fairly constant, and then by mid-2023, all of a sudden you see
the fraction of websites that have full restrictions has grown to almost 50%.

**[10:51]** And for terms, there's a similar trend, where in 2016 no one
really put any terms on their pages, and now most pages put some terms, and
most of the terms say you can't use this for AI. So even though it was
possible to crawl the internet back in, say, 2020, the internet that you can
actually legally crawl is much smaller. Of course, these are guidelines, and
sometimes, either intentionally or unintentionally,

**[11:38]** you could violate these things. Sometimes you get into these
situations where — there's this guy, I forget the name of the site, who was
complaining about Anthropic crawling them, hitting their servers a million
times in 24 hours, which was not good. And then Read the Docs was also
getting hammered. So there was a period — which I think has hopefully been
corrected by now — where crawlers were hammering websites. And what's
interesting here is that even before talking about copyright issues, there
are problems with crawling, because either you're violating terms of service,
or

**[12:25]** robots.txt, or you just generate a lot of server load, which
costs money for the person hosting it and also degrades service for everyone
else trying to use it. So there are issues with crawling, and then of course
there's copyright, which we'll talk about soon — that's a whole other complex
can of worms. And then finally there are these things called shadow
libraries, which are also part of the web. Examples are LibGen or Anna's
Archive, and these completely disregard copyright and bypass paywalls.
Someone has collected a ton of data —

**[13:11]** usually books and articles that are all copyrighted and that
people have to pay for, and made them available for free. There have been a
lot of takedown orders, lawsuits, and these get blocked, but they're
circumvented because they just make servers in other countries. The people
doing this argue that this is making freely available what should have been
free. But generally, from a legal perspective, this is piracy and copyright
infringement, and we'll come back to this point later. So there are a lot of
books and papers on these shadow libraries. So, the summary so far —

**[13:57]** the internet is a huge, messy, scary place, and you can't
actually get all of it — there are technical restrictions, there are legal
restrictions on what data one can access. So next time someone says, I
trained on the internet, you can tell them all of these things you just
learned. Now let's go to the second part — what data can we use? This has to
do with copyright. So suppose you were able to obey the terms of service, and
you're really a good citizen and you obey the rate limits — now there's still
a question of, are

**[14:43]** you allowed to train on this data? This is an ongoing question
that has not been fully resolved, but there have been a bunch of developments
in the last year. The legal context for thinking about this question falls
into intellectual property law, which is a system that is trying to
incentivize the creation of intellectual goods. Remember, the spirit of the
law is not just to technically say no to everything, but to incentivize the
creation of intellectual goods. And there are many forms of intellectual
property that are covered.

**[15:29]** There's copyrights, patents, trademarks, and trade secrets. The
thing that's most relevant for language model data is copyright law.
Copyright law has a fairly long history, going back to the 1700s in England —
1709 was the first time that governments and courts started actually
regulating copyright, for this goal of incentivizing innovation and creation.
And in the US, more recently, the Copyright Act of 1976 is what defines a lot
of modern copyright. Copyright protection applies to original works of
authorship

**[16:14]** fixed in any tangible medium of expression, et cetera. So, a few
notes. Not everything is copyrightable — in particular, collections are not
copyrightable. For example, if you have a telephone directory, unless there
is some creativity in how you arrange things, this is not really
copyrightable. And furthermore, copyright applies to an expression, not the
idea. You can't copyright an idea — you can't copyright the quicksort
algorithm. You can copyright an implementation of the quicksort algorithm,
but not the quicksort algorithm.

**[17:01]** One thing that happened in 1976 is that the barrier to getting a
copyright was relaxed a lot, so more things are copyrighted. It used to be
that you had to publish something to have it be copyrighted — now it just has
to be fixed. What that means is that registration of any sort is not required
for copyright protection. This is in contrast to patents — for patents, you
have to go pay a lot of money and file a patent. But the threshold for
copyright is very low — for example, you put something on your website, it's
copyrighted, that's it. Now, if you want to sue someone, you have to go get
it registered, but it only

**[17:47]** costs $65, which is much smaller than the lawyer fees you'll
probably pay. So it's really still a very low bar. One thing to know about
copyright is that it lasts 75 years, and then the copyright expires and that
content gets released into the public domain, and everyone can use it freely.
A lot of people who lived in the past — all of their great works are in the
public domain, which is great for everyone. The rationale here is that you're
incentivizing innovation. So, when an artist or a creator creates

**[18:33]** something, you want to protect it — you don't want to have
someone else just take it. So you protect them for a while, but after 75
years, presumably it's not worth protecting, or they've passed away or
something, so it doesn't make sense to keep things copyrighted anymore. So,
basically, everything on the internet is copyrighted. So you're saying, "Hm,
wait a minute — does that mean if I train on anything, that's a copyright
violation?" Well, not necessarily. Everything is copyrighted, but you can use
copyrighted works, and the way you do it is: A, you either get a license for
it, or B, you appeal to the fair

**[19:18]** use clause. A license, from contract law, is something that a
licensor grants to a licensee — basically it says, don't sue me, you can use
this work in the ways that the license permits. There's a really nice license
called Creative Commons, which allows something to act like it's in the
public domain — it enables free distribution of copyrighted work. Examples
include Wikipedia, Open Courseware, Khan Academy, all these things. It was
created in 2001 to essentially bridge public domain and

**[20:05]** existing copyright, because otherwise you have to wait 75 years.
But what if the creator says, actually, I want people to use it? They now
have a legal means of saying, I put a Creative Commons license on it — it's
going to act like it's in the public domain, and everyone can use it. So, so
far, things in the public domain and things that have Creative Commons
licenses — you can use those. But there are many other things where it's not
a Creative Commons license, it's not in the public domain — you have to get a
license. So if you have money, you can pay someone

**[20:51]** to give you a license. So there are a bunch of deals between
model developers and content platforms that license the data for training
foundation models. So that's the first thing — you can get a license, either
Creative Commons or a paid license. Now, if you don't want to pay, you can
appeal to fair use, which is a more complicated matter. This is Section 107
of the Copyright Act. Basically, fair use says, I can use this anyway, even
if I don't have a license, and there are four factors that determine whether
fair use applies. The first one is the purpose or

**[21:39]** character of the use — what are you trying to do with it? None
of these are hard rules — they're just tendencies, which have to be weighed
in court if it comes to that. For example, if you're trying to do something
for education, that's more likely to be fair use than if you're trying to
sell and make money off of it. If you transform something, that's more
favored than if you just literally host an identical copy. The nature of the
actual copyrighted work is also important — things that are more factual are
more likely to be fair use than fictional works, because if you write a page
of facts about World War II, that's less. There's

**[22:25]** less protection over my really creative poem. And then there's
the amount, or portion, of the original work — using a snippet is favored
over using the whole work. If you just take a little bit, that's not as bad
as taking everything. That's fairly straightforward. And the final thing
relates to the original motivation — why copyright at all? It's to protect
and align economic incentives. So, the effect that the use has on the market
for the original work. If you take an author's work and you provide

**[23:11]** an alternative and that decreases the amount the original author
could monetize their work, that's seen as potentially bad. But if you're
transforming it in a way that adds something, going to a new market or doing
something else with it, then that's going to be treated more favorably.
Here are some examples of fair use: you can watch a movie and write a summary
of it, you can reimplement an idea like an algorithm without copying the
code. And then there was this famous lawsuit between Authors Guild and
Google — if you go to Google Books, you can see snippets of various books,
which are copyrighted.

**[23:57]** There was a huge deal about whether this was deemed fair use,
because you're sort of redistributing other copyrighted work. And
eventually — meaning after 11 years — this lawsuit was settled in favor of
Google, which is why this thing exists. And this has set some precedent for
thinking about whether language model training is fair use. One thing to
note is that copyright is not about verbatim memorization. For an ML
audience, this might be a little bit new,

**[24:43]** because a lot of papers are focused on verbatim memorization, but
that's one way you can violate copyright, not the only one. For example,
plots and characters can be copyrightable — you can copyright Harry Potter,
the character, not any particular book or anything. But there are some
exceptions — for example, if you're parodying something, you're creating
something that looks like a derivative, but as long as you're making fun of
it, it's actually more likely to be fair use. So, a lot of copyright is about
semantics — it's definitely not about n-gram overlap — and also about the
economics. So, what is the implication for

**[25:29]** language models? First off, copying data — even if you're not
training, the mere fact of copying, which is in the word "copyright," is
potentially a violation already, even if you don't do anything with it.
Training a model — this is not necessarily a fact, but it
intuitively has a transformative flavor, because it certainly seems different
than just re-hosting another work. It's doing something transformative.

**[26:16]** In some sense, the models are trained on this data as a means to
an end — we're trying to extract the general idea, learn about the world and
how it works, rather than just the concrete expression. The other thing is
that, regardless of copyright, language models can definitely affect the
market. Remember, that's the fourth factor for fair use — by negatively
affecting the market, you're more likely to be ruled not fair use. So fair
use is a little bit slippery.

**[27:02]** Suppose you have fair use, or you have a license — remember that
there's still terms of service that can prevent you from actually getting a
piece of work. For example, YouTube has all these videos which are licensed,
but the terms of service prohibit downloading the videos using a bot, or
scraping — so that's another layer. There are multiple layers of restrictions
here. Let me talk a little bit about where things are on the legal landscape.
Back in 2023, New York Times filed suit against OpenAI, saying, "You trained
on our news articles, and look, here's

**[27:47]** evidence we were able to actually prompt ChatGPT to generate a
news article almost verbatim." I don't think this — this is still pending.
There's a case against Anthropic, where the allegation was, you pirated
millions of books and trained on these books to make Claude. Just last year,
there was a kind of landmark ruling that said, actually, this particular
instance of training is fair use — however, Anthropic, you're still in
trouble because you pirated all these books, and that is a no-no. So, notice
that this has nothing to do with training —

**[28:33]** just the mere fact of pirating — which is not a new thing — is
illegal. Anthropic, interestingly, had also bought and scanned all of the
books, which is actually fair use. The court said, you're allowed to scan —
you buy a bunch of books, rip off the binding, scan and digitize it for your
own use, that's fair use. But that doesn't absolve you of your sin of
pirating — you can't pirate and then buy it and say, actually, never mind
what I did first. So the outcome is that Anthropic paid $1.5 billion to
settle with the authors — that's about $3,000 a book. There's also a lawsuit
against

**[29:20]** Meta — the allegation was that you trained on our books, and
this was, as we'll see later, actually revealed in the Llama paper. The
judgment, which came right after Anthropic, was: yes, training is fair use.
But they also torrented some books, so that's still pending — and if
precedent holds, that's probably also not going to be good for Meta. So the
summary so far is that training, so to speak, has been deemed fair use, or at
least has not been deemed not fair use.

**[30:06]** The new rulings have so far been narrow — it's not to say that any
training on any copyrighted content is fair use, but in these cases, it's
fine. Pirating books is clearly illegal — I guess we already knew that. But
this is still a very active and evolving area. Any questions about the
sources of data? Before we get into the actual datasets, I just want to set
the stage with this more general social context.

**[30:51]** Yeah. The question is what I think of *[Ed: unclear — heard as
"voice dating"]* at ElevenLabs. I don't know too much about that, so we can
talk offline. Yeah. *[Question from the floor, inaudible]* Yeah, so the
question is, what happens with time if you have one license and

**[31:37]** then it gets changed — what happens? So, first, I'm not a lawyer.
However, second — with the first license, I believe you're still allowed to
train on it, but what happens is that the license usually doesn't apply to
just a fixed set of documents. Take Reddit, for example — there's constantly
new content being created, and the license being changed means you wouldn't
be able to train on the later things. Okay, so let's look at various sources
of data. Let's go back to crawling. Most model developers have

**[32:23]** their own crawler, because they want full control over what the
data is. Fortunately for the rest of us, if you don't want to build your own
crawler, there's something called Common Crawl, which has been around since
2007. Every month they run a web crawl and get about 3 to 5 billion web
pages — each crawl has some overlap with the previous crawl, but they try to
diversify and get new pages. So there are 300 billion pages so far, which
seems — this was from their website — it seems a little big, because if you
multiply this number by 20, you don't really get to 300 billion, but that's
what they say.

**[33:09]** So how many URLs are out there? That's really hard to estimate.
The Google search index is at least 100 petabytes, according to them, and
each Common Crawl dump has about two billion web pages — that's about 372
terabytes, and this isn't including images, this is mostly just the text.
Crawling is conceptually straightforward, but all the gory details are in the
implementation. You start with a bunch of URLs and then you iterate — it's
basically graph traversal. You pop a URL

**[33:55]** from a queue, you download it, then you look at all the
hyperlinks on that page and add them to the queue. Generally this is done in
parallel over many machines, and there are many decisions here — which pages
to download, respecting robots.txt, not overloading the server. Sometimes
websites change, so you want a policy that will download frequently-changing
pages, but not spend time downloading pages that don't change — there's some
policy around that. And, like I mentioned before, URLs are dynamic, so
sometimes the same URL leads to

**[34:42]** different content depending on some state of the browser, but
also, many different URLs might lead to the same content, so there's a lot of
duplication that happens if you're not careful — mirror sites in particular
are explicitly about duplication. Common Crawl releases their dumps in two
formats. One is a WARC file — this is the raw HTTP response. Remember, the
HTTP protocol is that you send a GET, here's a URL, and out comes an HTTP
response — the WARC file is that response. They also do some processing and
convert it to a WET file, which is

**[35:29]** necessarily a lossy process. It turns out this isn't necessarily
the best way to get text out of the web — for HTML to text, there are many
tools for this, including trafilatura and resiliparse. And the way you
convert it actually does matter. This is from the DataComp-LM (DCLM) paper,
which we'll talk about a little — there's an ablation where you look at the
WET files that Common Crawl releases versus these other tools, and it seems
like trafilatura and resiliparse are better. So one thing you can do is just
rely on general web crawls.

**[36:14]** But often the web is not a uniform place — it's not like there's
a bunch of websites and you just get some random fraction of them and that's
your dataset. There are specific pockets of really interesting high-quality
content, and I'll talk about three of those: Wikipedia, GitHub, and arXiv.
Wikipedia, all of you know about — it's been around since 2001, and now there
are 67 million articles across all these different languages. And Wikipedia
is not all content by any means — you can't have original thought in it,

**[36:59]** because everything has to be referenced and cited. So, in some
sense, you could argue that Wikipedia doesn't contain anything that isn't
already on the web. Well, actually, that's not true, because they can also
cite books, which, obviously, you can't easily get. So Wikipedia is that
good. And also, there are articles based on notability — not everyone can get
a Wikipedia article about them. Anyone on the internet can write this
content — this was the radical thing about wikis, and it's sort of still a
miracle, I

**[37:44]** think, to me, that this actually works. It just happens that any
vandalism gets reverted by administrators, or bots, these days. There's a
small number of Wikipedians — like any peer-production system, there's a
small set of people who do most of the work. For example, this guy has like 5
million edits. And one thing about Wikipedia is that every few weeks they do
a periodic dump — basically, take all of Wikipedia, package it into a nice
tar file, and then you can just download that. This is important, because
you don't need to crawl Wikipedia — in fact, they don't want you to crawl
Wikipedia, they want you to download this instead. And this is much

**[38:30]** better than crawling. As a fun aside, or just good to know —
there's this paper that shows you can actually poison Wikipedia. You might
think that vandalism gets reverted, so if an attacker tries to do something
fishy, it'll just get rolled back. But what Carlini realized is that these
periodic dumps happen at a regular cadence, so what you can do is go in and
edit it right before the dump happens. Then the dump happens, and then the
edits get rolled back, but the dump still has the malicious content.

**[39:15]** And if you're able to inject things into web pages, that means
you can cause — there's work that shows you can cause the model to, let's
say, ascribe negative sentiment to any trigger phrase, like "the iPhone." So
my one takeaway here is that, if you consider adversaries, even so-called
high-quality content might contain bad things in it. I think this has been
fixed since then. GitHub is a good place for code, and code is important, not
just if you want coding capabilities for your language model, but if you want
general

**[40:01]** reasoning. GitHub, all of you know — again, it's a live service
for hosting code repositories, been around since 2008, about the same age as
Common Crawl. It has 420 million repositories, 28 million of which are
public. Each repository — it's not a file, it's a directory with commit
history, issues, pull requests, and comments. Code generally has a lot of
duplicates, because of either copying code or forking code. And GitHub has
deemed that you're

**[40:46]** allowed to train on any public repository with a permissive
license — so MIT or Apache licenses are fine. Remember, there are two types
of data when you talk about GitHub. There's the repository data, which you
can just go and download through the GitHub Git protocol — again, you
shouldn't scrape GitHub, you should just download the repository. And then
there's the metadata associated with each repository — issues, pull requests,
and so on — which is made available via the GitHub Archive, which gives you
hourly snapshots of this event stream, which I think is really interesting
data. Basically every single comment, or star, or action on GitHub gets
recorded.

**[41:33]** There's this Software Heritage Foundation that's focused on the
repository, not the metadata — and GitHub isn't the only place where code
shows up, there's also smaller entities like GitLab, Bitbucket, and so on,
and they aggregate those repositories too. Okay, so arXiv — just very
briefly, everyone knows arXiv. This has been around as a site that allows
people to share papers since 1991, started with physics, and now has a lot of
other areas in it. Each of the 3 million submissions has the metadata, a PDF,
and optional LaTeX

**[42:20]** source. So already you should be thinking, well, what does it
mean to train on arXiv? It's not entirely clear, because you have to convert
the PDF into text, or you can use the LaTeX source, which is a bunch of
files. It's not peer-reviewed, but there is some approval process, and
authors can choose to maintain the rights, or put it under Creative Commons —
so everything is actually very clearly licensed. All the metadata is under a
permissive license, so you can use that regardless, and then you can go look
at all the papers that are Creative Commons licensed, download those, and use
them. And I think most arXiv papers are Creative Commons.

**[43:06]** And again, this is available — you don't crawl arXiv, you just
bulk-download it from some website. Okay, so any questions about sources of
data?

*[Question from the floor: Are there restrictions on data generated by
models?]*

Generated by models?

*[continued from the floor: Yeah, by using that data.]*

Oh, I see — we'll talk about that later on. So the question is, what about
model-generated, like synthetic data — are you allowed to use it? The short
answer is probably yes.

*[Question from the floor: Are you using crawlers like *[Ed: unclear — heard
as "Open or crawl"]* and stuff like that?]*

**[43:54]** *[continued from the floor: I guess, since you mentioned earlier
that there are websites that have a lot of pirated books on them — how do you
ensure that those types of websites don't show up in your crawl of the
internet?]*

Yeah, so the question is, there are a lot of websites with pirated books — if
you're just doing a web crawl, you can't look at all the websites, so how do
you make sure that everything is kosher? And that's part of the difficulty —
you can't. So there's most likely, in Common Crawl, copyrighted books and
content that you're not supposed to train on, and you can appeal to — well,
you can appeal to fair use. Remember, everything is copyrighted. Books are
not remarkably different from your

**[44:40]** website. For example, both are copyrighted. Probably a book
author has more of a — if they want to go into court, they'll probably be
able to protect that better, because it's published, than your website, but
still. And I'll talk a little bit about this
later — if you really want to be careful, you can do something I'll show
later. So, what I'm going to do now is talk about the various datasets that
are actually used in building models, starting all the way back to 2019.
We've talked about the origin of data and sources of data, so you have a feel
for the general data landscape.

**[45:26]** So let's start with BERT. BERT was from 2018, and back then they
just trained on Wikipedia and books. So what is "books"? There's this website
called Smashwords, created to let anyone publish an ebook, and they had about
half a million books by 2024. In 2015, there's this paper that scraped
Smashwords — took the books that were free and made a corpus. This was back
in the innocent days when no one was paying attention to any of

**[46:11]** this. So this BooksCorpus was around for many years in the
academic community. Since then it's been taken down, because it violated the
terms of service — just because it was free and you could get it doesn't
mean it was legally allowed. One thing to note about BERT is that the
sequences were documents rather than sentences — all the language modeling
research before that was focused on sentences. So in 2019, GPT-2 came
around — so what did they do? So they

**[46:58]** wanted to get high-quality web content. At that time people knew
about Common Crawl, but somehow the Common Crawl data was too messy. So what
they did was come up with this clever idea, where you look at pages that are
outgoing links from Reddit posts with greater than three karma — good posts
must link to good websites. You grab those pages and you get 40 GB of text, a
million pages. They never released this data, but there was an open
replication of WebText, which people used quite a bit.

**[47:43]** So that's GPT-2. As you're going through this, think about how
there are many methods of filtering and collecting data, and it's
interesting to look at these design choices. Okay, so CCNet was developed at
that time by Facebook. The goal was to create large, high-quality datasets
for pre-training, and they were especially interested in low-resource
languages — they didn't want some very manual process that only worked for
English. So they did a bunch of things. They did deduplication and

**[48:30]** language identification — they trained a classifier to detect
the language and keep only the language you're looking for. And for quality
filtering, the idea they had was to keep documents that look like Wikipedia.
Wikipedia at the time was deemed good, high-quality content, and they used a
language model trained on Wikipedia, and scored the probability of a new
document under that language model to gauge how Wikipedia-like it was. So
rather than outgoing links to Reddit, they used a language model on
Wikipedia. And they showed that, because you can get much more data,

**[49:16]** you can actually outperform just training on Wikipedia, and this
was a tool that would later appear in some other papers. So here's another
work — this is from Google, 2019, and it's called C4. This paper is actually
more famous for the T5 model, which pushes the idea that you should treat all
NLP tasks in the world as text-to-text. But actually a big contribution of
this paper, which is a very long report, was the C4 dataset. And the
observation, which was very clear at that time,

**[50:02]** was that Common Crawl is mostly not useful for natural language.
There was this idea that you had these small datasets, and they were
high-quality, and then you had Common Crawl, which was a mess — any attempt
to train on Common Crawl just led to junk results. So people were thinking,
"How do I get a larger but high-quality subset of Common Crawl?" We saw the
Reddit idea, we saw the Wikipedia-like language model idea — C4 used a
different idea, which is, "Let's just define a bunch of rules," which turned
out to be fairly effective. So basically, they keep lines that end in
punctuation, have more than five words,

**[50:49]** remove pages with fewer than three sentences, remove pages that
contain any bad words, remove things like terms-of-use boilerplate.
Interestingly, remove the curly brace, which filters out a lot of code —
clearly, at that time, they weren't thinking about code models. And they
filtered out anything that wasn't English. So the end result is that you have
156 billion tokens — 800 gigabytes of text. This is much larger than the
GPT-2 dataset, which was only 40 gigabytes of text. A bit later, there was
some analysis done on C4, which showed — here's the

**[51:37]** websites that were represented — a bunch of Wikipedia, and
patents was a very common thing. They also released a WebText-like
document — they used the same idea of outgoing links from Reddit posts with
greater than three karma, and filtered through those pages. Using this method
they were able to get 17 GB of text using 12 Common Crawl dumps. Remember,
WebText was 40 GB, which means that Common Crawl is maybe incomplete, or
doesn't

**[52:22]** have everything, because if it had everything, you should be
able to hit 40 GB. And they showed that, at that time, you could use this
data and improve on a bunch of NLP benchmarks. So we've seen using rules,
things that look like Wikipedia, Reddit links. Now let's talk about GPT-3.
For GPT-3, the dataset — they used Common Crawl, did their own processing.
There's WebText2, which was essentially the GPT-2 dataset but expanded —
notice there's some overlap, Common Crawl does overlap with this, but this is
a subset, a more targeted distribution. There's this

**[53:09]** — in the paper it's described as Books1 and Books2, which are
mysterious internet-based books corpora — this remains a mystery, exactly
what it is — and Wikipedia. So the result was about 500 GB of text, 400
billion tokens. And to do the Common Crawl processing, they trained a quality
classifier to distinguish things they deemed to be high quality from the
rest, and they did some fuzzy deduplication, because WebText2 and Common
Crawl had dupes.

**[53:54]** So this paper followed the idea of using a classifier to do
quality classification. After GPT-3 came out — I think this was a big
event — there were a number of efforts to try to do things in the open. One
of the initial efforts was The Pile, which is a grassroots effort from
EleutherAI, where they had a bunch of people in this Discord jamming and
figuring out high-quality sources, and they came up with this list in the
end. And today this is still pretty interesting and diverse. It has the
Common Crawl, has

**[54:41]** PubMed, this thing called Books3, which we'll talk about, arXiv,
GitHub, which we talked about, Wikipedia, IRC, another books corpus,
philosophy papers, and so on. And they included this Enron emails dataset —
Enron went bust in 2002, and all the emails were released, and this is one of
the few email datasets we have, which is a weird distribution for email, but
that's what you get. So, Project Gutenberg — let's talk about books a little
bit.

**[55:28]** Project Gutenberg was started in 1971, and it only includes
books that received copyright clearance — so, mostly books in the public
domain. And there's this packaging of Gutenberg into PG-19, which is
Gutenberg from before 2019 — which is, I guess, most of them, because for
them to be in the public domain you have to wait 75 years. So, Books3, which
is this interesting dataset that appeared in The Pile — this is described as

**[56:16]** 200K books from a shadow library called Bibliotik — it includes
all books from your favorite authors. And at that time, again, 2020, no one
was paying attention. It just sat there, people used it to train models. And
since then it's been taken down, so you cannot, or should not, use Books3
anymore. We'll come back to Books3 again. Stack Exchange, you've probably
used quite a bit — although, I guess these days maybe less so, because of AI.
There's a bunch of user-contributed questions and answers, starting 2008. And

**[57:04]** one thing that's interesting about this dataset is that it's a
Q&A format, and this is quite close to a real application. Normally you
think about pre-training data as, here's Wikipedia articles, just raw text.
But some parts of the web actually look supervised — like what you would want
to ask your language model — which I think maybe helps the model learn
certain types of question-answering behavior. So not everything is super
magically emergent — it's that this type of data does exist on the web. So,
there's also

**[57:51]** this data has metadata — like the number of votes — which is
useful for filtering. And again, this data is released in data dumps, which
lets you just download it without crawling. So, in 2021, there was another
paper from DeepMind — they trained a model called Gopher, which was never
released, and was actually sort of subsumed by Chinchilla. But the
description of the data, I think, is actually really good — you should read
this paper, because it's very thorough in how it describes the data
processing. Well, except for the parts where they don't tell you what's in
the data. So, there's this thing they created

**[58:36]** this MassiveWeb, and they had C4, which, remember, was the
dataset we looked at before from Google. They trained on books, news, GitHub,
and Wikipedia — they don't say anything about how they got that. For
MassiveWeb, they kept English, deduped, and their quality filtering was also
based on manual rules, and part of the reason for this is that they had more
control over it. So, as you can see, there's this sort of division between
people who wanted to use rules and people who wanted to use

**[59:22]** classifiers. So, as the name MassiveWeb suggests, they got a
massive amount of text, but their model was only trained on a very small
fraction of this dataset. So, in 2022, LLaMA 1 came out, and this was
actually a very nice paper in that it detailed their data processing — this
is probably one of the last non-open — fully open, I mean — models that
actually talk about their data processing. They have Common Crawl processed
with CCNet, which

**[1:00:07]** you know about. The classification, though, was whether the
page was referenced by Wikipedia, not whether it's actually a Wikipedia
article — the idea is that, well, maybe Wikipedia articles are too stylized,
but we know that Wikipedia articles reference a bunch of other articles,
which are presumably good, so let's call those the good websites. They had
C4, they did a bunch of processing on GitHub to get permissive licenses,
Wikipedia, and they trained on Books3 and Project Gutenberg. Books3 really
got them in a lot of trouble, because they were just announcing to the
world, "Hey, I trained on this dataset."

**[1:00:54]** And you look back, "Oh, this came from — where did it come
from?" It came from The Pile. "Oh, it came from a shadow library." So, that's
why people don't talk about their data anymore. So, they looked at arXiv and
actually used the LaTeX to do the processing, and Stack Exchange, and they
got 1.2 trillion tokens. They didn't actually release this dataset, but they
had enough of a description of their processing that this was reproduced by
Together's RedPajama v1 dataset, which was then used in other sources. So, as
you can see here, RedPajama v1 initially also contained Books3, which has now
been

**[1:01:40]** stripped out. So various decisions made early on about
copyright actually have a fairly big watershed effect. Okay, so let's talk
about RefinedWeb. This is a paper that tried to make the point that web data
is all you need — the web, in some sense, is everything. We talked about all
these specialized sources, like GitHub and arXiv and Stack Exchange, and they
said, "Well, what happens if we just stick with the web?" So they did a good
job of doing the transformation from HTML to text. They filtered using the
Gopher rules,

**[1:02:28]** which is basically, keep things that look like English. They
explicitly, at that time, still said, "Avoid ML-based filtering, to avoid
biases — I don't want to find an overly narrow subset of the web." They did
some deduplication, and they had 5 trillion tokens — they released about 600
billion of them. FineWeb from Hugging Face was a replication of RefinedWeb,
but improved it. They took all the Common Crawl dumps at that time, did some
filtering, again using manual rules, because they didn't want to inject
biases, deduped, did PII removal, and got, you

**[1:03:13]** know, 15 trillion tokens. So, you can see that the size of
these datasets is growing quite a bit. Then there's this Dolma dataset from
AI2, which includes their own processing of Common Crawl, The Stack — which
we'll talk about — C4, which you know about, and a bunch of other things. And
the Reddit dataset was from this project called Pushshift — at that time you
could still get this data and train on it, before things got locked down.
AI2

**[1:04:01]** has their own crawl of academic papers, called Semantic
Scholar, and they derive a dataset based on that. Let's look at their Common
Crawl processing — they use language identification, which was model-based,
but quality filtering still avoided model-based filtering. And then they
removed toxicity using rules and a classifier, and got 3 trillion tokens out
of that. So, DCLM, I think, was maybe the point where this idea of
model-based quality filtering really started to

**[1:04:47]** become the norm here. DataComp's initial motivation was to
define some sort of pipeline so that people can try different data methods
in a standard way and show results, but I think the main way people use this
is just using the dataset they released to train models and do other things.
So, they processed Common Crawl to produce DCLM-pool — this is completely
unfiltered, and if you look at it, this is massive, 240 trillion tokens. This
is probably more tokens than anyone really trains on. But a lot of this is
fairly

**[1:05:33]** low quality. So then they have a pipeline where all that gets
filtered — you keep only English, you have some sort of rules to really
narrow it down, you do dedupe, and then you do this model-based filtering,
and you end up with something that's 1.4%. And this is the dataset that
we'll see actually works pretty well. The model-based filtering — this was
kind of strangely good. So, to train a classifier, they took OpenHermes,
which is essentially instruction data that was generated by GPT-4,

**[1:06:18]** and ELI5, which is a subreddit with various questions and
answers. Let's see — these are the type of questions that are in ELI5. Okay,
anyway, you get the idea. It's kind of weird, but somehow this works. The
negative examples are anything from RefinedWeb, which, remember, is just a
very loosely filtered version of the web — it's basically the web. And it
got 3.8 trillion tokens. They train a fastText classifier — think of it as a
linear classifier. And then they show that this

**[1:07:04]** magical quality classifier outperforms a bunch of other things
that they tried. So DCLM, for a while, at least in the open community,
became a bit of a gold standard for quality filtering. Let's move on to
Nemotron. Nemotron is from Nvidia — they said, well, DCLM filters too
aggressively, it's removing most of the data, and remember, they only got 3.8
trillion tokens, right? So we need more tokens.

**[1:07:50]** So what do we do? We're going to do something more elaborate —
we're going to prompt our existing model to score FineWeb documents based on
educational value. So we prompt a language model and ask, is this
educational or not? Create a bunch of labels, train a fastText model — that's
one classifier. Another classifier is just using the DCLM classifier. So
we're going to use those classifiers. We're also going to use synthetic
data. This is probably — I don't know if it's the first, but probably not —
one of the main

**[1:08:35]** datasets that really lean into synthetic data for
pre-training. For low-quality data, as deemed by these classifiers, you use a
language model to rephrase it to make it look more like Wikipedia. For
high-quality data, you use a language model to generate various tasks — for
example, given a Wikipedia article, you can generate question-answer pairs,
or generate something like, please summarize this document, extract key
information from it, and so on. So they ended up with a dataset that was six
trillion tokens — substantially larger than DCLM.

**[1:09:20]** And just for reference, this is still kind of small by some
considerations — Llama 3 was trained on 15 trillion tokens, Qwen3 was
trained on 36 trillion. Although it's not clear how big these unique-token
counts really are, because when you look at token counts in language model
papers, some of it's repeated — if you do multiple epochs, two epochs, that's
twice the number of tokens, so you have to be careful when you look at those
numbers. And they show that this dataset is better — their high-quality
subset beats the previous datasets.

**[1:10:07]** So, up until now we've seen a bunch of different methods for
filtering. I think all of these look very similar at some level — you take a
web crawl, and you either decide, I'm going to use rules to filter, or I'm
going to use a model. If you're going to use a model, then you have to decide
what looks like good data, train a classifier, classify all your documents,
and select. And there seems to be a trade-off — you can take a lot of Common
Crawl, you can get 240 trillion tokens if you want, but that's probably going
to be really low quality, or you can get like one trillion tokens, and
there's some sweet spot in between.

**[1:10:54]** Let me talk about two final things — code, and going back to
this question of licensing. The Stack is a very nice project that was trying
to make a really good coding dataset, since it was clear by 2022 that coding
was going to be really important. In the initial version, what they did was
clone 137 million repos. They kept the ones that were permissively licensed,
removed near-duplicates, and resulted in 3 terabytes of code. And then in
2024 they had an update, and here they

**[1:11:44]** took more — the metadata, like the issues, comments, and PRs
from GitHub, and the repositories from Software Heritage, which we talked
about. They also scraped documentation from various websites by crawling
them. They did a bunch of processing — in GitHub repos you have binary
files, which you probably don't want to train on, and malware, so you get rid
of that. A lot of GitHub, especially these PRs, are bots, so you have to
filter that. Dedupe, do PII reduction. There are a lot of pull requests, so
they basically subsample to keep it representative and the dataset
manageable. They also did this thing, which I thought was kind of nice,
which is that

**[1:12:32]** there are many programming languages out there — a lot of
Python, a lot of C, but low-resource languages like Nim, which I hadn't even
heard of, aren't that common. So what they do is compile this code into a
low-level intermediate language, LLVM, which every C compiler, say, can
compile into, and they juxtapose the low-resource language and the
intermediate representation. So then the language model can actually learn
the mapping between the shared low-level representation, which has a lot of
data on it, and the thing that has less data. And just for fun, you add in
all the

**[1:13:19]** other good stuff that you can. So Stack v2 is mostly code,
plus other things that were used to train their coding models. And one thing
about pull requests is that they, and all their metadata, are not by nature a
linearized sequence, so there have to be some steps taken to linearize it.
One important thing you have to decide is how much context to provide — for
example, an event might just be, I changed one line of code — that could
just be one line, but presumably, to learn that, you want to provide some
context, maybe a few lines around it, maybe the

**[1:14:04]** entire file surrounding that diff. These are some design
decisions you have to make for linearization. So, at the end of the day, the
tokens they train on look something like this — sort of XML-like structured
data, where you have the PR, and then a bunch of diffs, and then for
comments, you have basically the events, like a comment was posted, what the
review state of that PR was, and so on and so forth. So it's not just
learning how to generate code, but also the software development process
around code. Okay, so finally, I'll talk about CommonPile.

**[1:14:49]** So, recall that almost all the data on the internet is
copyrighted. Some of it is permissively licensed, some of it is even in the
public domain. And while you can appeal to fair use and say, I'm going to
train on it anyway — this is not quite settled. So if you're very
risk-averse, then you say, well, if I don't know whether it's okay, that's a
no. If you take that attitude, then that's what CommonPile did. Can you train
a good model with only permissively licensed data? Or rather, how far can you
get? So

**[1:15:35]** this was a project that went and scoured the internet for all
the different types of data it could possibly find that was worth training
on and was permissively licensed. It includes Stack v2 and code. It turns
out that a lot of government proceedings are actually permissively licensed,
wikis, some things on the web. Some news sites are actually permissively
licensed. Academic papers, online forums, things that are in the public
domain, educational resources, and so on. So in the end there were 8
terabytes of

**[1:16:23]** data, which is actually pretty good for permissively licensed
data. Doing this project is actually much harder than it might sound at
first glance — it's not as simple as, oh, you look at the license and say,
yep, Apache, good, or CC, good. There are some subtleties. First is license
laundering — people are kind of sloppy with licenses, and might take some
copyrighted work and just slap a CC BY on it. Anyone can write this on the
internet, and it's kind of hard to tell whether it's real or not. There's
also this common practice

**[1:17:08]** where, for example, Dolma is permissively licensed —
remember, Dolma is the AI2 collection — but collection licenses don't extend
to the individual works. The collection itself — I mean, first of all,
there's a question of whether collections can be copyrighted, but I guess you
can put a license on it, anything — but the individual works aren't
necessarily covered. So you can't just look at datasets on Hugging Face —
many datasets on Hugging Face, you see they have a permissive license, but if
you dig deeper, it's actually not permissively licensed at the individual
level. And they also made a decision to

**[1:17:55]** forgo training on any synthetic data, because training
language models on synthetic data from unlicensed sources is unclear — is it
probably fine, because technically these are open-weight models with an MIT
license, so you can do whatever you want, use them however you want? But
these language models were presumably also trained on unlicensed data, so
it's a little bit of data laundering, if you're really honest about it. So,
with this data, they compared with a bunch of

**[1:18:42]** other models, like the first LLaMA, MPT, and Qwen. On a bunch
of benchmarks, they show that this is not bad — I'd say it's not nearly as
good as the Qwen models, for sure, but it's certainly outperforming the very
old models, from 2023, I guess — quite old models. So I'd say the conclusion
here is that you can do reasonably, but it's still pretty tough to compete
without more tokens. But I don't think this is the final word on it, and I
think you can probably eke more out of publicly permissive licenses if you
try.

**[1:19:29]** Okay, so, to summarize this lecture — hopefully I've now given
you some appreciation that data is a very rich topic. It's not that data
just falls from the sky, or you just go on Hugging Face and download a
dataset. Data has to come from somewhere, and there's also a technical, but
also a social, context in which data comes about — because at the top, the
internet is a bunch of live services, and someone has to do the work of
producing the raw data, whether it's the website that gives you dumps, or
someone has to build a crawler, or something. And then someone has to make a
decision about how to process it into

**[1:20:15]** usable form — filtering, transformation, deduplication. All of
these things have an impact on the quality of your final language model.
Notice that filtering is probably one of the most important things — how do
you go from 200 trillion tokens to less than 3 trillion tokens? That's
obviously a huge reduction, which I think merits a lot of attention. Data is,
in some sense, the key ingredient that differentiates language models — a
lot of language models are roughly the same transformer architecture, but
data, depending on how you process it, can make a pretty

**[1:21:01]** big difference. There are plenty of legal and ethical issues
around data, more than I can get into in this lecture. And also, this
process is very messy — unlike some of the other parts of this class, where
things are maybe more based on first principles, data processing right now,
at least, is a lot just based on vibes. You define this classifier, you
define this rule, you set some threshold. So there are many opportunities to
improve — as you do your assignment four, maybe think about whether there
are better ways to do this, and this could be a research direction. Okay, so
that's it for today. So, next

**[1:21:47]** time I'll continue talking about data — I'll talk a bit about
post-training data, and a bit more about filtering.

