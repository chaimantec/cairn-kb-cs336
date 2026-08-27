# Sources — CS336: Language Modeling from Scratch (Spring 2026)

Course website: <https://cs336.stanford.edu> · lecture source repo:
<https://github.com/stanford-cs336/lectures> · crawled 2026-08-27.

Nothing is committed into this repo as a binary. Every document below is recorded
at its canonical URL, because this knowledge base is read by an agent that
navigates markdown — a PDF blob in the repo would be unreadable to it — and the
current lecture decks alone total roughly 50 MB.

**This KB currently covers Lecture 1 only.** The inventory below spans the whole
course (and its two earlier offerings) so that provenance is recorded once, but
only Lecture 1 has been transcribed into `wiki/` and `raw/`. See `kb.json`.

## How CS336 publishes its lectures

CS336 splits its material by instructor, and the two halves are different kinds of
artifact. This matters for anyone citing them:

- **Percy Liang's lectures are executable Python programs** — `lecture_NN.py` —
  whose *execution* delivers the lecture. They are rendered by a trace viewer at
  `https://cs336.stanford.edu/lectures/?trace=lecture_NN`, which steps through the
  program and displays the value of each annotated variable as it runs. There are
  no slides and no slide numbers. The source file is the authoritative written
  form, and it is plain text.
- **Tatsunori Hashimoto's lectures are conventional PDF decks** — `lecture_NN.pdf`
  — in the same repository.

A crawl of the course website finds the PDFs and not the programs, since the
programs are not linked as documents. Both are listed below.

## Executable lectures (Percy) — `.py`

