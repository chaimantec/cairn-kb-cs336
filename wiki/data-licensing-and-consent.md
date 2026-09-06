# Data licensing and consent

[Copyright and fair use](copyright-and-fair-use.md) covers what the law says. This
page covers what website operators actually *permit*, how that has changed, and
why "permissively licensed" turns out to guarantee much less than it appears to.
From [lecture 13](13-data-sources-datasets.md).

## The layers of permission

A single page can be blocked at four independent levels, and clearing one says
nothing about the others:

| Layer | Mechanism | Enforceable? |
| --- | --- | --- |
| Technical | `robots.txt` | No — voluntary convention |
| Technical | Cloudflare, CAPTCHAs, IP blocks, rate limits | Yes, in practice |
| Contractual | Terms of service | Yes — contract law |
| Statutory | Copyright and licensing | Yes — copyright law |

The lecture's own example of how these stack: YouTube hosts Creative Commons
licensed videos whose **terms of service still prohibit downloading them** (≈27:02).
A permissive copyright license does not override a contract you accepted by using
the site. "There are multiple layers of restrictions here."

### robots.txt is a convention

Worth restating precisely, because it is easy to over-read: **"This is not a legal
restriction — this is just, you're supposed to be a good citizen and look at
robots.txt... That's basically the contract — not even a contract"** (≈7:46).

The entries the lecturer reads off `nytimes.com/robots.txt` are the AI crawlers
themselves — OAI-SearchBot, PerplexityBot, ChatGPT-User, ClaudeBot. Compliance is
voluntary, which is exactly why `robots.txt` is useful as a *measurement* of stated
intent even where it is not binding.

## The decline of consent

