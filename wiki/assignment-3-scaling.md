# Assignment 3 — Scaling Laws

CS336's third assignment puts you in charge of one large training run that you cannot
afford to get wrong, and asks you to decide its shape from small experiments. It has two
problems. The first is a warm-up on synthetic data: reproduce Chinchilla's IsoFLOPs method
on a file of 72 precomputed runs and extrapolate the compute-optimal model and dataset
sizes. The second carries 50 of the assignment's 55 points. You spend up to 12 B200-hours
of real training runs, submitted through a hosted training API, to fit your own scaling
law. Then you use it to choose the model and hyperparameters for a 48 B200-hour run and
predict the validation loss that run will reach. Your chosen configuration is then
trained for 48 B200-hours, and part of the grade is how well it does (handout §1, §2.1,
§3.3; the repository's example notebook).
Hashimoto's framing at the start of [Lecture 9](09-scaling-laws.md) ([0:51]): "You have
your very wealthy friend who has given you 10,000 B200s for a month".

> **What this page is built from.** The Spring 2026 handout, *CS336 Assignment 3
> (scaling): Scaling Laws*, **version 26.0.5** (PDF dated 2026-05-08), plus the starter
> repository at commit `03e9372` (2026-05-08): its `README.md`, `CHANGELOG.md`,
> `AGENTS.md`, `make_submission.sh`, `data/isoflops_curves.json`,
> `examples/client_example.ipynb`, the API client and server code in `cs336_scaling/`,
> and the reference training code in `cs336_scaling/training/`. Nothing here comes from
> a recording, except where a lecture is cited by timestamp. Handout claims cite the
> handout's own **section numbers** (§3.1) and **problem names**
> (`chinchilla_isoflops`). **The handout's full text is transcribed in this KB** at
> [`raw/handouts/assignment3-scaling.md`](../raw/handouts/assignment3-scaling.md), with
> page markers, so any section or problem named here can be read in the handout's own
> words.
>
> **This is the 2026 assignment, and it is not last year's.** Until 2026-04-29 the
> assignment answered queries from a cache of runs the staff had trained offline, under a
> FLOPs budget. Version 26.0.0 replaced that with an API that trains your configuration on
> a B200 and charges wall-clock seconds. [Lecture 1](01-overview-tokenization.md)
> describes the old design; see [where the sources disagree](#where-the-sources-disagree).
> If your copy of the handout prints a different version number, trust your copy.
>
> **This page describes the assignment. It does not solve it.** No problem's answer, no
> fitted exponent, no recommended configuration and no strategy for spending the budget
> appears here or anywhere else in this knowledge base. The handout's and the
> repository's rules for AI tools are [below](#rules-for-ai-assistants), and they apply
> to any AI assistant reading this page.

Handout: [`cs336_assignment3_scaling.pdf`](https://github.com/stanford-cs336/assignment3-scaling/blob/main/cs336_assignment3_scaling.pdf)
(the version this page was built from is pinned at
[commit `03e9372`](https://github.com/stanford-cs336/assignment3-scaling/blob/03e9372/cs336_assignment3_scaling.pdf)).
Full text in this KB: [`raw/handouts/assignment3-scaling.md`](../raw/handouts/assignment3-scaling.md).
Code: [stanford-cs336/assignment3-scaling](https://github.com/stanford-cs336/assignment3-scaling).
Changes: [`CHANGELOG.md`](https://github.com/stanford-cs336/assignment3-scaling/blob/main/CHANGELOG.md).

## The assignment at a glance

**What you produce** (§1). A methodology for fitting a scaling law, a prediction made with
it, and a write-up "detailed enough to reproduce your results". Unlike Assignments 1 and
2, there is nothing to implement against a test suite and no stubs to fill in: the code
you hand in is whatever you wrote to fit your laws and compute your estimates. The
repository's `tests/` exercise the API server, not your code.

**What you start from.** The repository is the training service itself, server and all.
For a student, three things in it matter:

- `data/isoflops_curves.json`, the synthetic runs for `chinchilla_isoflops`.
- `cs336_scaling/client.py`, a thin Python client for the API, with one function per
  endpoint (`get_budget`, `submit_experiment`, `list_experiments`, `get_experiment`,
  `save_final_submission`, `get_final_submission`). It reads your key from the
  `A3_API_KEY` environment variable, which the `README.md` says to set to your 8-digit
  student ID. The handout never mentions this module; its own examples call the
  endpoints with `requests`.
- `examples/client_example.ipynb`, which walks through checking the budget, submitting
  one run, polling it, watching the unused reservation come back, and making a final
  submission.

The `README.md` also links a web
[dashboard](http://hyperturing.stanford.edu:8000/dashboard) beside the API's
[documentation page](http://hyperturing.stanford.edu:8000/docs).

**Who can use the API** (§3.2). You authenticate with your Stanford ID, and "you must be on
the Stanford network to query this API, so you may have to use a VPN." Anyone following
the course from outside Stanford cannot reach it. The `README.md` has a "For non-students"
section for running the service yourself: a Modal script downloads the tokenized data,
and `cs336_scaling/training/run.py` runs training directly.

**The two budgets** (§1, §3.1, §3.3). Both are measured in B200 wall-clock hours, and every
run is on a single B200.

- **The scaling-law budget is 12 B200-hours**, which the handout's examples print as
  `total_budget_seconds` of 43200.0. It is "a hard cap that will be enforced by the
  API".
- **The target run is 48 B200-hours.** The handout calls the 12 hours "25% of our big
  run FLOPs budget".

**How you submit** (§1, §3.2). Two files to Gradescope: `writeup.pdf`, which describes
your methodology for fitting a scaling law and using it to predict the optimal model size;
and `code.zip`, with the code that fits your scaling law and computes your estimates.
Separately, you send your predicted optimal configuration and predicted final validation
loss to the API's `POST /final_submission` endpoint. On the zip's actual filename, see
[where the sources disagree](#where-the-sources-disagree).

### Problems and points

| § | Problem | Points | What it asks for |
| --- | --- | --- | --- |
| 2.1 | `chinchilla_isoflops` | 5 | Reproduce the IsoFLOPs method on the synthetic runs in `data/isoflops_curves.json`. (a) Plot your compute-optimal model-size law with the points it was fitted to, extrapolated to at least $10^{24}$ FLOPs, and state the predicted model size at $10^{23}$ and $10^{24}$ FLOPs. (b) The same for dataset size. |
| 3.3 | `scaling_laws` | 50 | Fit a scaling law from at most 12 B200-hours of API runs; use it to choose the model size and hyperparameters for a 48 B200-hour run, and predict that run's validation loss. A reproducible write-up, plus the final submission to the API. |

The handout prints no total. The points sum to **55 (computed)**. It does not say how
`scaling_laws`' 50 points divide between the write-up and the performance of the
submitted model, only that "Part of your grade on the assignment will be determined by
the performance of your predicted optimal model" (§1, §3.3).

## Part 1 — IsoFLOPs on synthetic data (§2)

§2 reviews one of Chinchilla's methods (J. Hoffmann et al. 2022): the second of the three
that [Lecture 9](09-scaling-laws.md#chinchillas-three-methods) walks through, and
Hashimoto's own favourite ([1:04:31]). The question it answers: given a compute budget
$C$, which split between model size and data gives the lowest loss? The handout's
notation follows. $N$ is the number of parameters, $D$ the number of training tokens, and
the compute to train is approximately

$$C = 6ND$$

([training FLOPs](training-flops.md) derives the factor of 6). For each budget $C_i$ you
train models of several sizes $N_{ij}$, each on $D = \frac{C}{6N}$ tokens so that every
run spends the same compute, and record each one's final loss $L_{ij}$. Plotted against
model size, the losses at one budget form what the handout calls an "IsoFLOPs profile".
Chinchilla observed a quadratic relationship there. The handout gives the intuition for
both ends. Too small a model "can’t fit the data regardless of how much compute we
spend". Too large a model cannot "take enough gradient steps within $C$ FLOPs to train
it effectively".

Each profile's minimum gives one point $\langle C_i, N_{\mathrm{opt}(C_i)} \rangle$, and
an analogous one for $D_{\mathrm{opt}}$. Through those points you fit power laws
$N_{\mathrm{opt}} \propto C^a$ and $D_{\mathrm{opt}} \propto C^b$, and extrapolate to a
budget larger than any you ran. (The handout writes the whole of `opt(C_i)` as a
subscript, as printed here. It means the optimal model size at budget $C_i$.)

**The data.** `data/isoflops_curves.json` is a JSON array of 72 runs, each with three
keys: `parameters`, `compute_budget` and `final_loss`. There are eight runs at each of
nine compute budgets, from $6 \times 10^{18}$ to $3 \times 10^{21}$ FLOPs. The handout
calls the data synthetic. It has not changed since the repository's version 1.0.0
(2025-04-29). Its budgets span the range Percy names when he introduces the method in
[Lecture 1](01-overview-tokenization.md): "6e18 all the way to 3e21" ([49:18]).

**What the handout recommends.** Use any curve-fitting method you like; it suggests
`scipy.optimize.curve_fit`. It also departs from the paper in one place. "While
J. Hoffmann et al. [2] fits a quadratic function to each IsoFLOP profile to find its
minimum, we instead recommend you simply take the run with the lowest training loss for
each compute budget as the minimum."

**Deliverables.** For each of (a) and (b), a plot of the fitted law showing the points
used to fit it, extrapolated to at least $10^{24}$ FLOPs, and a one-sentence answer for
budgets of $10^{23}$ and $10^{24}$ FLOPs.

## Part 2 — the training API (§3.1–3.2)

### What one run is

The API "runs real training jobs on B200 GPUs" (§3.1). Every run uses the same setup:

- **Data.** Tokenized DCLM with a 32K vocabulary, up to 500B tokens, never repeated ("There
  is no data epoching"). The data order is fixed: every configuration consumes "the same
  deterministic training-token order".
- **Sequences and loss.** Context length 512. The loss is "mean next-token cross-entropy
  loss over 512-token sequences", and there is no dropout.
- **Evaluation.** Validation uses $2^{18}$ tokens. A run is split into `n_evals` chunks
  of `total_train_tokens / n_evals` tokens each. After each chunk the API evaluates, and
  it appends the loss to `status.val_losses`. A completed run's final validation loss is
  the last entry. "The final leaderboard validation loss will use more tokens."
- **Seeds.** `model_seed` "controls only model initialization, not the data order."

**The model.** The handout says only that it "largely matches that of assignment 1, with
a few slight differences", and that the code is in the repository "for your reference"
(§3.1). From that code (`cs336_scaling/training/model/basic_model.py`), the model is
built from pieces [Assignment 1](assignment-1-basics.md) also uses: pre-norm blocks with
[RMSNorm](rmsnorm.md), a [SwiGLU](gated-activations.md) feed-forward with three bias-free
projections, and [RoPE](rope.md). Five differences from Assignment 1:

- It is written in JAX with Equinox, not PyTorch.
- It applies RMSNorm to each head's queries and keys before RoPE: the
  [QK norm](training-stability.md) from [Lecture 3](03-architectures.md#stability).
- Its attention projections can carry biases (`attention_bias`), which Assignment 1's
  model omits.
- Its input and output embeddings can be tied (`tie_word_embeddings`).
- The head dimension is a parameter of its own (`head_dim`).

The configuration's field names (`hidden_size`, `intermediate_size`,
`num_hidden_layers`, `num_key_value_heads`) are not Assignment 1's (`d_model`, `d_ff`,
`num_layers`). "The training code is only included as a reference. You should only train
models through the API."

**The optimizer** (§3.1, `cs336_scaling/training/optimizer.py`). AdamW or SGD, both under
a warmup-cosine learning-rate schedule. The schedule warms up from `init_value` to
`peak_value` over the first `warmup_frac` of the optimizer steps, then follows a cosine
down to `final_lr_frac` times the peak at the last step. The AdamW config carries
`weight_decay`, `beta1`, `beta2`, `eps`, `eps_root` and `grad_clip_norm`. The SGD config
carries only the schedule and `grad_clip_norm`, so it has no momentum or weight-decay
setting. The handout's example values are weight decay 0.01, clipping 1.0, warmup
fraction 0.05 and final fraction 0.1. No other schedule is offered.

### How the budget is charged

The handout spells out how runs are charged (§3.1). When you submit a run, it reserves its full `max_runtime_seconds` against
your 12 hours while it waits in the shared queue and while it runs. When it completes or
fails, the reservation is replaced by the runtime the run actually reported. That figure
is clipped to at least 1 second and at most `max_runtime_seconds`, and the difference
comes back. The handout's example: "if you reserve 30 minutes for a run that completes in
12 minutes, the unused 18 minutes are returned to your remaining budget." A run that
times out is charged its full `max_runtime_seconds`. It also does not count as completed,
even though the API may return the validation losses it reached before the timeout.

The handout adds three rules, and the code fills in one detail:

- A submission whose `max_runtime_seconds` exceeds your remaining budget is refused with
  a 400.
- Submitting a configuration you have already submitted returns a 409 and reserves
  nothing. The handout's "the same training configuration" means the whole of it: the duplicate check hashes
  every field, including `model_seed` and `max_runtime_seconds` (`unique_id` in
  `cs336_scaling/training/training_config.py`).
- A run can show as `"running"` for longer than its maximum runtime, because that state
  "includes startup time and preemption" (§3.2).

The queue is shared by the class. When workers free up, the dispatcher takes queued runs
from whoever has the fewest runs already going, and then by submission time
(`cs336_scaling/scheduler/experiment_selector.py`). It runs at most 100 at once under its
default settings.

### What the API accepts

A configuration has eight top-level keys (§3.2): `architecture_config`, `optimizer_config`,
`train_batch_size`, `val_batch_size`, `n_evals`, `total_train_tokens`,
`max_runtime_seconds` and `model_seed`. The architecture block has twelve:
`attention_bias`, `head_dim`, `hidden_size`, `intermediate_size`, `num_attention_heads`,
`num_hidden_layers`, `num_key_value_heads`, `rms_norm_eps`, `rope_theta`,
`tie_word_embeddings`, `dtype` (`"float32"` or `"bfloat16"`) and `vocab_size`. The API
fixes the sequence length at 512 and the validation set at $2^{18}$ tokens.

The handout names three of the consistency checks: `hidden_size` must equal
`num_attention_heads * head_dim`, the heads must divide evenly among the key-value heads,
and `total_train_tokens` must be divisible by `512 * train_batch_size`. The validator in
`training_config.py` checks more than the handout lists, and a rejected configuration
costs a round trip rather than budget:

- `num_key_value_heads` must **equal** `num_attention_heads`, with the message
  "GQA is not supported". This contradicts the handout; see below.
- The number of optimizer steps must be divisible by `n_evals`.
- $2^{18}$ must be divisible by `512 * val_batch_size`.
- `head_dim` must be even ("for RoPE").
- `intermediate_size` must be at least `hidden_size`.
- `total_train_tokens` is at most 500 billion.
- The sizes, `rms_norm_eps`, `rope_theta`, the peak learning rate, `eps` and
  `grad_clip_norm` must be positive. `warmup_frac` must lie in $[0, 1)$, `final_lr_frac`
  in $[0, 1]$, and AdamW's betas in $[0, 1)$.

A final submission is validated as a full configuration in the same way, and its
`predicted_final_loss` must be positive (`cs336_scaling/schemas/__init__.py`).

### The endpoints

Every request carries your key in an `X-API-Key` header (§3.2).

| Endpoint | What it does |
| --- | --- |
| `POST /submit` | Queue a run. Returns its `experiment_id` and a `budget_summary`. |
| `GET /budget` | `used_seconds`, `remaining_seconds` and `total_budget_seconds`. |
| `GET /experiments` | Every run you have submitted, with its configuration and status. |
| `GET /experiment/{experiment_id}` | One run. Poll it until `status.status_type` is `"completed"`. |
| `POST /final_submission` | Your predicted optimal `training_config` and `predicted_final_loss`. Resubmitting replaces the earlier one. |
| `GET /final_submission` | Your current final submission, or `None`. |

A run's status is `"queued"`, `"running"` (with the losses so far), `"completed"` (with
`used_runtime_seconds`, `val_losses` and `completed_at`) or `"failed"`. A failed run
"either crashed or timed out", and a timeout includes `partial_val_losses`.

## Part 3 — the leaderboard problem (§3.3)

`scaling_laws` asks you to "Construct a scaling law and use it to choose the model size and
hyperparameters that you expect to attain the lowest validation loss on 48 B200-hours",
and to predict the validation loss that model will reach. The handout places "essentially
no constraints" on the configuration you report for the 48-hour run. §3 adds one piece of
advice on method: "To set the hyperparameters for the predicted optimal model size, we
recommend analyzing how hyperparameters affect the validation loss for smaller-scale
settings." And it warns: "Be sure to carefully plan your runs before you get started".

The write-up has to describe your approach and methodology for fitting the law, how you
used it to predict the optimal model size, and your predicted values. It must include
"commentary about why you made particular design decisions", in enough detail to
reproduce the work. The handout lists five questions it should answer at least:

- Given the 12 B200-hour budget, how did you decide which runs to query?
- How did you fit your scaling law, by what concrete method or methods? The handout
  points to the approaches in J. Kaplan et al. and J. Hoffmann et al.
- How well does the law fit the experimental data?
- For the 48 B200-hour budget, what optimal model size does it predict, and what loss?
- What hyperparameters would you use for a model of that size?

On the last question, the handout supplies the parameter count to use: "To estimate the
number of non-embedding parameters for a given model hyperparameter configuration, use
$12 n_{\mathrm{layer}} d_{\mathrm{model}}^2$." Here $n_{\mathrm{layer}}$ is the number of
layers and $d_{\mathrm{model}}$ the model width, which the API calls `num_hidden_layers`
and `hidden_size`. Why the choice between counting all parameters and only non-embedding
ones matters is a thread of [Lecture 9](09-scaling-laws.md#why-kaplan-and-chinchilla-disagree)
([37:40]): it is one of the reasons Kaplan's and Chinchilla's exponents differ.

Finally, submit your configuration and predicted loss through `POST /final_submission`
(§1, §3.2). For this part §2 also invites ideas from "other references", naming J. Kaplan
et al. and G. Yang et al., and states the problem in one line: "The main challenge is how
to extrapolate from experiments done at a smaller scale to larger scale."

## Where the lectures teach each part

| Handout section | Lecture material | Wiki pages |
| --- | --- | --- |
| §1 the setting | [Lecture 9](09-scaling-laws.md), [0:51]–[1:37]; [Lecture 1](01-overview-tokenization.md), [45:30]–[48:32], on scaling recipes | [Scaling laws](scaling-laws.md) |
| §2.1 $C = 6ND$ | [Lecture 2](02-pytorch-resource-accounting.md) | [Training FLOPs](training-flops.md), [resource accounting](resource-accounting.md) |
| §2.1 IsoFLOPs, `chinchilla_isoflops` | [Lecture 9](09-scaling-laws.md#chinchillas-three-methods), [1:02:12]–[1:05:16]; IsoFLOPs as "always a good default" ([1:16:03]) | [The IsoFLOP method](isoflop-method.md), [compute-optimal scaling](compute-optimal-scaling.md) |
| §3.3 Kaplan's and Hoffmann's approaches; non-embedding parameters | [Lecture 9](09-scaling-laws.md#why-kaplan-and-chinchilla-disagree), [37:40] and Part 4 | [Compute-optimal scaling](compute-optimal-scaling.md) |
| §3 hyperparameters at smaller scale | [Lecture 9](09-scaling-laws.md#the-two-things-you-always-have-to-re-tune), batch size and learning rate ([41:27]–[47:37]); [Lecture 11](11-scaling-laws-in-the-wild.md#part-2--deepseek-fit-a-scaling-law-to-the-optimum-instead), DeepSeek's fits, and [Part 4](11-scaling-laws-in-the-wild.md#part-4--step-law-grid-search-the-hyperparameter-plane), Step Law | [Critical batch size](critical-batch-size.md), [learning-rate scaling and muP](learning-rate-scaling-and-mup.md), [Step Law](step-law.md), [published scaling recipes](published-scaling-recipes.md) |
| §2 G. Yang et al. [3], *Tuning Large Neural Networks via Zero-Shot Hyperparameter Transfer* | [Lecture 1](01-overview-tokenization.md), hyperparameter transfer ([47:47]–[48:32]); [Lecture 11](11-scaling-laws-in-the-wild.md#part-6--mup-derived), muP derived | [Maximal update parametrization](maximal-update-parametrization.md) |
| §3.1 model shape: depth, width, feed-forward size | [Lecture 9](09-scaling-laws.md#part-3--scaling-laws-for-model-engineering), aspect ratio ([36:07]); [Lecture 3](03-architectures.md#hyperparameters) | [Transformer hyperparameters](transformer-hyperparameters.md) |
| §3.1 the learning-rate schedule | [Lecture 11](11-scaling-laws-in-the-wild.md), cosine and WSD schedules ([10:00]–[12:17]) | [WSD schedules](wsd-schedules.md) |
| §3.1 the architecture | [Lecture 3](03-architectures.md); [Assignment 1](assignment-1-basics.md) | [RMSNorm](rmsnorm.md), [gated activations](gated-activations.md), [RoPE](rope.md), [training stability](training-stability.md) |
| §3.3 judging a fit | [Lecture 9](09-scaling-laws.md); [Lecture 11](11-scaling-laws-in-the-wild.md) | [Scaling law methodology](scaling-law-methodology.md) |

Hashimoto ties one plot in Lecture 9 to this assignment directly. Of the data scaling plot
he shows from Kaplan, he says "you'll make something that looks roughly like this in parts
of assignment three" ([14:35]; see [data scaling laws](data-scaling-laws.md)).

## Rules for AI assistants

The handout has a **Statement on AI tools** (§1, page 2), added in version 26.0.1. AI tools
are permitted "for answering high-level conceptual questions, or for providing low-level
programming documentation like function signatures and library APIs", and "are not
permitted for implementing any part of any assignment". That covers coding agents and AI
autocomplete alike. An agent must use the repository's `AGENTS.md`, and the handout links
the course's full [AI Policy](https://docs.google.com/document/d/1SZAlExB1qAc9izHt54gwunNpjKE6wXb8Y7yA_e-baK8/edit?tab=t.0).

[`AGENTS.md`](https://github.com/stanford-cs336/assignment3-scaling/blob/main/AGENTS.md),
shipped identically as `CLAUDE.md`, is Assignment 2's file with two lines added, both
aimed at this kind of assignment: an assistant may not "Write any part of the writeup" or
"Solve any of the math problems". It casts an AI assistant as "Teaching Assistant, Not
Solution Generator". It may explain concepts, point to
lecture material, review code the student wrote with general feedback, and suggest sanity
checks. It may not write any Python or pseudocode, give solutions, edit student code or
run commands. Among the
components it forbids an assistant to implement, it names this assignment's:
"scaling-law pipelines".

**This applies to Cairn's chat too, which is an AI tool.** For this assignment that rules
out more than code. Choosing which runs to spend the budget on, how to fit the law and
what to submit *is* the assignment, so this page, and the chat, explain the concepts
those choices rest on and say where each is taught. They do not make the choices.

## Where the sources disagree

The material around this assignment contradicts itself in several places. Each is recorded
here as printed rather than reconciled.

- **Grouped-query attention.** §3.2 says "`num_attention_heads` must be divisible by
  `num_key_value_heads`", which implies grouped-query attention is allowed. The validator
  in `cs336_scaling/training/training_config.py` rejects any configuration where the two
  differ: "num_key_value_heads must equal num_attention_heads; GQA is not supported".
- **The 12-hour limit on one run.** §3.2 says `max_runtime_seconds` "must be between 1
  second and 12 hours." Version 26.0.2's changelog entry removed that limit from the code
  "to allow 48 hour final submission", and the field now has only a lower bound of 1
  second. The handout's wording did not change. Comparing the text of all six versions of
  the PDF, only 26.0.1 changed any words; every later version changed only the printed
  version number. A run still cannot reserve more than your remaining budget.
- **Wall-clock or FLOPs?** §1 says the budget is "measured in wall-clock time", and the API
  charges seconds. The same handout calls the 48 B200-hour budget a "FLOPs budget" four
  times (twice in §1, twice in §3.3), and `chinchilla_isoflops` works in FLOPs throughout.
- **Training loss or validation loss?** §1 sets the goal as "a model with the lowest training
  loss (i.e., a *compute-optimal* model)". §3 and §3.3 set it as the lowest validation loss,
  which is what the API reports and the leaderboard scores.
- **What the final submission contains.** §1 and the endpoint ask for your "predicted final
  validation loss". The `scaling_laws` box asks you to submit "(2) the model’s validation
  loss to the api". The endpoint's field is `predicted_final_loss`.
- **What the submission is called.** §1 asks for `code.zip`. The repository's
  `make_submission.sh` writes `cs336-assignment-3-submission.zip`, under a comment calling
  it a "tar.gz". It excludes every `.json`, `.txt`, `.log`, `.out`, `.err`, `.pkl`, `.pt`
  and `.bin` file, among others. The handout does not mention the script.
- **Initialization.** Assignment 1's handout (§3.3) gives approximate initializations
  because "we will save the details for assignment 3". The 2026 Assignment 3 handout does
  not discuss initialization, and the API's configuration has no field for it; the only
  control is `model_seed`.
- **Notation.** §2.1 indexes model sizes as $N_{ij}$, then two sentences later as $N_i$. It
  writes the optimal model size as $N_{\mathrm{opt}(C_i)}$, with $(C_i)$ inside the
  subscript.
- **Lecture 1 describes the old assignment.** In the 2026 lecture Percy says "we train a bunch
  of models offline, and we provide this cache, so it looks like you're training" ([52:24]).
  Lecture 1's program says the API is "based on previous runs", that jobs run "under a FLOPs
  budget", and links the Spring 2025 PDF on a `master` branch. The repository has no
  `master` branch, and version 26.0.0 deleted that PDF; it survives at
  [commit `74ee390`](https://github.com/stanford-cs336/assignment3-scaling/blob/74ee390/cs336_spring2025_assignment3_scaling.pdf).
  The changelog dates the switch to real training to that same version, 2026-04-29:
  "change from offline to online model training api".
- **Small slips, transcribed as printed.** "it is your part of your job" (§1); "it will
  likely to be useful" (§3.3); no full stop after "(e.g., 06123456)" (§3.2); and "The
  training API has the following endpoints:" (§3.2), followed by a paragraph about headers
  rather than the list.

## Sources

- Handout: *CS336 Assignment 3 (scaling): Scaling Laws*, version 26.0.5, Spring 2026, 8
  pages —
  [current](https://github.com/stanford-cs336/assignment3-scaling/blob/main/cs336_assignment3_scaling.pdf),
  [as read, at commit `03e9372`](https://github.com/stanford-cs336/assignment3-scaling/blob/03e9372/cs336_assignment3_scaling.pdf);
  transcribed in full at [`raw/handouts/assignment3-scaling.md`](../raw/handouts/assignment3-scaling.md)
- Starter repository: [stanford-cs336/assignment3-scaling](https://github.com/stanford-cs336/assignment3-scaling)
  — `README.md`, `CHANGELOG.md`, `AGENTS.md`, `make_submission.sh`,
  `data/isoflops_curves.json`, `examples/client_example.ipynb`, `cs336_scaling/` (MIT
  licence)
- [Lecture 1](01-overview-tokenization.md) ([transcript](../raw/transcripts/01-overview-tokenization.md), [source](../raw/slides/01-overview-tokenization.md)),
  [Lecture 9](09-scaling-laws.md) ([transcript](../raw/transcripts/09-scaling-laws.md)),
  [Lecture 11](11-scaling-laws-in-the-wild.md) ([transcript](../raw/transcripts/11-scaling-laws-in-the-wild.md))
- [Assignment 1](assignment-1-basics.md) — the model this assignment's API trains, nearly
- [Course map](course-map.md#unit-3--scaling-laws) — where Assignment 3 sits among the five units
