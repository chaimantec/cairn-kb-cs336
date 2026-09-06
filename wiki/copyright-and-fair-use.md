# Copyright and fair use

Everything on the internet is copyrighted, so every dataset in
[lecture 13](13-data-sources-datasets.md) needs an answer to how it is allowed to
exist. This page collects that lecture's copyright material (≈14:43–31:37), which
is the longest single section of it.

**The lecturer states explicitly that he is "not a lawyer" (≈31:37).** That
disclaimer belongs on this page too. What follows is a language-modelling course's
working summary of a live and rapidly changing area, not legal advice.

## The frame: incentives, not prohibition

Copyright sits inside intellectual property law, whose goal is to **incentivize the
creation of intellectual goods**. The lecture is emphatic that this is the right
lens: "the spirit of the law is not just to technically say no to everything, but
to incentivize the creation of intellectual goods" (≈14:43). The four kinds of IP
are copyright, patents, trademarks and trade secrets; copyright is the one that
matters for training data.

The history is short: the Statute of Anne (England, 1709) is the first government
regulation of copyright, and in the US the Copyright Act of 1976 defines the modern
form. Protection applies to

> original works of authorship fixed in any tangible medium of expression, now
> known or later developed, from which they can be perceived, reproduced, or
> otherwise communicated, either directly or with the aid of a machine or device

Three consequences the lecture pulls from that sentence:

