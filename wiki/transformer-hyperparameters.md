# Transformer hyperparameters

The numbers you have to pick before you can train anything: how wide the
feedforward layer is, how many attention heads, how deep against how wide, how big
the vocabulary, and whether to regularize at all.

Hashimoto's framing at [43:44] is the reason this section exists: "when your
knowledge about language models is abstract, you don't have to care about any of
these. But once you have to instantiate it, you start to ask questions like, well,
how big should the feed-forward size be?" The space looks daunting and
high-dimensional — "but the space of things that people try is actually pretty
small."

The recurring finding is that most of these hyperparameters are **forgiving**:
there is a broad basin of near-optimal values, everyone has converged on roughly the
same point inside it, and what actually decides the choice is usually a systems
consideration rather than a modelling one.

## Feedforward ratio: $d_{ff} = 4\,d_{model}$

The width of the feedforward hidden layer relative to the model dimension. Slide 37
states the rule of thumb in the middle of the slide, in bold:

$$d_{ff} = 4\,d_{model}$$

and calls it "*almost always* true. There's just a few exceptions."

**Exception 1 — gated layers scale down by 2/3.** A GLU has a third matrix, so
keeping the parameter count fixed means shrinking $d_{ff}$ by $2/3$, giving
$d_{ff} = \tfrac{8}{3} d_{model} \approx 2.67\,d_{model}$. See
[gated activations](gated-activations.md). Slide 38's table shows where models
actually land:

| Model | $d_{ff}/d_{model}$ |
| --- | --- |
| PaLM | 4 |
| Mistral 7B | 3.5 |
| LLaMA-2 70B | 3.5 |
| LLaMA 70B | 2.68 |
| Qwen 14B | 2.67 |
| DeepSeek 67B | 2.68 |
| Yi 34B | 2.85 |
| T5 v1.1 | 2.5 |

The 3.5 values have a specific origin ([46:03]): the LLaMA 2 authors reasoned that
because MQA makes their attention heads cheap, they could afford to multiply the
ratio by roughly 1.33 and emphasize the MLPs more. Hashimoto calls this
"arbitrary." The practical summary is his: "either 2.6-ish or 3.5 for GLUs, or four
if you're doing non-GLU models."

**Exception 2 — T5, which is spectacular.** Slide 39: the 11B T5 sets
$d_{ff} = 65{,}536$ against $d_{model} = 1024$, "for an astounding 64-times
multiplier." The paper's own justification, quoted on the slide, is a systems
argument — "modern accelerators (such as the TPUs we train our models on) are most
efficient for large dense matrix multiplications."

Hashimoto plainly enjoys this one ([46:49]): "most people are just very boring in
their choice of architectures — they're like, we did LLaMA but we changed one
thing. But folks at Google are very bold sometimes."

**The evidence for the basin.** Slide 40 reproduces a sweep from Kaplan et al.
2020 with $d_{ff}/d_{model}$ on a log x-axis and "Loss Increase" on the y-axis.
The curve is essentially flat from ratio 1 to ratio 4 — under about 0.3% loss
increase — then climbs steeply: about 1.8% at ratio 8, 4.8% at ratio 25, and 8.4%
at the far right around ratio 50. Hashimoto describes the basin as running "from
about one and end up at about maybe 10" ([48:21]), which is the wider reading of
the same chart, and notes that above that "your loss starts really shooting up."

The punchline is slide 41's third bullet, and Hashimoto's at [49:07]: T5 v1.1, the
improved follow-up to T5, quietly went back to a 2.5 multiplier. Nobody says why,
"but clearly, when they tried to update T5, they decided they wanted to go back to
a more standard multiplier, which I find to be a little bit funny." Radical choices
can work — T5 was a good model — but they are probably compute-inefficient.

## Head dimension: $n_{head} \times d_{head} = d_{model}$

The convention is that the heads partition the model dimension: with $h$ heads,
each has dimension $d_{model}/h$, so multiplying back gives $d_{model}$. Slide 42
reproduces the CS224n slide explaining why this makes multi-head attention
essentially free — you compute $XQ \in \mathbb{R}^{n \times d}$, reshape to
$\mathbb{R}^{n \times h \times d/h}$, and treat the head axis like a batch axis, so
"the **matrices are the same sizes**."

Nothing forces this. As the slide says, "we can have head-dimensions > model-dim /
num-heads." But slide 43 shows almost everyone sticking to a ratio of about 1:

| Model | Num heads | Head dim | Model dim | Ratio |
| --- | --- | --- | --- | --- |
| GPT3 | 96 | 128 | 12288 | 1 |
| T5 | 128 | 128 | 1024 | 16 |
| T5 v1.1 | 64 | 64 | 4096 | 1 |
| LaMDA | 128 | 128 | 8192 | 2 |
| PaLM | 48 | 258 | 18432 | 1.48 |
| LLaMA2 | 64 | 128 | 8192 | 1 |
| Qwen 3.5 (27B) | 24 | 256 | 5120 | 1.2 |

