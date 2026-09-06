# MinHash and locality-sensitive hashing

How to find near-duplicate documents in a web-scale corpus in linear time. This is
the one genuinely algorithmic passage in CS336's data lectures — about twenty
minutes of [lecture 14](14-data-filtering-dedup-mixing.md) (≈27:18–48:34), with
running code — and it is the machinery behind the fuzzy deduplication step that
[deduplication](deduplication.md) describes at the pipeline level.

## The problem: deduplication is quadratic and must not be

Filtering asks a question about one document at a time — is this good? — so it
parallelizes trivially and runs in linear time. Deduplication does not:
"deduplication is fundamentally about comparing items to other items", and at web
scale "you can't do the $n^2$ thing where you compare everything to everything"
(≈28:03). Everything below exists to get that comparison down to linear.

Before choosing an algorithm, the lecture fixes a three-part design space, and
every scheme in this article is a choice within it (≈27:18):

1. **What is an item?** A sentence, a paragraph, or a whole document.
2. **How do you match?** Exact match, existence of a common sub-item, or a
   *fraction* of common sub-items.
3. **What action do you take?** Remove all copies, or remove all but one.

## Hash functions, and which kind to use

A hash function maps an item to a much smaller value; a **collision** is
$h(x) = h(y)$ for $x \neq y$. The relevant trade-off is speed against collision
resistance (≈28:49):

- **Cryptographic** hashes (SHA-256) are collision resistant and slow. They are
  what Bitcoin uses.
- **Fast** hashes (DJB2, MurmurHash, CityHash) are not collision resistant and are
  what hash tables use, "where hash collisions aren't the end of the world".

Deduplication uses the fast kind. The lecture's code uses **MurmurHash** through
the `mmh3` package, and `mmh3.hash("hello")` evaluates to `613153351`.

## Exact deduplication

The simplest instantiation: item = string, match = exact, action = remove all but
one. The lecture's six-item example is

```python
items = ["Hello!", "hello", "hello there", "hello", "hi", "bye"]
hash_items = itertools.groupby(sorted(items, key=mmh3.hash), key=mmh3.hash)
deduped_items = [next(group) for h, group in hash_items]
```

which yields `['hi', 'bye', 'hello', 'hello there', 'Hello!']` — five items from
six. Only the repeated `"hello"` goes; `"Hello!"` survives, because exact means
exact. The output is in hash order rather than input order.

The reason to write it this way is not elegance. It is "written in a sort of MapReduce style way, which makes it more easily parallelizable and scalable" (≈29:36):
sort-then-group by hash is a shuffle and a reduce, so it distributes across a
cluster. That is what makes exact deduplication linear.

**C4** applies exactly this with the item set to **3-sentence spans**, matched
exactly, all but one removed. The lecture flags the consequence rather than hiding
it: removing a shared 3-sentence span from the middle of a document "breaks the
coherence" of what is left (≈31:10). C4's documents can have holes cut in them.

Exact matching is "very clear what's happening, but this isn't really good enough
for the messy web data" (≈29:36), because most real duplication is *near*
duplication — the same page with a different date, a template with one entity
swapped, a licence pasted into a different repository.

## Jaccard similarity

The similarity measure for sets:

$$\text{Jaccard}(A, B) = \frac{|A \cap B|}{|A \cup B|}$$

It runs from 0 (disjoint) to 1 (identical). On the lecture's worked example
$A = \{1,2,3,4\}$ and $B = \{1,2,3,5\}$, the intersection is $\{1,2,3\}$ and the
union is $\{1,2,3,4,5\}$, so **Jaccard = 3/5 = 0.6** (≈31:58). Getting the union
right matters — it is 5, not 4 — because the MinHash argument below counts rows of
exactly that union.

Two documents are declared **near duplicates** when their Jaccard similarity
exceeds a threshold, "let's say 0.99" (≈31:58). The algorithmic problem is then:
find all such pairs in linear time.

## MinHash

A MinHash is a random hash function $h$ with the property

$$\Pr[h(A) = h(B)] = \text{Jaccard}(A, B)$$

which is the bridge the whole method rests on: hashing is what makes things linear,
Jaccard is the quantity you want, and this property connects them (≈32:45).

