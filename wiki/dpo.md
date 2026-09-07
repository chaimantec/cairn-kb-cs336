# DPO — direct preference optimization

DPO is [RLHF](rlhf.md) without a [reward model](reward-models.md) and without
on-policy sampling. It optimizes the policy directly on preference pairs with a
loss that looks almost like supervised fine-tuning, and it is what Llama and most
open post-training pipelines actually run.

CS336's lecture 15 calls it "the cool, fun bit" and gives the full derivation,
which is short (≈1:10:11 onward, slides 55–58).

## What it removes, and the intuition

> The algorithm is very simple: we're going to try to get rid of the complicated
> parts of PPO. What are we going to get rid of? We're going to get rid of the
> reward model, and we're going to get rid of anything that has to do with
> on-policy stuff.

The intuition underneath is almost crude: "everything in deep learning is just
taking gradient steps in the direction of good things. So we're going to take
steps in the direction of the log-loss of the good stuff, and then take negative
gradient steps in the direction of the bad stuff." SFT on the winner, negative
SFT on the loser. "It turns out that if you weight these two appropriately, you
get an algorithm that's actually pretty good."

### The shortcuts that do not work

Before DPO, several simpler ideas were tried, and lecture 15 lists them
explicitly so you don't rediscover them (≈1:09:24):

- **Control tokens.** Prepend `good` to preferred responses and `bad` to
  dispreferred ones, SFT on both, then condition on `good` at generation time.
  Does not work.
- **Train on the winners only.** Does not work very well.
- **Reward-model-filtered SFT.** Train a reward model, sample outputs, keep the
  ones it likes, SFT on those. "Doesn't work as well either, although it does
  somewhat work."

## The derivation

Three moves. Start from the KL-regularized RLHF objective:

$$\max_{\pi_\theta} \mathbb{E}_{x\sim\mathcal{D},\, y\sim\pi_\theta(y|x)}\left[r_\phi(x,y)\right] - \beta\, \mathbb{D}_{\text{KL}}\left[\pi_\theta(y \mid x) \,\|\, \pi_{\text{ref}}(y \mid x)\right]$$

**1. Assume the policy is nonparametric.** This is the one strong assumption, and
the lecture flags it as such: assume $\pi$ is "not a neural network at all — it's
the set of all possible policies, a nonparametric thing, it can approximate
anything" (≈1:11:43).

**2. Solve in closed form.** With that assumption the maximizer is the reference
policy exponentially tilted by reward:

$$\pi_r(y \mid x) = \frac{1}{Z(x)}\,\pi_{\text{ref}}(y \mid x)\exp\left(\frac{1}{\beta}r(x,y)\right)$$

which the lecture reads out in words: "every response gets upweighted or
downweighted with a weight of exp(1/β · r), based on how good the
reward is... if the reward is really bad, I'll exponentially downweight it; if
the reward's really good, I'll exponentially upweight it."

**3. Invert for the implied reward, and substitute.** Solve the above for $r$:

$$r(x,y) = \beta \log \frac{\pi_r(y \mid x)}{\pi_{\text{ref}}(y \mid x)} + \beta \log Z(x)$$

then put *that* into the pairwise preference loss the reward model would have
been trained with. The partition function $\beta\log Z(x)$ is intractable — and
it cancels, because the preference loss only ever sees reward *differences* for
the same prompt. What survives is the DPO objective:

$$\mathcal{L}_{\text{DPO}}(\pi_\theta; \pi_{\text{ref}}) = -\mathbb{E}_{(x,y_w,y_l)\sim\mathcal{D}}\left[\log \sigma\left(\beta \log \frac{\pi_\theta(y_w \mid x)}{\pi_{\text{ref}}(y_w \mid x)} - \beta \log \frac{\pi_\theta(y_l \mid x)}{\pi_{\text{ref}}(y_l \mid x)}\right)\right]$$

where $y_w$ is the preferred ("winning") response and $y_l$ the dispreferred one,
$\pi_{\text{ref}}$ is the frozen SFT policy, and $\beta$ controls how far the
policy may drift from it.

## The gradient — the part worth understanding

The lecturer calls this "the most intuitive form of DPO, at least to me"
(≈1:13:17). Slide 58 reproduces the DPO paper's annotated gradient:

