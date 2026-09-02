---
title: "Lecture 7: Parallelism"
lecture: 7
video: https://www.youtube.com/watch?v=SzpOcwdIL0Y
source: copy-edited from the YouTube auto-captions
verbatim_original: not committed - regenerate from the video (see below)
material: ../slides/07-parallelism.md
---

# Lecture 7: Parallelism — transcript

**This is the edited transcript.** The auto-captions have been repunctuated,
segmented into sentences, stripped of filler ("um," "uh," "you know," "sort
of," "kind of," "right?," "okay"), and had mis-heard technical terms
restored. No content was added, removed, or reordered, and every `[MM:SS]` /
`[H:MM:SS]` marker is preserved in its original position — content that was
under a marker is still under that marker. The verbatim captions are **not committed to this repo** — they are a
complete reproduction of the lecture and none of that text is this KB's own work.
They stay reproducible on demand: the `cairn-kb` skill's `fetch_transcript.py`
run on the video id in this file's front matter, piped through
`transcript_to_md.py`, reproduces them exactly — so the checks above can be
re-run by anyone who wants to verify this edit.

Terminology was cross-checked against
[the lecture's source program](../slides/07-parallelism.md) (`lecture_07.py`),
which is the authority for collective-operation names, interconnect hardware
names, library/API names (NCCL, `torch.distributed`, gloo, FSDP, ZeRO,
`all_gather_into_tensor`, `reduce_scatter_tensor`, `mp.spawn`), and numbers.

**Restorations made.** These are places where the captions produced a wrong
or garbled word and the text now reads differently.

| Caption | Restored | Confirmed by |
| --- | --- | --- |
| "you had um HPM which we lamented was so slow" / "this lecture HPM is going to be considered fast" / "don't fit on the HPM memory" (3 instances) | HBM | slide (HBM throughout); also spelled correctly by the captions themselves elsewhere in this same lecture (0:05, 0:51, 24:01) |
| "connected via NVLink and NVLink switch" / "NVLink and NVLink switch and InfiniBand" / "you can't have NV switch and NV link" / "a NV switch handling like 100,000... GPUs" / "NVLink land and NV Switch land" / "72 GPUs that are all kind of NV switched" / "a NV uh switch over some domain" / "connected to this... NV switch" / "NVLink and NV Switch. And another way... InfiniBand and NV Switch and NVLink" / "connected by NVLink, you know, switch" (NVSwitch, ~11 instances total) | NVSwitch | slide ("NVLink/NVSwitch", "connected by NVLink to an NVSwitch") |
| "we're not going to look too too much more into uh nickel" / "is nickel optimized for multi-node clusters" / "you don't have to explicitly think about nickel" / "use the nickel backend" (x2) / "data goes through nickel" / "it would be glue, but it could be nickel" / "topology, which is something that kind of nickel... figures out" (7 instances) | NCCL | slide (NCCL); also spelled correctly by the captions themselves at 29:27–30:15, immediately before the "nickel" instances start |
| "there's something called glue" / "I'm going to use uh the glue back end" / "in this case it would be glue" (3 instances) | gloo | slide ("gloo (CPU)", `dist.init_process_group("gloo", ...)`) |
| "algorithms such as uh uh SFTP" (36:23) | FSDP | slide (`FullyShardedDataParallel`); also spelled correctly elsewhere in this lecture |
| "pipe data parallel or SFDP" (1:17:22) | FSDP | slide, same as above |
| "to get to fancier things like, uh, zero or FSDP" / "FSDP and zero" / "next time we'll do FSDP and zero" (3 instances) | ZeRO | slide ("FSDP/ZeRO") |
| "important for MOEs" / "training MOEs" / "the key idea of the MOE" / "parallelize the experts for MOEs" (4 instances) | MoE / MoEs | slide's own casing ("MoEs", "MoE") |
| "There's a PCIE bus" / "GPUs on the same node you use PCIE to communicate" / "has to go through PCI E" (x2) | PCIe | slide ("PCIe", "PCI(e) bus") |
| "the question is what is the difference between our DMA and this?" | RDMA | context — the answer that immediately follows ("RDMA you can think about it as...") makes clear the question was about RDMA; RDMA is also spelled correctly throughout the rest of this lecture |
| "don't do it uh a sync" / "do a sync" (x2, 41:04) / "printed statements are coming in whatever order... because it's running a sync" (40:17) / "CUDA is already kind of a sync" / "all the processes being a sync" (44:07) | async | context (the whole passage is about asynchronous CUDA execution and process execution) plus the slide's own code, which names the parameter `async_op` |
| "you call uh um you know, torches.multiprocessing.spawn" | `torch.multiprocessing.spawn` | slide shows `mp.spawn(...)`; that `mp` is the conventional alias for `torch.multiprocessing` is outside knowledge of the PyTorch API, not spelled out in the slide itself |
| "if you just call dot backward" | `.backward()` | outside knowledge (standard PyTorch autograd API); not given in the slide's own code excerpts for this lecture, but uncontroversial |
| "I do the reduced scatter tensor" | `reduce_scatter_tensor` | slide's code block: `dist.reduce_scatter_tensor(output=output, input=input, ...)` |
| "after I do the all gather into tensor" | `all_gather_into_tensor` | slide's code block: `dist.all_gather_into_tensor(output_tensor=output, input_tensor=input, ...)` |
| "G stands for a grace" | Grace | **captions plus capitalization** — the word "grace" is present in the captions; the only editorial act is capitalizing it as the proper noun (NVIDIA's Grace CPU, the "G" of the "GB200"/"GB300" naming this same passage discusses a moment earlier). The slide file never names "Grace", so the identification rests on outside knowledge — but the word itself does not. |
| "remember Tatsuzou lecture" (19:25) / "and Toshi will talk more about this" (1:12:01) / "next Wednesday, Tatsuo will" (1:20:26) | Tatsu | context — this transcript's own captions spell the name correctly as "Tatsu" three other times (16:20, 1:01:58, 1:10:26); the slide file's front matter separately identifies the co-instructor by full name, "Tatsunori Hashimoto," of which "Tatsu" is the standard nickname (outside knowledge, matching the identical restoration made in the Lecture 6 transcript for the same person) |
| "RoCE, RDMA over a converged Ethernet" | RoCE, RDMA over Converged Ethernet | slide ("RDMA over Converged Ethernet (RoCE)"); the stray "a" was dropped and "Converged Ethernet" capitalized to match the acronym expansion |
| "you can simply define the model and the starting strategy" | sharding strategy | slide, verbatim: "the model, the sharding strategy, and the Jax compiler" |
| "requires very fast internet connects" | interconnects | slide, verbatim: "requires very fast interconnects (e.g., NVLink)" |
| "I can apply this non-linearity because this is element-wise anyway" | elementwise | slide's own casing: "GeLU is elementwise" |
| "could be some, could be max, could be min" | sum | slide's own list: "reduce... (sum, min, max)" |
| "the first three, broadcast, gather, gather, reduce" | broadcast, scatter, gather, reduce | context — matches the broadcast/scatter/gather/reduce ordering used everywhere else in this lecture; the duplicated "gather, gather" is restored to "scatter, gather" (the "three" vs. "four" count itself is left as heard — see [Ed:] notes below) |
| "for the NV uh 72" (31:00) | NVL72 | slide ("GB200/GB300 NVL72"); also spelled correctly elsewhere in this lecture (27:53) |
| "if you use NVLink uh five" | NVLink 5 | slide ("NVLink 5.0") |
| "has some tensor 0 1 0 1 2 3" | 0, 1, 2, 3 | slide's broadcast example: `tensor([0., 1, 2, 3])` — the leading "0 1" is a duplicated stumble, dropped |

**Capitalization and formatting only** (already the correct word in the
captions; only hyphenated, cased, or given code-font to match the slide's own
usage — not restorations): "all reduce" → "all-reduce", "all gather" →
"all-gather", "reduce scatter" → "reduce-scatter", "all to all" → "all-to-all"
(applied throughout, ~90 instances total — the slide headers themselves read
"All-reduce," "All-gather," "Reduce-scatter," "All-to-all"); "Nvidia" →
"NVIDIA" (slide casing); "param.grad" given code font (already spelled
correctly by the captions).

**`[Ed:]` notes inserted, and why:**

- *At 7:04*, "the first three, broadcast, scatter, gather, reduce" — the
  captions say "three" but four operations are then named, and the very next
  paragraph (7:50) explicitly calls all four of these "the warm-ups." The
  word "three" is left as heard rather than silently changed to "four."
- *At 7:04–7:50*, "how these are collective commission" — trails off into
  the next paragraph's "operations work"; "commission" does not parse and
  may be a segmentation artifact at the marker boundary rather than a real
  word. Left as heard, since content cannot be moved across the boundary.
- *At 10:08*, "the input is you have a dump for a bunch of pieces" — "a dump
  for" does not parse and no confirmable replacement is available.
- *At 32:33*, "like what what pieces like what cables are are and switch is
  are there" — does not resolve into a clean sentence; left close to
  verbatim rather than smoothed into invented phrasing.
- *At 48:47*, "then you get the bad effective bandwidth" — "bad" does not
  fit grammatically or logically; left as heard.
- *At 50:20*, "this uh world my size - 1 / world size" — "my" does not
  parse; likely a stray syllable or mis-segmentation, but not confirmable.
- *At 1:08:54*, "So, um >> So, in this So, what hack is done for U and
  versus automatically?" — reads as garbled crosstalk, possibly between the
  lecturer and a student (the ">>" itself looks like a caption artifact of a
  speaker change); does not resolve into a clean sentence. Left close to
  verbatim, in the same spirit as the Lecture 6 transcript's handling of a
  similar moment.
- *At 1:14:18*, "because I just did a full repass" — likely refers to the
  full forward-and-backward pass running to completion before any
  communication starts, but the exact intended word is not confirmable.
- *At 1:16:37*, "you wouldn't do tensor parallelism the kind of an NVLink...
  domain" — a preposition seems to be missing (e.g., "outside of," "beyond"),
  but the exact word is not confirmable; left close to verbatim.
- *At 1:13:32*, "if you put an 'i' before these" — read here as referring to
  prefixing `send`/`recv` with an "i" (i.e., `isend`/`irecv`, the standard
  `torch.distributed` async point-to-point calls). This reading rests on
  context plus outside knowledge of the PyTorch API; the slide file does not
  mention `isend`/`irecv` at all, so it is flagged as an editorial judgment,
  not a confirmed restoration.
- *At 1:17:22*, "the critical batch fact uh size" — read as "the critical
  batch size," dropping the intrusive "fact uh". The term is therefore
  present in the captions almost verbatim and this is a stutter collapse,
  **not** an outside-knowledge restoration. It is recorded here only because
  the term does not appear anywhere in the slide file, so a reader cannot
  confirm it against the course material; it is a standard term from the
  large-batch-training scaling literature.

**Genuinely unclear passages, not otherwise flagged inline:**

- *At 29:27*, "the NVIDIA Collective Communications Library, uh NCCL or
  NCCL, which translates..." — both words are already correctly "NCCL," so
  there is nothing to restore, but the duplication itself is unexplained.
  One possibility: the second occurrence was meant to be the colloquial
  pronunciation "nickel" (heard repeatedly later in this lecture), and the
  auto-captioner, having just locked onto "NCCL," transcribed it as "NCCL"
  again instead. Left as heard rather than guessed at.
- Several `*[Question from the floor: ...]*` tags (11:41 NumPy broadcasting;
  33:18 NCCL/multi-node; 1:08:08 "what happens in backprop?" and "is that
  done automatically by autograd?") have no independently-transcribed
  student utterance in the captions — only the lecturer's own "so the
  question is X" paraphrase survives. In these cases the tag reproduces that
  paraphrase rather than a separately-recorded utterance; this is a
  placement judgment call, not a wording uncertainty.
- *At 41:50*, "the question is the rank of the GPU" does not itself read as
  a coherent question, so it is tagged
  `*[Question from the floor, inaudible]*` rather than reconstructed.
- *At 31:46*, "Who is responsible for this one?" is tagged as a separate,
  short floor question from the RDMA question that immediately follows it.
  It is possible these are fragments of one question rather than two;
  the split is a judgment call.

**False starts are preserved, not completed** — e.g., "dis- disables" at
37:56, "ex- expensive" at 28:41, "rem-" at 1:02:44 (never completed), and
several self-corrections kept with both halves, such as "streaming multiple
— streaming multiprocessors" at 0:05 and "small matrix — smaller matrix
multiplications" at 1:07:21.

**Student questions.** The auto-captions run student questions together with
Percy's speech with no speaker labels. Wherever the recording makes clear
that someone from the floor is asking something, the transcript marks it
`*[Question from the floor: ...]*` (or `*[Question from the floor,
inaudible]*` when no usable words survive), using the captions' own words
with only light punctuation and the same filler-removal applied to the rest
of the transcript — never rewritten or completed. This lecture's questions
did not happen to straddle a `[MM:SS]` marker boundary, so the
`*[continued: ...]*` device used in the Lecture 6 transcript was not needed
here.

**Verification (run in the parent, against the verbatim original).** All three
checks pass.

1. *Timestamp sequence* — 105 markers in each file, identical and in order,
   `[MM:SS]` and `[H:MM:SS]` alike.
2. *Number inventory* — four differences, each adjudicated and deliberate:
   "1 trillion" written as "one-trillion" (3:14); "NVLink five" written as
   "NVLink 5" (23:14), matching the slide's "NVLink 5.0"; and a caption stutter
   at 7:50 where the speaker restarts the list he is reading, "has some tensor
   0 1 0 1 2 3", collapsed to "0, 1, 2, 3" — which is exactly the tensor the
   source program prints for broadcast, `tensor([0., 1, 2, 3])`. Nothing was
   lost.
3. *Per-paragraph word ratio* — retention 85.3% (9,636 of 11,299 countable
   words). Two paragraphs sit a hair below the 0.72–1.10 band, at 0.70 (45:37)
   and 0.71 (46:23); both were read against the original and are pure filler
   removal, with every substantive clause intact. That is the same signature
   Lecture 6 showed (0.72 and 0.71) for the same speaker.

Two claims in the drafting agent's own report were corrected here after checking
them against the captions: "Grace" and "critical batch size" were both reported
as resting on outside knowledge alone, but both words are present in the
captions ("G stands for a grace", "the critical batch fact uh size"). Only the
capitalization and a stutter collapse were editorial, and the rows above now say
so. Every other restoration's stated evidence was spot-checked and held.

---

**[0:05]** Okay, let's get started. So, welcome back, everyone. Today we're going to talk about parallelism. Remember, last week we introduced how to make a single GPU go fast by writing kernels, and we really looked inside this GPU. This week we're going to talk about how to leverage multiple GPUs to make your code go even faster. So, the picture you should have in your head is something like this. So, for the last week we focused on one of these boxes, where you have your GPU. Remember, you have your high-bandwidth memory, HBM, L2 cache, L1 cache, registers, and a bunch of streaming multiple — streaming multiprocessors.

**[0:51]** So, now the picture gets extended, because instead of having one GPU, you might have four, you might have a thousand GPUs, and those are going to be connected. I'll talk later about how those GPUs are going to get connected. And then you're going to have to figure out how to leverage all of this compute to train models. So, in both cases — meaning both in the single-GPU case and in the multi-GPU case, as we'll talk about today — the situation is similar if you zoom out: the compute, the arithmetic logic units, the tensor cores, and so on, is far away from your data. And far away for a single GPU means all the way over here in HBM. And now, if you have multiple GPUs,

**[1:38]** the thing that you need here might be all the way on a different GPU, and you're going to have to shovel that over somehow. But the same principles are going to be the same, because the game is to orchestrate the computation to try to avoid data transfer bottlenecks. It's very easy to use a ton of GPUs, but it's hard to use them effectively. So, taking a little bit of liberty, we can think about the generalized hierarchy, where at the local level, next to the SMs, is a single-node, single-GPU setting, where you have an L1 cache, shared memory. This was the fastest. And then you had HBM, which we lamented was so slow, but in

**[2:26]** this lecture HBM is going to be considered fast. Now we're going to think about the single-node, multi-GPU setting, where the GPUs are going to be connected via NVLink and NVSwitch. And then, finally, the multi-node, multi-GPU setting, where we have to resort to InfiniBand and Ethernet, depending on what network you have. So, last week we talked about various tricks for improving memory accesses — fusion and tiling: read into shared memory, do everything you can, and then write it back out. And this week we're going to talk about how you can reduce the amount of communication across GPUs by replicating and

**[3:14]** sharding appropriately. So, why do you do GPU — multi-GPUs? The obvious answer is, well, you want to scale, but to put a finer point on it, there's really two reasons. One is that your parameters, or activations, or gradients and optimizer state, don't fit on the HBM memory of a single GPU. So, a B200 has 192 GB. If you're training a one-trillion-parameter model, that's not going to fit on a single GPU. And the other reason is that even if your model could fit on a single GPU, you might want to leverage more GPUs by splitting everything up to train faster. So, sometimes there will be some decisions to be made, because

**[4:01]** you could fit everything on GPUs, but you have fewer cores. But if you spread out, then you're going to have to pay the communication bandwidth. So, that's some calculation you're going to have to do, to figure out how to parallelize. So, just one note here is that so far this lecture is Python — you execute it, and you can just show everything. Now, this lecture, if you run it directly, uses multiprocessing. But when I trace through it, I'm putting in some special single-process mode. So, if you want to see the standard out for this lecture, re- run in the multiprocessing setup, you can click here, and I'll show

**[4:46]** this as we go through the lecture. But just remember, as we step through this lecture, we're not actually doing multiprocessing, because we're just stepping through single lines of code. So, this lecture is going to include two parts. One is we're going to learn about the building blocks of distributed communication and computation, starting with the programming model, talk a little bit about the hardware, start to implement things in torch, which you're going to do on your assignment two. And then the second part, we're going to look at actual training. We're going to look at three types of parallelism: data parallelism, tensor parallelism, pipeline parallelism. Each

**[5:31]** is going to cut up our model in different ways. We're going to do this for MLPs rather than the full transformer, but it's really the core computation is going to be shown here. So, let's dive in. The first thing to talk about is these things called collective operations. Collective operations are these primitives from distributed programming that go back to the '80s. So, the idea of parallel programming is very old. It wasn't invented for LM training, and it's still the case that these primitives are the ones that we use today.

**[6:18]** And here, collective just means that you're specifying a general communication pattern, or a template, across multiple devices, rather than managing point-to-point how this GPU is going to communicate with another GPU. And this is going to be much easier, and the system can do a lot more work for you. So, this is a very tried-and-true interface for doing parallel programming. So, the general setup is as follows. The terminology here is, I find, a little bit strange, but this is standard in parallel programming. The idea is that you have a bunch of ranks, where a rank corresponds to a particular device — in our case a GPU, could be a TPU. But the point is that

**[7:04]** you have, let's say, four ranks here. The world size corresponds to the number of devices, so the world size here is four. So, there's a few operations we're going to go through: broadcast, scatter, gather, reduce, all-gather, reduce-scatter, all-reduce, and all-to-all. And each of these operations is going to specify how this set of ranks, or devices, is going to transfer some amount of data / compute with it to some other set of devices. So, the first three — broadcast, scatter, gather, reduce [Ed: the captions say "the first three" but then name four operations; the next paragraph explicitly calls all four of these "the warm-ups," so "three" is left as heard rather than silently changed to "four"] — are really just warm-ups, I would say. They allow you to get a sense of how these are collective commission [Ed: captions read "commission"; this trails off into the next paragraph's "operations work," so it may be a segmentation artifact at the marker boundary rather than a real word — left as heard since content cannot be moved across the boundary]

**[7:50]** collective operations work, but they're not really going to be the ones that are driving most of training. All-gather, reduce-scatter, and all-reduce are the ones that are going to show up again and again for distributed training of language models. And finally, all-to-all — I'll just mention here — which is important for MoEs, but we're not going to actually spend too much time on this lecture. So, let's dive in with the simplest operation, which is broadcast. In broadcasting, you have a rank zero — it could be any rank, but let's just say, for sake of picking one, rank zero — has some tensor 0, 1, 2, 3,

**[8:36]** and it broadcasts to all the ranks. So, at the end of this operation, we have that each of the ranks has the same tensor on it. That should be pretty straightforward. And this, again, doesn't really show up in the core path of training. Generally, broadcasts are used for initialization, where, let's say, you initialize — a load initial checkpoint — and then you want to broadcast it to all the ranks. So, something that's done once. So, the second operation is scatter, and scatter basically says: I have a tensor at rank zero,

**[9:23]** which is split up into the world size, and I'm going to basically scatter my tensor onto the other ranks. So, rank zero gets the zeroth component, rank one gets this, rank two gets this, and rank three gets that. So, again, this is not directly used, but scatter is an important stepping stone to understanding reduce-scatter. So, as the name implies, scatter just takes a big tensor at one place and spreads it out onto multiple places. And you can see how this might be helpful, because you want all the GPUs you're scattering to, to do some

**[10:08]** local computation on the different parts. So, the inverse of scatter is gather. This should be very predictable. The input is you have a dump for [Ed: captions read "a dump for"; this does not parse and no confirmable replacement is available — left as heard] a bunch of pieces, each of which reside on a particular rank. And then, when you do gather — that's with respect to a particular rank, rank zero — it's going to just concatenate all the pieces together. Again, gather isn't directly used, but it's going to be a stepping stone to understanding all-gather. So, next is reduce. Those of you who do functional programming are probably familiar with reduce — it's exactly the same. The idea is that

**[10:55]** you start in the same starting point as gather, and where the tensor is split up across multiple — well, you have some piece of data on each of the different ranks — and then you're going to apply your reduction operation to all of these and get that in — put that on rank zero. So, in this case, if you do a reduction with sum, then you add these all up, you get six. So, you can think about gather as a reduction where the operation is concatenation, if you will. And, of course, reduce is important to understanding all-reduce.

**[11:41]** So, let me pause there — those were just kind of the warm-ups, just in case people have any questions about what a collective operation is: broadcast, scatter, gather, reduce.

*[Question from the floor: Is this related to broadcasting in NumPy?]*

So, the question is, is this related to broadcasting in NumPy? I mean, I think it's the same — conceptually the same idea, where you have one thing that goes to many things. Like, in NumPy, if you have a scalar, it gets broadcast to a tensor. But the instantiation — this is for collective

**[12:26]** communication, so it's a bit different. So, let's move on to something more interesting. All-gather is what you do to basically — you perform gather to all ranks, not just rank zero. Remember what gather does? It basically takes all the different pieces and puts it on one rank, rank zero. And now all-gather just does it for every single rank. That's what the "all" word is for: all means do it for — all the output to all the ranks — and gather is what you're doing to all the ranks.

**[13:12]** So, this is going to come up a bunch. It's not important that you understand this statement precisely, but later we'll see that each rank holds part of the parameters, and then what you need to do is all-gather the parameters to get the full parameters for the full forward pass. So, in general, as we're doing training, we're going to see a lot of this: gather to do something, and then scatter, and then gather and scatter again. So, reduce-scatter is performing reduce on each dimension and then scattering the results. So, let's say you have four

**[13:59]** devices, and each of them has some vector. And so, when we did a reduce before, we just had 0, 1, 2, 3, and that got reduced to 6. But now, reduce-scatter says that for each component of this tensor, I'm going to do a reduction, and then I'm going to put it on a different rank. So, the first dimension, I'm going to add these up, I get six. Now, for the second dimension, I'm going to add these up, I get 10. For the third dimension, I'm going to process these. For the fourth dimension, I'm going to process those.

**[14:47]** So, where this is going to show up — just to foreshadow things — after the backward pass, when you sum the — what you're going to do is each GPU will be dealing with different data, right? And what you're doing is you need to sum all of the gradients from the different shards, and then you're going to distribute — redistribute — this storage. And then, finally, all-reduce: if you understand reduce-scatter and all-gather, it's basically you do one and then you do the other. So, what this does is reduce-scatter

**[15:33]** the same input as we had before. And remember, in reduce-scatter we had 6, 10, 14, and 18 sit on different ranks. And the all-gather part of all-reduce just puts them all on the same — everything on the same node. So, all-reduce is, in some sense, the easiest to understand. You have a bunch of tensors, you reduce — in this case, sum — and then you replicate them on all the nodes. So, we're going to see this one actually first when we do data-parallel, where we sum the gradients, and then we replicate the full — your parameters.

**[16:20]** So, that's where we're actually going to start. So, maybe just focus your attention on all-reduce. Later, we're going to see how to get to fancier things like ZeRO or FSDP — we need to break the all-reduce into reduce-scatter and all-gather, because then you can intervene and manage things a bit more. But for the basic version, all-reduce is fine. Finally, all-to-all. This one is, in some ways, the most general. You basically specify how each rank sends a particular message to another rank. And so, here's a simple example where you have the same input

**[17:07]** as before. And what this is saying is that I want to send zero — this element — to rank zero, meaning keep it myself. I'm going to send one to rank one, two to rank two, and three to rank three. And then, if I'm rank one, I want to send four to rank zero, five to rank one, six to rank two, and seven to rank three. So basically, the position here is going to denote where — which rank is going to be the ultimate destination. And so, if you look at the output, what happens is that the first rank is going to receive

**[17:54]** everyone who sent everything in column zero, because all these ranks are sending these things to rank zero. Similarly, for rank one, all of the ranks are sending this column to rank one, and so on and so forth. So, this is going to be useful for training MoEs, and the intuition here is that each rank has both a split of the data and also a subset of experts. And basically, you — the key idea of the MoE is that it's dynamic routing. You have to look at your data to figure out which experts

**[18:39]** are — you need to route those activations to. So, it ends up being an all-to-all communication. So, if you look at — if everything were balanced, meaning that every rank sent the same number of bytes to every other rank, then all-to-all, you can think about it as essentially a transpose. If you think about this as a matrix, all you're doing is transposing that matrix. But, in general, all-to-all also handles unbalanced splits. And I'm not showing this here, but you can configure it to send any number of bytes to any other rank. But, in general, you want the splits to

**[19:25]** be as balanced as possible. So, remember Tatsu's lecture, where we had load balancing to make sure that things were as balanced as possible. So, morally, the ideal goal is to have the all-to-all look like this. So, just to summarize — maybe a few helpful tips to remember the terminology, because I just went through quite a few different operations. So, reduce is — well, it's reduce. It performs some associative, commutative operation — could be sum, could be max, could be min. Scatter is the inverse of gather. Scatter distributes, gather centralizes.

**[20:12]** And "all" just means that the destination is all devices. So, that explains all-reduce and all-gather. Let me pause there to take any questions about collective communications.

*[Question from the floor: For operations such as gather and reduce, where we receive these smaller ranks to rank zero — is that rank zero the designated? Is it a particular GPU every time, or can that rank zero change?]*

So, the question is: when you do a gather or a reduce, the target where you write the output — right now I said

**[20:58]** rank zero. You'll see later in the code, you basically specify the GPU ID, or the rank, and it goes there. So, it doesn't have to be determined way in advance, but it has to be determined basically when you execute the call. Cool — anything else?

*[Question from the floor: These are just conceptual building blocks? Or they're not actually things that — are they really? These two actually represent —]*

Yeah, so the question is: are these just conceptual building blocks, or are they

**[21:43]** code? So, I'm showing these right now as just conceptual building blocks, but we'll very quickly see how these are implemented in code. So, before getting to the code, I want to talk a bit about the hardware — in particular, how GPUs are connected, because we already know what's inside a GPU. So, let's talk about networking in general. This is kind of a very classic picture — you can tell from this very old-looking image that this is how computers used — I mean, generally work. So, you have this server, and then you have a bunch of CPUs. There's a PCIe bus

**[22:28]** which you connect things like your — or used to connect things like your mouse and keyboard. And then you have a bunch of GPUs sitting off of them, and you have some RAM, and then this computer is connected to Ethernet, to another computer, and so on and so forth. So, this is a particular setup — it has a particular topology. And GPUs on the same node, you use PCIe to communicate, and GPUs on different nodes, you have to go all the way through Ethernet. So, this is like if you bought your gaming GPU and you had hooked it up with your friend, and he's like, "I'm going to train some big model" — that's what you would have to do. But,

**[23:14]** if you're really serious about training, then things look more like this. This is the picture I showed at the very beginning, where you have the GPU, and there's something called NVLink and NVSwitch and InfiniBand. So, the typical setup is this — and these numbers: eight is typical, but this 256 is made up. So, typically you have eight GPUs per node, and these are connected via NVIDIA's NVLink to an NVSwitch. And just for calibration, if you use NVLink 5, then you're getting 1.8 terabits — terabytes — per second of

**[24:01]** total bandwidth. And remember, HBM for the B200 was eight terabytes per second. So, it's about four — about four x — slower. So, I mean, this is still pretty fast if you think about going between devices, but obviously not as fast as high-bandwidth memory, which is much slower than shared memory or L1 cache. So, basically, NVLink connects to the NVSwitch, which means that, from a programming perspective, you can think about GPUs as connected to any other GPU — you go GPU to any other GPU, and the hardware takes care of transmitting that to the NVSwitch, and then

**[24:47]** the NVSwitch routes it. So, typically, what you will also have is that at some point you can't have NVSwitch and NVLink, and you have to — because what — as your clusters, the number of GPUs grows, then you're going to have to put these nodes into pods, which are connected by InfiniBand. And the way InfiniBand works is that now there's a bit more — so now the GPU doesn't connect directly to another GPU, it has to go through PCIe and goes through this kind of special InfiniBand cable, and you see that the

**[25:34]** the speeds are much lower. And then, finally, if you run out of InfiniBand and you have these huge pods, then you need to connect them via Ethernet. And in Ethernet, you have to go through PCIe, and that actually goes through your CPU, which, as we'll see, is even slower. So, it's analogous to the memory situation: the more nodes you have, then the slower it's going to be. You can't have an NVSwitch handling 100,000 GPUs. So, one note is that this — I mentioned

**[26:21]** alluded to — this bypassing the CPU, which is going to be an important thing from a hardware perspective. So, if you have traditional Ethernet, what happens is that the GPU has to talk to the CPU to get its data copied. So, it basically has to copy the data to this — the CPU has a kernel socket buffer. Here, "kernel" means — not the GPU kernel, but the CPU's traditional notion of a kernel — and then it has to build some network packets, copy to the network interface, and then ship it over. So, this generally introduces a lot of latency. And so, there's this technology called

**[27:06]** Remote Direct Memory Access, RDMA, which allows a GPU to directly write or read from another GPU's memory without using the CPU at all. So, obviously, if you're in NVLink land and NVSwitch land, then you have RDMA. InfiniBand also supports RDMA. So, if you're connected via InfiniBand, then you can directly have GPUs connect to each other without access — involving the CPU. But standard Ethernet does not. There's two notable advancements I will mention is that — NVIDIA has been really pushing the limits on what you can do with larger

**[27:53]** and larger pods. So, they have — for basically the B200s and the B300s — they have something called NVL72, which means that they have these trays of eight GPUs, but have nine of them. And so, basically, at the end of the — you have 72 GPUs that are all NVSwitched into one NVLink domain. And if you remember, the NVLink speeds are very fast. So, normally, if you're — if you're mortal — you think, "Well, okay, I have eight GPUs that are interlinked really fast, and then outside of that, things get slowed down a lot." But if you have a lot of money, you can buy this really fancy hardware, and you can get really fast

**[28:41]** interconnects up to 72 GPUs. And the other thing I'll mention is that — I said standard Ethernet doesn't support RDMA, but there has been progress on the Ethernet front as well. So, there's something called RoCE, RDMA over Converged Ethernet, where the Ethernet actually bypasses the CPU, and this is sort of their answer to InfiniBand. So, InfiniBand generally is very ex- expensive, as is a lot of NVIDIA products. But you can get pretty good performance by using — using

**[29:27]** RDMA over Converged Ethernet. And Meta had some papers showing that they were exploring this. So, Llama may or may not have been trained over Converged Ethernet. So, that's just a brief overview of what the hardware looks like. You have GPUs — they connect via NVLink to an NVSwitch over some domain, maybe eight, maybe 72, and then it's InfiniBand from there. And now, let's talk about how do you program this. So, at the very lowest level, there's something called the NVIDIA Collective Communications Library, NCCL, or NCCL — which translates the collective operations, the all-reduce,

**[30:15]** reduce, broadcast — into the actual low-level packets that are sent between GPUs. So, what NCCL does is, when you use NCCL, it's basically like saying, "I want to all-reduce." And then NCCL goes and figures out what the topology of the hardware is, figures out the path between different GPUs, and then it actually launches the GPU kernels to send and receive data. Because, at the end of the day, remember, everything that runs on the GPU is a kernel. So, there are communication kernels as well that actually do communication with other GPUs. So, we're not going to look too much more into NCCL, but just know

**[31:00]** that it exists. And then we're going to actually go to PyTorch. So, maybe before that — any questions about hardware?

*[Question from the floor: Can you like describe — is there anything like a rack? Uh, can I describe physically a rack and a tray? Like, what a rack is and what a tray is?]*

So, for the NVL72, I'm not a hardware expert, but a rack, I mean, is literally — I mean, I've — you've seen data centers, like, where you have a rack, and each tray is something that has — so, it's — so, G, the

**[31:46]** G stands for Grace. So, there's two CPUs, and each CPU is connected to four GPUs. So, each tray has eight GPUs on it, and they're stacked, and everything is connected to this NVSwitch.

*[Question from the floor: Who is responsible for this one?]*

Yeah, so the question is: what is the difference between RDMA and this? RDMA, you can think about it as more of a digital router. RDMA means that one GPU can read and write from another GPU's memory. And there's

**[32:33]** multiple ways to do RDMA. One is to use NVLink and NVSwitch. And another way is to use InfiniBand. So, InfiniBand and NVSwitch and NVLink are more of the hardware — like what pieces, like what cables are, are, and switch is, are there [Ed: captions read "like what what pieces like what cables are are and switch is are there"; this does not resolve into a clean sentence — left close to verbatim rather than smoothed into invented phrasing]. And RDMA is more operationally, like, what happens when you're communicating. Yeah, for example, there's an advance that —

**[33:18]** RDMA over Converged Ethernet is another way to do RDMA.

*[Question from the floor: Is NCCL optimized for multi-node clusters?]*

So, the question is, is NCCL optimized for multi-node clusters? So, my — so, I don't know the details of how

**[34:04]** they have or have not optimized it. All I can say is that NVIDIA has been optimizing the entire stack for inference and training of these large models, because their main customers are the major providers of language models. So, I would be surprised if they haven't thought of optimizing for those types of workloads.

*[Question from the floor: What happens if you have nine GPUs? How do you distribute the workload across those?]*

**[34:52]** I — I suppose, I guess, it kind of depends on how the nine lands in your setup. For example, a lot of times you'll have — let's say you have eight GPUs per node — so then the ninth one would be on a different node. And if you don't have NVLink connecting them, then that's going to be really bad, because that's going to be one node which is not providing that much compute and also is very expensive to communicate with. But if, let's say, everything were connected by NVSwitch, then it would be much more reasonable. One more question, and I'm going to move on.

**[35:37]** *[Question from the floor: TPUs? So, how is this different from TPUs?]*

So, how to describe TPUs a bit more — I mean, let's see. What can I say quickly about this? TPUs are generally much simpler objects. I'm not too familiar with the details of how — like, what each of these components corresponds to — but maybe we can talk about it offline. So, let's actually get down to some code to take advantage of this hardware. So, PyTorch conveniently has a

**[36:23]** torch.distributed library that provides a clean interface into these collective operations, so you don't have to explicitly think about NCCL. And, in fact, this library also supports different backends for different hardware: if you are on GPUs, then you would use the NCCL backend, and if you are on CPUs, then there's something called gloo, which still allows you — like I said, parallel processing has been around for a long time before GPUs, and you can do these collective operations on CPUs as well. This library also supports higher-level models and algorithms such as FSDP, but we're not going to use

**[37:08]** those in the course, because we're building things from scratch. All right, so let's walk through some basic examples of collective operations. So, this function called spawn, which takes another function I'm going to call, and it says, "I'm going to run this replicated four times, where four is the world size." So, let's see what this does. This is actually a wrapper I wrote just to hack around the fact that I can't do multiprocessing in this lecture. So, normally, what you would do is you call torch.multiprocessing.spawn, then you call the function. But you

**[37:56]** know, I'm going to do this branch, which dis- disables distributed. So, let's just go through that. So, now I'm in this function that's supposed to be running asynchronously for each process. So, remember, world size is the number of processes, and the rank is either zero, or one, or two, all the way up to world size minus one. And so there are world-size number of these functions that are — each running on a process at the same time. So, I'm on rank zero right now. So, what do I do? There's a setup — you basically configure the master address and port. Notice that this is not actually how the

**[38:44]** the GPUs are going to communicate. This is more for just general metadata and coordination — data goes through NCCL, otherwise it'll be very slow. And if you have CUDA available, then you can use the NCCL backend. I'm on my laptop, so I'm going to use the gloo backend. So, now I'm here — I'm running, pretend I have four of these processes running. So, there's this barrier function, which is useful, as it's a synchronization barrier. So, basically, if I see this, then it waits for all the processes to get to this point.

**[39:30]** So, basically, you can think of all the processes as running asynchronously. So, I don't really control — one could completely finish before the other. They might finish — be interleaving anyway. So, if I want to make sure that — certain — there's some code that is executed before other code, I put these synchronization barriers in. So, now the downside of putting more barriers in is that, well, you end up waiting potentially unnecessarily. So, let's try an all-reduce. So, I'm going to create this tensor 0, 1, 2, 3. And I have my rank. Just to make it more interesting, each rank is going to

**[40:17]** have a different tensor. I'm going to print out what I have before the all-reduce. So, now I'm going to skip over here and see what gets printed out. So, rank zero before all-reduce has 0, 1, 2, 3. Rank one has 1, 2, 3, 4, and so on — it's the same example as I showed before. Notice that the print statements are coming in whatever order the hardware feels like, because it's running async, but all the data is there. So, now, if I do an all-reduce — this you pass in — this is a PyTorch function. It passes in this tensor. You pass in the reduction operation, which is a sum.

**[41:04]** And I'm going to say don't do it async. And what this does is it calls — in this case it would be gloo, but it could be NCCL — and, in which case, it would spin up the CUDA kernels. It would do the communication — it takes care of everything for you. And then it basically writes in place to the data. So, after the all-reduce, I have the all-reduce operation — remember, which is the sum of each of these columns, but replicated across all the ranks. So, that's all-reduce. And if you want to do — be fancier — and do async, then you could say async

**[41:50]** equals true. But then it would screw up all these print statements. So, I'm trying to put more barriers than I normally would.

*[Question from the floor, inaudible]*

So, the question is the rank of the GPU. For this class, the rank is the GPU. Yes. So, let's try another example here — I'm going to do a reduce-scatter. So, here I'm going to create an input which is zero through the world size.

**[42:36]** And I'm going to have the output — I'm going to allocate the output. And so, what does this look like before I do the reduce-scatter? It looks like — let's see, this — where each — it's kind of the same input, and the output happens to be zeros, but it could be just anything. And then I do the `reduce_scatter_tensor`. Here, instead of writing in place, I have an output tensor and an input tensor. I say I want to do the sum. And then, afterwards, I get — basically, the input is not touched, but

**[43:21]** the output, I get the reduction of each component written into the respective ranks.

*[Question from the floor: How does it work to do the all-reduce asynchronous?]*

So, what this means is that this is sort of like a monolithic operation — you say, go do the all-reduce, and it's spinning up kernels. It's going to

**[44:07]** do the communication. And remember, CUDA is already async with respect to the processes, and now we have all the processes being async. And the point is that this code just would return, and then you could do other things. So, a typical thing — which I'm not going to talk about this class — is overlapping computation and communication. So, for example, you can do this operation, and then you can go ahead and load some other data for the next step, which is independent of this operation. And then, when you want to make sure that

**[44:52]** you actually are done, then you can call wait, or a barrier. So, let's do the final one, which is all-gather. So, by now, I think you get the idea. Here, I'm going to set as the input the output of the reduce-scatter. I'm going to allocate an output. So, before the all-gather, it looks like this: I have my result from the reduce-scatter here. The output is — it just allocated, happens to have some values in it, but don't worry about it.

**[45:37]** And then, after I do the `all_gather_into_tensor`, I will have all the different inputs gathered onto all the different ranks. And you can see here that, indeed — proof via example — that all-reduce is equal to reduce-scatter plus all-gather. And then, just to wrap things up — just as I started by a setup, then I clean up — which

**[46:23]** it's good practice to clean up. So, that was your first example of a torch.distributed program. So, let's do some benchmarking — this will be quick, since I want to actually move on to part two. So, how fast does communication happen? Let's do an all-reduce. So, here I'm going to all-reduce with 100 million elements. And — oops. Let me — sorry, mess this up.

**[47:14]** So, all-reduce — so, I'm going to create this tensor with this number of elements, call — and remember, just like before, when we do benchmarking, we warm up first. And here I'm going to call the CUDA synchronize, and also the barrier, just to make sure that — because there's two forms of asynchrony here, the CUDA kernels and the different processes — everything is not running in the — is done, before I start the time. And I'm going to do the all-reduce, and then wait again with the synchronizing, the barrier, and stop the

**[48:00]** time. So, remember, this is running for every single rank. So, if I look at the output here, for rank 0, 2, 1, 3, I have a different time, potentially, because they're all different processes. Each of them is going to report a certain measurement, and if you want to report one number, you can take the average, for example. So, now, one thing that is useful to do, which is analogous to when we were computing MFU, is to compute — measure — the effective bandwidth. And the idea here is that, well, this took 1.6 milliseconds.

**[48:47]** How much — is that good or bad? So, to compute the effective bandwidth, what we're going to do is compute essentially how many bytes were sent, and should be sent, during this computation. And then, if you divide by the total time, then you get the bad [Ed: captions read "bad"; does not fit grammatically or logically here ("you get the bad effective bandwidth") — left as heard rather than silently dropped] effective bandwidth. So, the size of what I'm sending around is the size of each element times the number of elements, so that's basically the number of bytes of this data tensor. How many bytes actually get sent? This needs some unpacking. So, for all-reduce,

**[49:33]** if you think about — let's just say, for simplicity, you do rank 0 plus rank 1 plus rank 2 plus rank 3 — you need to iterate this world-size-minus-one steps, because there's world-size-minus-one addition operations. So, that's this factor. There's a two, because you need to both send and reduce. And then you multiply by the size of the payload — so, that's the total number of sent bytes. And the total duration is the — the time that the — wall-clock time that it took. And then you multiply by the world size, because it's like the total amount that all the ranks have

**[50:20]** waited. And the bandwidth is the bytes sent divided by the total duration. So, in this case, you get something like about 400 GB per second. So, a few notes here. One is that the effective bandwidth — if you look at this expression, size * 2 * world size - 1 / world size * duration — so, as world size increases, this world my size - 1 / world size [Ed: captions read "world my size"; "my" does not parse — likely a stray syllable or mis-segmentation, but not confirmable, so left as heard] essentially converges to one. So, you're effectively left with two times the size bytes over the duration.

**[51:05]** So, this is essentially the bandwidth. Notice that this is independent of the world size, which is good. So, if you grow the number of GPUs you have, the bandwidth doesn't change. It is also independent of the topology, which is something that NCCL figures out — whether you're going to pass the messages in a ring or a tree topology. So, that is all-reduce. And for reduce-scatter, this is very similar — so, I'm going to create the inputs and the outputs, warm up, perform the

**[51:50]** operation, and time it. So, notice that the reduce-scatter has these timings, and you can also measure the effective bandwidth here. The number of bytes that were in the input — you have the number of bytes that were sent. Here, there is no 2x. And then the total duration — you divide by that, you get the bandwidth. So, here, the bandwidth — it should be very similar. I guess, sometimes, there's some stochasticity, but it's in the 400s.

**[52:36]** So, a few notes here. All-reduce is, remember, as we stated, reduce-scatter plus an all-gather. And so, all-reduce naturally is moving twice the amount of data, because reduce-scatter has some cost, all-gather has some cost, and all-reduce is doing twice as much work. But it takes twice the amount of time. But the two cancel out, so you get the same bandwidth. All right, so that is the end of part one. Maybe

**[53:24]** any questions before I move to part two?

*[Question from the floor: Why do we have to do the synchronize for the CUDA kernel?]*

So, at the end of the day, we are still doing CUDA operations — we have just multiple processes, each with a GPU, doing some CUDA operations. And then, so, if you're doing a CUDA operation, remember, by default, it's async. So, when you reach the next line in Python, that CUDA operation might not be done. So, we need to always wait until that's

**[54:10]** to make sure it's done, by synchronizing.

*[Question from the floor: Do you have to do barrier first and then synchronize?]*

Not sure — yeah, I think one problem is, if you barrier first, then the CUDA might not be done running, and you just immediately go to the barrier. And then, you are still each independently synchronizing the different CUDA kernels.

**[54:56]** Which means that you're not really synchronized — like, the barrier might, if all those operations just return, the barrier doesn't really do anything. Let me move on. So, now let's actually start thinking about how you train models. So, we're just going to walk through a very bare-bones implementation of training MLPs — multi-layer MLPs. I guess that's redundant, it's multi-layer perceptrons already. And remember that MLPs are the ones that are the actual compute bottleneck in a transformer, so this is actually pretty representative of what you'll see. So, data parallelism, tensor parallelism, and pipeline parallelism. And the picture I want to

**[55:42]** you to have in your head is this picture. So, this is a little bit of a schematic, so don't think too deeply about this, but it's more of a way to conceptualize how you're cutting your data and parameters. So, data parallelism says: "I'm going to split the data into pieces, and then each of the GPUs is going to be responsible for part of the data, and I'm going to just do normal — I'm going to act — keep track of all the parameters and do normal model training." And then I need to synchronize. So, let me explain how this works. So, I'm going to generate some sample data — there's a batch size of 128, number

**[56:29]** of dimensions, 1,024. So, this is a batch-size-by-num-dim matrix — data matrix. And let's jump into this data parallelism. So, the way that it's going to work is that you have this data matrix, and I'm going to break up the rows into a bunch of — into num-world-size pieces, in this case four — and each rank is going to get a piece. So, the number of dimensions, the batch size — so, each, I'm going to call the local batch size, basically

**[57:17]** the batch size divided by the world size, which is — every row is going to have — that GPU sees is going to have 32 data points. And this is just indexing: start index — data, start to end, gets you the slice of that data, and I'm going to just put it on that rank. So, at this point, each GPU has now a distinct data tensor, which is the part that they're responsible for. Now, in practice, each rank should probably load its own data, rather have this bottleneck, but this is just for illustrative purposes.

**[58:04]** So, let's instantiate the MLP. So, here, I'm — assume we have num-layers — and for each of the layers, I'm going to have just a num-dim-by-num-dim matrix. So, I'm just going to initialize a random set of parameters, and then I'm going to feed that into the optimizer. So, here's the training loop. So, in the forward pass, I take the data — remember, data is not all the data, it's just — if I'm rank two, then I only get the B2 part of the data.

**[58:51]** I'm going to just go through the number of layers and do a forward pass. And then I'm going to do a backward pass. And then, normally, this would be it. But now, remember, every rank has different data — therefore, the gradients are going to be different as well. So, this is the key step that makes data parallelism work: we're going to synchronize the gradients across all the workers. This is the only difference between standard training and DDP. It's actually pretty nice and elegant. So, basically, for all the parameters, I'm going to do an all-reduce of `param.grad`.

**[59:37]** And I'm going to average. And then, after this all-reduce is done, then, at this point, each of the ranks has the exact same gradients. And then I'm just going to update the parameters. So, it's really elegant, I find, because it's basically standard training where you apply it to your local batch, but you just insert this after the backward pass — let's just average all the gradients via this all-reduce. It's a one-line code change. And then the parameters get updated. So, as you're training,

**[1:00:24]** each rank is basically performing parameter updates as if it were — have all the data on it, but it's only actually processing a part of the data. So, that's basically DDP, or the first type of data-parallel. Any questions about this?

*[Question from the floor: Can you only do this with batch size greater than one?]*

Yes. So, your batch size has to be at least world size for this to really make sense, and usually it should probably be quite a bit larger.

**[1:01:12]**

*[Question from the floor: Should the batch size be a multiple of world size?]*

And that also would be nice — yes. I mean, if it's not, then you can pad it with zeros or something, so there's ways, but it's just easier for everyone if it is.

*[Question from the floor: What would this look like for a transformer?]*

It would actually be basically the same. DDP has a nice thing that is very modular — you do the forward pass. I mean, DDP just averages the parameters

**[1:01:58]** here — it doesn't care what your forward pass looks like. Let me move on. So, that's DDP. So, just to summarize: the losses are different across the ranks, the gradients are also initially different, but they're all-reduced to be the same across the ranks, and therefore the parameters all remain the same across ranks. So, next lecture, Tatsu is going to talk about fancier data parallelism, FSDP and ZeRO, and the idea there is, as I've alluded to — here we use all-reduce.

**[1:02:44]** It's a very simple monolithic operation, but it does require holding all the model's parameters in memory. But what if the model parameters don't fit in memory? Then you're going to have to be more clever, and that's the topic for next class. So, let me talk about tensor parallelism. So, here, the idea is we're going to cut this way — I mean, we're not going to cut the data, we're going to cut, essentially, each layer. And so, each rank is going to get part of each layer. And, generally, this rem- means that we're going to have to transfer a lot more data. We're going to discuss this a bit later.

**[1:03:31]** So, what does tensor-parallel look like? So, we're just going to assume we have this data — every rank has all the data, just for simplicity. And here, remember, the data is batch-size times num-dim. And I'm going to define a local-num-dim to be — for this rank, I only am responsible for a subset of the dimensions. So, the picture here is that each model still has all the layers — here, for all the layers — but the

**[1:04:17]** parameters now are num-dim times local-num-dim. So, if this were one of the parameter matrices for one layer, I would be doing — splitting down the columns. So, this is also known as column tensor-parallel. You can also do it by rows, but we're not going to talk about that right now. So, now what does the forward pass look like? So, go through all the layers, and I'm going to compute activations. So, I start with X, the data. And I'm going to access the parameters

**[1:05:03]** at layer — layer. And notice that this is only a slice of the — right? So, if I'm on rank one, I only get this part of the matrix. But I can still proceed — I can apply this nonlinearity, because this is elementwise anyway. But now, what I'm going to do is communicate activations. So, if I have a data matrix, rank one has activations for part of the activations for this part of the matrix — rank one has part of the activations for this matrix, and so on and so forth. And I need to basically put all the activations on all the

**[1:05:50]** the ranks. But we know how to do that — we introduced all-gather as a collective primitive. So, I have — this is all the activations. So, this is the batch size times local-num-dim, which is the shape of the activations. And then I'm doing an all-gather. Sorry — so, this is allocating memory for the activations. So, X is the actual part of the activation, so this is batch size times local-num-dim. And all-gather says each rank has X, and

**[1:06:36]** each rank is going to allocate activations, which is a list, one for each world size. And then, after the all-gather, X is going to be copied into each of the respective location activations. So, then, once I gather all the activations, I concatenate them to form the full-dimensional X, which is batch size times num-dim. Any questions about column tensor-parallel? So, this is done for every layer.

**[1:07:21]** We've — now, notice, one difference between — data-parallel is now we have to muck around with the model. Data-parallel is very elegant, because it's splitting by data — the model is treated as a module. But now we have to muck around with a model. And this is strongly leveraging the fact that if you want to do a matrix multiplication, you can split it up into a set of small matrix — smaller matrix multiplications. You can do those on different ranks, and then we can gather the results.

**[1:08:08]**

*[Question from the floor: What happens in backprop?]*

Now, in the backprop, you have your activations, and you have to reduce-scatter all the different gradients. So, in some ways, all-gather and reduce-scatter have this duality, where, in forward, if you're all-gathering, in the backward you're reduce-scattering.

*[Question from the floor: Is that done automatically by autograd?]*

So, none of this — well, okay, so if you just call `.backward()`, it's not going to do it, because there's no parallelism in that. But PyTorch has all these things that are done automatically for

**[1:08:54]** you. So — um — so, in this — so, what hack is done for U and versus automatically? [Ed: captions read this way; it reads as garbled crosstalk, possibly between the lecturer and a student, and does not resolve into a clean sentence — left close to verbatim] Here, we're managing things fairly explicitly, which means I'm not doing the backward pass, but you would have to manage and call the reduce-scatter yourself. And that's baked in by design, because this is 336 — building language models from scratch. In practice, you probably wouldn't have to do that. So, let's do pipeline parallelism

**[1:09:41]** quickly. So, the idea behind pipeline parallelism is we're going to split the network this way. So, each rank is going to get a subset of the layers now. Within each layer, it's going to get all the dimensions. And it's also going to get all the — well, one of the ranks is going to get all the data. It — every rank is going to see all the data in some form. So, this is the way it's going to work. So, let's — we have all the data, which is again the batch size times num-dim. And I'm going to split up the layers.

**[1:10:26]** So, local-num-layers is going to be the number of layers that a particular rank is going to handle. And so, I'm now — I'm going to have local-params, which is basically — there's local-number-of-layers number of them. But, within each layer, I'm going to do the num-dim by num-dim. So, one thing — I — maybe Tatsu will talk more about this on Wednesday — is this idea of micro-batches. So, maybe I'll just present this and explain why I'm doing things this way. So, in addition to splitting up the layers, I'm also going to split up the batch into a bunch of micro-batches.

**[1:11:13]** So, if I'm rank zero, then I get the data, and I'm going to chunk it up into a number of micro-batches. And for each micro-batch, what I'm going to do is receive it from the previous rank, and then do the feed forward pass only on the layers that are assigned to this rank. And then I'm going to send to the next rank. So, here, I'm actually using these receive and send, which are point-wise operations — I — so, I didn't cover those before, but they're fairly

**[1:12:01]** explanatory. This basically says: I'm rank — and I'm going to send this tensor over to — oh, sorry, I'm going to receive this tensor from rank minus one. And this says I'm going to send tensor X to rank plus one. So, basically, the reason why I'm talking about micro-batches is — and Tatsu will talk more about this on Wednesday — that in pipeline parallelism, you have one rank that gets the data. It processes some of the

**[1:12:47]** layers, and then it sends it to the next GPU, and it processes some of the layers, and then it sends it to the next GPU. So, this is a very natural way of dividing our deep network, but the problem is that you get these what are called pipeline bubbles, where, while you're not processing, you're waiting around for other tensors to process. And this ends up being quite inefficient. So, the idea behind micro-batches is that you break it up into smaller batches, so you can process it quickly, send it on to the next one. So, this can reduce the number of pipeline bubbles.

**[1:13:32]** So, the other thing I will mention that is not handled in this naive version is the idea of overlapping communication and computation, which is actually very important to pipeline parallelism. This basically gives you the right structure. If you put an "i" before these [Ed: captions read "if you put a I uh, before these" — read here as referring to prefixing send/recv with an "i" (i.e. `isend`/`irecv`), the standard torch.distributed async point-to-point calls; this reading is context plus outside knowledge of the PyTorch API, not confirmed by the slide file, which does not mention isend/irecv], then it becomes async. You have to add more things to manage the code. And the idea here is that, while you're computing here, you can be receiving data or sending data. So, computation and communication should be overlapped, so that reduces the amount of time you're actually spent

**[1:14:18]** waiting. So, a few things that are missing here, which will hopefully fill in next time. So, the communication-versus-computation overlap, which is especially crucial in pipeline parallelism. I didn't mention, in data parallelism, this also happens, because I just did a full repass [Ed: captions read "full repass"; likely refers to the full forward-and-backward pass running to completion before any communication starts, but the exact intended word is not confirmable — left as heard] and then, at the end, I'm just doing all these all-reduces. But if you're clever, then, on the backward pass, as soon as the gradients are done, you can start sending that. And that's something you'll be exploring in assignment two. And this, again, allows you to just

**[1:15:05]** overlap communication and computation more. We had some questions about — what about general models? Again, I think this MLP gives you essentially all — mostly, most of — what you need for understanding the basics. Some of the larger models just require a lot more bookkeeping, so it's harder to see the core algorithms. There are other types of parallelism that we haven't covered: sequence parallelism, which takes a whole sequence and chops it up into pieces, and that allows you to parallelize attention computation; expert parallelism, which

**[1:15:51]** allows you to parallelize the experts for MoEs, and this is where the all-to-all that I mentioned comes in. And then, also, different combinations of different parallelization techniques, which also will show up in the assignment. So, one thing to note is that which parallelism technique you choose is going to be strongly dependent on the hardware. For example, tensor parallelism — there's a lot of communication, because, for every layer, you need to send all these activations, which are fairly big. So, generally, tensor parallelism happens within a node, on NVLink or NV—

**[1:16:37]** where you have high bandwidth. Whereas you wouldn't do tensor parallelism the kind of an NVLink domain [Ed: captions read "you wouldn't do tensor parallelism the kind of an NVLink uh, you know domain"; a preposition seems to be missing (e.g. "outside of," "beyond" an NVLink domain), but the exact word is not confirmable — left close to verbatim]. Whereas pipeline parallelism — you'll see people using it, and, generally, this can tolerate much slower interconnects. So, some of the decentralized training work uses pipeline-parallel, because your nodes — your GPUs — are actually across, halfway across the world. But you wouldn't want to do tensor-parallel in that setting. So, sometimes, when you look at these combinations, it'll be tensor-parallel within a node, and then

**[1:17:22]** and then pipe — data-parallel, or FSDP — and then pipeline parallelism, if you need it. There's other effects, such as: if you do data-parallel, you might be able to do data-parallel quite a bit, but then you start hitting something called the critical batch size, where, if you start increasing the batch size too much, it doesn't actually help you — in which case, you're just wasting your compute, and then you're better off using tensor-parallel. So, there's a bunch of these considerations, which we'll talk more about as we go through the class. So, final note — TPUs came up a little bit.

**[1:18:07]** One thing to note is that, on purpose, we are using PyTorch — and not just using PyTorch, but really using the collective operations in a very primitive way, so you can see mechanically what's happening. Another approach, especially if you're in Jax-and-TPU land, is that you can simply define the model and the sharding strategy, and the compiler actually handles a lot of the decision of — basically, what kind of communication operations you need. You basically say, well, this piece of data needs to be here, and here, and here, and then the

**[1:18:54]** compiler does some magic to figure out what. So, that's appealing, but, obviously, it would take a lot of the joy out of actually building things from scratch. Just to summarize: there's many ways to parallelize. You can cut by data, cut by tensor, or expert, cut by pipeline, or sequence. We looked at data parallelism — we only did DDP; next time we'll do FSDP and ZeRO. Tensor parallelism, as I mentioned, requires very fast interconnects. Pipeline, less so, but you need to really work hard to

**[1:19:40]** reduce these pipeline bubbles. And then, maybe, at a high level, we see this pattern come up a lot. So, you can either recompute, or store in memory — when we were talking about things like activation checkpointing, or when we're working with GPUs. Or, in this case, you can think about an extension of this: that you can store on a different GPU. From that perspective, you look at data-parallel — you're doing redundant work, in some sense, because every rank is actually updating its parameters and keeping track of all the parameters. But the reason you're doing

**[1:20:26]** that is that you don't have to move the optimizer state across. So, one thing is that hardware is getting faster, but, in some sense, we'll always want bigger models. So, this idea of having a hierarchical structure will always be there. So, that's it for today. Next Wednesday, Tatsu will do more of a deep dive on more parallelism techniques.
