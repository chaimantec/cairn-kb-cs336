# Pruning and distillation

"you just take a large model, rip out pieces of it, and fix it up… It's a very
crude way, but it turns out to work" ([1:07:17]). Pruning removes
parts of a trained model; distillation repairs the damage. In
[lecture 10](10-inference.md) they are a single recipe, because pruning without
the repair leaves a model that is "not going to be very good."

## The recipe

From the NVIDIA paper the lecture links at [1:07:17] and works through to
[1:08:50]. The lecture never names it — its text says only "Paper from NVIDIA" —
but the results figure labels its models "Minitron 8B" and "Minitron 4B", which is
where the name used on this page comes from:

1. **Estimate importance** for each layer, attention head and hidden dimension, on
   a small calibration set — 1,024 samples is enough.
2. **Rank and trim.** Keep the top-scoring elements, drop the rest. The paper's
   figure shows this at the granularity of individual embedding channels, heads and
   MLP channels, not just whole layers.
3. **Distill** the original model into the pruned one, training it further "to kind
   of heal it."

Steps 2 and 3 iterate. What makes it cheap is that step 3 starts from weights that
are already most of a good model rather than from noise: the transcript is garbled at that exact point and carries an editorial note, but the sense is a training process that shrinks the model while initializing it from parts of a good one

The source calls this the **distillation recipe**, and contrasts it with the
from-scratch one:

> **From scratch:** 1. Define faster model architecture. 2. Train faster model.
>
> **Distillation:** 1. Define faster model architecture. 2. Initialize weights
> using original model (which has a different architecture). 3. Repair faster model
> (distillation).

Percy's description of step 2 is worth keeping, because the architectures genuinely
differ: "you kind of just make this Frankenstein thing — and then repair the
faster model with distillation" ([1:09:36]).

## How importance is measured

A student asks exactly the right question — how do you tell an important layer from
an unimportant one? The answer ([1:09:36]–[1:10:22]): run a calibration set through
the model and look at the magnitude of the activations. Units that are close to
zero, especially dead ones, go; large ones stay.

The follow-up is sharper still: what if a unit is *always* large — is that
meaningful, or an artifact? Percy concedes the point and gives the refinement: look
at variance too. "if it's 100, you can't just remove it, because then everything's going to be broken… but if it's high mean and low variance, maybe there's another way to just incorporate it as a bias, essentially" ([1:11:07]) — a constant-output unit is a bias
term in disguise, not a computation.

He also states the load-bearing empirical assumption plainly: this all works
because "this is an empirical observation that some of the channels will be much higher
than others. If this weren't true, these techniques wouldn't necessarily work"
([1:10:22]). The same observation underwrites [AWQ](quantization.md), which
allocates precision by activation magnitude for the same reason.

## The result

The Minitron figure, reproduced in
[`raw/slides/10-inference.md`](../raw/slides/10-inference.md), plots MMLU against
training cost in trillions of tokens. Minitron 8B, pruned from Nemotron-4 15B,
lands at roughly 64% MMLU at about 0.1T tokens of training — against Nemotron-3 8B,
trained from scratch at that size, at about 54.8% for roughly 3.7T tokens. The
figure's own annotation reads **"40x cheaper / 9% bettter"** (the typo is in the
source image). Minitron 4B sits near 58.5% at the same cost.

For context, that puts a pruned 8B in the neighbourhood of Gemma 7B (~64.5%) and
LLaMA-3 8B (~65%), both of which cost 30–150× more tokens to train.

The lecture's summary: "They were able to take a 15B model and reduce it to an 8B
model, and it doesn't really hurt accuracy by too much, and the amount you use
to train the model through this process is much less" ([1:08:50]).

## Where it sits

Pruning shrinks the *parameter* term of the memory bill, and, when heads or
key-value heads go, the [KV cache](kv-cache.md) term as well. It belongs to the
lossy group alongside [quantization](quantization.md) and the cache-reduction
architectures, and carries the same obligation to check accuracy afterwards.

It also feeds [speculative sampling](speculative-sampling.md) directly. A pruned,
distilled model that came out *almost* good enough is exactly what a draft model
is: "if you're not happy with it, then at least it can be a draft model, and you
can use your main model to fix things up" ([1:16:25]). Distilling the draft toward
the target is also how you raise the acceptance rate.

The lecture's own treatment of this section is brief — the source function ends
with a bare `# TODO` — so it is an outline of a method rather than a full
treatment.

## Related pages

- [Quantization](quantization.md) — the other lossy compression, and the same activation-magnitude observation.
- [Speculative sampling](speculative-sampling.md) — where a not-quite-good-enough pruned model finds a job.
- [KV cache](kv-cache.md) — what pruning heads also shrinks.
- [Inference](inference.md) — the workload.
- [Lecture 10 — Inference](10-inference.md)
