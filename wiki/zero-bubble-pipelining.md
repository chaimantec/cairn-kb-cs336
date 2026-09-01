# Zero-bubble pipelining

The cleverest idea in lecture 8's pipeline section, and the one that is not about
scheduling micro-batches at all. It attacks the
[pipeline bubble](pipeline-parallelism.md) by splitting the *backward pass* into
two operations with different urgency.

## The two halves of a backward pass

At every node of the computation graph, backpropagation does two distinct things
([37:30]):

1. **Propagate the partial derivatives further back** through the graph — call
   these the **B** operations.
2. **Compute the gradient with respect to this node's weights** — the **W**
   operations.

The insight is that these have completely different scheduling constraints
([38:16]):

> Propagating the partials backwards — that's very important, because your next
> stage can't do any work until you've propagated that signal back. On the other
> hand, the derivative with respect to the weights is kind of a leaf node, so to
> speak, on this graph, and you can do it whenever.

**B** is on the critical path; the upstream pipeline stage is blocked until it
arrives. **W** is a leaf — nothing waits on it. Standard implementations fuse the
two into one "backward" step and therefore schedule the whole thing as if it were
urgent.

## The schedule

Separate them, run every **B** as early as possible, and defer the **W**s into
whatever gaps remain ([38:16]):

> So, what you do is compute the B's as quickly as you can — those come first — and
> then you can defer the W's until later, and do them when you have a gap in your
> computation. When you do this, you can almost completely fill up the pipeline.

The bubble is not eliminated by making the pipeline deeper or the batch bigger; it
is *filled* with work that was always going to be needed and had no deadline. Slide
38 draws the computation graph alongside two hand-built schedules showing the B and
W cells placed separately.

## The cost

Complexity, and a lot of it ([39:02]):

> This zero-bubble pipelining thing is very complicated, much more complicated than
> you would normally like to deal with, but it allows you to deal with these
> pipelining issues almost entirely, depending on the workload involved in the W
> versus the B.

That last clause is the real caveat: the technique works to the extent that the
**W** work is big enough to fill the gaps and small enough not to create new ones.
It is not a free win independent of the model shape.

## The simpler predecessor

Before zero-bubble, slide 37 shows the more conventional route: interleaving, where
each device holds several non-contiguous stages and forward and backward micro-batch
steps are woven together ([36:45]):

> You can sequence different forward-pass elements in between different
> backward-pass elements, and by doing this clever scheduling, you can further
> reduce the bubble size. This was taken from the DeepSeek paper.

That buys "a little better in terms of bubble size" for extra communication
bandwidth — the slide's own title is "Trading communication bandwidth for
utilization". Zero-bubble is the step beyond it.

*(Slide 37's interleaved schedule visibly uses four cell colours for two model
chunks per device while its printed legend labels only two; the slide file records
that mismatch.)*

## See also

- [Pipeline parallelism](pipeline-parallelism.md) — the bubble this attacks.
- [3D parallelism](3d-parallelism.md) — where pipelines sit in the combination.
- [Lecture 8](08-parallelism-2.md) · [slides 37–38](../raw/slides/08-parallelism-2.md#slide-37--trading-communication-bandwidth-for-utilization) · [transcript](../raw/transcripts/08-parallelism-2.md)
