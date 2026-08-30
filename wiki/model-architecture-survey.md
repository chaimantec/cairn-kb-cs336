# The model architecture survey

Lecture 3 is built on a database rather than an argument. Tatsunori Hashimoto
maintains a table of the architectural choices made by every major dense
autoregressive language model from the 2017 transformer to the 2026 releases, and
the lecture's method is to read the trends off it. Five different views of that
table appear in the deck — slides 7, 9, 29, 51 and 67 — and all five are
transcribed cell by cell in [the slide file](../raw/slides/03-architectures.md).

This page explains what the table shows and how to use it. **For the actual values,
go to the slide file** — reproducing forty-plus rows here would create a second copy
to drift out of date.

## Why a survey at all

The lecture's opening thesis ([0:05]–[0:51]) is that architecture is not something
you can currently derive:

> I think we all wished we lived in a world where the only things you had to know
> were like VC dimension or something — very simple, theoretical tools — but that's
> not really where we are.

So the method is empirical, and explicitly second-best. Slide 2 states the theme:
"the best way to learn is hands-on experience / the second best way is to try to
learn from others' experience." Hashimoto is candid that the first is better
([0:05]): "The best thing to do, better than listening to this lecture even, is for
you to go out and train your own models."

What the survey buys you is a distinction that no single paper can give: **which
choices are fixed across all effective architectures, and which vary freely**
([0:51]). A hyperparameter that every successful model sets the same way is one you
should probably not spend your compute searching over. One that varies widely
across equally good models is one you can set on systems grounds.

That the corpus exists at all is a recent accident. Slides 5 and 6 are collages of
paper title pages — 19 new dense models in 2024–2025 alone, and a second wall of
2025–2026 releases including gpt-oss, LLaMA 4, DeepSeek-V3.2, Kimi K2, GLM-4.7,
Nemotron 3, MiniMax-M2 and Step-3. Hashimoto's running joke ([3:08]) is that he
expected the flow to slow down and it did not, though he notes most of the new
arrivals are mixtures of experts, which are next lecture's subject rather than this
one's ([3:54]).

## Which view is on which slide

The five screenshots show overlapping column sets. If you are looking for a
specific field, this is where it is:

| Slide | Columns |
| --- | --- |
| 7 | Name, Year, Vocab count, Norm, Parallel Layer, Pre-norm, Position embed., Activation, Other tricks, MLP factor, num_layers, model_dim |
| 9 | The same table, enlarged — used to show the colour trends visually |
| 29 | Name, Year, Norm, Parallel Layer, Pre-norm, Position embedding, Activations — sorted by year, and reaching further back (it is the only view with the 2017–2019 rows) |
| 51 | Name, Year, MLP factor, Aspect ratio (d/layer), weight decay, drop_rate |
| 67 | The complete table, every column — the only view carrying **num_heads** |

Slide 67 also carries two sparse columns worth knowing about: **MoE**, checked on
exactly two rows (GPT-4 and Mixtral), and **Parametrization**, filled on exactly one
(Phi-3, tagged MuP).

> These five views were transcribed independently, in different reading passes.
> Cross-checking them against each other compared **296 overlapping (model, column)
> cells** with **zero disagreements**, which is the strongest evidence available that
> the numbers in the slide file are right.

## What the table shows

Hashimoto reads the trends off the colour blocks at [29:58], and they are the
lecture's conclusions in compressed form:

**Settled — everyone does the same thing.**

- **RMSNorm** rather than LayerNorm, from roughly 2022 onward. See
  [RMSNorm](rmsnorm.md).
- **Norms outside the residual stream.** Only OPT-350M is post-norm in the harmful
  sense. See [pre-norm and post-norm](pre-norm-and-post-norm.md).
- **Serial rather than parallel** blocks — the Parallel Layer column is almost
  solid "Serial", with GPT-J, GPT-NeoX, PaLM, Falcon 2, Command R+ and Command A
  the visible exceptions.
- **Gated activations**, overwhelmingly SwiGLU with a Google-lineage GeGLU
  minority. See [gated activations](gated-activations.md).
- **A feedforward multiplier** of 4, or ~2.67 for gated models, and an **aspect
  ratio** near 100. See [transformer hyperparameters](transformer-hyperparameters.md).

**Still moving — this is where the interesting work is.**

- **Position embeddings.** The column runs Sine → Absolute → Relative → solid RoPE
  from 2021, and then breaks up again into "Hybrid" chips in the 2025–2026 rows.
  See [RoPE](rope.md) and [attention variants](attention-variants.md).
- **Stability tricks.** The `Other tricks` column is empty for most older models and
  fills with QK-norm, Z-loss and logit soft-capping only in the 2024+ rows. See
  [training stability](training-stability.md).
- **Vocabulary size**, which splits by whether the model is monolingual or
  multilingual rather than by year.

Hashimoto's own historical periodization ([6:12]–[6:58]) matches the table: broad
experimentation until GPT-3; convergence on "LLaMA-2-alikes" after LLaMA 2; a
stability-driven phase last year; and a long-context-driven phase this year. His
closing summary on slide 67: "Many aspects (arch, hparams) of transformers are in
common across the big LMs. Major differences? Position embeddings, activations,
tokenization."

## How to use it

Three ways this table is useful to someone actually training a model:

1. **As a default.** If forty models agree on a value, take it and spend your
   compute elsewhere. This is what slide 51's summary bullets are for.
2. **As a sanity check on a hyperparameter you are tempted to vary.** T5's 64×
   feedforward multiplier worked — and T5 v1.1 quietly reverted to 2.5. A value far
   outside the table's range is not impossible, but it needs a reason.
3. **As a reading aid.** Model reports increasingly omit these details.
   Hashimoto's advice to a student who asked how to acquire this knowledge
   ([39:10]) is that "reading any single paper in isolation is very, very difficult,
   especially now, because no single paper seems to give the full detail for a lot
   of language models these days." The table is the aggregation that no individual
   paper provides.

## A caveat on the table's own limits

The database records what models *did*, not what is *best*. Several of its
consensus columns have little controlled evidence behind them — slide 51 admits
this directly for head dimension: "Head dim\*Num head = D model is standard – but
low to no validation." Hashimoto makes the same point about parallel layers at
[40:42], where the honest answer to a student's question about the accuracy cost is
that "no one's done the ablations, as far as I know."

Consensus in this table is evidence about practice. Where it coincides with a
controlled study — as it does for gating (slides 24, 25), RMSNorm (slide 17) and
the feedforward basin (slide 40) — it is evidence about quality too. Those are
different things, and the lecture is careful to distinguish them.

## Related

- [Lecture 3 — architectures](03-architectures.md) — the lecture that reads this
  table.
- [MoE routing](moe-routing.md) — the companion table from
  [Lecture 4](04-attention-alternatives.md)'s slide 35, which does the same job for
  the *sparse* models: routed and active expert counts, shared experts and
  fine-grained ratios for twelve MoEs.
- [Course map](course-map.md) — where Lecture 3 sits in CS336.
- [The full slide file](../raw/slides/03-architectures.md) — all five views,
  transcribed.