| # | Lecture | Source | Rendered |
| --- | --- | --- | --- |
| 1 | Overview, tokenization | [`lecture_01.py`](https://github.com/stanford-cs336/lectures/blob/main/lecture_01.py) | [trace](https://cs336.stanford.edu/lectures/?trace=lecture_01) |
| 2 | PyTorch (einops), resource accounting | [`lecture_02.py`](https://github.com/stanford-cs336/lectures/blob/main/lecture_02.py) | [trace](https://cs336.stanford.edu/lectures/?trace=lecture_02) |
| 6 | Kernels, Triton | [`lecture_06.py`](https://github.com/stanford-cs336/lectures/blob/main/lecture_06.py) | [trace](https://cs336.stanford.edu/lectures/?trace=lecture_06) |
| 7 | Parallelism | [`lecture_07.py`](https://github.com/stanford-cs336/lectures/blob/main/lecture_07.py) | [trace](https://cs336.stanford.edu/lectures/?trace=lecture_07) |
| 10 | Inference | [`lecture_10.py`](https://github.com/stanford-cs336/lectures/blob/main/lecture_10.py) | [trace](https://cs336.stanford.edu/lectures/?trace=lecture_10) |
| 12 | Evaluation | [`lecture_12.py`](https://github.com/stanford-cs336/lectures/blob/main/lecture_12.py) | [trace](https://cs336.stanford.edu/lectures/?trace=lecture_12) |
| 13 | Data (sources, datasets) | [`lecture_13.py`](https://github.com/stanford-cs336/lectures/blob/main/lecture_13.py) | [trace](https://cs336.stanford.edu/lectures/?trace=lecture_13) |
| 14 | Data (filtering, dedup, mixing, synthetic) | [`lecture_14.py`](https://github.com/stanford-cs336/lectures/blob/main/lecture_14.py) | [trace](https://cs336.stanford.edu/lectures/?trace=lecture_14) |
| 17 | Alignment, multimodality | [`lecture_17.py`](https://github.com/stanford-cs336/lectures/blob/main/lecture_17.py) | [trace](https://cs336.stanford.edu/lectures/?trace=lecture_17) |

Supporting modules used by those programs: [`references.py`](https://github.com/stanford-cs336/lectures/blob/main/references.py)
(every paper citation, as structured data), [`lecture_util.py`](https://github.com/stanford-cs336/lectures/blob/main/lecture_util.py),
[`facts.py`](https://github.com/stanford-cs336/lectures/blob/main/facts.py).

**Transcribed in this KB:** `lecture_01.py` → [`raw/slides/01-overview-tokenization.md`](raw/slides/01-overview-tokenization.md).

## Slide decks (Tatsu) — `.pdf`, Spring 2026

| # | Lecture | Deck |
| --- | --- | --- |
| 3 | Architectures, hyperparameters | [`lecture_03.pdf`](https://github.com/stanford-cs336/lectures/blob/main/lecture_03.pdf) |
| 4 | Attention alternatives and mixture of experts | [`lecture_04.pdf`](https://github.com/stanford-cs336/lectures/blob/main/lecture_04.pdf) |
| 5 | GPUs, TPUs | [`lecture_05.pdf`](https://github.com/stanford-cs336/lectures/blob/main/lecture_05.pdf) |
| 8 | Parallelism | [`lecture_08.pdf`](https://github.com/stanford-cs336/lectures/blob/main/lecture_08.pdf) |
| 9 | Scaling laws | [`lecture_09.pdf`](https://github.com/stanford-cs336/lectures/blob/main/lecture_09.pdf) |
| 11 | Scaling laws | [`lecture_11.pdf`](https://github.com/stanford-cs336/lectures/blob/main/lecture_11.pdf) |
| 15 | Mid/post-training (SFT/RLHF) | [`lecture_15.pdf`](https://github.com/stanford-cs336/lectures/blob/main/lecture_15.pdf) |
| 16 | Post-training — RLVR | [`lecture_16.pdf`](https://github.com/stanford-cs336/lectures/blob/main/lecture_16.pdf) |

None of these has been transcribed yet.

## Assignments — Spring 2026

| # | Assignment | Repo | Handout |
| --- | --- | --- | --- |
| 1 | Basics (tokenizer, Transformer, optimizer, training) | [repo](https://github.com/stanford-cs336/assignment1-basics/tree/main) | [PDF](https://github.com/stanford-cs336/assignment1-basics/blob/main/cs336_assignment1_basics.pdf) |
| 2 | Systems (Triton kernels, DDP, optimizer sharding) | [repo](https://github.com/stanford-cs336/assignment2-systems/tree/main) | [PDF](https://github.com/stanford-cs336/assignment2-systems/blob/main/cs336_assignment2_systems.pdf) |
| 3 | Scaling laws | [repo](https://github.com/stanford-cs336/assignment3-scaling/tree/main) | [PDF](https://github.com/stanford-cs336/assignment3-scaling/blob/main/cs336_assignment3_scaling.pdf) |
| 4 | Data (Common Crawl → text, filtering, MinHash dedup) | [repo](https://github.com/stanford-cs336/assignment4-data/tree/main) | [PDF](https://github.com/stanford-cs336/assignment4-data/blob/main/cs336_assignment4_data.pdf) |
| 5 | Alignment (DPO, GRPO) | [repo](https://github.com/stanford-cs336/assignment5-alignment/tree/main) | [PDF](https://github.com/stanford-cs336/assignment5-alignment/blob/main/cs336_spring2026_assignment5_alignment.pdf) |

Lecture 1 links the Assignment 1 handout as
`cs336_spring2026_assignment1_basics.pdf`; the file currently served at that repo
path is `cs336_assignment1_basics.pdf`. Both are linked above — use the repo link
if one 404s.

## Other course links referenced in Lecture 1

| What | URL |
| --- | --- |
| Spring 2025 lecture playlist (2nd offering) | <https://www.youtube.com/playlist?list=PLoROMvodv4rOY23Y0BoGoBGgQ1zmU_MT_> |
| AI policy guide | <https://docs.google.com/document/d/1SZAlExB1qAc9izHt54gwunNpjKE6wXb8Y7yA_e-baK8> |
| Compute guide (Modal) | <https://docs.google.com/document/d/1cHE0iKVyXLJ3XpIs2XuXTmZ-HMmPk2hIPeCvy-AydMg> |
| Karpathy's tokenization video | <https://www.youtube.com/watch?v=zduSFxRajkE> |
| tiktokenizer (interactive) | <https://tiktokenizer.vercel.app/?encoder=gpt2> |
| GPT-5 tokenizer vocabulary dump | <https://github.com/stanford-cs336/lectures/blob/main/var/gpt5_tokenizer_vocab.txt> |
| *How to Scale Your Model* (recommended book) | <https://jax-ml.github.io/scaling-book/> |
| Assignment 1 leaderboard (Spring 2025) | <https://github.com/stanford-cs336/spring2025-assignment1-basics-leaderboard> |
| Stanford CGOE (lecture recording) | <https://cgoe.stanford.edu/> |
| Modal (compute sponsor) | <https://modal.com/> |

Every paper cited in Lecture 1 is listed with its URL inside
[`raw/slides/01-overview-tokenization.md`](raw/slides/01-overview-tokenization.md),
resolved from the course's own `references.py`.

## Earlier offerings

The course website links its two previous offerings, and the crawl picked up their
material as well. These are **not** the current course and should not be cited as
Spring 2026 content, but they are the published record of how the course has
changed.

- Spring 2025 (2nd offering): <https://cs336.stanford.edu/spring2025/> ·
  lectures repo <https://github.com/stanford-cs336/spring2025-lectures>
- Spring 2024 (1st offering): <https://cs336.stanford.edu/spring2024/> ·
  lectures repo <https://github.com/stanford-cs336/spring2024-lectures>

## Full crawl inventory

What follows is the mechanical output of the site crawl: 41 documents, all hosted
off-site on github.com, all recorded as links rather than copied. It includes the
2024 and 2025 archives listed above.

| Document | URL | Seen |
| -------- | --- | ---- |
| `cs336_assignment1_basics.pdf` | <https://github.com/stanford-cs336/assignment1-basics/blob/main/cs336_assignment1_basics.pdf> | 2026-08-27 |
| `cs336_spring2025_assignment1_basics.pdf` | <https://github.com/stanford-cs336/assignment1-basics/blob/main/cs336_spring2025_assignment1_basics.pdf> | 2026-08-27 |
| `cs336_assignment2_systems.pdf` | <https://github.com/stanford-cs336/assignment2-systems/blob/main/cs336_assignment2_systems.pdf> | 2026-08-27 |
| `cs336_spring2025_assignment2_systems.pdf` | <https://github.com/stanford-cs336/assignment2-systems/blob/main/cs336_spring2025_assignment2_systems.pdf> | 2026-08-27 |
| `cs336_assignment3_scaling.pdf` | <https://github.com/stanford-cs336/assignment3-scaling/blob/main/cs336_assignment3_scaling.pdf> | 2026-08-27 |
| `cs336_spring2025_assignment3_scaling.pdf` | <https://github.com/stanford-cs336/assignment3-scaling/blob/main/cs336_spring2025_assignment3_scaling.pdf> | 2026-08-27 |
| `cs336_assignment4_data.pdf` | <https://github.com/stanford-cs336/assignment4-data/blob/main/cs336_assignment4_data.pdf> | 2026-08-27 |
| `cs336_spring2025_assignment4_data.pdf` | <https://github.com/stanford-cs336/assignment4-data/blob/main/cs336_spring2025_assignment4_data.pdf> | 2026-08-27 |
| `cs336_spring2025_assignment5_supplement_safety_rlhf.pdf` | <https://github.com/stanford-cs336/assignment5-alignment/blob/main/cs336_spring2025_assignment5_supplement_safety_rlhf.pdf> | 2026-08-27 |
| `cs336_spring2026_assignment5_alignment.pdf` | <https://github.com/stanford-cs336/assignment5-alignment/blob/main/cs336_spring2026_assignment5_alignment.pdf> | 2026-08-27 |
| `cs336_spring2025_assignment5_alignment.pdf` | <https://github.com/stanford-cs336/assignment5-alignment/blob/master/cs336_spring2025_assignment5_alignment.pdf> | 2026-08-27 |
| `lecture_03.pdf` | <https://github.com/stanford-cs336/lectures/blob/main/lecture_03.pdf> | 2026-08-27 |
| `lecture_04.pdf` | <https://github.com/stanford-cs336/lectures/blob/main/lecture_04.pdf> | 2026-08-27 |
| `lecture_05.pdf` | <https://github.com/stanford-cs336/lectures/blob/main/lecture_05.pdf> | 2026-08-27 |
| `lecture_08.pdf` | <https://github.com/stanford-cs336/lectures/blob/main/lecture_08.pdf> | 2026-08-27 |
| `lecture_09.pdf` | <https://github.com/stanford-cs336/lectures/blob/main/lecture_09.pdf> | 2026-08-27 |
| `lecture_11.pdf` | <https://github.com/stanford-cs336/lectures/blob/main/lecture_11.pdf> | 2026-08-27 |
| `lecture_15.pdf` | <https://github.com/stanford-cs336/lectures/blob/main/lecture_15.pdf> | 2026-08-27 |
| `lecture_16.pdf` | <https://github.com/stanford-cs336/lectures/blob/main/lecture_16.pdf> | 2026-08-27 |
| `cs336_spring2024_assignment1_basics.pdf` | <https://github.com/stanford-cs336/spring2024-assignment1-basics/blob/master/cs336_spring2024_assignment1_basics.pdf> | 2026-08-27 |
| `cs336_spring2024_assignment2_systems.pdf` | <https://github.com/stanford-cs336/spring2024-assignment2-systems/blob/master/cs336_spring2024_assignment2_systems.pdf> | 2026-08-27 |
| `cs336_spring2024_assignment3_scaling.pdf` | <https://github.com/stanford-cs336/spring2024-assignment3-scaling/blob/master/cs336_spring2024_assignment3_scaling.pdf> | 2026-08-27 |
| `cs336_spring2024_assignment4_data.pdf` | <https://github.com/stanford-cs336/spring2024-assignment4-data/blob/master/cs336_spring2024_assignment4_data.pdf> | 2026-08-27 |
| `cs336_spring2024_assignment5_alignment.pdf` | <https://github.com/stanford-cs336/spring2024-assignment5-alignment/blob/master/cs336_spring2024_assignment5_alignment.pdf> | 2026-08-27 |
| `Lecture%2010%20-%20Scaling%20details.pdf` | <https://github.com/stanford-cs336/spring2024-lectures/blob/main/nonexecutable/Lecture%2010%20-%20Scaling%20details.pdf> | 2026-08-27 |
| `Lecture%2015%20-%20Alignment%20by%20SFT.pdf` | <https://github.com/stanford-cs336/spring2024-lectures/blob/main/nonexecutable/Lecture%2015%20-%20Alignment%20by%20SFT.pdf> | 2026-08-27 |
| `Lecture%2016%20-%20Alignment%20by%20RLHF.pdf` | <https://github.com/stanford-cs336/spring2024-lectures/blob/main/nonexecutable/Lecture%2016%20-%20Alignment%20by%20RLHF.pdf> | 2026-08-27 |
| `Lecture%2017%20-%20Evals.pdf` | <https://github.com/stanford-cs336/spring2024-lectures/blob/main/nonexecutable/Lecture%2017%20-%20Evals.pdf> | 2026-08-27 |
| `Lecture%203%20-%20architecture.pdf` | <https://github.com/stanford-cs336/spring2024-lectures/blob/main/nonexecutable/Lecture%203%20-%20architecture.pdf> | 2026-08-27 |
| `Lecture%204%20-%20details%20%2B%20MoEs.pdf` | <https://github.com/stanford-cs336/spring2024-lectures/blob/main/nonexecutable/Lecture%204%20-%20details%20%2B%20MoEs.pdf> | 2026-08-27 |
| `Lecture%205%20-%20GPUs.pdf` | <https://github.com/stanford-cs336/spring2024-lectures/blob/main/nonexecutable/Lecture%205%20-%20GPUs.pdf> | 2026-08-27 |
| `Lecture%207%20-%20Parallelism%20basics.pdf` | <https://github.com/stanford-cs336/spring2024-lectures/blob/main/nonexecutable/Lecture%207%20-%20Parallelism%20basics.pdf> | 2026-08-27 |
| `Lecture%209%20-%20Scaling%20laws%20basics.pdf` | <https://github.com/stanford-cs336/spring2024-lectures/blob/main/nonexecutable/Lecture%209%20-%20Scaling%20laws%20basics.pdf> | 2026-08-27 |
| `2025%20Lecture%2011%20-%20Scaling%20details.pdf` | <https://github.com/stanford-cs336/spring2025-lectures/blob/00191bba00d6d64621dc46ccaed9122681413a24/nonexecutable/2025%20Lecture%2011%20-%20Scaling%20details.pdf> | 2026-08-27 |
| `2025%20Lecture%207%20-%20Parallelism%20basics.pdf` | <https://github.com/stanford-cs336/spring2025-lectures/blob/4eff81bee0a853217209e163936b264f03572b66/nonexecutable/2025%20Lecture%207%20-%20Parallelism%20basics.pdf> | 2026-08-27 |
| `2025%20Lecture%2015%20-%20RLHF%20Alignment.pdf` | <https://github.com/stanford-cs336/spring2025-lectures/blob/61eddac004df975466cff0329b615f2d24230069/nonexecutable/2025%20Lecture%2015%20-%20RLHF%20Alignment.pdf> | 2026-08-27 |
| `2025%20Lecture%204%20-%20MoEs.pdf` | <https://github.com/stanford-cs336/spring2025-lectures/blob/98455ec198c9a88ec1ab2b1c4058662431b54ce3/nonexecutable/2025%20Lecture%204%20-%20MoEs.pdf> | 2026-08-27 |
| `2025%20Lecture%2016%20-%20RLVR.pdf` | <https://github.com/stanford-cs336/spring2025-lectures/blob/e94e33f433985e57036b25215dff2a4292e67a4f/nonexecutable/2025%20Lecture%2016%20-%20RLVR.pdf> | 2026-08-27 |
| `2025%20Lecture%203%20-%20architecture.pdf` | <https://github.com/stanford-cs336/spring2025-lectures/blob/e9cb2488fdb53ea37f0e38924ec3a1701925cef3/nonexecutable/2025%20Lecture%203%20-%20architecture.pdf> | 2026-08-27 |
| `2025%20Lecture%209%20-%20Scaling%20laws%20basics.pdf` | <https://github.com/stanford-cs336/spring2025-lectures/blob/fb79eb018fa047bf99c4c785dcbbd62fff361e54/nonexecutable/2025%20Lecture%209%20-%20Scaling%20laws%20basics.pdf> | 2026-08-27 |
| `2025%20Lecture%205%20-%20GPUs.pdf` | <https://github.com/stanford-cs336/spring2025-lectures/blob/main/nonexecutable/2025%20Lecture%205%20-%20GPUs.pdf> | 2026-08-27 |

