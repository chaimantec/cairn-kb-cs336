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
      wiki/scaling-laws.md 1. Run 26 found 5 more GitHub errors in wiki/minhash-and-lsh.md
      (lines 75-76 `\{ \}`, line 184 `\,`), on lines no run has changed since lecture 14. Most are `\,` `\;` `\|` that GitHub's markdown strips (use
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
- [x] wiki/assignment-4-data.md — all 13 problems (71 points, computed), the data given and
      where it lives, each primitive as specified with what its test checks, the English WET
      pool and the Paloma target, the fixed run and its config, lecture map, AI rules (the
      handout has none; AGENTS.md is A3's), nine source disagreements. Nothing computed
      beyond the points total
- [x] Quote check: 65 quotations against the A4 and A3 handouts, every transcript and slide
      file, and the repo's README, AGENTS.md, tests and submission script. One not found,
      "an empty module", which is from the Spring 2025 PDF (checked against its text layer)
- [x] Claims checked before publishing, three wrong in the draft: "nine problems have a
      coding part, seven tested" (ten and eight); the Gopher tests "one rule at a time" (one
      is a valid-input test, six test one bound each); the Jigsaw dataset "is made of"
      Wikipedia comments (the handout says "includes")
- [x] Disagreements verified at source: both §2.1 `wget` addresses 404 (2026-10-12) and
      download_data.py fetches CC-MAIN-2026-12's first listed file instead; the config is
      24 layers / 1024 / 16 heads, which A1's handout calls GPT-2 medium; footnote 5 prints
      U+00A0 where the link has `~`
- [x] Inbound links from 12 pages (01, 13, 14, assignment-1-basics, assignment-3-scaling,
      course-map, data-filtering, deduplication, html-to-text-extraction, minhash-and-lsh,
      quality-classifiers, web-crawling) and INDEX (start-here, Assignments, raw material, citing)
- [x] Fix: wiki/13 said lecture 14 is "not yet covered" (covered since run 15)
- [x] Fix: course-map.md said Unit 4 is "the only one in the course" covered from its own
      lectures; every unit is, since the KB reached 18/18
- [x] Fix: kb.json's A2 caveat said "Assignments 3-5 have no page"; license.assignmentHandout
      named only A1 and A2
- [x] sources.md (A4 paragraph; the 2025 PDF's inventory link now 404s on main, survives
      under the `spring2025` tag and at 4d364a0), LICENSE.md (MIT exception), AGENTS.md (two
      general lessons, layout rows), kb.json (178 topic pages, materials.assignments."4", one
      caveat, builtBy)
- [x] verify_kb.py clean (hard checks; link checker 5052 links, 0 broken). Its math pass
      returned no JSON, so check_math was run on all 19 changed files: every finding is on a
      line older than this run (INDEX 10, minhash-and-lsh 5), recorded in the open item above
- [ ] Commit and push
- [ ] Move this section to BUILD_LOG.md
