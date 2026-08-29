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

**This KB now covers one of each.** Lectures 1 and 2 are executable lectures,
transcribed from source text; [Lecture 3](03-architectures.md) is a 67-page deck,
transcribed from the rendered page images. The citation rules differ accordingly —
cite a *function name and line range* for the first kind, a *slide number* for the
second. One wrinkle specific to `lecture_03.pdf`: it prints no page numbers on any
page, so its "slide N" labels are PDF page numbers, and the slide file says so in
its front matter.

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