The inversion is worth pausing on. Ordinarily you design a hash function so that
distinct items land in distinct buckets — collisions are the failure mode. Here
"you actually want collisions — not arbitrary collisions, but you want to control the collisions in a certain way to align with a similarity" (≈33:33). Similar
things should collide *more often* than dissimilar ones.

The construction is one line — hash every element of the set and keep the smallest:

```python
def minhash(S: set[str], seed: int):
    return min(mmh3.hash(x, seed) for x in S)
```

Taking the minimum is arbitrary: "you can take a max too, it doesn't really
matter, it's just a way to break ties" (≈33:33).

### Why the property holds

The lecture's proof is the characteristic-matrix picture (≈34:18–35:50). Write the
union as rows, with a column per set:

```
item | A | B
1    | 1 | 1
2    | 1 | 1
3    | 1 | 1
4    | 1 | 0
5    | 0 | 1
```

A random hash function **induces a random permutation over the items** — it "elects
one of these rows to be first". Taking the minimum hash within a set is the same as
taking whichever of its members comes first under that permutation. Now ask when
the two minima agree:

- If row 1, 2 or 3 comes first, it is present in both sets, so it is the first
  element of A *and* of B — they agree.
- If row 4 or 5 comes first, it is present in only one set, so the two minima are
  different.

Every row is equally likely to come first, three of the five rows are shared, so
the minima agree with probability $3/5$ — which is the Jaccard similarity. The
rows are the union, which is why the union's size is the denominator.

The lecture then checks this empirically with 100 hash functions, seeded
`range(100)`:

```python
matches = [minhash(A, seed) == minhash(B, seed) for seed in range(100)]
estimated_jaccard = len([m for m in matches if m]) / len(matches)
assert abs(estimated_jaccard - jaccard) < 0.01
```

**The estimate comes out at exactly 0.6** — 60 of the 100 seeds match — so the
assertion passes with a difference of zero. Landing exactly on the true value is
luck at $n = 100$; the guarantee is only that the estimate concentrates as $n$
grows. Because the seeds are fixed, this reproduces identically on any machine.

The payoff is the one that matters: "the key thing with a MinHash is that I don't
have to do the $n^2$ thing — I can compute the MinHash of A and the MinHash of B
and the MinHash of a different set, and I just look for collisions" (≈36:36).

## Why MinHash alone is not enough

A single MinHash collision is one Bernoulli draw with probability equal to the
Jaccard similarity. So "a collision doesn't tell us that Jaccard is above some
threshold, which is what we want" (≈37:29). On average more similar items collide
more often, "but it's very stochastic, right? The variance is quite large here" (≈38:20).

What is needed is a **phase transition**: collide with probability near 1 above the
threshold and near 0 below it. Since a single hash gives probability exactly equal
to similarity, the probabilities have to be *sharpened*.

## Locality-sensitive hashing

Use $n$ hash functions and break them into **$b$ bands of $r$ hash functions each**,
with $n = b \cdot r$. With $n = 12$, $b = 3$, $r = 4$ the bands are

```
h1 h2 h3 h4  |  h5 h6 h7 h8  |  h9 h10 h11 h12
```

**A and B collide if for *some* band, *all* of that band's hash functions return
the same value** (≈39:53). That is an **and-or structure** — and within a band, or
across bands — and it is "doing the lifting" (≈40:38).

The probability follows in two steps (≈41:26–42:11). A fixed band matches only if
all $r$ of its hashes agree, which happens with probability $s^r$ where
$s = \text{Jaccard}(A,B)$ — "generally quite low; it's exponential in $r$". The
pair collides if *some* band matches, which is one minus the probability that all
$b$ bands fail:

$$\Pr[\text{collide}] = 1 - \left(1 - s^{\,r}\right)^{b}$$

At $s = 0.8$, $b = 5$, $r = 10$ this evaluates to **0.4333** — two documents that
are 80% similar collide less than half the time under those parameters.

Plotted against $s$, this is an **S-shaped curve** running from 0 at $s = 0$ to 1
at $s = 1$, and the S is exactly what was wanted: "we're trying to get this to be
like a phase transition" (≈42:59).

### What b and r each do

The lecture evaluates the same formula at seven similarity values under three
settings. All three are reproduced here from the lecture's own code:

