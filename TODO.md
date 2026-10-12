# KB build — CS336 (Language Modeling from Scratch, Stanford, Spring 2026)

Coverage: **all 18 lectures**, plus **Assignments 1, 2, 3 and 4**, each with a wiki page and a
full handout transcription. See `kb.json` for machine-readable coverage.

**This file holds open work only.** Every run reads it in full before doing anything. While a
run is in progress its checklist lives here; once the run is committed and pushed, the whole
section moves verbatim to [`BUILD_LOG.md`](BUILD_LOG.md), which holds runs 1-26. An unchecked
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
      (lines 75-76 `\{ \}`, line 184 `\,`), on lines no run has changed since lecture 14.
      Run 27 found 14 more on old lines of pages it only gave a see-also line: wiki/15 3,
      wiki/16 2, advantage-estimation-and-baselines 1, dpo 5, ppo 3 (mostly `\,` and `|`
      that GitHub alters).
      Most are `\,` `\;` `\|` that GitHub's markdown strips (use
      `\thinspace`, `\mskip{5mu}`, `\Vert`), and formulas GitHub shows as raw text. Run
      `check_math.mjs` over the whole repo first; run 20 counted 22 chat errors repo-wide.
- [ ] **Assignment 5** — no page and no transcription. Not scheduled; when asked,
      follow runs 22, 24 and 26 (transcription first, then the page). Run 27, below.

## Run 27 — Assignment 5: both handouts and the page

Source: the starter repo `stanford-cs336/assignment5-alignment` at commit `c2734a2`
(2026-06-04; v2.0.1 per its changelog). It carries two handouts, both Typst 0.14.2: the main
`cs336_spring2026_assignment5_alignment.pdf` (39 pages, dated 2026-05-29) and the "completely
optional" `cs336_spring2026_assignment5_supplement_safety_rlhf.pdf` (18 pages, dated
2026-06-03). Neither PDF has a raster image. Scope agreed with the user: both handouts, one
wiki page with the supplement as its own marked part.

- [x] Inputs: per-page text layer, rows with box/code flags, links, outline; all five 2026
      builds of the main PDF diffed (none changed the printed "Version 26.0.0"; 495ea2b,
      "Typo fix", reverted 10ee9bc's question_only sentences and dropped raw_rewards from
      compute_group_normalized_rewards' return; cd5f183 restored the sentences)
- [x] raw/handouts/assignment5-alignment.md — script draft from span data (run 26's tools,
      adapted: page continuations detected, box rows missed at page feet restored); math,
      the Notation table and Algorithm 1 by four Sonnet readers (pp. 1-10, 11-27, 28-33,
      34-39), appending per page. Verified: 39 markers, 28 problems (90 points, computed),
      186 code lines, 24 links, prose ratio 0.945 (every difference read), numbers.
      New check: per-page math symbols, script levels and sub/superscript side against the
      PDF's span sizes and positions. It found 13 reader errors: 11 expectations with the
      policy's argument inside its subscript (reported by reader 1 as a "printed slip"; the
      span sizes say otherwise), Algorithm 1's x_{(i)} for x^{(i)} (reported as a slip by
      reader 2), and (55)'s limit L/2 for a stacked fraction. Reader 3 inserted a stray
      "img" line and reported it as present in the chunk; it was not. Three reported slips
      confirmed against crops (f' x/f(x) on p. 9, pi_{theta(A=1)}, (65)'s extra paren).
      368 formulas clean in KaTeX and on github.com (next_state via `\verb`; `\char`
      fails in MathJax)
- [x] raw/handouts/assignment5-supplement-safety-rlhf.md — no reader; parent fixed the draft
      and rebuilt the formulas, equations (1)-(3) read against a crop. 18 markers, 16
      problems (61 points, computed), 88 code lines, 5 links, prose ratio 0.961; symbol
      and level check clean; 50 formulas clean in both renderers
- [x] wiki/assignment-5-alignment.md — 28 problems (90 points, 26 B200-hours, computed), the
      seven functions with what each test checks, the derivation as §4.1 builds it, the
      variants and off-policy section, the supplement (16 problems, 61 points), lecture map,
      AI rules (neither handout has any; AGENTS.md lacks A4's "Solve any of the math
      problems"), nine disagreements. Nothing computed beyond the totals
- [x] Quote check: 55 quotations against both handouts, the A3 handout, repo files, every
      transcript and slide file. Three misses were the parent's own: a column label and a
      paraphrase ("right format, wrong answer") set as quotes, and an HTML anchor attribute.
      One artifact (a quote whose source sentence carries link markup), verified by eye
- [x] Claims verified before publishing, six wrong in the draft: REINFORCE's estimate is
      equation (14), not (15); five problems carry GPU budgets, not four; fourteen code
      problems, not thirteen; the handout was revised four times after release, not five
      (also in the transcription header); every test_grpo.py test uses a snapshot, the
      reward test included; and "the tests follow torch.std, not the formula" could not be
      checked without computing a problem's answer, so it was cut
- [x] Disagreement verified at source: `pytest -k test_compute_policy_gradient_loss_off_policy`
      collects the GSPO test too (checked with dummy tests under uvx pytest)
- [x] Licence: the repo's main branch has no licence (MIT only on a stale master branch with
      the 2025 code). User decided: publish both transcriptions as Layer 2 course material,
      and say so in LICENSE.md, both headers, AGENTS.md and kb.json
- [x] Inbound links from 13 pages (01, 15, 16, advantage-estimation-and-baselines,
      assignment-4-data, course-map, dpo, expert-iteration, grpo, length-bias-in-rl, ppo,
      rl-infrastructure, rlvr) and INDEX (start-here, Assignments, raw material, citing)
- [x] Fix: course-map.md's Assignment 5 paragraph carried only Lecture 1's preview; INDEX
      said raw/handouts/ were all MIT-licensed
- [x] sources.md (A5 paragraph; the 2025 supplement's inventory link now 404s on main,
      survives under the spring2025 tag), LICENSE.md (new subsection: not MIT), AGENTS.md
      (layout rows, two lessons), kb.json (179 topic pages, materials.assignments."5", licence
      note, one caveat)
- [x] verify_kb.py clean (hard checks: 197 pages indexed, 245 files' links resolve to tracked
      files, kb.json counts). Its math pass returned no JSON again, so check_math was run on
      all 21 changed files: this run's files are clean; the failures are on older lines
      (INDEX 10, already open; 14 new ones in five pages, added to the open item above)
- [x] Commit and push (2026-10-12)
- [ ] Move this section to BUILD_LOG.md