$$\nabla_\theta \mathcal{L}_{\text{DPO}}(\pi_\theta; \pi_{\text{ref}}) = -\beta\, \mathbb{E}_{(x,y_w,y_l)\sim\mathcal{D}}\Big[\sigma\big(\hat{r}_\theta(x,y_l) - \hat{r}_\theta(x,y_w)\big)\big[\nabla_\theta \log \pi(y_w \mid x) - \nabla_\theta \log \pi(y_l \mid x)\big]\Big]$$

The paper's own underbrace annotations name the three pieces:

- $\nabla_\theta \log \pi(y_w \mid x)$ — **increase the likelihood of the winner**
- $-\nabla_\theta \log \pi(y_l \mid x)$ — **decrease the likelihood of the loser**
- $\sigma(\hat{r}_\theta(x,y_l) - \hat{r}_\theta(x,y_w))$ — **higher weight when
  the reward estimate is wrong**

That last factor is the adaptive step size, and it is what distinguishes DPO from
naive *SFT on good, negative SFT on bad*:

> Did my model already assign a very high reward to the winning one? If so, I'll
> take a small step. But if I was very wrong, and said, "Oh, these two are almost
> equal, same probability" — in that case, I'll actually take a much bigger step
> size.

## In practice

**Llama uses it, inside an outer loop.** The Llama tech report's recipe is: SFT,
then DPO, then use the DPO'd model to generate candidates, rejection-sample them,
and repeat. "So there's an outer loop on top of it, but basically the core RLHF
primitive for Llama was DPO" (≈1:14:02). This blurs the SFT/RL boundary — a point
the lecturer makes earlier when a student asks whether SFT and RL are really
distinct: "the lines are very blurry between the two, to be honest, especially
when we get into something like expert iteration" (≈35:26).

**The variants mostly do not matter.** Slide 60 shows two from the Tulu 3 paper:

*SimPO* drops the reference policy entirely and normalizes by length, adding a
margin $\gamma$:

$$\mathcal{L}_{\text{SimPO}}(\pi_\theta) = -\mathbb{E}\left[\log\sigma\left(\frac{\beta}{|y_w|}\log\pi_\theta(y_w \mid x) - \frac{\beta}{|y_l|}\log\pi_\theta(y_l \mid x) - \gamma\right)\right]$$

*Length-normalized DPO* keeps the reference log-ratio but divides each side by
response length, to blunt length-hacking:

$$\max_{\pi_\theta}\mathbb{E}_{y_c,y_r\sim\mathcal{D}}\left[\log\sigma\left(\frac{\beta}{|y_c|}\log\frac{\pi_\theta(y_c|x)}{\pi_{\text{ref}}(y_c|x)} - \frac{\beta}{|y_r|}\log\frac{\pi_\theta(y_r|x)}{\pi_{\text{ref}}(y_r|x)}\right)\right]$$

The lecturer's verdict: "there have been a lot of different variants of DPO...
But really, none of these variants seem to matter very much" (≈1:14:49). What
matters is the core move — signed gradient steps with a step size set by the
implied reward model's error — "as long as you set the step sizes right"
(≈1:16:22).

*(The transcript marks one phrase here as unclear: the lecturer's spoken
description of SimPO's normalizer. Slide 60 settles it — SimPO replaces the
reference log-ratio with a $\beta/|y|$ length-normalized log-probability.)*

## DPO versus PPO

Unsettled, and lecture 15 presents the dispute as a lesson about empirical
fragility rather than a question with an answer (≈1:15:36). AI2 published a paper
finding that moving from DPO to PPO improves results; the same group's Tulu 2
paper found that DPO done right beats PPO. "So, depending on how you execute
this, one can be better than the other. I'm sure you have experience reading
deep-learning papers where the results are very fragile."

The practical position: "it maybe doesn't matter very much, unless you're at the
frontier, training the very best model. DPO is reasonably good, and it's good
enough for Llama, it's good enough for me" (≈1:14:02).

## See also

- [RLHF](rlhf.md) — the objective DPO is derived from
- [PPO](ppo.md) — what it replaces
- [Reward models](reward-models.md) — why the $\log Z(x)$ term cancels
- [Style and length bias](style-and-length-bias.md) — what length-normalized variants are defending against
- [GRPO](grpo.md) — the other escape from PPO, for verifiable rather than pairwise rewards
- [Kimi K1.5](kimi-k1-5.md) — a DPO-style derivation that lands on GRPO
- [Lecture 15](15-mid-post-training.md), [Lecture 16](16-post-training-rlvr.md)
- [Course material for lecture 15](../raw/slides/15-mid-post-training.md)
