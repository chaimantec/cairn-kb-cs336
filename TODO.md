# KB build — CS336 (Language Modeling from Scratch, Stanford, Spring 2026)

Coverage: **all 18 lectures**, plus **Assignments 1 to 5**, each with a wiki page and a full
handout transcription (Assignment 5 also its optional supplement). See `kb.json` for machine-readable coverage.

**This file holds open work only.** Every run reads it in full before doing anything. While a
run is in progress its checklist lives here; once the run is committed and pushed, the whole
section moves verbatim to [`BUILD_LOG.md`](BUILD_LOG.md), which holds runs 1-27. An unchecked
box anywhere in the repo means work left to do, and the lines below that start with
`- [ ]` are the complete list.

## Open

- [ ] **Figure audit, lecture 9 — the pages still not audited.** Carried from run 9's "Not
      done (future runs)" list. Its highest-value pages were the ones the wiki quotes: 37, 38,
      39 (critical batch size) and 43, 44, 46, 47 (joint scaling, Chinchilla methods 1-2).
      Pass 3 audited 37, 39, 43 and 47; **38 and 46 remain, and 44's panels (b) and (c)**.
      In all, 21 of the 57 pages are not fully audited (see raw/slides/09's front matter).
      Not scheduled: the run-order rule in AGENTS.md is one audit pass, then stop, and lecture
      9 has had three. Do it if a wiki claim comes to rest on 38, 44 or 46. When re-running,
      have the agent append findings per page.
- [ ] **Decision, not work: the verbatim captions in git history.** Carried from run 10. They
      were removed from the working tree, but remain in git history and on GitHub, since they
      were committed and pushed first. Removing them for real needs a history rewrite
      (`git filter-repo` or BFG) and a force push, which would change every commit SHA,
      including the ones BUILD_LOG.md cites, and would not purge GitHub's unreferenced objects
      without asking support. Blocked on the user's decision; left as-is until then.

## Run 28 — Math rendering, repo-wide (2026-10-12)

Started from `check_math.mjs . --github`: 263 files, 7118 formulas, **chat 22 errors, 125
warnings; github.com 959 errors, 176 warnings**. Ended at 6972 formulas, **chat 0 errors, 5
warnings; github.com 0 errors, 3 warnings**. 109 files changed. Most of it was scripted against the
checker's JSON (formulas matched by file, line and TeX), every stage re-checked live.

- [x] Carried line, ticked here:
      - [x] **Math rendering: 19 errors on lines older than run 24.** Found by `check_math.mjs
            --github` over the files run 24 touched: INDEX.md 10, wiki/09-scaling-laws.md 3,
            wiki/11-scaling-laws-in-the-wild.md 2, wiki/data-scaling-laws.md 3,
            wiki/scaling-laws.md 1. Run 26 found 5 more GitHub errors in wiki/minhash-and-lsh.md
            (lines 75-76 `\{ \}`, line 184 `\,`), on lines no run has changed since lecture 14.
            Run 27 found 14 more on old lines of pages it only gave a see-also line: wiki/15 3,
            wiki/16 2, advantage-estimation-and-baselines 1, dpo 5, ppo 3 (mostly `\,` and `|`
            that GitHub alters).
            Most are `\,` `\;` `\|` that GitHub's markdown strips (use
            `\thinspace`, `\mskip{5mu}`, `\Vert`), and formulas GitHub shows as raw text. Run
            `check_math.mjs` over the whole repo first; run 20 counted 22 chat errors repo-wide.
- [x] **Identical-TeX rewrites, 576 formulas.** `\,` `\;` `\!` `\|` `\{ \}` `\\` to
      `\thinspace` `\mskip{5mu}` `\negthinspace` `\Vert` `\lbrace \rbrace` `\cr`; `*` to
      `\ast`; a `_` after a non-letter to `_ `; inline `< >` to `\lt \gt`; code identifiers
      such as `\text{num\_tokens}` and `lm\_head` to `\verb|num_tokens|` (run 24's precedent;
      `\verb+…+` in the three table rows of raw/slides/11 slide 8, where `|` split the cell);
      `\#` to `\verb|#|`. `\textunderscore` was tried and rejected: KaTeX allows it only in text
      mode, MathJax only in math mode.
