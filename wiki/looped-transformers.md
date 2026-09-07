# Looped transformers

Architectures that run a block of transformer layers **repeatedly** on the same
activation instead of stacking more distinct layers — trading parameters for
FLOPs. Also called recurrent-depth models. Introduced in
[Lecture 18](18-serving-megakernels-recurrence.md) (≈41:42–≈44:46) as the setting
for [PARSE](parse.md), the lecture's stabilization method.

## The construction

> This was basically our take on a technique called loop transformers, where you
> take some blocks of your transformer and run them in a loop. So instead of
> having your tokens go through the model one layer at a time, at some point you
> say, as you're going through, just send it back through that loop. (≈42:27)

Concretely: an activation flows through the early layers, reaches a designated
**recurrent block**, and passes through that same block some number of times
before continuing. "That purple block is the recurrent block, and at the end you
get the thing that comes back" (≈43:14). The number of iterations is a knob, set
independently of the parameter count.

## Why anyone wants this

**A dial for compute that is not a dial for parameters.** "You can keep your
parameters constant, but it gives you a dial to increase your flops. So, if you
think that more flops equals higher quality, this is a way to increase your
quality without paying a higher parameter cost" (≈43:59).

**An expressivity argument.** The lecture cites older work — "'old' is a relative
term, from a few years ago" — suggesting "there are things you can't express with
the same number of parameters that you can express with these looped models"
(≈43:59).

**The question behind both**, which is the one that connects this to the rest of
CS336: "what's the best quality per parameter, what's the best intelligence per
parameter, or intelligence per parameter and data, that these things will allow
you to do?" (≈43:59). Against
[scaling laws](09-scaling-laws.md) that spend on parameters and data, this
proposes a third thing to spend on — see
[scaling laws for recurrence](recurrence-scaling-laws.md).

**And a serving argument**, which is why this thread belongs in a lecture about
inference at all (≈1:01:52):

> One of the reasons that I was personally very excited about these loop things is
> that one of the big bottlenecks to serving inference efficiently actually ends
> up being GPU memory. If you have fewer parameters, you can fit, for example,
> more KV cache, or you can do less communication, because you need to split your
> model across fewer GPUs.

Fewer parameters is not just cheaper — it changes what fits where. See
[KV cache](kv-cache.md), [tensor parallelism](tensor-parallelism.md), and the
small-memory endgame in
[decode-specialized hardware](decode-specialized-hardware.md).

## Why it did not work

Looped models were unstable to train, in a way that made them impractical rather
than merely fiddly:

> If you looked at any of them, and you tried to train them, and then changed
> anything about the training algorithm at all — say, if you changed the learning
> rate by a little bit — you'd suddenly start to see these models blow up. If you
> did a simple thing like a learning rate sweep, you'd see that nine times out of
> ten this model just isn't going to converge — it's going to blow up, you're
> going to get NaNs, you're going to get these big loss spikes. (≈45:32)

Prior practice worked around it rather than through it: "you can put norms in
every layer to figure out what's happening, or just pick the learning rate of
2e-4, don't pick any of the other learning rates" (≈46:19). A model that trains at
exactly one learning rate is not a model you can scale.

The lecture's response is a research disposition worth extracting on its own:

> If you're ever training a model, and you're scaling it up, and you see these big
> loss spikes that suggest something has gone very wrong with your training
> process, you should take a deeper look and try to figure out what happened.
> (≈45:32)

That is what produces [PARSE](parse.md): the instability is diagnosed as a
property of the loop's dynamics — see [spectral radius](spectral-radius.md) —
rather than accepted as a fact about the architecture.

## Prior work and context, as the lecture gives it

- **Recurrent-depth models**, "a paper from Tom Goldstein's group at Maryland that
  suggested, hey, this thing might be better than transformers, with some results
  on the [ARC](reasoning-benchmarks.md) tasks" (≈44:46). This is the baseline
  PARSE is compared against (≈53:18).
- **A Twitter episode**, which the lecture recounts as an illustration of hype
  rather than evidence: about a week before PARSE's release, "some dude from
  OpenAI" claimed a frontier model was a looped language model. The lecturer's own
  assessment: "I don't think it's right, I think he was just making it up," and
  the claimant "had to write this blog post being like, 'Hey, my bad, I just made
  that up, none of it's true'" (≈44:46). The model name in that anecdote is
  garbled in the recording and the transcript marks it unresolved rather than
  guessing.
- **Looping a pretrained model**, from the Q&A, which is the strangest result
  mentioned: someone "won some leaderboard competition without training a single
  thing — what he did was he actually looped two or three layers in a Qwen model,
  and just saw that on some math things it started having higher quality"
  (≈1:01:05). The lecturer neither endorses nor explains it — "it kind of disturbs
  me, I don't know why that would possibly be the case" — and says work on it "may
  be coming out soon."

## Relation to other ways of spending compute at inference

Looping adds depth-wise compute inside a single forward pass, which distinguishes
it from spending compute on *more tokens* — the chain-of-thought and
[reasoning-model](reasoning-models.md) route that
[Lecture 16](16-post-training-rlvr.md) develops. The lecture does not directly
compare them; it does report that under a fixed FLOP budget, converting some of
that budget from data into recurrence lowers validation loss
([scaling laws for recurrence](recurrence-scaling-laws.md), ≈58:41).

## See also

- [PARSE](parse.md) — the stabilization that makes these trainable
- [Spectral radius](spectral-radius.md) — the criterion the instability violates
- [Scaling laws for recurrence](recurrence-scaling-laws.md)
- [Lecture 18](18-serving-megakernels-recurrence.md) ·
  [Architectures (Lecture 3)](03-architectures.md)
