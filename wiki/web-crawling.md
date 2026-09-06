# Web crawling

How text physically gets from live web servers into a training corpus. From
[lecture 13](13-data-sources-datasets.md) (≈4:42–13:57, ≈32:23–35:29).

The starting point is that **"language models are trained on the entire internet"
is wrong**, and the lecture's first objection is conceptual: the claim "doesn't
really quite type-check, because for that to be true, it would have to be an agent
— like an RL agent that goes on the internet and does stuff" (≈4:42). Pre-training
does not browse. The web is a set of **live servers**, and you cannot train on a
live server — only on what a crawler captured.

## What a crawler cannot reach

Four categories, and together they are why "the public web" is much smaller than it
sounds:

**Dynamic content.** Many sites are applications, where "the URL isn't even as full
a specification of the content — it's just an app that you interact with," needing
clicks and form submissions (≈6:14). Discord and wandb are the examples. This is
the *deep web*: not reachable by following hyperlinks.

**Authentication.** Facebook, X, LinkedIn and the New York Times hold "huge amounts
of content locked up behind these walled gardens" (≈7:00). The lecture notes the
asymmetry this creates: if you are Facebook you already have the data, and if you
are xAI you have X — "but if you're anyone else, you can't actually access this
content."

**Technical restrictions.** `robots.txt`; Cloudflare bot detection and CAPTCHAs
("some algorithm has detected that you might be a bot"); IP and country blocks;
rate limits (≈7:46–8:32).

**Legal restrictions.** Terms of service, and licensing — see
[copyright and fair use](copyright-and-fair-use.md) and
[data licensing and consent](data-licensing-and-consent.md).

### robots.txt is a convention, not a law

The lecture is precise about this and the precision matters: **"This is not a legal
restriction — this is just, you're supposed to be a good citizen and look at
robots.txt, and if your name is on that list, then you should not crawl. That's
basically the contract — not even a contract"** (≈7:46).

It is a file at a domain's root — `nytimes.com/robots.txt` is the worked example —
and the entries the lecturer reads off are the AI crawlers themselves:
OAI-SearchBot, PerplexityBot, ChatGPT-User, ClaudeBot.

