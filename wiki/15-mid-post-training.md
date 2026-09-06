# Lecture 15 — Mid/Post-Training

**The lecture that gets you from GPT-3 to ChatGPT.** Everything in the course up
to here builds a base model: an architecture, a training loop, a data pipeline,
a scaling law. Lecture 15 argues that a base model, however good, is of limited
use — and that the process which turns it into something a person can actually
instruct has two parts, *supervised fine-tuning* on demonstrations and
*reinforcement learning from human feedback* on preferences. Lecture 16 then
takes ChatGPT to o1.

Two framings run through the whole lecture and are worth holding onto:

1. **Post-training is a data problem, not an algorithm problem.** "Algorithms are
   not really the secret sauce — it's not where a lot of the leverage is. It's
   going to be the data" (≈5:28). The SFT *method* section of this lecture is
   about ninety seconds long, because the method is `loss.backward()`.
2. **SFT and RLHF are conceptually different games.** SFT fits a distribution;
   RLHF maximizes a reward. That distinction is not pedantry — it is what makes
   [mode collapse](mode-collapse-and-calibration.md) a coherent outcome rather
   than a bug (≈43:10).

The lecturer is unusually candid about how little is publicly known here.
Frontier post-training information "is honestly pretty sparse," most of the
citable material predates the competitive scramble, and "basically none of the
vendors want to release any information about their post-training processes"
(≈3:10, ≈3:55).

![Slide 5 — Caveat: post-training information is pretty sparse!](../raw/images/15-mid-post-training/slide-5.png)
*Slide 5 — the leaked Scale AI documents the lecturer uses to make the point that post-training data is a trade secret.*

