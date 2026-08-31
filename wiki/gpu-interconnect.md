# GPU interconnect — NVLink, InfiniBand, Ethernet

[Lecture 5](05-gpus-tpus.md) described the memory hierarchy *inside* a GPU.
[Lecture 7](07-parallelism.md) extends the same picture outward, and the shape is
the same: each step away from the compute is roughly an order of magnitude
slower, so the whole art is arranging for most traffic to stay near ([2:26]).

| Level | Link | Bandwidth (as the lecture states it) |
| --- | --- | --- |
| Inside one GPU | L1 / shared memory | fastest |
| Inside one GPU | HBM | 8 TB/s on a B200 |
| GPUs in a node | NVLink → NVSwitch | 1.8 TB/s (NVLink 5) |
| Nodes in a pod | InfiniBand | ~0.05 TB/s |
| Pod to pod | Ethernet | ~200 MB/s |

The single most useful calibration: NVLink is "about four x" slower than HBM
([24:01]). Crossing between *devices* costs about what a 4× slowdown on local
memory would — which is far better than the jump to InfiniBand, roughly another
36× down.

## The classic picture, and the modern one

The **classic** arrangement is the one in an ordinary computer: CPUs, a PCIe bus
that you "used to connect things like your mouse and keyboard" to, some GPUs hanging off
it, and Ethernet to the next machine ([21:43]–[22:28]). GPUs in a box talk over
PCIe; GPUs in different boxes go all the way out through Ethernet. Percy's gloss:
"this is like if you bought your gaming GPU and you had hooked it up with your
friend, and he's like, 'I'm going to train some big model' — that's what you would
have to do" ([22:28]). PCIe 7.0 at 16 lanes gives 242 GB/s; Ethernet, ~200 MB/s.

The **data-center** arrangement replaces the bottom two rungs ([23:14]):

- **8 GPUs per node**, connected by NVLink to an **NVSwitch**.
- **Nodes into pods**, connected by **InfiniBand** — out through PCIe to an HCA /
  InfiniBand NIC and down an InfiniBand cable.
- **Pods into a cluster**, connected by **Ethernet**, through PCIe and the CPU.

Percy flags which of his own numbers to trust: "eight is typical, but this 256 is
made up" ([23:14]).

## Why NVSwitch matters

An NVSwitch makes a node look flat. "NVLink connects to the NVSwitch, which means
that, from a programming perspective, you can think about GPUs as connected to any
other GPU — you go GPU to any other GPU, and the hardware takes care of
transmitting that to the NVSwitch, and then the NVSwitch routes it" ([24:01]).

That flatness is what makes [tensor parallelism](tensor-parallelism.md) viable at
all, and it is also why the hierarchy has to exist. Switches do not scale
indefinitely: "you can't have an NVSwitch handling 100,000 GPUs" ([25:34]). Past
the switch's reach you are on InfiniBand, and past that, Ethernet — "it's
analogous to the memory situation: the more nodes you have, then the slower it's
going to be."

## RDMA, and getting the CPU out of the way

On ordinary Ethernet, sending a tensor is not just slow, it is *ceremonious*
([26:21]). The GPU hands data to the CPU, which copies it into a **kernel socket
buffer** — "here, 'kernel' means — not the GPU kernel, but the CPU's traditional
notion of a kernel" — builds network packets, and copies again to the network
interface. "So, this generally introduces a lot of latency."

**Remote Direct Memory Access (RDMA)** removes that path entirely: it "allows a
GPU to directly write or read from another GPU's memory without using the CPU at
all" ([27:06]).

Where you get it:

- **NVLink / NVSwitch** — yes.
- **InfiniBand** — yes.
- **Standard Ethernet** — no.

Asked how RDMA relates to NVLink and InfiniBand, Percy separates the layers:
InfiniBand, NVSwitch and NVLink "are more of the hardware" — the captions garble
his next clause, but the sense is which cables and switches are physically present
— while "RDMA is more operationally, like, what happens when you're communicating"
([32:33]). RDMA is a capability; there are several ways to
provide it.

## Two developments

**NVL72** ([27:53]). For B200s and B300s, NVIDIA packages "trays of eight GPUs,
but have nine of them" — 72 GPUs inside a single NVLink domain, all NVSwitched
together. The significance is how much of your job can stay on the fast tier:
"if you're mortal — you think, 'Well, okay, I have eight GPUs that are
interlinked really fast, and then outside of that, things get slowed down a lot.'
But if you have a lot of money, you can buy this really fancy hardware, and you
can get really fast interconnects up to 72 GPUs."

On the physical layout, from a student question ([31:46]): a tray holds two CPUs,
each connected to four GPUs — eight GPUs to a tray — and the trays stack in a rack,
everything wired to the NVSwitch. The "G" in GB200 is **Grace**, NVIDIA's CPU.
(Percy prefaces this with "I'm not a hardware expert".)

**RoCE — RDMA over Converged Ethernet** ([28:41]). Ethernet that bypasses the CPU:
"this is sort of their answer to InfiniBand. So, InfiniBand generally is very
ex- expensive, as is a lot of NVIDIA products. But you can get pretty good
performance by using — using RDMA over Converged Ethernet." Meta has published on it, and "Llama may
or may not have been trained over Converged Ethernet" ([29:27]) — the hedge is
Percy's own.

## What follows for your parallelism strategy

The interconnect decides the technique, not the other way round
([1:15:51]–[1:16:37]):

- **[Tensor parallelism](tensor-parallelism.md)** moves activations every layer,
  so it "happens within a node, on NVLink" — trailing off mid-word
  as he reaches for NVSwitch — "where you have high bandwidth".
- **[Pipeline parallelism](pipeline-parallelism.md)** "can tolerate much slower
  interconnects. So, some of the decentralized training work uses
  pipeline-parallel" — the GPUs being, quite literally, "halfway across the world".
- **[Data parallelism](data-parallelism.md)** moves gradients once per step, which
  is cheap enough to cross nodes — until the critical batch size stops you.

A related practical point, from a student asking what happens with nine GPUs
([34:52]): if the ninth lands on another node without NVLink, "that's going to be
really bad, because that's going to be one node which is not providing that much
compute and also is very expensive to communicate with." Topology is not a
detail you can round off.

## See also

- [GPU architecture](gpu-architecture.md) — the hierarchy inside the chip.
- [torch.distributed and NCCL](torch-distributed.md) — the software that
  discovers this topology and routes around it.
- [Collective operations](collective-operations.md) — what gets carried over these
  links.
