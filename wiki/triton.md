# Triton

Triton is the language CS336 writes GPU kernels in, and Lecture 6 is its
introduction. It was developed by OpenAI, and Percy Liang's summary of why it is
the default now is short: "I think by now it's pretty standard. You basically
specify what each thread block does" ([38:32]).

## The one idea: program the block, not the thread

The distinction from CUDA is the whole language ([36:59]–[38:32]):

| | CUDA | Triton |
| --- | --- | --- |
| You write | what each **thread** does | what each **thread block** does |
| Pro | "very closely related to what is actually happening underneath the hood... fine-grained control" | "generally it's powerful enough, especially for this class and getting started" |
| Con | threads that must cooperate have to synchronize and manage shared memory by hand | less control at the frontier — "if you really want to exploit every single new feature of the latest hardware, it might not give you the full flexibility" |

For a purely elementwise kernel this buys you nothing, and the lecture says so
when a student asks: "CUDA's even simpler, I think, because it really is
elementwise — you wake up, you identify the thread, and then you just operate on
that element" ([46:18]). The payment comes later: "if you do something more than
elementwise, then CUDA's going to be a lot more annoying to work with" ([47:05]).

The conceptual framework for a block is one sentence — it "is going to load data
into shared memory, operate on it, and write it back to global memory" ([38:32]) — and Triton's position in the stack is a hybrid: PyTorch's atomic unit
is a whole matrix, CUDA's is one element, "and at some level, Triton is a hybrid
between that and the individual elements" ([39:18]).

## The skeleton every kernel shares

Percy states it outright, and all four of the lecture's kernels follow it:

> "You generally have your inputs, your outputs — you wake up, you figure out
> which index you're going to look at, you read, you do some stuff, and then you
> write to HBM." ([50:09])

In code, from
[`triton_gelu_kernel`](../raw/slides/06-kernels-triton.md#triton-gelu-elementwise):

```python
@triton.jit
def triton_gelu_kernel(x_ptr, y_ptr, num_elements, BLOCK_SIZE: tl.constexpr):
    pid = tl.program_id(axis=0)      # Identifies the block
    start = pid * BLOCK_SIZE         # Starting index of this block
    offsets = start + tl.arange(0, BLOCK_SIZE)
    mask = offsets < num_elements    # Don't read/write past the end
    x = tl.load(x_ptr + offsets, mask=mask)
    # ... compute on the whole vector ...
    tl.store(y_ptr + offsets, y, mask=mask)
```

Four things in that skeleton are worth naming separately, because they are what a
PyTorch programmer has to unlearn.

**You get pointers, not tensors.** "Before, we had X and Y, so now these are
pointers. You can think of these as just like integers, they're addresses — you
have to get comfortable with that" ([42:24]). Indexing is arithmetic on those
addresses: `x_ptr + offsets`.

**There is no return value.** The output tensor is allocated by the *host* side
before launch, and the kernel writes into it. "In Triton we're not thinking
functionally anymore — we're just thinking about moving, you have to explicitly
read and write. There's no returning value" ([40:05]).

**"Who am I?" is the first question.** `tl.program_id` gives the block its index —
0, 1, 2, 3 — and everything the block touches is derived from it ([43:10]).

**Masking is not optional.** The tensor's length rarely divides the block size, so
the last block would run off the end. "You'll often see, in Triton code, this
masking, which says: well, let's say the tensor only goes up to here, then I'm
going to form a mask which is going to be true up until that point and false after
that point. And for blocks that are not the final block, it's going to be just all
ones" ([44:42]). The padding *value* is chosen to be the identity of whatever
reduction follows — `-inf` for a max, `0.0` for a sum. See [fused
softmax](fused-softmax.md).

Inside the load and the store, though, the code goes back to looking ordinary:
"now you can just think about this as a vector. And then you do your normal
computation" ([45:30]). On the softmax kernel he puts it more strongly — "Triton
makes this very easy, because this is as if you were just writing normal PyTorch
code. So, if everything fits in a block... you can just write normal PyTorch,
almost" ([1:04:05]).

## Launching a kernel

The host side allocates the output, works out the grid, and calls the kernel with
a bracketed grid shape:

```python
y = torch.empty_like(x)
num_blocks = triton.cdiv(num_elements, BLOCK_SIZE)
triton_gelu_kernel[(num_blocks,)](x, y, num_elements, BLOCK_SIZE=BLOCK_SIZE)
```

"This, basically, in brackets, tells me essentially the shape of the grid... this
says the grid has `num_blocks` blocks. And I'm going to, for every one of these
blocks, invoke this function" ([41:38]). The grid can be multi-dimensional — the
[tiled matmul](tiling.md) kernel uses a 2-D grid and reads both
`tl.program_id(0)` and `tl.program_id(1)`.

## What Triton decides for you

Two questions from the floor draw the boundary of your control, and in both the
answer is that the compiler and the hardware decide.

**Where a value lives.** Asked to walk through HBM → shared → register, Percy's
answer is that the Python you are reading is "in some sense, a lie. It's not like
the GPU actually calls the Triton library on the GPU and is actually executing this
code — this is basically for our consumption, to specify the computation. The
compiler takes this... and writes it into something called PTX" ([47:51]). The
local variable you name "is going to generally be a register or shared memory.
Actually, you don't actually — Triton figures out what to do there" ([48:37]).
Asked again about the row-sum accumulator: "you don't explicitly say, and this is
up to the Triton compiler to figure out where to put it. But in general, if the
block size is large enough, it has to go in shared memory" ([1:10:18]).

**Whether you get the tensor cores.** "You don't control that — the hardware
figures out where to put things" ([47:05]).

The corollary is that a Triton kernel is compiled **once** and every thread runs
the same code, distinguishing itself only by its ids ([53:13]). That is the same
SIMT bargain described in [the GPU execution model](gpu-execution-model.md).

## Below Triton, and beside it

Below is [PTX](ptx.md), which Triton compiles to and which the lecture reads for
two minutes to show what actually executes.

Beside it are other DSLs. Asked for alternatives, Percy frames the choice as
inductive bias rather than a ranking: "every language has an inductive bias, that
makes certain things easier and certain things harder. Most of what we do — Triton
was built by people who train transformers. So anything involving transformers, I
think, is going to be relatively easy there" ([1:24:09]). He names
**ThunderKittens** and **CuTe** as examples, "various DSLs that give you — they're
not necessarily comparable, either up or down the stack, they just give you
different characteristics" ([1:24:54]).

> The captions render the second name as "cute"; this KB writes it **CuTe** on the
> editorial judgement recorded in the [transcript
> header](../raw/transcripts/06-kernels-triton.md), since the lecture's own
> material never prints either library's name.

## Related

- [Lecture 6 — Kernels and Triton](06-kernels-triton.md) — the four worked kernels.
- [PTX](ptx.md) — what this compiles to.
- [torch.compile](torch-compile.md) — which emits Triton on your behalf.
- [Fused softmax](fused-softmax.md), [tiling](tiling.md) — the reduction and
  matmul kernels in detail.
- [The GPU execution model](gpu-execution-model.md) — threads, blocks, warps.
- [Transcript](../raw/transcripts/06-kernels-triton.md),
  [lecture source](../raw/slides/06-kernels-triton.md).