The exceptions are again Google models — T5 at 16 and LaMDA at 2.

> Two details worth flagging, both from the deck rather than the speech. PaLM's head
> dimension is **printed as 258**, not the round 256 you would expect; this KB
> transcribes it as printed rather than correcting it. And slide 51's own summary is
> unusually candid about the evidence base for this rule: "Head dim\*Num head = D
> model is standard – but low to no validation."

## Aspect ratio: $d_{model}/n_{layer} \approx 100$

How deep against how wide. Hashimoto rates this the most conceptually interesting
of the hyperparameters ([51:25]), because it is the one you hold fixed when you
scale a model up: "the way you usually do that is you fix an aspect ratio ... and
then you make the whole model bigger."

Slide 44's table, with a margin bar marking the "Sweet spot?" band from 87 to 128:

| Model | $d_{model}/n_{layer}$ |
| --- | --- |
| BLOOM | 205 |
| T5 v1.1 | 171 |
| PaLM (540B) | 156 |
| GPT3/OPT/Mistral/Qwen/OLMo 3 | 128 |
| LLaMA / LLaMA2 | 102 |
| Gemma 3 | 87 |
| Gemma 4 | 61 |
| T5 (11B) | 33 |

**Why the sweet spot is where it is, is a systems answer.** Depth is hard to
parallelize and width is easy ([52:12]–[52:58]). Splitting a model across devices
by layer means pipeline parallelism, where each device waits on the one before it —
slide 45 draws exactly this, four layers on four GPUs with forward arrows running
left to right and backward arrows returning. Hashimoto's verdict: pipeline parallel
"is something that most people really, really do not want to deal with." Splitting
by width is tensor parallelism, "much, much simpler to deal with." Slide 45 also
quotes Tay et al. 2021 making the same point about depth being "non-parallelizable
across different machines."

So: "there's systems reasons to go wide, and maybe there's expressiveness reasons
to go deep, and you end up at roughly a hundred."

**The evidence** is on slide 46, and it is the most-cited chart of the section.
Kaplan et al. plot loss against aspect ratio for three model sizes (50M, 274M and
1.5B parameters), with two vertical rules at roughly ratio 9 and ratio 330 and the
annotation between them reading **"A wide range of architectures achieve similar
performance"**. All three curves are close together and nearly flat between the
rules, rising steeply outside them.

> The y-axis of that Kaplan panel carries gridlines but **no tick labels and no axis
> title** in the deck's paste. That was verified against the PDF at high
> magnification during this KB's figure audit — it is a genuine property of the
> slide, not a limitation of the transcription. So the chart establishes the *shape*
> of the relationship, not the size of the loss penalty in nats.

Tay et al.'s panels on the same slide sweep $d_{model}$ and number of layers
independently, plotting negative log-perplexity and SuperGLUE accuracy against
FLOPs. Hashimoto's reading at [54:31]: "as you sweep the depth-to-width tradeoffs,
you find that really, the only thing that matters, in some sense, is FLOPs."

## Vocabulary size

Slide 47 splits the world in two, and the split is by what the model is *for*
rather than by year:

**Monolingual models — 30–50k.** Original transformer 37,000; GPT 40,257;
GPT-2/3 50,257; T5 32,128; LLaMA 32,000.

**Multilingual and production systems — 100–250k.** mT5 250,000; PaLM 256,000;
GPT-4 100,276; Gemma 4 262,144; DeepSeek 100,000; Qwen 152,064; Yi 64,000.

Hashimoto's account at [55:18]: early open-source work was largely monolingual
English, and post-LLaMA interest shifted to multilingual and production systems,
which "really do need a much larger vocab to cover the whole space." Two secondary
effects at [56:04] — Google models consistently carry more vocabulary than
everyone else, and larger models can support larger vocabularies, so part of the
trend is a scaling effect rather than a language-coverage one. "No one's training
... large monolingual models anymore."

This connects back to Lecture 1 — see [tokenization](tokenization.md).

## Regularization: dropout and weight decay

Slide 48 poses the question and gives the textbook answer for *not* regularizing a
pretraining run: there are trillions of tokens, more data than parameters, and SGD
makes only a single pass, so memorization is hard. Hashimoto endorses the reasoning
at [59:57]: "overfitting is not really a problem, almost ever, during
compute-constrained language modeling. Now, some people even only look at training
loss."

And yet slide 49 shows people doing it anyway:

| Model | Dropout | Weight decay |
| --- | --- | --- |
| Original transformer | 0.1 | 0 |
| GPT2 | 0.1 | 0.1 |
| T5 | 0.1 | 0 |
| GPT3 | 0.1 | 0.1 |
| T5 v1.1 | 0 | 0 |
| PaLM | 0 | (variable) |
| OPT | 0.1 | 0.1 |
| LLaMA | 0 | 0.1 |
| Qwen 14B | 0.1 | 0.1 |