The *Consent in Crisis* paper by **Shayne Longpre**
([arXiv 2407.14933](https://arxiv.org/abs/2407.14933)) examined both `robots.txt`
and terms of service for URLs appearing in C4, RefinedWeb and Dolma, and found that
restrictions have risen sharply (≈10:05):

- Through 2023 the picture was "fairly constant"; by **mid-2023 the fraction of
  sites with full restrictions had grown to almost 50%.**
- Terms of service moved the same way: "in 2016 no one really put any terms on
  their pages, and now most pages put some terms, and most of the terms say you
  can't use this for AI" (≈10:51).

The conclusion is the one to carry: **even though it was possible to crawl the web
in 2020, the web you can legally crawl now is much smaller** — and this is a moving
target, so a corpus's legality is partly a function of when it was collected.

![Three panels: robots.txt composition over time, ToS composition over time, and log-scale restriction rate by crawler organization](../raw/images/13-data-sources-datasets/decline-consent.png)

*From* Consent in Crisis. *[Source](https://github.com/stanford-cs336/lectures/blob/main/images/decline-consent.png)*

> **What this figure does and does not show.** The lecture introduces it as
> examining restrictions "for URLs in common datasets (C4, RefinedWeb, Dolma)" —
> that is the paper's scope, **but the figure breaks out nothing by dataset.** Its
> three panels are robots.txt category composition over time, ToS category
> composition over time, and restriction rate by **crawler organization**: OpenAI
> 25.9%, Anthropic 13.3%, Common Crawl 13.3%, Google 9.8%, "False Anthropic" 6.0%,
> Cohere 4.9%, Meta 4.1%, Internet Archive 3.2%, Google Search 1.0%. The third
> panel's y-axis is **logarithmic**, so the post-ChatGPT rise is steeper than a
> linear reading suggests. Verified against the image directly — see the
> [figure audit](../raw/slides/13-data-sources-datasets.md#figure-audit).

The "False Anthropic" category is worth noting on its own: crawlers impersonating a
known agent string are common enough to need their own series.

## Crawling harms, before copyright

Consent is not only a legal matter. The lecture describes a site operator
complaining about a crawler "hitting their servers a million times in 24 hours," and
Read the Docs "also getting hammered" (≈11:38). The costs are borne by the host:
server load "costs money for the person hosting it and also degrades service for
everyone else trying to use it" (≈12:25).

## Shadow libraries

LibGen, Z-Library, Anna's Archive and Sci-Hub are "technically part of the web" and
hold copyrighted, paywalled books and papers — LibGen ~4M books (2019), Sci-Hub ~88M
papers (2022). They receive takedown orders and lawsuits and are blocked in various
countries, but persist by relocating servers.

The lecture presents both readings without adjudicating: defenders "argue that this
is making freely available what should have been free," and "from a legal
perspective, this is piracy and copyright infringement" (≈13:11).

They matter to this course for two concrete reasons: [Books3](pretraining-datasets.md#books3)
came from Bibliotik, and **the acquisition question is what cost Anthropic $1.5
billion** — see [copyright and fair use](copyright-and-fair-use.md#bartz-v-anthropic-2024--the-landmark).

## You cannot filter your way to certainty

Asked how you keep pirated books out of a web crawl, the lecture's answer is blunt
(≈43:54):

> That's part of the difficulty — you can't.

Common Crawl most likely contains copyrighted books and content you are not
supposed to train on, and the only available recourse is fair use. The lecturer
adds that books are "not remarkably different from your website" — both are
copyrighted — though a book author "they'll probably be able to protect that better,
because it's published" (≈43:54–44:40) — both phrases running across the
paragraph break, which splits contiguous speech.

## CommonPile

The alternative to that resignation: build a corpus from permissively licensed
material only. The framing is explicitly one of risk tolerance — "if you're very
risk-averse, then you say, well, if I don't know whether it's okay, that's a no"
(≈1:14:49).

The key question: **can you train a good model using only permissively-licensed
data?**

![CommonPile composition across permissively licensed source categories](../raw/images/13-data-sources-datasets/commonpile.png)

*CommonPile's sources — including Stack v2 and code, government proceedings, wikis,
permissively licensed news, academic papers, online forums, public-domain works and
educational resources.
[Source](https://github.com/stanford-cs336/lectures/blob/main/images/commonpile.png)*

Result: **8 TB**, "which is actually pretty good for permissively licensed data"
(≈1:16:23). [Code](code-data.md) is a large component, because code carries
explicit machine-detectable licenses.

### Three ways "permissively licensed" fails

The project is "much harder than it might sound at first glance — it's not as simple
as, oh, you look at the license and say, yep, Apache, good, or CC, good" (≈1:16:23).
Each of these is a distinct failure of the label:

1. **License laundering.** People "might take some copyrighted work and just slap a
   CC BY on it. Anyone can write this on the internet, and it's kind of hard to
   tell whether it's real or not." **A permissive label is not evidence of
   permissive origin.**

2. **Collection licenses do not extend to the individual works.** Dolma is ODC-By,
   but that covers the collection, not the documents in it. This connects back to
   the doctrinal point that [collections are not original works](copyright-and-fair-use.md#the-frame-incentives-not-prohibition).
   The practical warning is direct and immediately actionable (≈1:17:08):

   > You can't just look at datasets on Hugging Face — many datasets on Hugging
   > Face, you see they have a permissive license, but if you dig deeper, it's
   > actually not permissively licensed at the individual level.

3. **Synthetic data does not launder provenance.** CommonPile forwent
   [synthetic data](synthetic-data.md) entirely: an open-weight model may carry an
   MIT license, "but these language models were presumably also trained on
   unlicensed data, so it's a little bit of data laundering, if you're really
   honest about it" (≈1:17:55).

### Does it work?

![Bar chart comparing Comma v0.1-1T against LLaMA, MPT, RPJ-INCITE and Qwen3 across 11 benchmarks](../raw/images/13-data-sources-datasets/comma-results.png)

*Comma v0.1-1T against four baselines across 11 benchmarks. The six gold stars mark
where Comma leads the three ~1T-token models — an inference from the pattern, since
nothing in the figure labels them.
[Source](https://github.com/stanford-cs336/lectures/blob/main/images/comma-results.png)*

The verdict is measured: "not nearly as good as the Qwen models, for sure, but it's
certainly outperforming the very old models, from 2023" (≈1:18:42). So **you can do
reasonably well, but it is tough to compete without more tokens** — with the
lecturer adding that he doesn't think this is the final word, and "you can probably
eke more out of publicly permissive licenses if you try."

Set 8 TB against Nemotron-CC's 6.3T tokens and Qwen3's 36T to see the size of the
gap being described.

## See also

- [Copyright and fair use](copyright-and-fair-use.md) — the legal frame
- [Web crawling](web-crawling.md) — the technical layer
- [Pre-training datasets](pretraining-datasets.md) · [Code data](code-data.md) ·
  [Synthetic data](synthetic-data.md)
- [Lecture 13](13-data-sources-datasets.md)