- **Material:** [`lecture_15.pdf`](https://github.com/stanford-cs336/lectures/blob/main/lecture_15.pdf),
  transcribed in full at [`raw/slides/15-mid-post-training.md`](../raw/slides/15-mid-post-training.md)
- **Transcript:** [edited transcript](../raw/transcripts/15-mid-post-training.md)
- **Video:** [Lecture 15](https://www.youtube.com/watch?v=2oH6PWPrYFo)

---

## Where this sits

The organizing picture is InstructGPT's three-step diagram, which the deck
returns to three times (slides 6, 31, 35). Step 1 is SFT; steps 2 and 3 are the
reward model and the RL against it. This lecture does step 1 first and steps
2–3 second, and [lecture 16](https://www.youtube.com/watch?v=dIFAi87Ws4E)
replaces the human preference signal with a verifiable one.

![Slide 6 — Where today's lecture fits in](../raw/images/15-mid-post-training/slide-6.jpg)
*Slide 6 — the InstructGPT pipeline with Step 1, supervised fine-tuning, boxed in red as this lecture's first half.*

Why bother, when pre-training is where the scale is? Because you cannot
post-train your way out of a weak base model — "if we ignore pre-training and
just try to post-train our way to victory, we will get none of the things that we
want. Absolutely none of it" — but the base model on its own is a "primordial
soup" whose useful behaviours have to be extracted deliberately (≈2:24).

## Part 1 — Supervised fine-tuning

### The method is boring; the data is not

SFT is next-token prediction on a different distribution. "We all know how to SFT
a model, that's basically exactly the same as pre-training, so the only real
difference, with some minor variation, is going to be the training data"
(≈6:14). Slide 28 makes the joke explicit with a screenshot of `loss.backward()`.

Everything interesting is therefore in
[what data you collect](supervised-fine-tuning.md) and
[which datasets exist](instruction-tuning-datasets.md).

### Six generations of open SFT data

Slide 9 lays out the progression chronologically, and the lecture walks it top-left
to bottom-right (≈7:01).

![Slide 9 — Progression of SFT data (in the open world)](../raw/images/15-mid-post-training/slide-9.jpg)
*Slide 9 — the chronological grid of open SFT datasets the lecture walks through.*

**FLAN** was "the oldest, and in some ways the most visionary": NLP already had
hundreds of supervised input/output datasets, so train on all of them (≈7:46).
It was also wrong in an instructive way. Its examples are reconstituted
benchmarks — an Enron email with "write a subject line for this email" *appended
after* the body — a shape nobody has ever actually prompted a model with. Its
summarization targets are short and, on inspection, "often hallucinated"
(≈11:36, ≈12:23). FLAN inherits the deficiencies of the datasets it was built
from, and it bet on scale at a moment when scale was the wrong axis.

**Self-Instruct** asked why the model could not write the data itself.
**Alpaca** and **Vicuna** distilled ChatGPT traces, and produced "more
natural-looking inputs, longer, chattier outputs" — but the lecturer is careful
that this only worked "when we did it on the original Llama models. So
pre-training and post-training both need to work out" (≈14:41).

**Open Assistant** was the maximal human effort: a volunteer, Wikipedia-style
project to write high-quality prompts and responses at scale. "A very admirable,
very impressive effort that generated a decent amount of data... before stalling
out as a project" (≈16:15).

**WizardLM and Tulu 3** returned to synthesis, with increasingly elaborate
LM-driven generation pipelines. And the newest generation has moved off chat
entirely: **Nemotron**'s SFT data is largely agentic, with parallel tool calls
supervised alongside the assistant text, because the product people want is no
longer a chat box but an agent that keeps a to-do list (≈17:00).

The three high-level shifts, in the lecture's own summary (≈17:47): toward
**chattiness** (people "don't really want to talk to an NLP benchmark"), toward
**higher-quality annotators and more detail**, and toward **tool use**.

### You need far less data than you think

The single most practically useful finding in the SFT half: "if you have a
sufficiently capable model, it does not take very many examples to steer these
systems" (≈32:23). Five hundred safety examples measurably move a model's
refusal behaviour across four separate harm benchmarks.

![Slide 26 — Safety-tuning with just a little data](../raw/images/15-mid-post-training/slide-26.png)
*Slide 26 — mean harmfulness score against added safety examples. The steep drop is between 0 and 300–500; after that the curves flatten.*

The mechanism the lecturer proposes is that "models already have a 'will I be a
safe model or an unsafe model' axis inside them after pre-training, so it does
not take very many examples to pull this out" (≈33:09). This is the SFT-as-
extraction view: if the behaviour is already latent in pre-training, you are
selecting a mode, not teaching a skill.

The important qualification, stated immediately: few examples suffice to *steer*,
but not to enforce fine-grained distinctions. "If you're OpenAI or Anthropic and
you want to enforce really fine-grained distinctions about what is safe and not
safe, you do need very large-scale data collection."

### Style is a decision, and it distorts your evaluations

Response length and style vary enormously across these datasets, and those
differences are "conscious decisions made by the data collection folks" — the
reason Claude and ChatGPT have different tones (≈19:18).

They also break preference evaluation. Raters "will very often select responses
that have bullet-pointed lists in their outputs, or responses that have more,
longer detail" (≈20:03), so training on some datasets moves AlpacaEval a great
deal while leaving standard benchmarks flat: "your models aren't necessarily
smarter because you've trained on certain kinds of post-training data... But you
can very much shift the engagement signals" (≈21:35).

![Slide 18 — What about benchmarks?](../raw/images/15-mid-post-training/slide-18.jpg)
*Slide 18 — the same instruction datasets scored on open-ended preference evaluation and on standard benchmarks; the two columns disagree.*

The conclusion is a design rule: **think about style control separately from
capability control**. See [style and length bias](style-and-length-bias.md), and
[chat benchmarks](chat-benchmarks.md) for the evaluation-side treatment from
lecture 12.

### Teaching a model facts it does not know teaches it to hallucinate

This is the subtlest argument in the lecture, and it starts from a single
Open Assistant example that contains an academic reference (slide 19).

![Slide 20 — Knowledge extraction and alignment](../raw/images/15-mid-post-training/slide-20.jpg)
*Slide 20 — Schulman's "Hallucination and Behavior Cloning" framing, the argument that behaviour cloning on unknown facts is what produces confident fabrication.*

SFT'ing on that example teaches two things at once: the *content* of the Bivens
and Mishel citation, and the *format* — that a good response cites a reference.
"These are two different mechanisms going on at once" (≈22:22), and the second
generalizes beyond the first. The model learns to emit a reference whether or not
it has one, which is "forcibly emit unknown knowledge" (≈23:55).

So the counterintuitive rule: **you might not want to train on the highest-quality
data if the model does not already know it.** Tail knowledge "can actually
sometimes be actively harmful" (≈24:41).

John Schulman's separate argument is that this is precisely why RL is needed: to
be calibrated about what it knows, a model has to be trained on its *own*
outputs. "You can't have an external person shoving knowledge down your throat if
you want the model to be calibrated about what it knows and doesn't know"
(≈23:55). Asked to expand, the lecturer offers a folk story worth reading in full
at ≈26:14: if the model has an internal "I know this" direction, RL can reward
citing in that direction and penalize citing outside it — but "if you don't know
what you know at all, at any level, RL cannot help you there."

Full treatment: [hallucination and knowledge extraction](hallucination-and-knowledge-extraction.md).

### Safety as an SFT problem

The framework is a two-sided trade-off: minimize the **violation rate** (bad
queries answered) without inflating the **false refusal rate** — the model that
won't tell you how to "kill a Python process" (≈29:18). Published detail is even
thinner here than for capabilities; Llama 2's description, "one of the more
detailed descriptions," does not say how many examples were used.

Tulu 3 is the exception and the lecture's recommended reference. Its safety
component is ~50k examples, mined from **WildChat** — free chat access exchanged
for the logs — filtered for attempted unsafe behaviour and jailbreaks, with
refusals and jailbreak-resistant responses written as the preferred outputs
(≈30:50). The lecturer believes the closed labs do the same thing: "look at usage
information, find unsafe behaviors, and get your annotators to play whack-a-mole."

See [safety tuning](safety-tuning.md), and
[safety evaluation](safety-evaluation.md) for lecture 12's measurement side.

## Midtraining — the boundary that stopped existing

The one genuinely important *method* point in the SFT half. Instruction data is
increasingly mixed into the **decay phase of pre-training** rather than applied
afterwards, "and so everyone, as far as I know, is doing it" (≈36:13).

![Slide 30 — 'Midtraining' / 'Two-phase training'](../raw/images/15-mid-post-training/slide-30.jpg)
*Slide 30 — miniCPM's two mixtures. The stable phase is general web data; the decay phase swaps in Stack Exchange QA, UltraChat and other SFT-style data.*

Two consequences the lecture draws out:

- **"Base model" has become a misleading term.** "Base models today are
  pre-trained on things like UltraChat and who knows what else. Those are chat
  datasets that are synthetically designed to make you good at chat. So it's very
  hard to say this is a base model in the traditional sense of the word" (≈36:59).
  The lecturer flags this as a personal pet peeve.
- **Midtraining is where mixture ablations actually happen.** Because the decay
  phase is short, "you can run something like ten of these for each one" full
  pre-training run (≈40:05) — and the results are then reflected *back* into the
  pre-training mix. The reason you don't simply make all of pre-training
  high-quality is token supply: "you run out of tokens if you try to make
  Wikipedia your whole pre-training set" (≈40:51).

The lecture's honest summary of mixture selection is that mixtures are "very
trial-and-error... We do have algorithms... but I think they're fairly
unreliable" (≈39:19) — which is the position
[data mixture selection](data-mixture-selection.md) records from lectures 9 and
14 as well. The leaked Meta book-ablation court documents are offered as a
concrete example of what the process really looks like (≈41:38).

Full treatment: [midtraining](midtraining.md).

## Part 2 — RLHF

### The conceptual shift

Pre-training and SFT are generative modelling: fit a distribution, predict the
next word. RLHF is not. "We are no longer playing a 'fit a distribution' game —
we are playing a 'maximize a reward' game" (≈43:10).

The consequence the lecturer wants remembered: under a reward objective, "I can
totally collapse my distribution onto a single point for every input... and that
would be okay, as long as it got a good reward." Mode collapse is not a failure
of the objective; it is permitted by it.

![Slide 31 — The second part of RLHF](../raw/images/15-mid-post-training/slide-31.jpg)
*Slide 31 — the InstructGPT pipeline again, now with steps 2 and 3 in focus.*

### Why optimize rather than collect more demonstrations?

Two arguments (≈43:55):

**People judge better than they generate.** In a study of freelance writers
summarizing news documents, some annotators preferred Instruct Davinci's
summaries to *their own writing* — and on interview, stood by it: "Huh, I did not
realize that was actually a better way of doing it." Their own summaries were
independently checked for quality. The gap between what people produce and what
they endorse is a reason to collect ratings rather than demonstrations.

![Slide 33 — Why optimize? G-V gap](../raw/images/15-mid-post-training/slide-33.jpg)
*Slide 33 — the generation–verification gap: freelance writers' summaries against Instruct Davinci's, judged by the writers themselves.*

**Verification is often easier than generation.** Checking a proof is easier than
finding one — which is the door to
[lecture 16's RLVR](https://www.youtube.com/watch?v=dIFAi87Ws4E), explicitly
deferred here (≈45:27).

Full treatment: [RLHF](rlhf.md).

### Collecting preferences

The standard shape: sample several outputs per prompt at temperature 1 (the
post-SFT model is still diverse enough for this), have a rater rank them, train a
reward model on the rankings, then optimize against it (≈46:14).

![Slide 36 — RLHF and data – standard setups](../raw/images/15-mid-post-training/slide-36.jpg)
*Slide 36 — a pairwise annotation interface of the kind most preference collection uses.*

The **InstructGPT appendix** is "kind of the last point at which we have a glimpse
into data-collection processes from industry" (≈47:00). Its guidelines ask raters
to balance three things — helpful, truthful, harmless. The only other public
example is a leaked set of **Google Bard** guidelines, which use a Likert scale
rather than pairwise comparison but are structurally similar (≈48:33).

See [preference data](preference-data.md).

### The annotators matter, in four separate ways

The lecture spends roughly fifteen minutes here, and it is the part least
available elsewhere. See [human annotation](human-annotation.md) for the full
treatment; the four findings are:

**The workforce has moved upmarket.** On Scale AI's platform, ~70% of annotators
hold bachelor's or master's degrees, with a modal age around 35, doing creative
and technical writing (≈49:19).

![Slide 39 — Modern worker distribution](../raw/images/15-mid-post-training/slide-39.jpg)
*Slide 39 — education, age and task breakdowns for one annotation platform's workforce.*

**Pay has bifurcated.** Median wages above $50/hour across many topics, with
specialists above $100 — because labs now want doctors and lawyers annotating
for professional deployment. But "it's also kind of a pyramid, it's not like the
lower-cost, scalable annotation has gone away" (≈50:52). The lecture does not
soften the ethics: Scale AI "got into quite a bit of trouble" over outsourced
low-cost annotation, and Google Bard annotators publicly disputed being asked to
fact-check long responses "in under a minute" (≈51:38, ≈52:25).

**Who annotates shapes what the model believes.** Asking models standard opinion-poll
questions, base models sat near Protestant and Roman Catholic response patterns;
post-trained models moved toward Buddhist, Hindu and atheist ones. The
explanation was in the InstructGPT appendix: the annotator pool was heavily
Southeast Asian and US West Coast (≈53:12). The lecturer adds the *emergent
misalignment* result — a model trained on innocuous data from a model that
"likes owls" inherits the preference — as evidence that "very subtle biases can
transmit through data" (≈54:44), while cautioning that this literature "can be
quite fragile and sketchy at times."

**Expertise changes which errors get caught.** Hosking et al. compared expert
annotators against crowdworkers: non-experts over-weight *formatting*, experts
catch *factuality* and *inconsistency*. Which is unsurprising — "it's much, much
harder to test for factuality, so it's no surprise that if you don't have
experts, you don't actually end up checking for these" (≈55:31).

![Slide 44 — RLHF and style – annotators matter (a lot)](../raw/images/15-mid-post-training/slide-44.jpg)
*Slide 44 — Hosking et al.'s error-rate differences between crowdsourced and expert annotation. Factuality under high assertiveness is the most negative cell on the slide, at -22.3%.*

Asked how you would even measure annotator quality, the lecturer gives two
imperfect answers and rejects both as sufficient: a sufficiently detailed
guideline can be made semi-objective, and inter-annotator agreement measures
variance — but "it doesn't tell you what the bias is," and if every annotator is
quietly using ChatGPT, "the variance will also be zero" (≈57:01).

### Models now do most of the annotating

The empirical answer to "human or model feedback?" is settled, and the lecture
says so plainly: "there's basically no space for human-collected data if all you
want to do is catch up to the frontier in capabilities" (≈1:00:53).

The instructive case is **Zephyr**. Hugging Face deliberately set out to build an
open model with no distillation, went to the same annotation vendors OpenAI uses,
and found the human data "extremely time-consuming, costly, and the results they
were getting were not actually better than using model-based annotations." They
switched to AI feedback. "They gave it a very real, good shot" (≈1:01:39).

![Slide 46 — RLHF and data - LM-generated](../raw/images/15-mid-post-training/slide-46.jpg)
*Slide 46 — UltraFeedback, Zephyr, Tulu 3 and OLMo: the open pipelines that ended up on model-based annotation.*

UltraChat and UltraFeedback are now standard, and Tulu 3 uses model-based
annotation throughout. The limit is sharp, though: "if you want to push the
frontier out, you don't necessarily get to play these games" (≈1:02:27), and
world knowledge from lawyers or scientists still has to come from lawyers or
scientists (≈1:04:01).

Two related uses of models on themselves: Anthropic's **Constitutional AI**,
which prompts a model to generate its own safety data and trains on it, and
**Self-Instruct**, the capability-centric version of the same loop (≈1:03:14).

Models inherit human biases too, sometimes worse — you can push response length
out and keep gaining win rate, and a separate result shows you can "RLHF on
length alone and do quite well on many of these benchmarks" (≈1:04:48).

See [model-based annotation](model-based-annotation.md).

### PPO, briefly

The RLHF objective, which "appears almost exactly in the InstructGPT paper...
equation two" (≈1:05:34):

$$\max_{\pi_\theta} \mathbb{E}_{x\sim\mathcal{D},\, y\sim\pi_\theta(y|x)}\left[r_\phi(x,y)\right] - \beta\, \mathbb{D}_{\text{KL}}\left[\pi_\theta(y\mid x)\,\|\,\pi_{\text{ref}}(y\mid x)\right]$$

Maximize reward, stay near the reference policy. The KL term exists "because I
don't want to go too far and become degenerate" (≈1:06:21) — and it turns out to
be the main defence against
[overoptimization](reward-overoptimization.md).

The route to PPO is three steps (≈1:07:06), and the lecture is explicit that a
fuller treatment comes next lecture:

1. **Policy gradients** — differentiate the expected reward, and you get
   log-probability gradients weighted by reward. "This really just looks like SFT,
   but with weighted examples."
2. **Off-policy / TRPO** — sampling is expensive, so reuse a rollout for several
   steps, but constrain how far you move or the local reward estimates blow up.
3. **PPO** — replace the hard trust-region constraint with clipped importance
   ratios, "a heuristic clipping thing."

See [PPO](ppo.md) and [reward models](reward-models.md).

### DPO

Before DPO, several reasonable-sounding shortcuts were tried and are listed so
you don't repeat them (≈1:09:24): prepending `good`/`bad` control tokens and
conditioning on `good` at generation time; training only on the winning
responses; and reward-model-filtered SFT. The first two do not work; the third
"does somewhat work" but not as well.

![Slide 55 — DPO – RLHF without tears?](../raw/images/15-mid-post-training/slide-55.jpg)
*Slide 55 — what DPO removes from the RLHF pipeline: the separate reward model, and the on-policy sampling loop.*

DPO removes both the reward model and the on-policy sampling. The derivation is
three moves (slides 56–57, ≈1:10:58):

1. Assume $\pi$ is **nonparametric** — the set of all policies. This is the one
   strong assumption.
2. Then the KL-regularized objective has a closed-form maximizer: the reference
   policy exponentially tilted by reward,
   $\pi_r(y\mid x) = \frac{1}{Z(x)}\pi_{\text{ref}}(y\mid x)\exp\!\left(\frac{1}{\beta}r(x,y)\right)$.
3. Invert it for the **implied reward**,
   $r(x,y) = \beta\log\frac{\pi_r(y\mid x)}{\pi_{\text{ref}}(y\mid x)} + \beta\log Z(x)$,
   and substitute that into the pairwise preference loss. The intractable $Z(x)$
   cancels, because the loss only ever sees reward *differences*.

$$\mathcal{L}_{\text{DPO}}(\pi_\theta;\pi_{\text{ref}}) = -\mathbb{E}_{(x,y_w,y_l)\sim\mathcal{D}}\left[\log\sigma\!\left(\beta\log\frac{\pi_\theta(y_w\mid x)}{\pi_{\text{ref}}(y_w\mid x)} - \beta\log\frac{\pi_\theta(y_l\mid x)}{\pi_{\text{ref}}(y_l\mid x)}\right)\right]$$

The gradient is the part the lecturer calls "the most intuitive form of DPO":
raise the likelihood of the winner, lower the likelihood of the loser, and scale
the step "by how much my implied reward model is wrong" — small steps where the
model already ranks the pair correctly, large ones where it thought they were
equal (≈1:13:17).

**In practice.** Llama's RLHF primitive is DPO, wrapped in an outer loop: SFT,
DPO, then generate candidates with the DPO'd model, rejection-sample them, and
repeat (≈1:14:02). Variants — SimPO, length-normalized DPO — exist in quantity
but "none of these variants seem to matter very much" (≈1:14:49).

**DPO versus PPO** is the lecture's model of a fragile empirical dispute. AI2
published a paper finding PPO beats DPO, and a Tulu 2 paper finding that
well-executed DPO beats PPO. "So, depending on how you execute this, one can be
better than the other" (≈1:15:36), and the practical answer is that it "maybe
doesn't matter very much, unless you're at the frontier."

![Slide 61 — But PPO does too (and sometimes better?)](../raw/images/15-mid-post-training/slide-61.jpg)
*Slide 61 — "Unpacking DPO and PPO", the other side of the dispute.*

Full treatment: [DPO](dpo.md).

## What to watch out for

**Overoptimization.** The historical hope was that enough thumbs-up/thumbs-down
could be scaled into superintelligence. It cannot: push RLHF hard and "you'll
start overfitting to your learned reward model" (≈1:16:22). The KL regularizer is
"really critical... at least if your optimization process is very good" — a nice
formulation of the problem, since a *better* optimizer makes this *worse*.

![Slide 63 — Things to watch out for - Overoptimization](../raw/images/15-mid-post-training/slide-63.jpg)
*Slide 63 — gold-standard reward turning over while the proxy reward keeps climbing: Goodhart's law with a learning curve.*

**Mode collapse and calibration.** RL'd models "have much less diversity —
they're concentrated on a few possible outputs" (≈1:17:08), which follows
directly from the objective shift at ≈43:10. GPT-4's own release materials list
post-RLHF miscalibration as an open problem, and "I don't think anyone has really
solved that yet" (≈1:17:54).

![Slide 64 — Things to watch out for - mode collapse](../raw/images/15-mid-post-training/slide-64.jpg)
*Slide 64 — the entropy histogram: RLHF'd models concentrate their probability mass.*

Both matter more next lecture, "where the entropy and exploration is actually
quite critical for the model to explore all the possible solutions" (≈1:18:41).

See [reward overoptimization](reward-overoptimization.md) and
[mode collapse and calibration](mode-collapse-and-calibration.md).

## The recap, and the handoff

The lecture closes on four points (≈1:18:41): RLHF data collection is hard for
the same reason all data work is hard; RLHF algorithms are complex, PPO
especially; GRPO is the simpler variant the assignment uses; and
**overoptimization is the big open problem**.

The transition to lecture 16 is posed as a question: "is there a reward where we
won't overoptimize, where we can just dump compute in and model performance just
keeps monotonically getting better? And that's one of the reasons why what people
call RLVR has been so impactful" (≈1:19:29).

## Related

- [Supervised fine-tuning](supervised-fine-tuning.md) — the method and its data
- [Instruction-tuning datasets](instruction-tuning-datasets.md) — FLAN through Nemotron
- [Midtraining](midtraining.md) — the decay-phase mixture
- [RLHF](rlhf.md) — the pipeline and the conceptual shift
- [Preference data](preference-data.md) · [Human annotation](human-annotation.md) · [Model-based annotation](model-based-annotation.md)
- [Reward models](reward-models.md) · [PPO](ppo.md) · [DPO](dpo.md)
- [Reward overoptimization](reward-overoptimization.md) · [Mode collapse and calibration](mode-collapse-and-calibration.md)
- [Style and length bias](style-and-length-bias.md) · [Hallucination and knowledge extraction](hallucination-and-knowledge-extraction.md) · [Safety tuning](safety-tuning.md)
- [Post-training data](post-training-data.md) — lecture 14's data-side treatment
- [Lecture 14](14-data-filtering-dedup-mixing.md) — the pre-training data unit this follows
- [Course material for lecture 15](../raw/slides/15-mid-post-training.md)