- **Collections are not original works**, so a telephone directory is not
  copyrightable — unless there is creativity in the selection or arrangement.
  (This matters later for [dataset collection licenses](data-licensing-and-consent.md#three-ways-permissively-licensed-fails).)
- **Copyright covers expression, not ideas.** "You can't copyright the quicksort
  algorithm. You can copyright an implementation of the quicksort algorithm, but
  not the quicksort algorithm" (≈16:14).
- **The 1976 Act widened scope from "published" to "fixed."**

## Everything is copyrighted

The practical consequences of "fixed" rather than "published":

- Registration is **not required for protection** — unlike patents.
- The threshold is "very low — for example, you put something on your website,
  it's copyrighted, that's it" (≈17:01).
- Registration *is* required before you can sue, and costs **$65** — "much smaller
  than the lawyer fees you'll probably pay" (≈17:47).
- Copyright lasts **75 years**, then the work enters the public domain. This is why
  most of Project Gutenberg is usable at all.

Which yields the lecture's pivot sentence:

> Summary: *basically everything on the Internet are copyrighted.*

And therefore exactly two ways to use a copyrighted work (≈18:33):

1. **Get a license for it.**
2. **Appeal to fair use.**

## Licenses

A license comes from contract law: a licensor grants it to a licensee, and
"effectively, 'a license is a promise not to sue'."

**Creative Commons**, created by Lessig and Eldred in 2001, lets a work behave as
if it were in the public domain without waiting 75 years — the lecture frames it as
bridging public domain and existing copyright for creators who *want* their work
used (≈20:05). Wikipedia, OpenCourseWare and Khan Academy are examples, along with
307M Flickr images, 39M MusicBrainz images and 10M YouTube videos.

Where Creative Commons does not apply and the work is not public domain, you buy a
license. The lecture names three such deals: **Google–Reddit**,
**OpenAI–Shutterstock**, and **OpenAI–StackExchange**.

One audience question is worth recording because the answer is non-obvious. If you
train under a license and **the license later changes**, you keep the right to what
you already trained on — but "the license usually doesn't apply to just a fixed set
of documents." With Reddit as the example: new content created after the change is
out of bounds, because the license covered an ongoing stream, not a snapshot
(≈31:37).

## The four factors

Fair use is Section 107. Four factors determine whether it applies, and **"none of
these are hard rules — they're just tendencies, which have to be weighed in court"**
(≈21:39):

1. **Purpose and character of the use.** Educational favoured over commercial;
   transformative favoured over reproductive.
2. **Nature of the copyrighted work.** Factual favoured over fictional; the lecture's
   contrast is a page of facts about World War II against "my really creative poem"
   (≈22:25).
3. **Amount and substantiality used.** A snippet is favoured over the whole work.
4. **Effect on the market** for the original. This one returns to the original
   motivation — copyright exists to align economic incentives, so providing an
   alternative that "decreases the amount the original author could monetize their
   work" counts against you (≈23:11).

Worked examples of fair use: watching a movie and writing a summary; reimplementing
an algorithm rather than copying the code; and **Authors Guild v. Google**, where
Google Books' index-and-snippets was held fair use — a case that ran, as the lecture
notes, for **11 years** before settling in Google's favour (≈23:57). That precedent
is part of how people reason about training today.

### Copyright is not about verbatim memorization

The lecture flags this specifically as a correction for an ML audience, "because a
lot of papers are focused on verbatim memorization, but that's one way you can
violate copyright, not the only one" (≈24:43).

- **Plots and characters can be copyrighted** — Harry Potter the character, not
  merely a particular book.
- **Parody is likely fair use**, even though it produces something derivative.

> Copyright is about semantics (and economics).

"It's definitely not about n-gram overlap" (≈24:43). A memorization metric is
neither necessary nor sufficient evidence.

## What this means for language models

Applying the four factors to training (≈25:29–26:16), two points cut in opposite
directions and both matter:

- **Copying is already the violation.** "The mere fact of copying, which is in the
  word 'copyright,' is potentially a violation already, even if you don't do
  anything with it." Acquisition is a separate act from training — and this is the
  distinction the Anthropic judgment turns on.
- **Training is plausibly transformative.** "This is not necessarily a fact, but it
  intuitively has a transformative flavor, because it certainly seems different
  than just re-hosting another work." Models are trained on data "as a means to an
  end — we're trying to extract the general idea... rather than just the concrete
  expression."
- **But market effect is factor four**, and "language models can definitely affect
  the market" regardless of how transformative training is. Nothing about how a
  model works answers this factor.

Hence: **"fair use is a little bit slippery"** (≈26:16).

### Terms of service are a separate layer

Even with a license or a fair-use argument, terms of service can still forbid
acquisition. The lecture's example: YouTube's ToS prohibits downloading videos
**even when those videos are Creative Commons licensed** (≈27:02). Fair use is a
defence to copyright infringement; it is not a defence to breaching a contract you
accepted. "There are multiple layers of restrictions here."

## The lawsuits

### NYT v. OpenAI (2023)

Allegation: training on and reproducing NYT articles. The complaint's evidence was
that they could "prompt ChatGPT to generate a news article almost verbatim"
(≈27:47). **Still pending.**

### Bartz v. Anthropic (2024) — the landmark

The judgment splits in two, and the halves point opposite ways:

- **Training on the works was fair use.**
- **Pirating the copies was not** — "notice that this has nothing to do with
  training — just the mere fact of pirating... is illegal" (≈28:33).
- Anthropic had *also* bought and scanned the books, and **that** was fair use too:
  the court accepted that you may "buy a bunch of books, rip off the binding, scan
  and digitize it for your own use." But doing so afterwards "doesn't absolve you
  of your sin of pirating — you can't pirate and then buy it and say, actually,
  never mind what I did first."
- **Outcome: Anthropic paid $1.5 billion to settle — "that's about $3,000 a book."**

This is the case to remember, because it establishes that **how you acquired the
data is a separate question from whether you were allowed to train on it**, and in
this instance the acquisition is what cost $1.5B.

### Kadrey v. Meta

Allegation: training on the plaintiffs' books, "revealed in the Llama paper" —
which is the [Books3](pretraining-datasets.md#books3) chain. Summary judgment (2025)
held training in this instance fair use. **The torrenting allegation is still
pending**, and the lecture's read is that "if precedent holds, that's probably also
not going to be good for Meta" (≈29:20).

## Where this leaves things

The lecture's summary is deliberately hedged and worth quoting at its actual
strength — training "has been deemed fair use, or at least has not been deemed not
fair use" (≈29:20). The rulings "have so far been narrow — it's not to say that any
training on any copyrighted content is fair use, but in these cases, it's fine"
(≈30:06). Pirating books is clearly illegal. It remains "a very active and evolving
area."

Two practical consequences run through the rest of the course's data material:

- **This is why model developers stopped documenting their data.** The chain from
  LLaMA's paper naming Books3, to Books3 coming from The Pile, to The Pile taking
  it from a shadow library, is exactly what a plaintiff needs — "that's why people
  don't talk about their data anymore" (≈1:00:54).
- **You cannot filter your way to certainty.** Asked how to keep pirated books out
  of a web crawl, the answer is "you can't"; Common Crawl most likely contains
  copyrighted books, and the only available recourse is fair use (≈43:54).
  [CommonPile](data-licensing-and-consent.md#commonpile) is the attempt to
  sidestep the question entirely.

## See also

- [Data licensing and consent](data-licensing-and-consent.md) — robots.txt, terms
  of service, and what "permissively licensed" fails to guarantee
- [Pre-training datasets](pretraining-datasets.md) — including the three that no
  longer exist
- [Lecture 13](13-data-sources-datasets.md)
