---
title: "Lecture 6: Kernels, Triton"
lecture: 6
video: https://www.youtube.com/watch?v=xnDHaNUvHBg
source: copy-edited from the YouTube auto-captions
verbatim_original: original/06-kernels-triton.md
material: ../slides/06-kernels-triton.md
---

# Lecture 6: Kernels, Triton — transcript

**This is the edited transcript.** The auto-captions have been repunctuated,
segmented into sentences, stripped of filler ("um," "uh," "you know," "sort
of," "kind of," "right?"), and had mis-heard technical terms restored. No
content was added, removed, or reordered, and every `[MM:SS]` / `[H:MM:SS]`
marker is preserved in its original position — content that was under a
marker is still under that marker. The verbatim captions are kept at
[`original/06-kernels-triton.md`](original/06-kernels-triton.md); that file
is the record of what was actually said, and this one is the readable
version of it.

**How that was checked.** Three mechanical comparisons against the verbatim
captions, the same three as lectures 4 and 5. The 113 timestamp markers match
exactly in sequence and form, `[MM:SS]` and `[H:MM:SS]` alike. The inventory of
numbers in the two bodies was diffed: nothing was lost, and the single addition
is the `run_operation2` restoration recorded below. Every paragraph's word count
was compared with its original to catch content drifting across a marker
boundary; retention is 82.9% (10,928 of 13,181 words), and the two lowest
paragraphs — 25:24 at 0.72 and 50:55 at 0.71, both a hair under the usual band —
were read against the originals and are pure filler removal, this speaker's
"you know" appearing ten times in the first of them alone.

One note for anyone re-running that third check: **a paragraph here is
everything from one marker to the next, not the single block the marker
starts.** This transcript lifts each student question onto its own line for
readability, so a checker that stops at the first blank line reports wild false
outliers — it produced fifteen of them, including a 0.00, before the splitter
was fixed.

Terminology was cross-checked against
[the lecture's source program](../slides/06-kernels-triton.md)
(`lecture_06.py`), which is the authority for GPU hardware terms, Triton API
names (`tl.load`, `tl.store`, `tl.program_id`, `tl.arange`, `tl.dot`,
`tl.constexpr`, `triton.cdiv`, `@triton.jit`), PTX register names (`%ctaid.x`,
`%tid.x`), library names (CUTLASS, Triton, PTX, HBM, SM, SMEM, CTA, GeLU,
ReLU, softmax), and numbers.

**Restorations made.** These are places where the captions produced a *wrong
word* or dropped a word, and the text now reads differently. Each was
confirmed against [the lecture's source program](../slides/06-kernels-triton.md)
unless the note says otherwise.

| Caption | Restored | Confirmed by |
| --- | --- | --- |
| "we've had multiple generations from M100s to H100s to B200s" | A100s | slide (the hardware table's columns are "A100, H100, B200"; there is no M100 generation) |
| "there's H100s and B200s also have thread block clusters... enable some amount of distribution memory" | distributed memory | slide ("H100s and B200s also have thread block clusters that enable distributed shared memory") |
| "it's like FI for I equals you know, ranging over your data set" | f(i) for i equals | slide ("each thread processes one element — $f(i)$ for $i = 0,\ldots,N-1$") |
| "each bank can only be accessed by one thread, at most one thread. Assuming that's not a location" | not the same location | slide ("Each cycle, each bank can only be accessed by one thread (if not the same exact location)") |
| "the reason I'm doing benchmarking and profiling as opposed to teaching you try and earlier" | Triton | context — this is exactly the kind of Triton mishearing this lecture's own caption stream produces elsewhere, and only "Triton" makes sense as the thing not yet taught at that point in the lecture |
| "in your assignment you're using site which gives you more details" | nsight | slide ("In your assignment, you will use nsight to get more details") |
| "SM100 corresponds to the Blackwall you know, architecture" / "specifically designed for Blackwall" (2 instances) | Blackwell | slide ("sm100: corresponds to the NVIDIA Blackwell architecture (B200)") |
| "let's start with a the value example here... Triton value kernel" (3 instances, 39:18–42:24) | the GeLU example / triton_gelu_kernel | slide — the numbers that follow (an "8,000"-element vector, block size 1024, "eight blocks") match `triton_gelu_kernel`'s actual parameters (`num_elements = 8192`, `BLOCK_SIZE = 1024`, `num_blocks = 8`) exactly; no other kernel in the lecture fits |
| "this is the you know, the Triton um you know, library A range zero block size" | tl.arange | slide (`offsets = start + tl.arange(0, BLOCK_SIZE)`) |
| "you do TL store Y pointer plus offsets on Y and and the mask" | tl.store | slide (`tl.store(y_ptr + offsets, y, mask=mask)`) |
| "I'll get to that TF.load" / "Now that the TF.load is done" (2 instances) | tl.load | slide (`x = tl.load(x_ptr + offsets, mask=mask)`) |
| "here the CTA.x is the block index, and TID.x is the thread index" | %ctaid.x | slide ("`%ctaid.x` is block index, `%tid.x` is thread index") |
| "now you know, warming up to the mammal, suppose that your row doesn't fit into a block" | matmul | context — the sentence is explicitly building toward the matmul section that follows |
| "There's you know, ThunderKittens, there's uh you know, cute you know, various DSLs" | CuTe | context **plus outside knowledge** — grouped with ThunderKittens as a named low-level GPU DSL, which is what NVIDIA's CuTe (part of CUTLASS) is. The slide never mentions either library by name, so the specific identification of "cute" as **CuTe** rests on knowledge of the library, not on anything in this lecture's material. Treat as an editorial judgement. |
| "I think Tatsuo showed this as well" | Tatsu | context — the lecturer is named "Tatsu" everywhere else in this transcript (four other instances); "Tatsuo" is an isolated caption variant of the same nickname, not a different person |
| "Again, run operation creates two random matrices" (26:57) | `run_operation2` | source program — this is the one place the captions give the name with no "two" at all, so the digit is supplied rather than re-cased. The sentence itself says the function "creates two random matrices", which is exactly what `run_operation2` does and what `run_operation1` does not; the same call at 23:07 is spoken in full as "run operation two". This is the single added numeral flagged by the number-inventory check. |

**Capitalization and formatting only.** The following were already the
correct words in the captions and were only cased, joined, or given the
punctuation of their code form to match the source program's own usage —
they are not restorations: "GELU" → "GeLU" and "relu" → "ReLU" (slide's own
casing throughout), "Nvidia" → "NVIDIA" (slide casing), "element-wise" →
"elementwise" (slide: "elementwise operations"), "run operation two" →
"run_operation2" where the captions do say "two" (the actual function name,
matching the lecture-5 precedent of "max auto tune" → "max-autotune"; the one
place they do not say it is listed as a restoration above), "NN functional GELU" →
"nn.functional GeLU" (PyTorch's actual call), "FP 32" → "FP32", "the program
ID is is basically the PID" kept as spoken prose (matches `tl.program_id`
conceptually; not converted to code form since it's describing the concept,
not naming the call).

**`[Ed:]` notes inserted, and why:**

- *At 8:33*, "Technically, you can also exceed the warps" — "exceed" does not
  make sense in context (you cannot "exceed" a warp), but no confirmable
  replacement (e.g. "access," "address") is given by the slide or context;
  left as heard.
- *At 16:15*, "memory coalescing, which talked to you talked about" — reads
  like a garbled reference to an earlier explanation (possibly Tatsu's, from
  lecture 5), but the words don't resolve to a clean sentence; left close to
  verbatim.
- *At 17:01*, "combined into a transaction of 20 to 128 bytes" — the slide
  states only "128 bytes," with no range; "20" is not confirmable as a real
  qualifier (32-byte and 64-byte transactions also exist on NVIDIA hardware,
  so "20" is very plausibly a mis-hearing of "32," but that is a guess, not a
  restoration); left as heard.
- *At 28:30*, "there is this long name... Um color F32 F32 64 64 16" — "color"
  does not parse as a word in this position (presumably part of the CUTLASS
  kernel name quoted a few lines later in the source, e.g. "cutlass3x_..."),
  but no confirmable reading is available; left as heard.
- *At 47:51*, "what is actually happening when this uh um the same end is
  executed" — "the same end" does not parse; likely refers to the kernel or
  code just discussed, but no confirmable replacement; left as heard.
- *At 50:55*, "PTX code... is this intermediate assembly language for new
  GPUs" — the slide says only "an assembly language for GPUs," with no "new."
  "New" may be a mishearing of "NVIDIA," but that is not confirmable; left as
  heard.
- *At 49:24*, a passage beginning "and this is like so when this is invoked,
  it's not really clear..." is flagged inline as garbled crosstalk between
  the lecturer and a student, in the same spirit as lecture 5's two
  "exchanges left close to verbatim" — it is not clear where the student's
  words end and Percy's resume, so the passage is left close to verbatim
  rather than split and attributed.
- *At 36:13* and *53:13*, short leading fragments ("Uh is CUDA kernel What
  uh" and "it makes this for each individual So, all of these are So,
  probably the same one for the whole thing") are marked as
  `*[Question from the floor, inaudible/garbled: ...]*` — they are clearly
  the tail end of a student's question run into the transcript by the
  auto-captions, but too garbled to render as a clean sentence without
  inventing wording.

**False starts are preserved, not completed** — e.g. "com— compilation" at
33:53, "65 — 64 warps" at 13:12, "400 — 4,000 columns" and "cons—" at 1:05:39,
"T— TID.x" at 54:01, and the self-correction "semantic, not semantic,
streaming multiprocessors" at 6:15 (the lecturer's own on-air flub and
correction, not a caption error, so both words are kept).

**Student questions.** The auto-captions run student questions together with
Percy's speech with no speaker labels. Wherever the recording makes clear
that someone from the floor is asking something, the transcript marks it
`*[Question from the floor: ...]*`, using the captions' own words with only
light punctuation — never rewritten or completed. Several questions are cut
across a `[MM:SS]` marker boundary by the recording itself; where that
happens, the quote is split at the same point and the continuation is marked
`*[continued: ...]*` in the next paragraph, exactly as the marker boundary
requires.

---

**[0:05]** Welcome back, everyone. On Monday, Tatsu did an excellent job of describing a high-level overview of GPUs and how to think about performance and all the quirks that come with GPUs. This lecture is going to be a continuation of that, where we're going to dive more deeply into the code, write some Triton kernels, and do some benchmarking and profiling. Just to start, just to refresh what's going on with GPUs, here is a simplified diagram of what a typical GPU looks like. There's memory, and then the actual GPU chip. And

**[0:50]** just to keep in your head what are the characteristics of a GPU. Of course, we're talking about NVIDIA GPUs, but Tatsu also talked about TPUs, and there's AMD and so on. But focusing on NVIDIA GPUs, we've had multiple generations from A100s to H100s to B200s. And in each generation, you have a GPU — it has a number of streaming multiprocessors, or SMs. And the number of SMs is about 100, between 100 and 200. That hasn't really changed that much. And then within an individual SM, there's a set of

**[1:35]** registers. B200s have 65,000 registers, for a total of 256K per SM. This also hasn't changed that much. In addition, there's an L1 cache as well as shared memory — remember, these are the same memory. Shared memory you can control; L1 you can't. This is per SM, and it's in the same ballpark size. And then there's an L2 cache — this is not per SM, this is for the whole chip, and it's a bit larger. And then finally, you have a high-bandwidth memory, HBM, which is large. And you see

**[2:23]** that this is the number that's actually going up quite a bit. In addition to size, you can think about what's happening with the bandwidth, and essentially it's inversely correlated: registers are very fast, L1 is slightly less fast, L2 is less fast, and high-bandwidth memory is the slowest. Although 8 terabits — terabytes — a second is still not that slow in the grand scheme of things. So this is the main hierarchy you should have in your head: large memory is slow and far, but big, and fast memory like registers and L1

**[3:10]** resides on the SM — it's local and it's fast, but small. That's the environment we're dealing with. So then, how do you program a GPU? The programming model is as follows. There's a notion of threads, where each thread executes a piece of code on a small part of the data — each one of these arrows, you can think of as a thread. These threads are organized into thread blocks, also known as concurrent thread arrays, or CTAs — a group of threads. And then finally, you have a collection of thread blocks, which forms the grid. When you launch a kernel, you're basically launching a grid of threads and thread blocks to all, in parallel, simultaneously do some

**[3:56]** computation. This is simplified — there are some other things that are happening. For example, H100s and B200s also have thread block clusters, which are clusters of thread blocks that enable some amount of distributed memory. Also, B200s have tensor memory now, for tensor cores, which is somewhere between the registers and shared memory. Some of these things are invisible to the programmer, but they are in the hardware. But we won't worry about them for this intro lecture. So, one thing you might ask about is: why do we have thread blocks? Why can't we just have a grid of threads, and each thread just takes a piece of

**[4:43]** data and does something? This would normally be fine if all you want to do is elementwise operations, which we'll see — for example, GeLU is an activation function that applies elementwise. And threads are pretty natural: each thread processes one element. So it's like f(i) for i equals — ranging over your data set. But for operations that involve communicating between threads, such as softmax or matrix multiplication, this view isn't really enough. And the reason is that, well, it could be enough if you were willing to pay the cost of writing and reading HBM.

**[5:28]** Because then you can still have — for example, in a matrix, we'll see later — every element in a cell just goes and computes that element of the matrix, and it can just read and write HBM. But as we've noted, HBM is very slow, so this would not be a good strategy. Instead, what we should do is use shared memory, which is local to an SM. And what the thread block allows you to think about is a collection of threads that are all going to access the shared memory. So consequently, you can think about a thread block — one of these thread blocks is being scheduled on one

**[6:15]** of these — semantic, not semantic, streaming multiprocessors — and it does its thing. What it's going to do is read a bunch of data from HBM and then process it, where the processing might involve communication between the threads via the shared memory, and then write it back out. This is a critical piece that comes up — later we'll talk about tiling, and that's the whole game here. In Triton, in fact, which we'll see as the primary way that we're going to write kernels these days, we're going to think natively in thread blocks. I think once you get the hang of thinking in thread blocks, this makes a lot of sense and makes your life a lot

**[7:02]** easier, as we'll see. So the programming model is fairly simple, I think — there's threads, thread blocks, and grids. Now, where I think things get a bit more complicated is the interaction between the programming model and the hardware. The programming model is actually very nice — it provides an abstraction of the hardware. When you write a kernel, all you have to know is that there's a bunch of thread blocks: you define them, and then define what computation each thread within the thread block has to do. And that's just like writing — it's like writing Python.

**[7:47]** That part isn't hard. And in fact, if you just care about correctness, that's all you need to know. But in practice, the performance is very sensitive to the hardware, and so you need to really deeply understand the hardware to obtain high performance. In fact, the whole reason we're talking about GPUs and kernels is that you're trying to squeeze out performance. So there are these two levels here: you're trying to understand the computation, and that's only part of the programming model, but how fast it runs is going to be strongly dependent on the hardware. So I'm going to give you some examples of why this matters — some of this will be review from what Tatsu

**[8:33]** talked about, but hopefully it'll reinforce some of these concepts. I think there will be around five different examples — this is just to give you a flavor of the considerations. There's something called warps, which I didn't talk about. In the very simplified view, warps are not really part of this clean picture of the programming model — you have threads, and thread blocks, or grids. Technically, you can also exceed the warps *[Ed: unclear — captions read "exceed the warps"]*, but you don't have to when you're just programming. So what is a warp? Each thread block, remember, is a collection of threads, but these threads are actually grouped into warps. There's basically 32 threads

**[9:21]** per warp. For example, if you have 64 threads in a thread block, there are two warps — the first 32 and the second 32. And as Tatsu mentioned last time, all threads within a warp — all of these Ts — must execute the same instruction in lockstep on an SM. Every cycle, they have to execute exactly the same instruction. Control divergence is when different threads within a warp need to actually execute different instructions — for example, if you have branching: if something, then A, else B. What happens is that you can only do the A's in your threads in your warp, and then you do the B's. So this gets sequentialized.

**[10:07]** This is bad and inefficient, and that's why branching is something you generally want to avoid. One thing that's really cool about warps is that an SM actually runs multiple warps. There's a warp scheduler, and there's a bunch of resident warps that are about to run — each warp, and threads, have registers — and the SM can switch between them with zero cost. This is not true in general, like on a CPU, but the way it's designed is to hide latency. This is important because one of the warps is, for example, reading from

**[10:53]** HBM, which, remember, is very expensive — it can take like 100 cycles or something. You don't want to just sit around waiting for that warp to do nothing — you switch immediately to another warp where it can actually do some tensor core operations. So that's one thing to note about the warp. Another reason why the warp comes up is this idea of occupancy, more specifically warp occupancy. The hardware constraint says that each thread can use at most 255 registers. And so what happens is that the SM has a fixed number of registers, so the more registers each thread is using, the

**[11:39]** fewer threads you can have — that's just math. And that can reduce your occupancy. But that's not necessarily bad, because if you have fewer threads but each thread is doing more work, that can actually be good. Occupancy is something you can measure, but it's not necessarily the larger the better, because there are some other trade-offs here. In fact, an example where you might want to have fewer threads is this idea of thread coarsening, where let's say you have an elementwise operation, and you can have one thread just perform

**[12:26]** work on one element. So then you have a lot of threads. But you can also say each thread processes multiple elements, like a constant, maybe eight. That gives you fewer threads, which makes scheduling and these things easier, but each thread is doing more work. So if your threads are very light, maybe you want to fatten them up a little bit. Just as an example of what might happen here: suppose you have a thread block with 128 threads in it, and each thread is using 160 registers. There's a hardware constraint — B200s have a maximum of 65,000 registers per

**[13:12]** SM. And it also has a constraint that says you can't run more than 65 — 64 warps at a time. So then you can go and do some simple math to compute what the occupancy is here. The number of registers per thread has to be less than 255 — that's fine. The number of registers you're using per block is the number of threads per block times the number of registers per thread — that's 20,000. That means you can run at most three blocks on your SM concurrently, and that corresponds to 12 warps. And because the maximum number of warps is 64, that means you have an occupancy of

**[13:57]** 18%. So you're only using 18% of the total number of warps you have, and this is because you have a lot of register use per thread. So this is an example where memory is constraining you in terms of how much compute you can do. Here's another example: something called bank conflicts. This applies to shared memory — remember, shared memory is like L1, it's on an SM. And just the way the hardware works is that shared memory is divided into 32 banks, each one 4 bytes wide. So

**[14:42]** these are my 32 banks, and this is my memory here — each bank has many elements. There's a constraint that says every clock cycle, each bank can only be accessed by one thread — at most one thread — assuming that's not the same location. So, for example, you can't access this location and this location at the same time. Which means that if you have multiple threads trying to access the same bank, the accesses have to be serialized — this is called a bank conflict. In the worst-case example, imagine this were a matrix that was kind

**[15:29]** of laid out like this, and you had some operation where 32 threads — you think, wow, this 32 is so great, I can massively parallelize this — and you try to all access this first column. They're just going to wait in line. This is a 32-way bank conflict, which is the worst possible setting you can be in. Now you could say, well, okay, why do you do that? Just access rows, of course — but this is unavoidable, because, for example, if you're doing a matmul, you have to access rows of one matrix and columns of the other matrix, and sometimes you do transposes. So you can't always just get away with choosing the order in which you go down. For elementwise operations it's fine — you can go in any

**[16:15]** order. But if you're doing matmuls, you can't control, for every matrix, whether it's row-major or column-major. There are some solutions here, which I won't get into — something called swizzling, which arranges your shared memory so that when you're going through, you can avoid bank conflicts. So that's another consideration. And when you're profiling, you can look at the bank conflicts, you can look at occupancy, and you can see what's happening. A final note, which is actually two more things: memory coalescing, which — talked to you, talked about *[Ed: unclear — captions read "which talked to you talked about"]* — so I'll just quickly remind people what this is about. When you have 32

**[17:01]** threads in a warp try to access HBM, the memory accesses actually get combined into a transaction of 20 to 128 bytes *[Ed: unclear — captions read "20 to 128 bytes"; the slide gives only "128 bytes"]*, which are called cache lines, and it goes and fetches it all at once. So imagine you have memory laid out like this — this is 32 wide. And in the best case, which is called full coalescing, all the threads are accessing the same cache line. So you have thread one accessing M00, thread two accessing M01, and so on, which means that all at once you're going to

**[17:46]** grab this entire cache line. Whereas if you go down the columns, then you're going to fetch a lot of memory that you're not going to use — same for the second row, and so on. This can feel similar to bank conflicts, but it's a very different constraint: that one is about shared memory, and this one is about HBM. So, final thing: block occupancy. Thread blocks, remember, are scheduled onto SMs. Logically, you can define as many thread blocks as you want, but physically, on chip, you only have a

**[18:31]** certain number of SMs — for example, 148. So if you launch 160 thread blocks, then you can only schedule 148 of them, and then you have to wait until they're done, and then you schedule the 12. But what happens if you schedule the 12 is that a bunch of the SMs are just not doing anything. That's called low occupancy — when the last wave of thread blocks has fewer than the total maximum number of thread blocks. So, in general, maybe it's a good idea to make the number of thread blocks divide the number of SMs. So, just to summarize here.

**[19:18]** There is a very elegant programming model where you have a grid of thread blocks, and the thread blocks have individual threads in them. And in terms of memory, HBM is global to everyone, shared memory is local to a thread block, and registers are local to a thread. But again, all the details of the hardware — warps, bank conflicts, memory coalescing, occupancy — really determine performance. And I think many of these details are hard to know, because the profiling tells you a bunch of information, but you have to know exactly how many

**[20:03]** SMs there are, and exactly the sizing of everything, and sometimes the scheduler does something you don't really have control over. So it's a lot messier than the programming model. I'll stop there for questions.

*[Question from the floor: Is there a scenario in which the kernel can share an SM — like, for example, in the problem we see, 148 SMs and 165 blocks, and we have to launch in two different ways? Is there no way in which a block can share an SM?]*

So the question is, can a block share an SM? I think the issue

**[20:50]** is — if you're doing things right, then the — I guess it depends on the block. If you have a block that's, for example, using most of the tensor cores on the SM, then putting another block there isn't really going to speed things up. I think fundamentally there is this jagged problem here of unevenness — because the blocks have to stay together. You can't take this and spread it out over here. You can define your — you can — I think the thing — the thing you would do is change your block size, so you change the number of blocks, so you don't get this tail here.

**[21:36]** Any other questions? Okay, let's move on. Hopefully you guys are getting more comfortable with GPUs now. Now I'm going to talk about benchmarking and profiling. I'm not going to actually say that much in terms of content, but I do want to emphasize the philosophy here, which is: here's a recipe for success. You benchmark and profile your code, you make changes, and you benchmark and profile your code again. And the reason I'm doing benchmarking and profiling as opposed to

**[22:22]** teaching you Triton earlier is because you should always just measure what's going on in your code and figure out what the bottlenecks are before you start writing kernels. So, benchmarking is basically how long things take, and it gives you this end-to-end time. It doesn't tell you where things are spent, but nonetheless it's pretty useful, because ultimately that's the thing you care about — how long things are running for. And because it distills things into one number, you can see how things are scaling, let's say with dimension. So there's a nice

**[23:07]** tool for benchmarking, but because this class is language models from scratch, I'm going to do it from scratch. I'm doing it just to highlight a few gotchas with benchmarking. So let's say we have this operation, matrix multiplication. run_operation2 is this wrapper that basically instantiates two random square matrices of size dimension by dimension, and then returns a function to perform the operation. So matmul, if you call this, basically does the matmul of those two random matrices — the random matrices are already generated, it's just like multiplying the two.

**[23:53]** So how do you benchmark? The naive thing to do is just: start time, run it, then stop time. But there are a few things — one is to always remember to warm up. I think I mentioned this on the second lecture, but it's worth emphasizing. This is because some things are lazily compiled, and you want to make sure that time doesn't factor in, because most of the time you care about how fast something is because you're going to run it over and over again, so the initial conditions don't really matter. And then you often want to time it multiple times, because there is some

**[24:39]** variance. The proper way to time things is to use these CUDA events — a start event and an end event, which you call record on, actually do the computation, and then hit the end event's record. And remember to synchronize, to wait for the CUDA threads to finish, because everything on the GPU is happening asynchronously, and this is a synchronization barrier. And then you record the time. And then you can do this again, and again, and here we're just taking the average. I think you might want to,

**[25:24]** if you are being very particular, look at the whole distribution, the P95, or whatever — but we'll just do the average here. And then one thing you can do with benchmarking is scale up your matrices and see how the time changes. You can see that, in this case, matrix multiplication, as expected, should grow cubically. But notice that there is this floor where, up until you get to almost 2000-dimensional matrices, things are basically constant. And this is because, as we've discussed, the shapes of

**[26:10]** these GPUs are built for fairly large matrix multiplications, and if you have like a 2-by-2 matrix, it's going to be very inefficient. So, really quickly on profiling: profiling tells you where time is actually spent. Hopefully all of you are familiar with profiling and are doing it. Maybe less obviously, profiling — even if you don't care about the time — helps you figure out what's actually happening under the hood, because especially with these high-level languages, you write some code, it runs, you get some result, and sometimes it can just be good to understand what's actually going on. PyTorch has a built-in profiler; in your assignment you're using nsight, which

**[26:57]** gives you more details, but we're going to skip that in the interest of time. So let's start with just — if you add two numbers, or sorry, two tensors, in PyTorch. Again, run_operation2 creates two random matrices and applies the operation. So, in profiling, I'm warming up, and then I'm putting this in the profiling context and doing the run. And then let's look at what the profile looks like for just A plus B in PyTorch.

**[27:43]** If you're not normally doing PyTorch, you probably don't think about it — it's like, okay, well, these two tensors just get added. So what's actually going on underneath the hood? If you look at the things it's calling, there is this long name — kernel at CUDA functor add *[Ed: unclear — captions read "kernel at CUDA functor add"; the exact profiler kernel name is machine-dependent and not given in the slide]*. This is basically a kernel that adds two tensors, and the times aren't going to be that interesting, because I'm only adding, so that's going to take 100% of the time. But this tells you that, underneath the hood, there is this thing called add. What about matmul? I'm going to, in PyTorch, do A at B.

**[28:30]** Similarly, there is this long name that describes this particular matmul kernel — color, F32, F32, 64, 64, 16, and so on *[Ed: unclear — captions read "color"; likely part of the CUTLASS kernel name]*. Notice that if you change the dimensions — now I'm doing a 128-by-128 matmul — you get a different one. If you look closely, this is 64, 64, 16; this is 32, 32, 16. So underneath the hood, in PyTorch, it looks like you're just doing add, but underneath the hood there could be all sorts of things happening. So, observations: here you can see which CUDA kernels are

**[29:17]** actually being called — these are generally the ones with the long names, and different CUDA kernels are invoked depending on the tensor dimensions. The name actually tells you something about the implementation as well. So, this name — CUTLASS is NVIDIA's CUDA library for linear algebra. SM100 corresponds to the Blackwell architecture, so this is a kernel that's specifically designed for Blackwell. This is FP32, and then 64, 64, 16 is the shape of the tile, which we're going to talk about later when we talk about tiling in the context of matmuls.

**[30:02]** So, last on benchmarking and profiling: just remember to do it. I think we make you do it on the assignment, so you have no choice. So let's apply this to another example here, on the GeLU. Remember, the GeLU activation function is this function, which is a typical non-linearity that's used. Often it's approximated with this tanh approximation, which is more compute-friendly. So, naively, if you implement the GeLU, you can do it in

**[30:50]** PyTorch just like this. You take a tensor, you take this equation, and you just put it into PyTorch — that's fine, and you get some number. PyTorch also has a built-in version — if you call nn.functional GeLU, you can get that version as well. And you can check that they're the same — you can run the two GeLU versions and make sure the answer is the same for a random input. There's also something that some of you have probably discovered — if you haven't encountered this, this is an important thing to know — which is that you can

**[31:36]** take any PyTorch function, call torch.compile on it, and it generates another function that does the same thing. So we have three horses in this race — we have the naive implementation, we have the built-in implementation, we have the compiled implementation. So, let's benchmark them. Naive takes — I guess this is 3.75. Built-in is much faster, and compiled is much faster as well, but not quite as fast as built-in. So, what's happening here? Like

**[32:23]** why are these things different? They all compute the same answer, but they have wildly different performance characteristics. So, here we can pull up the profiler and see what's actually going on underneath the hood. If you look at the naive GeLU here, and just do a profile — something's wrong with this view, so I'm going to go to this. You see that the profiler shows how time is being spent. There's a bunch of different kernels: binary functor, unary, add, tanh is a kernel

**[33:08]** here. And this corresponds to the fact that, in PyTorch, when you write the PyTorch expression, you look at the computation graph, and each primitive in the computation graph is actually realizing a kernel. And the reason this is slow is that when you launch a kernel, the kernel has to read from HBM, pull it all the way over to your SM, do the computation, and write it back. And then the next kernel picks it up from HBM, and writes it back, and so on and so forth. So you're doing a lot of reads and writes back and forth, because between kernel invocations, things have to go back to HBM.

**[33:53]** And if you look at what the built-in is doing, this is actually not that interesting. There's this GeLU CUDA kernel implementation, which is a single kernel that just implements the GeLU. So why does this exist? Well, because people use GeLU, so someone wrote a kernel for it and put it in the standard library — there's nothing magical about it. So, com— compilation is really interesting here. I'm not going to say too much about how it works, but it's really, I think, a fascinating topic, where you can take a naive implementation, which, remember, in PyTorch, it

**[34:39]** has a computation graph, and run a compiler, which, if you look at what's underneath the hood, is just a single kernel. And this is because it's figured out how to look at the computation graph and essentially write that kernel in Triton. So you can see that this is actually a Triton kernel. So: naive implementation, multiple kernels, requires multiple reads and writes to and from HBM — there's no kernel fusion here, this is slow. The built-in and compiled version, there's one kernel — basically all the operations in the GeLU have been fused together into one kernel. So you read from HBM once, you write to HBM once, per element.

**[35:27]** And you see that the compiled kernel is a Triton kernel. So that maybe is a good segue to talk about what this Triton thing is all about.

*[Question from the floor: What kernel is this, the built-in? Is it also — do the indices go straight out of CUDA?]*

So, I don't — let's see. I mean, I guess this says CUDA kernel implementation, so I imagine someone wrote it in CUDA. Any other questions?

*[Question from the floor: Why is Triton kernel faster than this one?]*

**[36:13]** *[Question from the floor, inaudible]* So, why is a Triton kernel faster? So, a Triton kernel is actually not faster in this case — oh, sorry, the compiled kernel is one Triton kernel, and this is slower than the built-in. I think last year when I did this, it was actually closer, but these things change, and it's very hardware-dependent. I think none of this is terribly optimized — this is just giving you the general idea here. So, let's write some Triton kernels. Remember our

**[36:59]** programming model — you have a bunch of threads organized by thread groups, by thread blocks, and there's a grid of thread blocks. And if you were to write in CUDA, which was originally developed by NVIDIA, and that's been, for years, the thing you do when you write kernels — the mental model is: what does each thread do? So you write a piece of code which essentially has some ID that identifies which thread you're talking about, and it just executes the code. The nice thing about this is that it's very closely related to what is actually happening underneath the hood, and it gives you fine-grained control,

**[37:46]** but there are cons here, which is that, remember, all these threads are in a thread block — some operations require them to communicate. So what has to happen is that they have to synchronize, they read from HBM all at once, they have to synchronize and do the computation, and then you have to basically do that bookkeeping. So if you were doing all elementwise operations, CUDA is just fine, it doesn't really matter. But as you get more complex operations, then Triton provides some value, the abstraction. So, Triton

**[38:32]** developed by — was developed by OpenAI. I think by now it's pretty standard. You basically specify what each thread block does. Generally it's powerful enough, especially for this class and getting started. If you really want to exploit every single new feature of the latest hardware, it might not give you the full flexibility, but let's not worry about that. And the conceptual framework to think about in Triton is: think about what does a block do. A block is going to load data into shared memory, operate on it, and write it back to global memory. So, in some sense, these blocks are an intermediate point between thinking about what individual

**[39:18]** elements are doing, as well as thinking about the general operation. In PyTorch, you basically define these huge matrices, and you say multiply them together, and that's the atomic operation — a lot of what you're thinking about is how do I get things into big matmuls. And at some level, Triton is a hybrid between that and the individual elements, as we'll see. So, let's start with the GeLU example here. Let's define an 8,000-dimensional vector and start writing some Triton. So, Triton is basically going to be — you write Python. How many of you have written Triton

**[40:05]** before? Just as a show of hands. Okay. Hopefully — I'll try — not that many, which is good, because then you won't be bored. So, this is normal — this is normal PyTorch, there's no Triton here, but I'm just preparing. So what I'm going to do is take this tensor, and I'm going to allocate an output tensor. Because in Triton we're not thinking functionally anymore — we're just thinking about moving, you have to explicitly read and write. There's no returning value. So I'm going to allocate the output tensor, which the kernel is going to write

**[40:50]** to. And so this tensor can be arbitrarily big, and I can't generally fit this all into one SM, because it's just too big. So I need to break it up into blocks. What I'm going to do is — you can think about this X as this array — I'm going to chop it up into blocks. So the total number of elements is 8,000, I'm just going to set the block size to 1024 for now, and then I have eight blocks. So, then I'm going to

**[41:38]** use this sort of weird syntax to call the kernel. So, triton_gelu_kernel — this, basically, in brackets, tells me essentially the shape of the grid. So basically this says the grid has num_blocks blocks. And I'm going to, for every one of these blocks, invoke this function, triton_gelu_kernel, which I'll talk about in a bit. I'm going to pass in an x, y, num_elements, and the block size. All right, so, unfortunately, I'm not going to be able to trace through this, so I'll just show you what the code

**[42:24]** looks like here. So let me get rid of that. This is probably the simplest kernel you can imagine. So, when you're looking at triton_gelu_kernel — now, before, we had X and Y, so now these are pointers. You can think of these as just like integers, they're addresses — you have to get comfortable with that. And then we have the number of elements and the block size, which are basically passed — actually passed in from here.

**[43:10]** So the way to think about it is that for every block we're going to have this function being called. And what happens is the first thing I'm going to do is — the block wakes up and says, who am I? The program ID, the PID, is basically identifying the block. So PID would be zero for this block, one, two, and three here. So now I have to figure out what data I'm going to operate on — that's PID times block size, this is the offset into the X pointer. So if PID is zero, then start is zero. If

**[43:56]** PID is one, then start is block size. If PID is two, then it's two times block size, and so on. So now I'm going to figure out the span I'm operating on. So offsets is start plus — this is the Triton library, tl.arange, zero, block size — which conceptually gives you the integers zero through block size minus one. So offsets is going to be, essentially, for let's say block one, it's going to be block size, block size plus one, all the way to two times block size minus one. So

**[44:42]** in this case, num_elements divides block size, but in general that's not the case. So you'll often see, in Triton code, this masking, which says: well, let's say the tensor only goes up to here, then I'm going to form a mask which is going to be true up until that point and false after that point. And for blocks that are not the final block, it's going to be just all ones. So now I've done the setup — what I do is I read, and this is basically pointer arithmetic. X pointer, remember, is the integer that specifies the memory location of where X is. I'm going to add offsets to that, which gives me the first block-size number of elements

**[45:30]** there — according to the mask, if I'm masked out, I don't read it. And then now you can just think about this as a vector. And then you do your normal computation, and then you get Y — Y is the same size as X — and then you do tl.store, Y pointer plus offsets, on Y, and the mask. So this loads from HBM, does some stuff, and then writes back to HBM. I'm going to stop there and take any questions about this

**[46:18]** first Triton kernel.

*[Question from the floor: What is the difference between this and CUDA? For this elementwise case it looks pretty much the same.]*

And, in fact, CUDA's even simpler, I think, because it really is elementwise — you wake up, you identify the thread, and then you just operate on that element. Now, the only thing here is that it's sort of like the vectorized version, where you have a block and you operate on that block. Later we'll see that

**[47:05]** operating on blocks is doing more — if you do something more than elementwise, then CUDA's going to be a lot more annoying to work with. There's a bunch of questions. Yeah, back there.

*[Question from the floor: How is this related, if you want to use the tensor units?]*

I'll later show you what this code actually compiles down into, and maybe I'll get back to that. But the short answer is that you don't control that — the hardware figures out where to put things.

*[Question from the floor: Can you actually walk through what's happening at the level from HBM to shared to register? What's]*

**[47:51]** *[Question from the floor, continued: like the step-by-step, where things are actually going?]*

So, the question is: what is actually happening when this — the same end *[Ed: unclear — captions read "the same end"]* is executed? So, the short version is that this is, in some sense, a lie. It's not like the GPU actually calls the Triton library on the GPU and is actually executing this code — this is basically for our consumption, to specify the computation. The compiler takes this, as we'll see later, and writes it into something called PTX, and then it will actually do the work. But conceptually — conceptually — so that's mechanically what's happening, which I'll get to later.

**[48:37]** If you're asking about the conceptual question, the way to think about this is that this X pointer is a memory location in HBM, and this basically specifies a range of memory locations, and load takes those memory locations and returns the data associated with them, and it gets — here I have a local variable I called X. In practice this is going to generally be a register or shared memory. Actually, you don't actually — Triton figures out what to do there. The thread level

**[49:24]** *[Ed: the following passage is garbled crosstalk between the lecturer and a student; speaker boundaries are unclear]* — and this is like, so when this is invoked, it's not really clear — did I line up what is going to be in shared, and what's going to be at the register level, beforehand, or is it happening now, which is way late? And so that's just — we're all the threads are sitting idle to get memory to flush in — execute, so that we're not sitting there twiddling our thumbs, just waiting for data to come in from HBM. Yeah, yeah. So the question is: when does this actually get executed? Doesn't this block? Let me try to come back to this question while I show you the PTX, and maybe it'll provide a bit more context on what's happening.

**[50:09]** Any other questions? So, all the kernels are going to look something like this. I do want to make sure people understand the general form, which is that you generally have your inputs, your outputs — you wake up, you figure out which index you're going to look at, you read, you do some stuff, and then you write to HBM. So, let's talk about PTX briefly. Let's see, how does this work — does this link work? So,

**[50:55]** when you write Triton, the compiler generates PTX code, which is this intermediate assembly language for new GPUs *[Ed: unclear — captions read "new GPUs"; possibly "NVIDIA GPUs"]*. I am obviously not going to go through all of this, but just to give you the flavor of what this looks like. Let me actually start with an observation — a few things. This is now what a thread is actually doing, not a thread block, because that's been compiled away. And so, if you look at

**[51:40]** a few notes here. This LD global is basically saying: load from HBM into some registers, and the registers are denoted, like, the R's are integer registers, FR — floating point registers. And then you have statements like: move zero into R5, move zero into R6, and so on. And then you have multiplication — you're going to multiply this register by this constant, and then put it in this. So this code gets executed, and at the bottom you should see a global store — so this is writing back into the HBM.

**[52:28]** This gets executed — this is actually the code that gets executed on a thread, and Triton is basically a layer above that. One other thing to notice is that you see all these blocks, and what's going on there is what I'm alluding to — thread coarsening — which is that this is one thread, but rather than processing a single element, it's actually processing eight elements. So the compiler decided that, well, actually this thread is pretty lightweight, it doesn't do that much, so let's just try to thicken it up a little bit. So, looking at PTX can give you some sort of appreciation for

**[53:13]** what's going on underneath the hood.

*[Question from the floor, garbled: does it make this for each individual — are all of these — probably the same one for the whole thing?]*

So, the question is: does it make this for each thread? This is compiled once, and it's the same piece of code that each thread runs. And the way that the thread distinguishes itself is that this piece of code gets passed, basically, the thread ID. So here, %ctaid.x is the block index, and TID.x is the thread index. So this basically, if

**[54:01]** this piece of code is running, this tells you which block I'm in, and T— TID.x tells you which thread inside that block I'm in. Any other questions? And I'm trying to figure out — okay. So that's generally the flavor of your first Triton kernel — load from HBM, compute, write back to HBM — and then we saw the PTX, which is like

**[54:48]** the grungy — what actually happens underneath the hood. There's still a lot of things that are not specified in the PTX, like, for example, which SMs things are operating on, and the warps and everything — a lot of that is hardware controlled, so you don't even see it.

*[Question from the floor: PTX code is generated by the compiler, not something that you would normally go write in — it looks like assembly code?]*

So the question is: is PTX generated by the compiler? There are people who do write PTX, if you really think you're better than the compiler. And I think the NVIDIA

**[55:36]** compilers are generally pretty mature, but some other accelerators that are less developed, I think, sometimes you just have to reach in and actually hand-hold a bit more. But generally, you shouldn't need to do that.

*[Question from the floor: So, when I look at the PTX — so, when a warp gets scheduled onto an SM, I'll get to that tl.load, and it's almost like a CPU trap call, where I'm just waiting for something to happen, and now I am twiddling my thumbs, so some other warp will get scheduled over me, and then they will do operations. Now that the tl.load is done, then I'll reschedule that warp, and then I'll continue on. Is that kind of —]*

Yeah, that's right. So, just

**[56:22]** to repeat the question, or the comment: if you look at this Triton code, these statements — like this load — are going to block for some number of cycles. And so, this is running on some thread, on some warp, on some SM. And remember, the SM is running multiple warps at the same time. So when you get to that point, it can just find another warp to run, and then, when this is done, the warp scheduler comes back and takes over.

*[Question from the floor: Why would four warps be — so, I'm sorry — why would there be four warps in the same kernel on each SM, if four warps are scheduled —]*

Yeah, so what the question is, why four

**[57:08]** warp schedulers? I don't know exactly the reason behind that. So, now let's go through some other examples — maybe just as a quick preview, we're going to do three more examples. GeLU — GeLU is the simplest form, even though the computation has a lot of messiness, it's just elementwise, so conceptually, in the context of this lecture, it's actually very simple. Now we're going to look at softmax, where you're going to have to do a reduction — in this case, we're going to think about the case where the row fits on a block, and then we're going to consider the case where the row doesn't fit

**[57:54]** on a block, and then we're going to go up to matmul. And hopefully, by that point, you'll have all the ingredients you need to do the assignment and implement flash attention. So far we've looked at elementwise operations — now let's think about other operations that aggregate over multiple values. Remember what softmax does — it ex— takes, let's just think of it as a matrix — you exponentiate and normalize each row of a matrix. This is used in attention, it's used in generating probability — probabilities. Generally a good thing to do.

**[58:39]** So let's just start with a naive implementation, and keep track of what's happening. Here I'm defining this tensor, and here's the naive implementation — actually, I think this is on assignment one. Anyway, here I have an M-by-N matrix, and so, for every row, I need to compute — I'm going to compute the max of each row, and this is for numerical stability. I'm going to subtract off the max, and then I'm going to exponentiate elementwise, and I'm going to sum

**[59:26]** and compute the normalization constant for every row, and I'm going to divide, and then that's it. So, if you count the number of reads and writes — remember, this is just plain PyTorch, so this is a different kernel, this is a different kernel, and unless you call torch.compile, these are going to be different operations, and each operation is going to read and write, read and write, from HBM. So you have five MN reads, three MN writes, and in principle you should really only have

**[1:00:11]** much fewer. So this piece of code makes sense — everyone should be familiar with what a softmax is doing. So, let's write now the Triton kernel. In some sense, the form is going to be very similar, just like in GeLU — once you do the scaffolding, the core computation looks very much like the naive version. So, what we're going to do is say each row is a block. And why do I make each row a block? Well, because, remember, each row I have to normalize and sum. So it's not elementwise — softmax is not

**[1:00:57]** elementwise, but it is sort of row-wise. So the blocks don't interact — they don't need shared memory across blocks, so that's fine, there's no shared memory across blocks. So then, within each row, I'm just going to do some stuff. So let's see what happens. Again, in Python, I'm going to allocate my output tensor — I have this M-by-N matrix. I'm going to define the block size as, basically, the number of columns — go to the next power of two, for good luck. And then the number of blocks is just the

**[1:01:44]** number of rows. And then I'm going to call this kernel. So, how many blocks are there? M — one for each row. And each block, I'm going to pass the input pointer, the output pointer, and then I'm also going to pass these strides, which tell me how far to move down. Okay, let me actually go and show you the softmax kernel. So what does this look like here? So

**[1:02:30]** I wake up, I'm on a particular row. And this is going to give me all the columns from zero to the block size, which, I'm assuming, is all the columns. I'm going to read — so basically, I need to figure out where to read from memory, and that's going to be the start of my data plus — which row, and the row stride basically gives me — every row is basically this is the number of columns, essentially, it's going to tell me how far to go down. And then this, X pointers, is basically the addresses of all the data I need to load. I load them up.

**[1:03:18]** And here I do this thing where, if it's masked out, then I put minus infinity, because I know that's going to be the equivalent of a zero for the softmax operation. And then this part is essentially the same as the naive softmax — I'm just going to compute the max, subtract it off, exponentiate, sum, and divide, and then write it back. Yeah, so this is the version where each block can just span the entire row.

**[1:04:05]** So, yeah, this is sort of — and you can see that Triton makes this very easy, because this is as if you were just writing normal PyTorch code. So, if everything fits in a block — the thing is, if you can fit anything through a block, you can just write normal PyTorch, almost.

*[Question from the floor: What if my number of columns and number of rows are bigger than the block size?]*

Yeah, so we'll get to that. Now, if the number of columns, rows, in general, is going to be much larger than that block size, so we'll come back to that.

*[Question from the floor: So, if you wanted to do like a softmax like]*

**[1:04:52]** *[continued: by column?]* So, the question is: if you wanted to softmax by column, I think that should be fine, because here we're tracking these pointers, and the pointers can be anything. And here, I think all you would have to do is change the stride here, to basically access the columns. Actually, it would be here — the column offsets, you would just multiply this by the row stride, I think. Okay, let's move on. All right, so now, warming up to the matmul, suppose that your row doesn't fit into a block. So what do you do in this case? For example, if you have, let's say,

**[1:05:39]** a row that's — 400 — 4,000 columns, but the block size is only 1,024. So you can't cons— you have to do something here. So here's the strategy: we're going to break up the row into tiles. In this case there's going to be four tiles, and each thread is going to iterate over the tiles and accumulate a sum. And then, finally, at the end, we're going to do the reduction, by summing everything that each thread produced. I'll show you an example of this. And now I'm switching from softmax to row sum, because it's just easier to think

**[1:06:25]** about. So, here — well, this is not very interesting. The built-in row sum does what you would expect — the row sum operation just takes a matrix and computes the sum of each row. So, no surprises there. So, conceptually, what we're going to do here is as follows: each block is still in charge of one row, that part hasn't changed. So, suppose I'm block one, row one — I wake up, and what do I do? I'm going to — remember, now there's tiles, and suppose tile zero is columns zero through three, tile one is columns four through seven, and then tile two

**[1:07:12]** is columns eight through 11. So what I'm going to do here is iterate. First, I basically process — all the threads, basically, each thread keeps an accumulator, and processes the first tile, and it's going to move on to the next tile, and add the current element to the accumulator. So, here I'm putting 3, 1, 4, 1, and then the second loop iteration, I'm going to add five to this, I get eight; I add nine to this, I get 10; two to four; and six to one. So each of these four threads is going to be accumulating its own thing.

**[1:08:00]** And then, at the end, for tile two, I add five and three to their respective accumulators. So, at the end of the day, I have a vector of accumulators, and that I can just sum up. So let's see what the code looks like here. I'm just going to call the row sum kernel, and let's see. So here's what it looks like: wake up, I am on a particular row, and this is what one row looks like — there's one tile, a second tile, a third

**[1:08:45]** tile, and so on. N is the number of elements of that row, and block size is the size of this tile. Remember, block size is the number of threads. I'm processing more data — larger than the block size — I just have to do it iteratively now. So I'm going to loop over all of the tiles: start is going to go from zero, to block size, to two times block size, and so on and so forth. Each time I'm jumping across, I'm doing — basically getting the offsets

**[1:09:32]** at the particular tile. And then I'm going to load the data from HBM, and add it to the accumulation. This accumulation is going to be either in registers or shared memory. And then, finally, after I loop over all the tiles — I process a whole row — I get, basically, for every thread, an accumulator of what that thread has picked up, and I can just do a sum to get a scalar, and write it out. So this is a little bit more complicated than before, because now we have a for loop within a thread, and this is necessary when your data doesn't fit within a block.

**[1:10:18]** So, the question is, I think, if you can control where the accumulator resides. At least in this Triton program, you don't explicitly say, and this is up to the Triton compiler to figure out where to put it. But in general, if the block size is large enough, it has to go in shared memory. All right, does this make sense? It might, just to,

**[1:11:04]** make sure people are on the same page. Remember, in GeLU, we also split a row into a bunch of pieces, but those were blocks — each of those pieces was a block that was processed independently. These are not blocks, these are tiles. The block corresponds to this whole row, and has to process all the tiles. So this is where it starts to not look like PyTorch, because you're not able to process all your data in one nice — not everything fits into shared memory.

*[Question from the floor: I— let's say the accumulator is stored in shared memory?]*

Yeah.

**[1:11:51]** All right, so let's now go on to our finale, which is matmul. Matmul, of course, is the bread and butter of deep learning — it's been optimized to death, and it's, in some sense, a very fundamental operation. You take two matrices, you multiply them. I'm going to add a little bit of a twist here — I'm going to do a matmul followed by a ReLU, just for kicks. And this happens, right, because if you have one linear layer, it's a matmul, and then you apply a ReLU activation. So this is not completely out of nowhere.

**[1:12:37]** But I'll show you later why I did this. So how do you build a matmul kernel? Here's the naive approach: here's my — let's say — A matrix, I'm multiplying A times B, and I'm trying to write out the matrix C. A is M-by-K, B is K-by-N, so C is M-by-N. So what I'm going to do is fix one of these elements — let's say M equals 1, N equals 2. So I'm processing — actually, let's do M equals 1, N equals 1, so I'm doing C5. And then, basically, for every K, I'm going — so I'm going to

**[1:13:25]** I'm going to iterate over K, the rows of A and the columns of B. I'm going to read from HBM, multiply them, accumulate that, and at the end write out to this element. So that is a valid matmul kernel. So what's wrong with that? It's correct, but if you look at how many reads and writes it's doing, this is not good. Basically, for every M and N and K, I have to read from HBM. So it's on the

**[1:14:10]** order of M times K times N reads. The number of writes doesn't matter, but this is a bottleneck. And if you remember from the second lecture, if you look at the number of operations you're doing divided by the number of bytes transferred, that's the arithmetic intensity, which you want to be high. The number of operations is on the order of M times K times N, and the number of reads is also the same, so the arithmetic intensity is a constant, which is not good. If you look carefully, you notice that there are a lot of redundant reads. Imagine computing C4 — you needed to read A4, A5, and A6, and

**[1:14:56]** if you compute C5, you're going to have to read those over again as well. So if you can just read those once, then you really save on reads, and that's great. So let's try to use shared memory to do that. Here's the idealized approach: I'm going to load all of A and B into shared memory, and then just compute C. If I can do that, then I don't have this cubic number of reads, I only get a quadratic number of reads, which means I get an arithmetic intensity of order N, which, in the second lecture, I said was an ideal

**[1:15:42]** thing you could hope for. So, if you can do that, that's great, because there are basically no redundant reads — you read everything once into shared memory, do the computation, and write it back. But what's the problem with this idealized approach? The problem is that A and B are usually too large to fit into shared memory. So then what do you have to do? So the idea here is this very classic idea of tiling, and the idea, basically, is to fit as much as you can into shared memory. So, in some sense, it's going to be

**[1:16:29]** look like the naive — it's going to globally look like the naive approach, but locally look like the idealized approach, is the way to think about it. So here's the picture — I think Tatsu showed this as well. So what we're going to do is take this matrix C, and instead of — remember, the naive approach just said, for every element, I'm going to compute it — now I'm just going to say, for every tile, I'm going to do it. So I break it up into tiles, and each of these tiles is going to be a thread block. I'm going to have a bunch of threads responsible for computing this. Now, a different tile is going to be computed completely separately by another thread block.

**[1:17:14]** So, imagine I'm in Triton, I wake up, I'm looking at this thread block. So what do I have to do? Well, in some sense, it's the same as the naive approach, where, for every row tile of A, I'm going to scan across the rows, and for every column tile of B — I'm going to load the corresponding tile of A and the corresponding — so say I'm here — the corresponding tile from B into shared memory. I'm going to multiply these two together, and that's going to be, like, the idealized approach, and I'm going to accumulate

**[1:17:59]** that in the partial sum, and this is all sitting in shared memory. And then, after I finish the whole sweep of the row and the column here, then I can finally write this output tile to HBM. So that's conceptually what's happening. And then the arithmetic intensity here now goes up to order tile size. So you can generally not reach order N, because that would require you to fit everything into shared memory, but if your tiles are big, then that's still not too bad. So just as a bonus, while you're writing a kernel for this anyway, sometimes if you want to apply an

**[1:18:46]** elementwise activation function, it's very easy to just put it on at the end — and this is kernel fusion. So, very quickly, the implementation here — oops. Just as a reminder about strides, since that's going to show up: a tensor is a multi-dimensional array, but in memory it's linearized, and the strides of a tensor tell you how to map from a multi-dimensional index — such as a row, comma, column — into an actual index. And basically, what you do is multiply the row by the stride, plus the column by the stride

**[1:19:31]** of the column. So, in this case, every time you advance to the next row, you go four positions in memory, and every time you advance a column, you go one. If it were a transpose, it would be flipped. So what does this kernel look like here? So — the launch is not interesting. You wake up, and you are on tile M, N — it's like, "Okay, I'm responsible for computing the C matrix, but for the M-comma-N tile." There's a bunch of index manipulation, which I'll just gloss through, but it's

**[1:20:19]** straightforward, but a little bit — you just have to track the indices. So this basically tells you which rows of matrix A I'm looking at, which — oh, sorry — which columns of B I'm looking at, and then this is just the numbers one through K. Then I'm going to get the pointers into A and B at my tile location, and then I'm going to set up this accumulator matrix — this is going to be in shared memory, this is M-by-N. And then this is going to look kind of like the row reduction — you have this sum over all of the tiles, but instead of just going across the tiles, I'm now going across the row

**[1:21:06]** tiles, and also, simultaneously, down the column tiles of B. I load A, the small matrix, I load B, the small tile, and then I perform this dot. So, remember, whenever things are in shared memory, things look like PyTorch, and I can just say, "matmul it," and it'll do the thing. And then I advance to the next row tile of A and the next column tile of B. And this is — the bonus is that, if I wanted to apply an elementwise non-linearity, I might as well do that here, before I write it out to HBM — I can do any sort of operation on it. And then, finally, I just write it out.

**[1:21:51]** So there are some indices you have to pay attention to, but hopefully the form of the algorithm is clear. Any questions about that? All right, maybe I'll just summarize, and you can ask me later. So today we talked about — there's a programming model, which is either PyTorch, or Triton, or PTX — this is what the programmer can control. And even PTX, you can write PTX, and you can control every — specialize however you want.

**[1:22:37]** But this is not the full picture, because the reality is that your code has to run on hardware, and there's only a finite number of SMs, a finite number of banks, and the sizing of memory and registers are all finite. So you come in with your big matrix and transformer, and it has to fit in with the constraints of the hardware. So that's why benchmarking and profiling are really important, to understand how the messiness of the hardware translates to performance. We talked about Triton, which I think is a pretty nice, neat language to think about thread blocks. Hopefully,

**[1:23:24]** by now you can appreciate that things are easier to think about with thread blocks than individual threads, because you don't have to think about explicitly synchronizing threads or doing shared memory. And the way to think about it is: you have your computation, you break it down into these thread blocks, where you just need to read from shared memory, do some stuff, and write it back into HBM. And then we saw some examples of increasing difficulty — elementwise is the easiest, then reduction over a row, then reduction where it doesn't fit into a row — and that's where we introduced the baby tiling — and then matmul is the canonical example, where you actually do

**[1:24:09]** tiling. So that's all I'm going to say about how to program a single GPU. Next time we're going to go to more GPU and talk about multi-GPU programming. Yeah, question?

*[Question from the floor: If I'm able to write my own kernels, what are the alternatives that I have to Triton, and what's the range of choices that I have? How close can I get to the optimal performance?]*

So the question is, what are the alternatives to Triton? There's a trade-off — every language has an inductive bias, that makes certain things easier and certain things harder. Most of what we do — Triton was built by people who

**[1:24:54]** train transformers. So anything involving transformers, I think, is going to be relatively easy there. Of course, in the extreme, you can always go to PTX and write that, but I wouldn't advise that as a first step. There are a bunch of other language — sort of libraries. There's ThunderKittens, there's CuTe — various DSLs that give you — they're not necessarily comparable, either up or down the stack, they just give you different characteristics.

*[Question from the floor: When you have a high-dimensional tensor — multiplications, or high-dimensional tensor processing,]*

**[1:25:39]** *[continued: what is the best approach? Is there — what I'm thinking is, can I load — unload — the whole tensor, both of the tensors, on a bunch of threads or thread blocks on my GPU at the same time, and compute them? Or would it be better to take each element, just like what we did here, for the particular component of my tensor, then process it, and write it back into HBM?]*

So, briefly, the question — and we should probably wrap up — is: is it better to read all at once, or maybe process individual elements at a time? I think it's hard to

**[1:26:25]** answer this in the abstract — it depends on the nature of the computation. Maybe we can talk offline about that. All right, see you next time.