Because compliance is voluntary, `robots.txt` measures *stated intent*, which is
what makes it usable as a research instrument — see
[the decline of consent](data-licensing-and-consent.md#the-decline-of-consent).

### Crawling has costs even before copyright

The lecture recounts a site operator complaining about a crawler "hitting their
servers a million times in 24 hours," and Read the Docs "also getting hammered"
(≈11:38). Three distinct harms, none of which is a copyright question (≈12:25):

- violating terms of service,
- violating `robots.txt`,
- **server load**, which "costs money for the person hosting it and also degrades
  service for everyone else trying to use it."

## Common Crawl

The alternative to building your own crawler. Most model developers do build one,
"because they want full control over what the data is" (≈32:23) — but Common Crawl,
a non-profit founded in 2007, publishes crawls for everyone else.

- A crawl roughly **every month**, adding **3–5 billion pages**; crawls overlap but
  are diversified.
- **300 billion pages so far** — though the lecture does not take this at face
  value: "it seems a little big, because if you multiply this number by 20, you
  don't really get to 300 billion, but that's what they say" (≈32:23). Reading a
  dataset's own marketing sceptically is the transferable habit here.
- The **April 2026 crawl** is 2.19 billion pages and **372.2 TB** — "and this isn't
  including images, this is mostly just the text" (≈33:09).
- For comparison, the Google search index is "at least 100 petabytes."

Crawling uses **Apache Nutch**, and the algorithm is graph traversal: start from a
seed set (hundreds of millions of URLs), pop a URL from the queue, download it, add
its hyperlinks to the queue, parallelised across machines. The lecture's framing is
that it is "conceptually straightforward, but all the gory details are in the
implementation" (≈33:09).

### The three policies

The design decisions that make a crawler more than a loop:

- **Selection policy** — which pages to download at all.
- **Politeness policy** — respect `robots.txt`, don't overload servers.
- **Re-visit policy** — how often to re-check a page. You want to re-download
  frequently-changing pages and not waste requests on static ones.

Plus the standing difficulty of **URL dynamism**: the same URL can serve different
content depending on browser state, and many different URLs serve the same content.
"There's a lot of duplication that happens if you're not careful — mirror sites in
particular are explicitly about duplication" (≈34:42). That is where
[deduplication](deduplication.md) begins.

## WARC vs WET, and why HTML-to-text is a modelling decision

> **Lecture 14 measures this.** DCLM's comparison puts Common Crawl's own WET text
> behind both dedicated extractors on both evals — 20.7/12.2 against resiliparse's
> 24.1/13.4 and trafilatura's 24.5/12.5. Taking WET costs more than the choice
> between extractors does. See
> [HTML-to-text extraction](html-to-text-extraction.md).

Common Crawl publishes each crawl in two formats (≈34:42):

- **WARC** — the raw HTTP response, i.e. the HTML as served.
- **WET** — Common Crawl's own conversion to plain text, and "necessarily a lossy
  process."

The temptation is to treat WET as a convenience and move on. **It is not a neutral
choice**, and this is one of the lecture's recurring claims: how you convert HTML
to text has a measurable effect on downstream task accuracy. The tools named are
[trafilatura](https://trafilatura.readthedocs.io/en/latest/) and
[resiliparse](https://resiliparse.chatnoir.eu/en/stable/), and the DCLM paper's
ablation puts both ahead of WET (≈35:29).

![Table comparing CORE and EXTENDED accuracy across resiliparse, trafilatura and WET files](../raw/images/13-data-sources-datasets/dclm-wet.png)

*The DCLM extraction ablation. trafilatura leads CORE at 24.5, resiliparse leads
EXTENDED at 13.4, and WET files come last in both (20.7, 12.2). The figure does not
define what CORE and EXTENDED measure or in what unit.
[Source](https://github.com/stanford-cs336/lectures/blob/main/images/dclm-wet.png)*

Four datasets in this course make this choice explicitly and differently, which is
the clearest evidence that it is treated as a real decision:

| Dataset | Choice |
| --- | --- |
| The Pile (Pile-CC) | WARC + jusText — "better than WET" |
| RefinedWeb | WARC + trafilatura, not WET |
| DCLM | ran the ablation above |
| Nemotron-CC | jusText **rather than** trafilatura, because it returned more tokens |

Nemotron-CC's reasoning is worth noting because it optimises a different objective:
not the cleanest text, but the **most surviving tokens** — consistent with its
general position that [everyone over-filters](data-filtering.md#the-counter-argument-everyone-over-filters).

## Sources that publish dumps

A structural point that undercuts much of the above: several of the best sources do
not need crawling at all, because they publish bulk dumps.

- **Wikipedia** — every few weeks. "You don't need to crawl Wikipedia — in fact,
  they don't want you to crawl Wikipedia, they want you to download this instead"
  (≈37:44).
- **GitHub** — repositories over the git protocol, metadata via GitHub Archive's
  hourly event snapshots. See [code data](code-data.md).
- **arXiv** — bulk download from S3.
- **Stack Exchange** — anonymised XML dumps including metadata.

Where a dump exists it is strictly better: complete, structured, cheaper for both
sides, and unambiguous about permission. But the dump schedule is also an attack
surface — see the Wikipedia poisoning result in
[lecture 13](13-data-sources-datasets.md#wikipedia-github-arxiv), which works
precisely *because* dumps happen on a predictable cadence.

## See also

- [HTML-to-text extraction](html-to-text-extraction.md) — what happens to the bytes
  once a crawler has them, including the PDF and OCR path

- [Pre-training datasets](pretraining-datasets.md) — what gets built from crawls
- [Data filtering](data-filtering.md) — what happens to the text afterwards
- [Deduplication](deduplication.md) · [Code data](code-data.md)
- [Data licensing and consent](data-licensing-and-consent.md)
- [Lecture 13](13-data-sources-datasets.md)
