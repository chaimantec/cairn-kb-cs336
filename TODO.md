# KB build — CS336 (Language Modeling from Scratch, Stanford, Spring 2026)

Coverage: **all 18 lectures**, plus **Assignments 1, 2 and 3**, each with a wiki page and a
full handout transcription. See `kb.json` for machine-readable coverage.

**This file holds open work only.** Every run reads it in full before doing anything. While a
run is in progress its checklist lives here; once the run is committed and pushed, the whole
section moves verbatim to [`BUILD_LOG.md`](BUILD_LOG.md), which holds runs 1-25. An unchecked
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
- [ ] **Math rendering: 19 errors on lines older than run 24.** Found by `check_math.mjs
      --github` over the files run 24 touched: INDEX.md 10, wiki/09-scaling-laws.md 3,
      wiki/11-scaling-laws-in-the-wild.md 2, wiki/data-scaling-laws.md 3,
      wiki/scaling-laws.md 1. Most are `\,` `\;` `\|` that GitHub's markdown strips (use
      `\thinspace`, `\mskip{5mu}`, `\Vert`), and formulas GitHub shows as raw text. Run
      `check_math.mjs` over the whole repo first; run 20 counted 22 chat errors repo-wide.
- [ ] **Assignment 5** — no page and no transcription. Not scheduled; when asked,
      follow runs 22, 24 and 26 (transcription first, then the page).

## Run 26 — Assignment 4: handout transcription and page

Source: the Spring 2026 handout, *CS336 Assignment 4 (data): Filtering Language Modeling
Data*, version 26.0.1 (16 pages, Typst 0.14.2, PDF dated 2026-05-07), and the starter repo
at commit `0555bea` (2026-05-07; MIT). Same shape as run 24: no reader, the parent
transcribes from the marked-up text layer.

- [x] Ticks the old "Assignments 4 and 5" line for its A4 half; A5 carried above
- [x] Inputs: per-page text layer, monospace spans, 15 link targets, outline, rows with
      box/code flags; both 2026 PDF versions diffed (only the version number differs);
      the Spring 2025 PDF diffed for what changed
- [x] raw/handouts/assignment4-data.md — 16 page markers, 13 problems, 80 code lines,
      15 links, 47 formulas read against seven crops (no errors); math clean in both renderers
- [ ] wiki/assignment-4-data.md
- [ ] Quote check of the new page
- [ ] Inbound links (data topic pages, 13, 14, course-map, 01, assignment pages) and INDEX
- [ ] Fix: wiki/13 says lecture 14 is "not yet covered"
- [ ] sources.md, LICENSE.md, AGENTS.md, kb.json
- [ ] verify_kb.py clean, review section read
- [ ] Commit and push
- [ ] Move this section to BUILD_LOG.md