| Jaccard | b=10, r=10 | b=10, r=20 | b=20, r=20 |
| --- | --- | --- | --- |
| 0.70 | 0.249144 | 0.007951 | 0.015838 |
| 0.75 | 0.439885 | 0.031263 | 0.061549 |
| 0.80 | 0.678860 | 0.109491 | 0.206993 |
| 0.85 | 0.888356 | 0.326527 | 0.546434 |
| 0.90 | 0.986261 | 0.726449 | 0.925170 |
| 0.95 | 0.999892 | 0.988195 | 0.999861 |
| 0.98 | 1.000000 | 0.999984 | 1.000000 |

**Increasing $r$ sharpens the threshold and moves the curve right** — everything
becomes harder to match, "because you have more hash functions inside a bucket, so
it sharpens that exponent" (≈45:21). Column 1 → column 2 drops 0.70 from 0.25 to
0.008 and 0.80 from 0.68 to 0.11.

**Increasing $b$ moves the curve left** — easier to match, because "if you have more bands, then there are more chances of matching" (≈46:09). The lecture reads the
0.90 row across: it "was only 0.72 and now it's 0.92", with the top of the range
rising too but "not by that much".

The two knobs are not redundant. Doubling $b$ undoes some of the *shift* from
doubling $r$ but not the *sharpening*: column 3 still separates 0.80 (0.21) from
0.90 (0.93) far more decisively than column 1 does. "You can drive this phase
transition to be as sharp as you want by increasing $b$ and $r$" — at a cost, since
more hash functions is more work (≈46:09).

### The design rule

The phase transition sits at

$$\text{threshold} = \left(\frac{1}{b}\right)^{1/r}$$

so the recipe is: pick the Jaccard cutoff you want, choose $b$ and $r$ so that this
expression equals it, and then scale both up to sharpen the transition around it
(≈46:58).

At the threshold, a fixed band matches with probability exactly $1/b$, so the
collision probability is $1 - (1 - 1/b)^{b}$, which tends to
$1 - 1/e \approx 0.632$ as $b$ grows — "it's a constant" (≈47:45). The centre of
the phase transition is therefore **not a coin flip and not a certainty**: a pair
sitting exactly on the threshold is caught about 64% of the time, and everything
below goes to 0 and everything above to 1 as $b$ and $r$ grow.

### A real setting

*Deduplicating Training Data Makes Language Models Better*
([arXiv 2107.06499](https://arxiv.org/pdf/2107.06499)) uses **n = 9000 hash
functions, b = 20 bands, r = 450** per band. Putting those into the formula:

- threshold = $(1/20)^{1/450}$ = **0.9934**
- probability a fixed band matches at the threshold = $1/20$ = **0.05**
- probability of collision at the threshold = $1 - (1 - 1/20)^{20}$ = **0.6415**

So this configuration is hunting for documents that are **99.3% similar** — near
duplicates in the strict sense, not merely related pages.

The combination has a name: "this method is called MinHash LSH, because LSH really
works for any hash functions. For deduplication of language model processing, we're
using the MinHash, which approximates the Jaccard" (≈48:34).

## One operational warning

Deduplication is usually run *within* each dataset, and that is not sufficient:
"you actually have to do deduplication across your entire dataset, because often
datasets can be redundant with each other. Sometimes that's not done, but it should be" (≈48:34).
Two separately-deduplicated corpora combined into one training mix are not a
deduplicated corpus.

## See also

- [Deduplication](deduplication.md) — why it matters and where it sits in the
  pipeline, from [lecture 13](13-data-sources-datasets.md)
- [Lecture 14](14-data-filtering-dedup-mixing.md) — the lecture this comes from
- [Data filtering](data-filtering.md) — the step before, and the reason it is
  linear-time while this one is not
- [Benchmark contamination](benchmark-contamination.md) — decontamination is the
  same machinery pointed at the test set, and the lecture calls it "arguably even
  more important" (≈26:30)
- [Data repetition](data-repetition.md) — what happens if the duplicates stay
- [Course material for lecture 14](../raw/slides/14-data-filtering-dedup-mixing.md#jaccard-similarity-and-minhash)
  — the source program, with every computed value
