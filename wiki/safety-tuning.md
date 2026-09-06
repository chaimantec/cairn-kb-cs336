# Safety tuning

The training-time side of model safety: the post-training data and procedure that
make a model refuse things. This is distinct from
[safety evaluation](safety-evaluation.md), which is lecture 12's treatment of how
you *measure* whether it worked.

CS336's lecture 15 frames safety as an SFT data problem and is candid that the
public record is thin — "safety SFT information seems even more sparse than
capabilities SFT information in publicly available documents" (≈28:32).

## Why post-training owns this

The lecture's framing is organizational as much as technical (≈27:46):

> I think pre-training people live in this ivory tower of "let's just compress the
> world." But, post-training people have to say, "Oh, yes, but what if people are
> using our system for political manipulation and disinformation?"... You're the
> last line of defense, in some ways, between the system and its misuse.

The de facto mechanism companies use is refusal: "you have to apply safety
controls to the system that will refuse to respond to malicious inputs."

![Slide 22 — Safety](../raw/images/15-mid-post-training/slide-22.jpg)
*Slide 22 — the misuse categories that safety tuning is meant to address.*

## The two-sided objective

Every safety-tuning approach is balancing exactly two quantities (≈29:18):

- **Violation rate** — how often genuinely bad queries get answered.
- **False refusal rate** — how often benign queries get refused.

The second is not a footnote. The lecture's example is the over-trained model that
will not explain how to "kill a Python process": "if your bot's like, 'No, I can't
let you kill anything,' that's very frustrating to the user." You want "a good
Pareto trade-off between the two," and you navigate it by constructing data
tailored to each side.

This is the same construct-validity problem the course raises about benchmarks —
see [construct validity](construct-validity.md) — appearing on the training side:
"safety" is not one thing, and a model tuned on one operationalization of it will
be wrong about others.

## How much data

Order of "a few thousand or tens of thousands of examples" (≈30:04). For Llama 2,
"I think it's about a few thousand examples used for this process" (≈30:04). The
lecturer pulled out Llama 2's own account of its safety SFT and noted, with some
exasperation, that even though "this is one of the more detailed descriptions...
you'll see that they don't even talk about how many examples they used for safety
tuning, which is kind of crazy to me" (≈29:18).

![Slide 23 — Safety SFT in the wild](../raw/images/15-mid-post-training/slide-23.jpg)
*Slide 23 — Llama 2's description of its safety fine-tuning, among the most detailed published accounts.*

## The one good public pipeline: Tulu 3

Tulu 3, the post-training pipeline behind Allen AI's OLMo models, is the lecture's
recommended reference for post-training generally and safety specifically —
"given the lack of details about post-training in many places, this is one of the
few places where you can see a reasonably performant post-training pipeline in
action" (≈30:04). Its safety component is roughly **50,000 examples**.

![Slide 24 — Pipeline with most details](../raw/images/15-mid-post-training/slide-24.jpg)
*Slide 24 — the Tulu 3 pipeline in full.*

### The recipe: mine real misuse, then write the refusals

The strategy is "fairly simple" and worth knowing because the lecturer believes
the closed labs do the same thing (≈30:50):

1. **WildChat** — give people free chat access, and collect what they do with it.
2. **Filter** those logs for attempted unsafe behaviour and for jailbreak attempts.
3. **Write the preferred response** for each: resist the jailbreak in one case, and
   decline in the other.

> From what we see — at least in the technical reports and model cards of the
> various closed-source companies — similar things are happening: you look at
> usage information, find unsafe behaviors, and get your annotators to play
> whack-a-mole with these bad behaviors.

"Whack-a-mole" is the lecturer's word, and it is a fair description of a
fundamentally reactive process — you can only defend against the misuse you have
already observed.

![Slide 25 — Main safety approach – extract scenarios from users](../raw/images/15-mid-post-training/slide-25.jpg)
*Slide 25 — WildTeaming's attack-composition pipeline and the non-compliance taxonomy that organizes what a model should decline and why.*

The non-compliance taxonomy on slide 25 is worth noting on its own: refusal is not
one behaviour. Incomplete requests, subjective matters and indeterminate requests
are separate categories calling for different responses, which is what makes the
false-refusal side of the trade-off tractable at all.

## Five hundred examples is enough to move the needle

The most quotable result in the SFT half of the lecture, and the clearest evidence
for the [SFT-as-extraction view](supervised-fine-tuning.md#sft-as-extraction-not-instruction).

Take ~500 examples — find or synthesize unsafe prompts, write refusals — and "the
rate at which you follow malicious instructions or other kinds of hate-speech
instructions drops dramatically... you're really applying a fairly small push to
the model, but that results in a fairly nice, across-the-board reduction in these
safety issues" (≈32:23).

![Slide 26 — Safety-tuning with just a little data](../raw/images/15-mid-post-training/slide-26.png)
*Slide 26 — mean harmfulness score against the number of added safety examples, across four harm benchmarks. The drop from 0 to 300–500 examples is steep in every group; beyond that the curves flatten.*

The proposed explanation: "models already have a 'will I be a safe model or an
unsafe model' axis inside them after pre-training, so it does not take very many
examples to pull this out" (≈33:09).

**And the qualification, which matters just as much.** Cheap steering is not the
same as policy enforcement:

> Just because you can steer the model with very few examples doesn't mean there's
> no benefit to more examples. If you're OpenAI or Anthropic and you want to
> enforce really fine-grained distinctions about what is safe and not safe, you do
> need very large-scale data collection — this doesn't really change that story.

Which is consistent with Tulu 3 spending 50k examples on safety rather than 500.
Five hundred buys you the broad axis; the remaining 49,500 buy you the boundary.

## See also

- [Safety evaluation](safety-evaluation.md) — lecture 12's measurement side, including jailbreaking and HarmBench
- [Supervised fine-tuning](supervised-fine-tuning.md) — the extraction argument this exemplifies
- [Construct validity](construct-validity.md) — why "safety" resists a single operationalization
- [Instruction-tuning datasets](instruction-tuning-datasets.md) — Tulu 3 in its wider role
- [Model-based annotation](model-based-annotation.md) — Constitutional AI's self-generated safety data
- [Lecture 15](15-mid-post-training.md)