- [x] **Structural fixes.** 159 author-affiliation and footnote marks (`Hu$^1$`) became
      Unicode superscripts (`Hu¹`), as the handouts already write them (raw/slides 04, 09, 11,
      15). 21 inline formulas wrapped across a line were joined. 81 runs of adjacent formulas were
      merged (`$10^5$–$10^6$` to `$10^5\text{–}10^6$`, `$a$/$b$/$h$` to `$a/b/h$`, a leading
      `~` or `≈` taken in as `\sim` or `\approx`). 29 `$$` blocks were given their own
      paragraph; two that ended inside a quotation (raw/slides 04 slide 12, 09 slide 6) now close
      the quote before the block.
- [x] **Quoted formulas.** 119 quotation marks around a lone formula were dropped
      (`labelled "$F_0$"` to `labelled $F_0$`); 30 quotations that start with a formula were
      reworded by hand (legend labels into parentheses, `$W^{UK}$ "can be merged…"`). 37
      `top-$k$` forms became `$\text{top-}k$`. BUILD_LOG.md's three lines about LaTeX inside
      quotations now show it as code.
- [x] **GitHub quirks found this run**, each confirmed against the markdown API: no inline
      `\begin{…}` (the three `bmatrix` captions in raw/slides/08 are `\left[{B_1 \atop
      B_2}\right]`); a formula ending in `)` right before `)` fails (8 wrapped in braces); any
      formula containing "Select" fails (`\mathrm{Sel{}ectBatch}`, raw/slides/11 slide 56); math
      in link text (wiki/02, resource-accounting, efficiency); math in italics (five captions
      in wiki/17-multimodality, one in dpo, split around the formula).
- [x] **The chat's 22 errors.** INDEX.md and raw/slides/05: inline formulas wrapped across a
      line (joined, above). wiki/tiling.md: a wrapped formula's second line began `+ `,
      which markdown read as a list item. wiki/data-mixture-selection.md: a space before the
      closing `$`. raw/slides/12 (an adversarial suffix containing `\comment`) and raw/slides/17
      (a model's reply quoting LaTeX) are literal printed text and are now code spans.
- [x] **Literal dollar signs.** raw/slides/05 slide 10 prints `L1$` and `L2$` (cache); they
      paired into a seven-line "formula", now `\$`. raw/slides/15 `~$12-15 … ~$260` escaped.
      raw/slides/04 had a stray `\$` after `\$0.7`, removed.
- [x] **Notation warnings.** 103 `N×` in wiki prose became `$N\times$` (`$336\times 336$` for
      dimensions), 6 ranges `$1.18\text{–}1.41\times$`, two `$\text{pass@}100$`;
      wiki/training-flops.md's "2 × (# data points) × (# parameters)" gloss became prose.
- [x] **Slip, caught and reverted.** The first rewrite pass took the paired `L1$ … L2$` prose in
      raw/slides/05 for a formula and turned the bold `**one**` inside it into `\ast\ast`. Before
      rewriting a flagged formula's TeX, check that it is one: words inside it, or `**`, mean
      two literal dollars have paired.
- [x] **Left as warnings, deliberately.** wiki/17-multimodality.md:142 "(4×)" and "(3×)" sit
      inside an italic caption, where GitHub would not render math; wiki/zero-and-fsdp.md
      "8×A100" is a hardware name. Three `$$` blocks indented inside list items (raw/slides/03
      slide 31, twice; wiki/speculative-sampling.md step 3) render as inline math on github.com;
      moving them out would break the list.
- [x] AGENTS.md — the math convention gains the GitHub rules above, in general form.
- [x] `verify_kb.py` clean; `check_math.mjs . --github` 0 errors in both renderers.
- [ ] Commit and push; `check_math.mjs --github-published` on the pushed commit.
- [ ] Move this section to BUILD_LOG.md.