Dropout has largely gone; **weight decay has not**. "This is very mystifying — why
is this?" ([1:00:43]).

**The resolution: weight decay is not acting as a regularizer.** Slide 50
reproduces Andriushchenko et al. 2023. Its left panel scatters validation loss
against training loss for three weight-decay settings ($\lambda = 0.0$, 0.1, 0.3);
all three colours lie interleaved along a single diagonal band, with no colour
systematically above or below. The slide's caption: "It's not to control
overfitting."

The middle and right panels show what it *is* doing. Under a 10× cosine
learning-rate decay, the stronger weight-decay runs start out worse — the blue
$\lambda = 0.3$ curve is highest for most of the run — and then converge to a
*lower* final loss (about 3.28 against 3.30 for $\lambda = 0.0$), with the dashed
"tiny LR" branches ending lowest of all at about 3.23. Under a constant learning
rate the effect largely disappears. The slide's caption: "Weight decay interacts
with learning rates (cosine schedule)."

Hashimoto's summary at [1:02:15]: the stronger weight-decay runs "start out slow
but essentially end up converging to a much better minimum later. And this is
generally true when we decay the learning rate, not necessarily true when we're at
a constant learning rate, which is maybe more where your intuition is coming from."

Asked afterwards why regularization helps at all ([1:04:34]), he distinguishes the
two: dropout is out of favour "because it doesn't really interact well with
optimization", whereas weight decay "might allow you to use a higher learning rate,
or it might allow you to decay faster."

> The transcript carries an `[Ed:]` note at [1:03:01], where the captions have
> Hashimoto saying weight decay is an optimization intervention "which is what you
> would expect here" — while the surrounding passage frames the finding as
> surprising throughout, suggesting a dropped negation. The substance is not in
> doubt; slide 50's own captions say the same thing.

Weight decay is also the one hyperparameter people change *during* training
([1:24:28]): "people change it in concert with learning rate — that's actually a
heuristic that people do that works very well." The architecture hyperparameters
cannot be changed mid-run at all, since doing so would make the training
incompatible.

## The summary slide

Slide 51 collects the four answers, and they are worth memorizing as defaults:

- **Feedforward** — factor-of-4 rule of thumb, 8/3 for GLUs.
- **Head dim** — $n_{head} \times d_{head} = d_{model}$, though with "low to no
  validation."
- **Aspect ratio** — a wide range of good values, 100–200; systems concerns pick
  the value.
- **Regularization** — you still do it, but its effects are primarily on
  optimization dynamics.

Hashimoto's own gloss at [53:45] is the sentence to keep: "there are a lot of
hyperparameters that seem quite important, but are also fairly forgiving, and
people have converged roughly on the minimum."

## The scaling-law view (lecture 9)

Lecture 3 answers these questions by surveying what people do.
[Lecture 9](09-scaling-laws.md) answers them from first principles, and the
reframing is worth having: the question is not "what is the best value" but
**"is this quantity scale-invariant?"** ([36:07]).

- **Number of layers is not scale-invariant.** One layer is catastrophic — "with one
  layer, you're not going anywhere, that's a terrible scaling trend" — and beyond
  that, more layers is better at every compute level, so the optimum keeps moving as
  you grow ([36:07]).
- **Aspect ratio $d_{model}/n_{layer}$ roughly is.** Across model sizes the minima
  land in the same place, "around 100 d-model per layer, or maybe a little less",
  drifting only slightly toward smaller ratios for deeper models ([36:54]).
  Slide 34's middle panel shows the three model-size curves dipping to a shared
  near-zero minimum, with a bracket annotating the flat region: "a wide range of
  architectures achieve similar performance."

That distinction is what licenses the Lecture 3 advice. "If your strategy is 'I'm
going to fix my aspect ratio and scale up,' you can make plots like this and
convince yourself you're probably good, because your optimum isn't shifting too much
as you go to larger and larger models" ([36:54]). The basins Lecture 3 reads off
Kaplan are not just broad — they are broad *in a way that survives scaling*, which
is the property you actually need.

Slide 34's caption from Kaplan puts a number on it: aspect ratio "can vary by a
factor of 40 while only slightly impacting performance", and a $(6, 4288)$ model
comes within 3% of the $(48, 1600)$ model used in GPT-2.

The two hyperparameters that are **not** inheritable this way are batch size and
learning rate — see [critical batch size](critical-batch-size.md) and
[learning rate scaling and muP](learning-rate-scaling-and-mup.md).

## Related

- [Gated activations](gated-activations.md) — where the 2/3 factor comes from.
- [Scaling laws](scaling-laws.md) — the hub. Kaplan et al. supplies the evidence for
  two of the basins here, and [Lecture 9](09-scaling-laws.md) supplies the
  scale-invariance argument above. Lecture 11 is not yet covered.
- [Efficiency](efficiency.md) — the course's organizing frame, which is why so many
  of these answers turn out to be systems answers.
- [Lecture 3 — architectures](03-architectures.md).
