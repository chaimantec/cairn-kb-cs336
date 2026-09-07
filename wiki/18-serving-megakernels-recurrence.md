# Lecture 18 — Guest Lecture (Dan Fu): Serving, Megakernels, and Looped Recurrence

**Dan Fu** (UCSD, and Together AI).
[Transcript](../raw/transcripts/18-serving-megakernels-recurrence.md) ·
[video](https://www.youtube.com/watch?v=9EEm4iMAF5s)

Every other lecture in CS336 builds a language model. This one starts the moment
the model is finished and asks what happens next: "once you have one of those
models, what it looks like from the other side — what it looks like to actually
serve these models, to do inference, turn these things from electricity into
tokens, into intelligence" (≈0:05). It is the last lecture of the course, and the
only one delivered by an outside speaker.

> **A note on this page's sources.** This is the one lecture in CS336 with **no
> course material of any kind** — no `.py` program, no slide deck, no handout.
> That was checked against the course schedule, the
> [lectures repo](https://github.com/stanford-cs336/lectures), both sibling-year
> archives and the course site repo; see [sources](../sources.md). Everything on
> this page therefore comes from the transcript alone, and every claim is cited
> to a timestamp rather than to a slide. Where the recording is garbled the
> transcript says so rather than guessing, and this page repeats those gaps
> rather than filling them.
>
> The speaker adds a caveat of his own that is worth carrying: his slides "are
> completely AI-generated, from Nano Banana Pro… they're pretty good, as long as
> you don't look too closely at the text" (≈7:45). He then points at one slide
> where the generator **invented content that he had to correct live** (≈28:36),
> and another where "some of the fives turned into S's" (≈9:16). So even the
> unpublished deck would not have been a citable authority for this lecture.

> **A note on the numbering.** Cairn's catalog lists this as Lecture 18, and this
> KB follows that. The course's own schedule calls it **session 19** (Wed June 3);
> its session 18 was a second guest lecture, by Daniel Selsam, which was never
> recorded and is not in the course playlist. So the course had 19 sessions, 18
> of which exist on video, and this is the last of them.

## The claim the whole talk is built to support

The talk states its thesis twice, in almost the same words, at the start and at
the end — which is the clearest signal available of what the speaker wanted the
room to keep:

> If you understand inference, and understand the inference engines, if you
> understand the GPU kernels that underlie a lot of the core technology, you can
> enable full-stack innovation in machine learning algorithms. (≈6:14)

The structure follows from it. First a walk down the whole serving path — "the
lifetime of a token" — so the audience knows what the pieces are. Then two
research projects, each of which is an instance of the thesis: one that changes
the *kernels* and gets a large speedup ([megakernels](megakernels.md)), and one
that changes the *architecture* in a way that only makes sense if you know what
serving costs ([PARSE](parse.md) and [looped
transformers](looped-transformers.md)). The closing restatement names exactly
those three levers: "a new routing algorithm… new kernels… new architectures"
(≈59:30).

That is also what makes this lecture a fitting end to CS336 rather than an
appendix. The course spends five units taking the stack apart —
[tokenization](tokenization.md), [architecture](03-architectures.md),
[kernels](06-kernels-triton.md), [parallelism](07-parallelism.md),
[scaling laws](09-scaling-laws.md), [data](13-data-sources-datasets.md),
[alignment](15-mid-post-training.md) — and this lecture argues that the payoff of
knowing all of it at once is the ability to change any layer of it.

## 1. Inference is the engine, not the afterthought

The motivation is economic before it is technical. Scale is what produced the
capabilities: 100-million-parameter models in 2018, GPT-2 in 2019 ("we thought
that they were too dangerous to release"), and today "open-source models that are
a trillion parameters and more; the frontier is probably at 5 to 10 trillion
parameters" (≈1:38, ≈2:24). That scale is bought with GPUs, and "in a very real
sense you could say GPUs are the new oil" (≈3:56).

The analogy is then completed, and it is the one the speaker clearly likes best.
In 1902 there were 130,000 working horses in Manhattan, and enough manure that
academic conferences convened on the problem; an 1898 conference concluded there
was nothing to be done but "hold your nose and deal with it." **"Ten years later,
by 1912, cars had already outnumbered horses in Manhattan"** (≈2:24, ≈3:11). His
reading of where we are: "for language models… that 1912 moment was probably last
year. At least for me, last year I started writing the majority of my code using
these language models" (≈3:56).

The technical point the analogy is carrying is this one:

> You can think of inference as the engine that turns electricity into
> intelligence — the same way that oil is only useful in a car if you actually
> have an engine that turns that oil into useful kinetic motion. (≈4:43)

A model, on this view, is not a running system. "These machine learning models
are really just DAGs of operations — there's some mathematical object that exists
in the ether. The inference engines, the GPU kernels, all these pieces, are the
things that you actually have to program and map down to ML operations" (≈5:29).
The course has already built the mathematical object; the engine is what this
lecture is about.

## 2. The lifetime of a token

The spine of the first half is a walk through what happens to one request. The
stages, in order (≈7:45–≈16:13), are on their own page —
[the inference request lifecycle](inference-request-lifecycle.md) — and in brief:

1. **Scheduling.** The request is assigned to GPUs, possibly with
   [prefill and decode disaggregated](prefill-decode-disaggregation.md) onto
   different machines.
2. **Cache lookup.** "You'll run that request against a KV cache to see: have I
   seen this request, or versions of this request, before? Is there some compute
   that I can actually save?" (≈8:31)
3. **Execution**, split across machines, nodes and GPUs by whichever
   [parallelism](07-parallelism.md) the model's size demands.
4. **Sampling and post-processing** — stop tokens, and "you might run a safety
   check to say, oh, is my user trying to hack into the system in a bad way"
   (≈15:26).

The engine "is really running in this loop, waiting for these requests to come
in, so it's running this scheduling, execution, token sampling loop, and
repeating" (≈15:26).

## 3. Production traffic looks like neither training data nor a benchmark

This is the part of the lecture with no counterpart anywhere else in CS336, and
it has its own page: [inference workloads](inference-workloads.md). The framing
claim is that a real serving workload

> doesn't necessarily look like, certainly doesn't look like the type of tokens
> that you see during training, but it also doesn't look like if you just made up
> a traffic workload in your head. (≈9:16)

What varies is not one number but a shape: the distribution of input against
output lengths, the number of turns, the gap between turns, and the latency
target. A coding agent with the codebase in context has "tens of thousands of
input tokens" and possibly a short output (≈10:02). Narrative summarization —
"pasting entire books into a chat window" — is different again, and a one-shot
"explain to me first-order calculus" is different from both (≈10:48).

Two observations here are easy to miss and are the reason the page exists. First,
**agentic workflows change the cadence, not just the volume**: an agent left to
iterate alone produces a different arrival pattern than an interactive chat, and
"if at some point your agent gets stuck and it's, 'Hey, hey, help, I need to ask
for advice,' and you don't notice it, there might be another gap between turns"
(≈12:22). Second, **session structure is a serving parameter**. The speaker's own
example is a ChatGPT conversation about his workout plan that he returns to "about
once every other week… a very different traffic pattern than some of the other
ones" (≈13:08). Those gaps are what decide whether a cache entry is still worth
holding — which is what §7 and §8 are about.

## 4. Prefill and decode: the split that organizes everything downstream

The single most load-bearing distinction in the lecture. The course has already
derived it in [Lecture 10](10-inference.md); this lecture is about what it does
to a *deployment*.

**Prefill** processes the prompt: "let's say you have 10,000 tokens that you've
never seen before, and you want to go compute the activations and what the logits
should be. So, 10,000 tokens in, one token out" (≈13:54). It is compute-bound,
and "that actually looks pretty close to the things that you guys have been
looking at for training time… Prefill is very similar, you just don't run the
backwards pass" (≈14:39).

**Decode** generates one token at a time, and re-runs the whole model to do it.
"If you do the math on that, there's actually not too many flops that you need to
compute. So it's going to be a relatively light computation, but it's going to be
very memory-bandwidth-bound. What that means is you have to load up the model
every time just to generate a single token" (≈14:39, ≈15:26). See
[arithmetic intensity](arithmetic-intensity.md) and
[GPU architecture](gpu-architecture.md) for why that is the binding constraint.

Everything else follows from the asymmetry. The two phases have different
bottlenecks, run for different lengths of time, and occur in different
quantities: "you run prefill once for a prompt, and you run decode once for every
token that you generate" (≈20:05). So they get
[split onto different workers](prefill-decode-disaggregation.md), and — §6 — they
increasingly get split onto **different silicon**.

## 5. Continuous batching, and the KV cache as a shared asset

[Continuous batching](continuous-batching.md) is presented through a figure with
time flowing downward, requests entering as they arrive: a long request running,
a short one joining it, the short one finishing, a new one starting (≈16:13,
≈16:59). Two kinds of resource are contended, and the second is the interesting
one: "These resources are first compute resources… They can be memory resources
if you're filling up a KV cache." When the cache fills, "you might start queuing
for that reason" (≈16:59).

The [KV cache](kv-cache.md) is then reframed from a per-request optimization into
a **cross-user asset**:

> You probably have a lot of users who are saying "hi" to ChatGPT or "hi" to
> Claude, and theoretically you don't need to compute new activations and run that
> again for every single user. (≈17:46)

The mechanism is prefix sharing over a tree: "use a very traditional data
structure like a basic tree, look at which tokens you've seen before, which
tokens are new, and then do a lookup" (≈18:33). This is the same structure
[paged attention](paged-attention.md) and prefix caching serve, arrived at from
the traffic side rather than the memory-allocator side.

## 6. Splitting the model, and splitting the hardware

A trillion-parameter model does not fit on one GPU — the lecture's stated figure
is "280-gigabyte GPUs" (≈18:33) — so it is split by
[tensor parallelism](tensor-parallelism.md), or, for the mixture-of-experts
models that are now "a lot of the state-of-the-art," by distributing
[experts](expert-parallelism.md) across GPUs (≈19:18). "The choices that you make
at this point will determine what the bottlenecks are."

Then the claim that generalizes the prefill/decode split beyond software.
Because the two phases stress different parts of the machine, they are starting
to run on **different chips** — see
[decode-specialized hardware](decode-specialized-hardware.md):

> Some things you might have heard of, like when NVIDIA bought Groq: NVIDIA, the
> king of GPUs, buys this new kind of inference chip. One of the reasons is that
> the decode workload is so different from the prefill workload that if you're
> looking at decode, you can be using very different chips. (≈20:51)

He describes NVIDIA planning to use "its GPUs for the prefill side, using these
LPU Groq chips for the decode," an OpenAI compute partnership with Cerebras —
"another chip that's much better at decode" — and SambaNova making bets in the
same space (≈20:51, ≈21:38).

> **What this KB does and does not vouch for.** These are corporate and
> commercial claims made in passing in a June 2026 talk, with no slide, citation
> or handout behind them. This page records **that the lecturer said them**,
> because they are the reasoning behind a technical point that does not depend on
> them — that decode's memory-bandwidth profile makes it a target for
> non-GPU silicon. Do not cite this KB as a source for who acquired or partnered
> with whom.

## 7. What breaks when you serve trillions of tokens a day

A short, memorable section, and the one with the least equivalent in a normal
systems course. Its own page: [serving at scale — failure modes](serving-at-scale-failures.md).

> One of the characteristics of these large-scale systems is that something that
> will work well at a small scale will inevitably start breaking at a large
> scale. We're talking about events that happen 0.001% of the time, or less.
> (≈22:23)

Three real incidents, which he places in open-source inference engines around
late the previous year (≈21:38):

- **NaN loops.** "Sometimes you'll have a kernel that is very slightly wrong, but
  the conditions for triggering it are very rare, and you'll start having some of
  your logits turn into NaNs halfway through the computation." The visible
  symptom is degenerate repetition — "the output just starts saying 'hi hi hi hi
  hi' after a while, or starts outputting exclamation points" (≈22:23).
- **The tool-call doom loop.** A change to tool-call handling stopped returns
  being processed, so the model would "say, 'Hey, make an internet search, hey,
  make an internet search, hey, I don't know why there's no internet search going
  on' — it would just get into this very long doom loop for tens of thousands of
  tokens." The *monitored* symptom was completion length shooting up (≈23:10,
  ≈23:56).
- **The Chinese-character bug**, which is the best story in the lecture because
  the plausible explanation was wrong. Models began emitting Chinese characters
  unprompted; it "actually got blamed on a quantization issue," and there was
  speculation that someone had fine-tuned on a Chinese model. The real cause was
  "an off-by-one error in one of the kernels, and it was a very subtle bug —
  sometimes you would read in some extra, uninitialized memory space from your
  GPU, run it through attention, and then at the end of that whole process you
  get a random Chinese character." The
  model then rationalizes: "Why did I start suddenly thinking Chinese? I must
  be — the user must be asking me a question in Chinese," and continues in
  Chinese (≈23:56, ≈24:42).

The last one carries a lesson the lecture states plainly: "sometimes when this
happens it's because the model has legitimately been trained to think in Chinese;
sometimes it can just be an off-by-one bug in somebody's code" (≈24:42). A
behavioural symptom does not imply a behavioural cause.

## 8. The KV cache down the memory hierarchy

Serving wants the largest possible cache, so the cache descends: GPU memory, then
CPU DRAM, then SSD, then "some other global store" (≈25:30, ≈27:03). Details on
[KV cache offloading](kv-cache-offloading.md).

This is where a hardware aside becomes a real constraint. "If you're paying
attention to Jensen's keynotes, he's recently started getting very obsessed with
CPU performance," because a slow CPU bottlenecks the reads: "if your
$500,000,000 machine is being bottlenecked by the thousand-dollar CPU that you
purchase to put on top of it, that's not a great place to be" (≈25:30, ≈26:16).
He connects the same pressure to reports of "OpenAI buying up all the SSDs, all
the DRAM in the world" (≈27:03).

Asked whether particular workloads get offloaded, he gives the answer that
organizes the section (≈27:48):

> I'm sure none of you guys have taken an operating systems class, but you should
> take your operating systems classes — we actually get to some pretty classic
> scheduling things. This diagram, except for the GPU things on the right, looks
> exactly like an operating systems diagram that you might have seen in the '70s
> or the '80s… It's exactly the same workload.

And so the eviction policy is the classic one. LRU is "actually a pretty decent
heuristic. There's probably some OS paper somewhere that says LRU is within 2x of
what's optimal" (≈28:36). The ideal is prefetching against a prediction, and here
the lecture makes a genuinely nice observation: **the UI gives you the
prediction**. "When you go into your chat app and bring up some old conversation
from a month ago, that's a very strong signal that you're going to start asking a
question about it, and you might then want to go load that up onto a GPU"
(≈28:36).

This passage is also where the AI-generated slide misfires and he corrects it in
real time: "on the left side, Nano Banana hallucinated 'evictions for least
recently used'" (≈28:36).

## 9. Rack scale, fault tolerance, and one very cheap optimization

The last generation of Blackwell parts ships "NVL72 Grace Blackwell chips…
72 GPUs that are connected with really fast interconnect" (≈29:21), which raises
questions the course's [parallelism](08-parallelism-2.md) lectures set up: how to
split a trillion-parameter model across all 72, how to handle a million-token
context, and — the one with no clean answer — **fault tolerance**: "if I've taken
a model, split it across 64 GPUs, I'm serving production traffic against millions
of users, trillions of tokens — what do I do when a single GPU goes down?"
(≈30:56).

The failure mode he actually names is gloriously physical: "the connectors are
kind of flimsy, made of plastic, not metal, so if you jam the thing in too much,
your cables are going to bend a little bit and then you get really flaky NVLinks"
(≈30:09). (This is also the slide where the image generator "put a fan into the
chips, which doesn't quite make sense.")

Against that, the section closes with the smallest intervention in the lecture and
the best effort-to-payoff ratio in it —
[cache-aware routing](cache-aware-routing.md), "a piece of work that we put out
together a few months ago called cache-aware prefill/decode disaggregation. It's a
very simple optimization — it's like two lines of code in the routing layer"
(≈31:44).

The argument is entirely about traffic. If a conversation lasts ten turns on
average, then "10% of your requests are going to be very fresh, very new
requests" — and those cold requests are the expensive ones, thousands of tokens
of prefill. Mixing them with warm short turns is what hurts:

> You don't want that very short question-and-answer to happen at the same time,
> on the same GPUs, as the very long request. (≈32:29)

So route by cache-hit rate: cold requests to one pool of prefill nodes, warm ones
to another. **"Turns out, if you do this, you can get up to 40% faster serving
with these very simple optimizations"** (≈33:14). No new kernel, no new
architecture — just a scheduler that knows the shape of the traffic from §3.

His own summary of the state of the art is worth keeping: "we're very early —
this is the type of thing that, in 10 or 20 years, people are going to look back
on and be like, 'Oh, why are these guys talking about this? Isn't this already
obvious to folks?'" (≈33:14).

## 10. Research thread 1 — megakernels

Full treatment: [megakernels](megakernels.md), with the underlying problem on
[kernel launch overhead and tail effects](kernel-launch-overhead.md).

The problem is decode's, and it is stated sharply: because you must run the whole
model to produce one token, "instead of using all this big parallelism that you
get with a GPU… you've now turned this massively parallel system into basically a
glorified memory loader" (≈34:47).

Compounding it is how kernels are written. One kernel per operation is what makes
them tractable to write — "you just have to write your norm kernel, or your map
kernel, or attention kernel" — "but it ends up introducing a lot of downtime into
your system" (≈35:33). The lecture visualizes this with time on the x-axis and
the GPU's streaming multiprocessors on the y-axis — "on an H100 there are 132 of
these; on the B200 there are, I think, 148" — where bars are useful work "and the
empty space is just waiting" (≈36:18). Three sources of empty space: **kernel
launch and teardown**, **tail effects** when a batch mixes short and long inputs,
and **the gaps between kernels**, which "add up" (≈36:18, ≈37:06).

A **megakernel** is the aggressive answer: "instead of treating each operation in
the model as its own operation and writing a kernel for it, let's write a single
kernel to cover multiple operations at once. This is similar to the fusion that
you see in flash attention, except done more aggressively" (≈37:06). The
conceptual move is to stop treating the GPU as one device:

> You start thinking of the GPU as a massive distributed system, and saying,
> okay, I have all this work that I need to get done, some of it has dependencies
> on other stuff… how can I schedule it, how can I distribute the work to
> maximize my GPU utilization? (≈37:54)

The results, as stated: **30–70% speedups** on the attention inference kernel
alone, and then the whole thing applied to one layer of a Llama 1B model (≈37:54).
What fusion buys at that scale is overlap that crosses operation boundaries —
"a weight load from the next layer into the attention," a reduction started
"before the attention operation is over" (≈38:39). Two concrete instances:
loading the KV cache into attention while QKV plus [RoPE](rope.md) is still
running (≈39:25), and starting the O-projection's weight load before attention
finishes (≈39:25).

The implementation is "a relatively complex CUDA framework, with basically an
instruction-based abstraction where we can implement each subkernel in its own
file, and then have a big virtualized shared-memory system to orchestrate the
running of these operations," built on the **ThunderKittens** library — "you can
think of it as almost like [Triton](triton.md), except more low-level, with a lot
more fine-grained control" (≈40:11).

The payoff is stated as a fraction of the hardware limit rather than a speedup
over a baseline, which is the right way to state it: **"on the H100, it's
achieving 72% bandwidth utilization, which is near the speed of light on the
GPU"** (≈40:56). Stating it as a fraction of achievable bandwidth rather than as
a multiple of some baseline is the same discipline
[Lecture 5](05-gpus-tpus.md) applies to
[arithmetic intensity](arithmetic-intensity.md): it says how much room is left,
which a speedup number does not.

The cost is paid in labour, and the Q&A puts a number on it (≈1:03:26):

> The trade-offs are people's blood, sweat, and tears… I think a fully talented
> kernel engineer, over the course of a year, will probably be able to write
> megakernels for one hardware, for two or three models, for batch sizes one to
> 16. You go to batch size 17, you're like, nope, start over.

## 11. Research thread 2 — PARSE and looped recurrence

Full treatment on three pages: [looped transformers](looped-transformers.md) for
the architecture, [PARSE](parse.md) for the stabilization, and
[scaling laws for recurrence](recurrence-scaling-laws.md) for the empirical
claim. The stability criterion itself is on
[spectral radius](spectral-radius.md).

The question is posed against the whole first half of CS336: capability has come
from scaling parameters and data, so "is this the only way you have to scale, or
is there potentially some other way that you can get this quality" (≈41:42,
≈42:27). PARSE is work from his UCSD lab, "led by Hayden, and also in
collaboration with two folks, Zachary and Taylor" (≈41:42).

**The architecture** is a loop transformer: "you take some blocks of your
transformer and run them in a loop. So instead of having your tokens go through
the model one layer at a time, at some point you say, as you're going through,
just send it back through that loop" (≈42:27). The appeal is a
parameter/compute decoupling — "you can keep your parameters constant, but it
gives you a dial to increase your flops" — plus an older expressivity result
that "there are things you can't express with the same number of parameters that
you can express with these looped models" (≈43:14, ≈43:59). The driving question
is "the best intelligence per parameter" (≈43:59).

**The problem** is that they will not train. Change anything — "say, if you
changed the learning rate by a little bit — you'd suddenly start to see these
models blow up… you'd see that nine times out of ten this model just isn't going
to converge" (≈45:32). Prior practice was workarounds: norms in every layer, or
"just pick the learning rate of 2e-4, don't pick any of the other learning
rates" (≈46:19). The lecture draws a general moral here that applies far beyond
this architecture:

> If you're ever training a model, and you're scaling it up, and you see these big
> loss spikes that suggest something has gone very wrong with your training
> process, you should take a deeper look and try to figure out what happened.
> (≈45:32)

**The analysis** is the elegant part, and it is a modelling decision rather than a
theorem. Analyzing the block directly is hopeless — "tons of parameters, there's
all sorts of nonlinearities, there's a softmax, there's RoPE" (≈46:19). So: look at
the *residual* instead, observe empirically that "it actually doesn't change that
much" from block to block, then write a dynamical system over that residual
(≈47:06). All the nonlinear machinery is pushed into a box called $R$ and set
aside, leaving two matrices — a $B$ that transforms the initial vector into the
loop, and an $A$ that transforms the residual at each iteration (≈47:52, ≈48:38).
That framing recovers the prior work as special cases: "in one case you just
treat it as the identity, you're just going to add things; in another case it's a
fully learnable matrix" (≈48:38).

Dropping $R$ leaves a system solvable "if you use high school calculus," and its
closed form is "dominated by these $A$ matrices, and especially this $A$ matrix
that you are powering up to a large degree" (≈49:24). Hence the diagnosis, in
terms of the [spectral radius](spectral-radius.md) $\rho(A)$:

> If this matrix can learn to be something like — let's say, if you go to
> scalars, imagine this matrix is two, and then this t is like 16 or something —
> you've now taken this activation and blown it up to 2 to the 16th, and it's
> really big. This starts to explain some of those big loss spikes. (≈50:11)

That is $2^{16} = 65536$, from a per-iteration factor of two applied sixteen
times.

The prior choices of $A$ and $B$ are, in his words, "marginally stable, or
unstable" (≈50:59).

**The fix** is to constrain the two matrices so the math cannot explode: make $A$
"a negative diagonal matrix — if you power that up, the term eventually goes to
zero," and put "a really simple linear norm" on $B$, which is safe because "the
$B$ matrix actually only gets applied once, so it doesn't really blow up." Then
$\rho(A) < 1$ and "it's now actually going to be a stable system" (≈50:59,
≈51:45). Trained, "even with the 6e-4 learning rate that was so bad for the other
models, you actually got a stable model at the end" (≈51:45).

There is a subtlety worth preserving, because it explains why norms alone were
not the answer. Norming an unconstrained model sets up a tug-of-war: the model
"is actually trying to expand the activations, because it's saying, oh, with more
room I can represent different things better," while the norm pulls it back, "and
that manifests in loss spikes." The tell is that "even though on the right your
norms are very good, you're not seeing the activation actually blow up — you do
see that the loss can do some pretty gnarly things" (≈52:32). Constraining the
dynamics removes the pressure instead of fighting it. The unconstrained baseline
"blows up, goes to 10 to the 19th" (≈51:45).

**The results**, as stated: PARSE beats prior loop transformers ("recurrent-depth
models") and "also outperforms a strong transformer baseline — this transformer
is like one of the nanochat ones," on both perplexity and end-to-end quality
(≈53:18). The prior work that motivated the area is "a paper from Tom Goldstein's
group at Maryland… with some results on the ARC tasks" (≈44:46).

**The scaling claim** is the part with consequences for the rest of the course.
Recurrence is proposed as a *third axis* alongside parameters and data. Holding
parameters fixed and sweeping data against recurrence count on iso-parameter,
iso-FLOP curves, the curves go "down and to the right" — the same signature that,
in the classic parameters-vs-data plots, means scale both together (≈54:51,
≈56:23). So: "for these fixed-parameter training runs, as you increase the amount
of data you should actually also be increasing the amount of recurrences you
have," and the recurrences "follow some pretty classic power laws," which makes
joint prediction possible (≈56:23). A fixed-FLOP comparison makes the same point
directly: spend some of the budget on recurrence rather than all of it on data and
"you start to get smaller validation losses" (≈58:41).

The observation he draws from it is the sharpest claim in the lecture:

> As far as I know, all of our models today have no recurrence in them, so they're
> all at the very left of these curves, and they all have a ton of data, which
> suggests that there might be something slightly better that we could be doing
> when training these models. (≈57:54)

And the conclusion, stated as a suggestion rather than a result: "it might be the
case that we should be looping all of our big pre-training runs" (≈58:41).

Two honesty notes the page keeps because the lecture does. He is explicit that
the three-way version of the scaling figure was inconclusive to read: "we had this
really complex 3D figure that showed recurrences, data, and parameters… but that
figure was just really hard to look at" (≈57:09). And on the pretrained-model
result in §12 he says plainly, "it kind of disturbs me, I don't know why that
would possibly be the case" (≈1:01:05).

## 12. Questions from the floor

Six exchanges, and several carry material found nowhere in the body of the talk.
Where a question was inaudible the transcript marks it rather than reconstructing
it; the speaker restates most of them before answering, and those restatements are
kept verbatim.

**Does offloading apply to particular workloads?** (≈27:48) Answered in §8 — it is
classic OS paging, and the answer is the LRU/prefetch discussion.

**Can you loop a model that is already pretrained?** (≈1:00:18–≈1:01:52) Yes,
apparently, and nobody knows why. He cites "a really troll blog post from someone
a few months ago, where he was like, hey, I won some leaderboard competition
without training a single thing — what he did was he actually looped two or three
layers in a Qwen model, and just saw that on some math things it started having
higher quality." Work on this "may be coming out soon."

**What are the inference implications of looping?** (≈1:01:52–≈1:02:38) This is
where the two halves of the talk join. Fewer parameters means "you can fit, for
example, more KV cache, or you can do less communication, because you need to
split your model across fewer GPUs." He then names the dream case: a recurrent
block small enough to be its own megakernel, kept resident in memory. "So far we
haven't been able to make those blocks small enough" — but on next-generation LPU
parts with "like 250 megabytes of memory," if you can design a model to fit, "you
can just keep your weights in memory the whole time and run your activations
through as quickly as you can" (≈1:02:38).

**What are the trade-offs of megakernels?** (≈1:03:26–≈1:04:11) The labour answer
quoted in §10, plus the state of tooling: at Together "we're trying to put
together some compilers that can automate some of that process, but it's a very
challenging thing to do." And a note that the idea is not new — it "has kind of
gone in peaks and troughs over the last few decades when it comes to GPU
programming."

**If you know the serving hardware, how should the architecture change?**
(≈1:04:11–≈1:05:43) Memory first: "if you know you're going to be taking a model
and serving it on a particular Cerebras chip, you want to go look at the Cerebras
wafer, figure out how much memory you have, and then size your model so that it
can fit there with enough KV cache." Then numeric format, with a concrete
example — NVIDIA's Nemotron trained in **NVFP4**, "an FP4 format that is
proprietary to NVIDIA chips," against **MXFP4** for AMD. He also reads Chinese
model releases as showing quantization choices that "suggest they might be
starting to think about the Huawei chips that are coming out." See
[quantization](quantization.md).

**Is looping ever compute-optimal, or is it only an inference trick?**
(≈1:06:29–≈1:07:17) A careful answer that partly deflates the framing:
"compute-optimal is always: given some flop budget, figure out what you want to
hit. It's almost a little bit contrived, in that sense, because if you want a
higher-quality model, you should just increase your flops budget." The real
choice is made under external constraints — model size fixed, data exhausted, or
"if you're going to release it open-source, what is the size of model that people
can serve on their laptop today." See [compute-optimal
scaling](compute-optimal-scaling.md).

**How different are optimal architectures across use cases?**
(≈1:07:17–≈1:09:35) The most architecturally interesting answer in the Q&A.
Agentic workflows want the KV cache "as hot as possible"; batch processing that
sees each document once does not care about it at all. That single axis explains
[MLA](multi-head-latent-attention.md) — "a radical compression of the KV cache" —
and FP8/FP4 KV cache formats. But "the biggest one is causal attention versus
non-causal attention": for batch processing "Google was just using BERT models,
and I think probably still uses BERT models on search… you just do that big
bidirectional attention once, get your vector out, and then stick that in the
database," whereas chat "there's always going to be this decode portion of it."
T5 gets named as the intermediate design.

**Do megakernels survive multi-GPU communication?** (≈1:10:22–≈1:11:09) Partly.
"Turns out you can also fuse the NCCL calls into the megakernel, if you set it up
correctly," though "we haven't found a really great killer use case for that yet,
where sometimes you're just bound by the latency of the NCCL call itself." He
cites a DeepSeek megakernel for the mixture-of-experts inference layer that fused
some communication. The forecast is the last technical statement of the course:

> You'll have a little megakernel for part of the computation, but you won't
> necessarily have a megakernel for the whole model, unless, of course, you pay
> the blood, sweat, and tears price, and then you really get the whole thing
> going. (≈1:11:09)

## Where this sits in the course

This lecture is the mirror image of [Lecture 10](10-inference.md), and the two
should be read together. Lecture 10 **derives** inference: the three metrics, the
arithmetic of a forward pass, KV cache size, quantization,
[speculative sampling](speculative-sampling.md). Lecture 18 **operates** it —
the same objects seen from a production serving stack, where the constraints come
from traffic and hardware procurement rather than from a derivation. Almost
nothing overlaps.

It also closes loops opened much earlier:

- [Kernels and Triton (Lecture 6)](06-kernels-triton.md) taught fusion and asked
  you to write flash attention; megakernels are that idea pushed to the whole
  model, with ThunderKittens as the tool.
- [GPUs and TPUs (Lecture 5)](05-gpus-tpus.md) established occupancy, SMs and
  memory bandwidth; §10 is those numbers used as a budget, ending in a
  bandwidth-utilization figure rather than a speedup.
- [Parallelism (Lectures 7](07-parallelism.md) and
  [8)](08-parallelism-2.md) built the splitting strategies; §6 and §9 apply them
  to serving, where fault tolerance and flaky cables become part of the design.
- [Scaling laws (Lectures 9](09-scaling-laws.md) and
  [11)](11-scaling-laws-in-the-wild.md) fitted parameters against data;
  §11 proposes recurrence as a third axis and fits power laws on it.
- [Architectures (Lecture 3)](03-architectures.md) and
  [attention alternatives (Lecture 4)](04-attention-alternatives.md) surveyed the
  design space; the Q&A argues the space should be searched with the serving
  platform already in hand.

The through-line, and the reason it is the last lecture: CS336's premise is that
you should be able to build the whole stack, and this lecture's thesis is that
the *return* on knowing the whole stack is being able to innovate at any layer of
it — routing, kernels, or architecture — which is precisely what its two research
threads demonstrate.

## Topics from this lecture

- [The inference request lifecycle](inference-request-lifecycle.md)
- [Inference workloads](inference-workloads.md)
- [Prefill/decode disaggregation](prefill-decode-disaggregation.md)
- [Cache-aware routing](cache-aware-routing.md)
- [KV cache offloading](kv-cache-offloading.md)
- [Serving at scale — failure modes](serving-at-scale-failures.md)
- [Decode-specialized hardware](decode-specialized-hardware.md)
- [Megakernels](megakernels.md)
- [Kernel launch overhead and tail effects](kernel-launch-overhead.md)
- [Looped transformers](looped-transformers.md)
- [PARSE](parse.md)
- [Spectral radius](spectral-radius.md)
- [Scaling laws for recurrence](recurrence-scaling-laws.md)

## See also

- [Lecture 10 — Inference](10-inference.md), which derives what this lecture operates
- [Inference](inference.md) · [KV cache](kv-cache.md) ·
  [Continuous batching](continuous-batching.md) ·
  [Prefill and generation](prefill-and-generation.md)
- [Course map](course-map.md)
