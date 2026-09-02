# Licence and copyright

This repository is a **study aid compiled from Stanford CS336 course material**. It is
not a work with a single owner, so it carries no single repository-wide licence: three
layers of authorship sit here with three different copyright positions. Read this before
reusing anything from it.

If you only want the short version: **the explanatory writing is yours to reuse with
attribution; the course material and the paper figures are not mine to give you.**

---

## Layer 1 — the compilation's own editorial work

Copyright (c) 2026 chaimantec.

Licensed under **Creative Commons Attribution 4.0 International** (`CC-BY-4.0`),
<https://creativecommons.org/licenses/by/4.0/>. You may share and adapt it, including
commercially, with attribution.

This covers only the material that is original to this compilation:

- `INDEX.md`, `AGENTS.md`, `SEE_ALSO.md`, `TODO.md`, `kb.json`, `sources.md`, this file
- the original explanatory prose in `wiki/` — the topic explanations, the connective
  argument, the organisation of the material into pages, and the editorial judgements
  about what is worth recording

It does **not** extend to the lecture content those pages describe, nor to any passage
in `wiki/` that quotes a lecturer, a slide or a paper. Those are quotations, marked as
such, and they belong to Layer 2 or Layer 3.

### Not offered under CC BY 4.0, even though I wrote the words

`raw/transcripts/` and `raw/slides/` are derivative works. The copy-editing, the `[Ed:]`
adjudications and the figure descriptions are mine; the lecture that is being transcribed
and the slides that are being described are not. I cannot license the underlying
expression, so I do not offer these directories under CC BY 4.0. Read them, cite them,
quote them as you would quote the lecture itself — but do not treat them as freely
relicensable text.

The verbatim auto-captions these were edited from are **deliberately not published here**.
They were a complete reproduction of nine lectures with no editorial contribution at all —
the weakest thing in the repository on every fair-use factor except purpose — so they are
gitignored rather than committed. Nothing is lost for verification: they are regenerable
from the public video with the `cairn-kb` skill's `fetch_transcript.py`, using the video id
recorded in each transcript's front matter, so anyone can re-derive them and audit an edit.

---

## Layer 2 — Stanford CS336 course material

**No licence exists, and none is granted here.**

The lectures, slide decks and figures are the work of Percy Liang and Tatsunori Hashimoto
for CS336, *Language Modeling from Scratch* (Stanford, Spring 2026), published at
<https://cs336.stanford.edu> and <https://github.com/stanford-cs336/lectures>.

As checked on 2026-09-03, no licence is stated anywhere: the lectures repository has no
`LICENSE`, `LICENSE.md`, `COPYING` or `NOTICE` file; its README says nothing about reuse;
the course landing page carries no licence, copyright or reuse statement; no page of any
transcribed deck contains licence text, and none of the PDFs records one in its metadata;
and the YouTube recordings carry no Creative Commons marker, which is consistent with the
default Standard YouTube License.

Publicly downloadable is not the same as licensed. Absent an express grant, the default
applies: copyright arises automatically and no reuse rights pass to anyone.

The images in `raw/images/` and the material described in `raw/slides/` are reproduced
here for study and commentary, relying on fair use (17 U.S.C. § 107) rather than on any
permission. Every deck is linked at its canonical URL beside the material drawn from it.
**That reliance is mine and does not transfer to you.** Fair use is a defence to
infringement, not a licence: it can justify a particular use by a particular user for a
particular purpose, and it gives me nothing to pass downstream. If you want to reuse
Layer 2 material, make your own assessment or ask the authors.

At US universities copyright in teaching materials commonly rests with the faculty author
rather than the institution, so the people able to grant permission here are most likely
the instructors themselves.

---

## Layer 3 — third-party figures reproduced inside the decks

The CS336 decks reproduce figures from published papers, with a credit line on the slide.
Those figures belong to their authors and publishers. Stanford could not license them to
me, and I cannot license them to you — a licensor can only grant what they own, which is
why even a Creative Commons licence on the decks would not have reached this layer.

Of the ten most-reproduced sources, checked against arXiv's licence metadata on
2026-09-03, **nine are under arXiv's `nonexclusive-distrib/1.0`** — which grants arXiv
the right to distribute and grants third parties nothing — and **one, Wei et al.,
*Emergent Abilities of Large Language Models* (arXiv:2206.07682), is CC BY 4.0** and so
is freely reusable with attribution.

Across the 236 rendered deck pages there are 32 distinct cited sources. The heaviest is
Kaplan et al. at 8 pages, spread over three different lectures; Fedus et al. is 8, all in
one; only 3 of the 32 appear on more than three pages, and most appear once or twice. A
handful of figures from any one paper, presented inside original explanatory commentary
and credited to the source, is the same order of use the lectures themselves make.

`AGENTS.md` names these sources; the slide files carry the credit line printed on each
slide. Cite the original paper, not this repository, when you use one.

---

## Takedown

If you are an author, rightsholder or member of the CS336 teaching staff and you want
something removed, open an issue or contact the repository owner through GitHub. Anything
you identify will be deleted — the file and every reference pointing at it — and the
removal noted in `kb.json`. No argument, no delay.
