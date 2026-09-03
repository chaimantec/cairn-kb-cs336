# Executable lectures

CS336 does not have slides for half its lectures. Percy Liang's lectures are
**Python programs whose execution delivers the content** — he calls them
*executable lectures* — and understanding this is necessary to cite the course
material correctly.

He introduces the format at [19:26] in [Lecture 1](01-overview-tokenization.md):

> If you think it looks like a Python program, it is — because it actually is a
> Python program, but it has been rendered for your viewing pleasure. So when I
> step through it, it is actually executing the lecture.

## How it works

A lecture is a file like
[`lecture_01.py`](https://github.com/stanford-cs336/lectures/blob/main/lecture_01.py).
Prose is delivered by `text()` calls; figures by `image()`; citations by `link()`,
resolved against a structured
[`references.py`](https://github.com/stanford-cs336/lectures/blob/main/references.py).
The lecture's structure *is* the program's call graph — `main()` calls
`tokenization()`, which calls `bpe_tokenizer()`, and so on — which is what Percy
means at [19:26] about seeing the hierarchical structure and returning to `main`
when a section ends.

The reader is a trace viewer at
`https://cs336.stanford.edu/lectures/?trace=lecture_01`, which steps through
execution. Variables annotated `# @inspect` have their live values displayed at
each step, and `# @stepover` marks calls the viewer should not descend into. So
when the lecture shows you a compression ratio, that number was *computed* as the
page rendered, not typed into a slide.

```python
total = 0  # @inspect total
for x in [1, 2, 3]:  # @inspect x
    total += x  # @inspect total
```

That snippet is the lecture demonstrating itself at [19:26].

## Why it matters for citation

Three practical consequences.

**There are no slide numbers.** You cannot cite "slide 34" for a Percy lecture. The
stable references are the function name and the source line range; this knowledge
base's transcriptions of
[`lecture_01.py`](../raw/slides/01-overview-tokenization.md) and
[`lecture_02.py`](../raw/slides/02-pytorch-resource-accounting.md) each provide a
section-to-line table for exactly this reason. Cite a Percy lecture as, for
example, "`arithmetic_intensity_matmul()`, lines 449–468" — that reference stays
valid as long as the file does.

**The worked numbers are not in the source.** Because values appear at runtime via
`@inspect`, reading `lecture_01.py` does not show you the compression ratios, the
merge sequence, or the token ids — the code that produces them is there, the
outputs are not. This KB's transcription reports those values, obtained by
executing the lecture's own code, and says so explicitly at the head of the
relevant section.

Lecture 2 sharpens this point, because its runtime values come in two kinds. Most
are ordinary arithmetic on constants — `6 * 70e9 * 15e12`, or a byte count divided
by a bandwidth — and can be recomputed exactly by anyone, which is what this KB
did. But a handful are *measurements*: `benchmark()` wall-clock timings, the
measured FLOP/s they imply, the resulting MFU, and `get_max_memory_usage()`
readings. Those depend on which GPU the program is running on, so there is no
"the" value for them at all — the number Percy showed in class is a fact about the
machine in front of him. This KB marks them *machine-dependent, not reproduced*
and gives no number. If you need one, run the lecture on your own hardware; the
figure you get is the answer for your hardware, which is the point of the
exercise.

**The material is exact.** Unlike a slide deck read by a vision model, the source
text is unambiguous — no OCR, no misread axis labels, no figures that must be
described in prose. The one thing genuinely lost is the *images*: `image()` calls
reference files this KB records by path but does not describe, because doing so
would mean inventing content. Where a claim depends on a figure, the KB says the
figure exists and stops there.

## Which lectures are which

CS336 splits by instructor, and the split is exactly the format split:

- **Percy Liang → executable Python.** Lectures 1, 2, 6, 7, 10, 12, 13, 14, 17.
- **Tatsunori Hashimoto → conventional PDF decks.** Lectures 3, 4, 5, 8, 9, 11,
  15, 16.

Both live in [`stanford-cs336/lectures`](https://github.com/stanford-cs336/lectures).
Full inventory in [`sources.md`](../sources.md).

**This KB covers five executable lectures and five decks.** Lectures 1, 2, 6, 7 and 10
are executable lectures, transcribed from source text; [Lecture 3](03-architectures.md)
(67 pages), [Lecture 4](04-attention-alternatives.md) (60 pages),
[Lecture 5](05-gpus-tpus.md) (55 pages), [Lecture 8](08-parallelism-2.md) (73 pages)
and [Lecture 9](09-scaling-laws.md) (57 pages) are decks, transcribed from the
rendered page images. The citation rules differ accordingly — cite a *function name and line
range* for the first kind, a *slide number* for the second.

One wrinkle applies to all five decks: **none prints a page number on any page**,
so their "slide N" labels are PDF page numbers, and each slide file says so in its
front matter. For `lecture_03.pdf` an automated scan reported a printed number that
turned out to be the numerator of a fraction on page 61; for `lecture_04.pdf` the
scan reported none, and all 60 pages were checked by eye as they were read; for
`lecture_05.pdf` the scan again reported none, a corner-position scan returned only
mid-page body text, and each of the three readers who split the deck confirmed the
absence independently over its own range.

[Lecture 6](06-kernels-triton.md) is the first executable lecture in this KB whose
subject is code the students will themselves write — its four Triton kernels are
quoted from the program rather than described — and it is a good illustration of why
the format is cited by function and line range. Two of its numbers also show the
runtime/source split at its sharpest: the deterministic ones (an occupancy of 0.1875,
an eight-block grid) are recomputed from the program's own expressions, while its
benchmark timings and profiler tables are measurements of one GPU on one afternoon
and are not reproduced at all.

[Lecture 7](07-parallelism.md) is the exception to that last rule, and the reason is
worth recording: **the course publishes this one lecture's runtime output.** The
program's own standard output from a real four-GPU run is committed to the lectures
repo as `var/traces/lecture_07_stdout.txt`, so its measured collective bandwidths,
its per-rank losses and its printed tensors are quoted in
[`raw/slides/07-parallelism.md`](../raw/slides/07-parallelism.md) rather than
withheld. They are marked "(recorded run)" and are measurements of that machine
(Modal, CUDA 13.2, four GPUs) — not values a reader's own run will reproduce.

Lecture 7 also breaks the format in a second way: it genuinely launches multiple
processes, so it **cannot be traced**. Its `spawn()` helper detects a tracer and
falls back to running in one process with every `torch.distributed` call replaced by
a no-op, with rank hard-coded to 0. The stepped-through view therefore shows the code
with no communication happening — which is exactly why the separate stdout file
exists. See [torch.distributed](torch-distributed.md#a-wrinkle-in-reading-the-lecture).

[Lecture 10](10-inference.md) is the executable format at its most unusual: almost
none of its numbers are runtime measurements, because **it computes symbolically**.
FLOPs and bytes are accumulated as sympy expressions in $B, S, T, D, F, N, K, H, L,
V$, simplified, and only then substituted with a Llama 2 13B configuration — so
where lecture 6 had to withhold machine-dependent timings and lecture 7 could only
quote them because the course published a stdout file, lecture 10 has nothing
machine-dependent to withhold at all. Every `@inspect` value in
[`raw/slides/10-inference.md`](../raw/slides/10-inference.md) was reproduced by
evaluating the lecture's own expression, and each matches the `assert` the source
makes about it.

That format has a second consequence worth knowing when reading it: the source's
`assert` statements are the lecture *checking its own claims*, so they are the
authoritative statement of what each derivation should come out to. Where a
hand-written comment disagrees with the computed value — lecture 10 has three such
slips, recorded in its slide file's front matter — the arithmetic is the part to
trust.

A crawl of the course website finds the PDFs and misses the programs entirely,
since the programs are not linked as documents — worth knowing if you are
extending this knowledge base.

## Sources

- [Lecture 1](01-overview-tokenization.md) at [19:26], where the format is
  introduced and demonstrates itself
- [`lecture_01.py` transcription](../raw/slides/01-overview-tokenization.md)
- [`lecture_02.py` transcription](../raw/slides/02-pytorch-resource-accounting.md)
  — the second worked example of the format, and the one with machine-dependent
  values in it

## The other half: the deck lectures

Four of the eight lectures covered so far are **not** executable programs. Lectures
3, 4, 5 and 8 are Tatsunori Hashimoto's PDF slide decks, transcribed page by page
into `raw/slides/` from the rendered pages.

Citation practice differs accordingly, and getting it wrong makes a citation
unfollowable:

- **Executable lectures** (1, 2, 6, 7) — cite the **function name and source line
  range**. There are no slide numbers.
- **Deck lectures** (3, 4, 5, 8) — cite the **slide number**, which for all four
  decks means the **PDF page number**, since none of them prints a folio on any
  page.

Lecture 8's deck is the largest so far at 73 pages, and the most figure-dense
relative to its text: 86 raster images against about 2,749 words, roughly 38 words
per page. As with lectures 3–5, the figures *are* the content rather than an
illustration of it, so the slide file describes every chart, diagram and table in
prose. See [`raw/slides/08-parallelism-2.md`](../raw/slides/08-parallelism-2.md).

One consequence worth knowing when reading a deck transcription: the decks contain
their own typos and self-contradictions, which are transcribed **as printed** and
flagged rather than silently repaired. Lecture 8's deck misspells FSDP as "FDSP" on
three separate slides, and slides 70 and 72 disagree about the same model's
pipeline-parallel degree.
