---
title: "Lecture 13: Data I — Sources and Datasets"
lecture: 13
video: https://www.youtube.com/watch?v=-qm0ln33G24
source: copy-edited from the YouTube auto-captions
verbatim_original: not committed - regenerate from the video (see below)
material: ../slides/13-data-sources-datasets.md
---

# Lecture 13: Data I — Sources and Datasets — transcript

**This is the edited transcript.** The auto-captions have been repunctuated,
segmented into sentences, stripped of filler ("um," "uh," "you know," "kind
of," "right?") and false starts, and had mis-heard technical terms restored.
No content was added, removed, or reordered, and every `[MM:SS]` marker is
preserved in its original position — content that was under a marker is
still under that marker.

The verbatim captions are **not committed to this repo** — they are a complete
reproduction of the lecture and none of that text is this KB's own work. They stay
reproducible on demand: the `cairn-kb` skill's `fetch_transcript.py` run on the
video id in this file's front matter, piped through `transcript_to_md.py`,
reproduces them exactly, so the checks below can be re-run by anyone who wants to
verify this edit.

Terminology was cross-checked against
[the lecture's source program](../slides/13-data-sources-datasets.md)
(`lecture_13.py`), which is the authoritative written transcription of this
lecture's content — dataset names, paper citations, author/organization
attributions, and numbers.

**Verification.** All three checks pass.

- **Timestamps** — 107 markers, identical sequence, in order, diffed against the
  verbatim captions.
- **Numbers** — every number in the captions is preserved. Where a spoken number
  differs slightly from the deck's own figure (e.g. "a million pages" for WebText,
  where the deck says 8 million; "200K books" for Books3, where the deck says
  196K; "six trillion tokens" for Nemotron-CC, where the deck says 6.3T), that is
  treated as the lecturer's own spoken approximation, not a caption error, and is
  left as said rather than corrected to match the deck.
- **Word ratios** — all 107 paragraphs read as plausible filler/false-start
  removal against the verbatim captions; nothing was trimmed beyond disfluencies
  and the restorations documented below.

**Restorations made.** These are places where the captions produced a *wrong
word* or dropped/mangled a word, and the text now reads differently. Grouped by
kind; unless noted otherwise, each is confirmed against
[the material file](../slides/13-data-sources-datasets.md).

*Dataset/format/tool names:*

| Caption | Restored | Confirmed by |
| --- | --- | --- |
| "WAT file" (34:42, 35:29) | WET file | material: "Two formats: WARC ... WET: converted to text (lossy process)" — the paragraph describes exactly this lossy HTML→text conversion, not WAT (which is metadata, not text) |
| "Brazilia Parser" (35:29) | resiliparse | material: "Tools to convert HTML to text: trafilatura, resiliparse" — named as a pair with trafilatura, exactly as heard |
| "Data Comp at WLM paper" (35:29) | DataComp-LM (DCLM) paper | material's own section "DataComp-LM / DCLM (2024)" cites the same WET-vs-tools ablation, arXiv 2406.11794 |
| "archive" as the preprint server, throughout (36:14, 41:33 ×2, 42:20 ×2, 43:06, 1:00:54, 1:01:40) | arXiv | material's "arXiv" section; distinguished from the legitimate "GitHub archive" (→ GitHub Archive) which is a different thing |
| "Archive GitHub" (54:41) | arXiv, GitHub | material lists arXiv and GitHub as separate Pile components; the caption ran the two list items together |
| "capture" (8:32) | CAPTCHA | material: "present CAPTCHAs" |
| "GitHub archive" (41:33) | GitHub Archive | material: "GitHub Archive" (hourly event-stream snapshots) |
| "software heritage" (41:33, 1:11:44) | Software Heritage | material: "Software Heritage" |
| "Olmo" (3:10) | OLMo | material: "OLMo from AI2" |
| "Qwen 3.5 397B" (3:10) | Qwen3.5-397B | material: "Qwen3.5-397B-A17B is an instruct model" (the "-A17B" suffix is not spoken, so not restored) |
| "Gwen 3" (1:09:20) | Qwen3 | material: "Qwen3 trained on 36T" |
| "Refined Web" (1:01:40, 1:06:18) | RefinedWeb | material's own section header "RefinedWeb" |
| "fine-web" (1:02:28) | FineWeb | material's own section header "FineWeb" |
| "Doma" (1:17:08 ×2) | Dolma | material's own section header "Dolma"; spelled correctly elsewhere in this same transcript (1:03:13) |
| "Bibliotek" (56:16) | Bibliotik | material: "the shadow library Bibliotik" |
| "Books 3" (throughout) | Books3 | material's own spelling, "Books3" |
| "Red Pajama V1" (1:00:54 ×2) | RedPajama v1 | material's own spelling, "RedPajama v1" |
| "web text" for the GPT-2 dataset (47:43, 51:37, 52:22) | WebText | material's own section header "WebText and GPT-2 (2019)" |
| "web text" for the GPT-3-era expanded version (53:09, 53:54) | WebText2 | material: "WebText2 (WebText expanded with more links)" |
| "books one and books two" (53:09) | Books1 and Books2 | material: "(Mysterious) Internet-based books corpora (Books1, Books2)" |
| "fast text model" (1:07:50) / "fast class classifier" (1:06:18) | fastText model / fastText classifier | material: "Trained a fastText classifier" (DCLM section) |
| "DCLAM" (1:04:01) / "DCL M" (1:07:04 ×2, 1:07:50) | DCLM | material's own section header "DataComp-LM / DCLM (2024)" |
| "data on pool" (1:04:47) | DCLM-pool | material: "Processed CommonCrawl to produce DCLM-pool (240T tokens)" |
| "Nematron" (1:07:04 ×2) | Nemotron | material's own section header "Nemotron-CC (2024)" |
| "Open or Hermes" (1:06:18) | OpenHermes | material: "OpenHermes-2.5" |
| "Eli5" (1:06:18 ×2) | ELI5 | material: "ELI5" |
| "score find" (1:07:50) | score FineWeb | material: "Prompt Nemotron-340B-instruct to score FineWeb documents based on educational value" |
| "PI removal" (1:02:28) / "PI reduction" (1:11:44) | PII removal / PII reduction | material: "Anonymize email and public IP addresses (PII)" |
| "fear it" (43:54, in "you appeal to ... fear it") | fair use | matches "appeal to fair use," used correctly elsewhere in this same transcript (e.g. 19:18, 44:40) |
| "PushShift" (1:03:13) | Pushshift | material: "the Pushshift project" |
| "137 new repos" (1:10:54) | 137 million repos | material: "git clone'd 137M repositories" |
| "open way models" (1:17:55) | open-weight models | standard term, matches "open weight model" used correctly earlier in this same transcript (0:04) and material's "Open-weight models" |
| "the pile" (53:54, 1:00:54) | The Pile | material's own section header "The Pile (2021)" |
| "Stack V2" (1:13:19) | Stack v2 | material's own spelling, "Stack v2" |
| "clone... 137" / "since it was cleared by 2022" (1:10:54) | "since it was clear by 2022" | grammar smoothing only, not a name — "cleared" doesn't parse; "clear" does |
| "paralyze that effort" (1:36) | parallelize that effort | homophone correction — sense of the sentence (spreading work across many people) requires "parallelize," not "paralyze" |

*Person/organization names not tied to a single named dataset:*

| Caption | Restored | Confirmed by |
| --- | --- | --- |
| "Shane Lampray" (9:18) | Shayne Longpre | the paper title itself, "Consent in Crisis," is already correct in the captions; Shayne Longpre (MIT) is the lead author of "Consent in Crisis: The Rapid Decline of the AI Data Commons" (arXiv 2407.14933), the paper material cites for this exact section |

**Restorations made that are NOT attested in this lecture's own material file**,
listed separately with evidence so a reader can weigh them:

- **Read the Docs** (11:38), for "read the docs was also getting hammered" — a
  real, well-known hosted documentation platform (readthedocs.org) that publicly
  reported being hit hard by AI-crawler traffic in 2024. The material file doesn't
  cover this specific anecdote.
- **OAI-SearchBot**, **PerplexityBot**, **ChatGPT-User**, **ClaudeBot** (7:46) —
  the captions read "OAI search bot," "perplexity bot," "chat GPT user," "Claude
  bot." These are the actual, publicly documented `robots.txt` user-agent strings
  for OpenAI's search crawler, Perplexity's crawler, OpenAI's browsing agent, and
  Anthropic's crawler, respectively — phonetically near-exact matches, and the
  kind of list a robots.txt discussion would name. Not in the material file, which
  doesn't enumerate specific bot names.
- **EleutherAI** (53:54), for "a grassroots effort uh from Eleuther AI" — the
  well-known organization behind The Pile (arXiv 2101.00027, cited in material),
  standard one-word branding.
- **ElevenLabs** (30:51), for "11 labs" — a well-known AI voice-synthesis company;
  "11 labs" is a direct phonetic rendering. Not covered in the material file (this
  is a student question, off the lecture's planned content).
- **MPT** (1:18:42), the weakest of these restorations — the captions read "the
  first LLaMA, uh, MBT, um, and Qwen" in a list of comparison baselines. MPT
  (MosaicML Pretrained Transformer) is a well-known open LLM family from the same
  era, and a very plausible peer to "the first LLaMA" and "Qwen" in this kind of
  comparison table; "MBT"/"MPT" are a one-consonant ASR slip. But this is an
  editorial reading, not a certain identification — treat it as such.

**`[Ed: unclear]` markers left in place, because guessing felt riskier than
flagging:**

- **7:46**, "Cora bot" — sits in the same robots.txt list as OAI-SearchBot,
  PerplexityBot, ChatGPT-User and ClaudeBot (all restored above), so it is very
  likely a mangled crawler name, but no candidate name matched with confidence.
  Left verbatim with a flag rather than guessed.
- **30:51**, "voice dating at 11 labs" — "11 labs" is confidently ElevenLabs (see
  above), but "voice dating" does not parse as a coherent ElevenLabs product or
  feature. Left verbatim with a flag.
- **43:06**, "Are you using crawlers like Open or crawl and stuff like that?" —
  part of a student's rambling question about crawler tooling; "Open or crawl"
  does not resolve to any tool name with confidence (Apache Nutch, the crawler
  material names for Common Crawl, was considered but doesn't fit phonetically).
  Left verbatim with a flag.

**Student questions.** Following the KB's convention (see lecture 12's
transcript), wherever the recording makes clear someone from the floor is
speaking — as distinct from Percy paraphrasing a question in his own words —
the transcript marks it `*[Question from the floor: ...]*`, using the captions'
own words with only light punctuation. Unlike lecture 12, this lecture's captions
mostly capture only Percy's own paraphrase of what he was asked ("the question
is...") rather than the student's actual words, so most of the "any questions?"
exchanges in this lecture are left as ordinary running prose — there is nothing
of the student's own speech to bracket off. Two exceptions:

- **30:51–31:37**: a `>> [clears throat] >>` speaker-turn marker in the raw
  captions brackets an inaudible exchange with no recoverable words; marked
  `*[Question from the floor, inaudible]*` rather than invented.
- **43:06–43:54**: two questions where the captions do run the student's actual
  words together with Percy's, without a speaker break — "Are there restrictions
  on data generated by models?" / "Yeah, by using that data" (about synthetic
  data), and "Are you using crawlers like [unclear] ... how do you ensure that
  those types of websites [with pirated content] don't show up in your crawl?"
  (about filtering shadow libraries out of a crawl). The second question's own
  words span the 43:06/43:54 marker boundary and are marked
  `*[Question from the floor: ...]*` / `*[continued from the floor: ...]*`
  accordingly, split exactly where the captions split it.

---

