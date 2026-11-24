# Prior work

Draft for §8, Related work. Conventions are in [outline.md](outline.md). The
evidence behind each statement, with links and verification status, is in
`../RELATED_WORK.md`. Paper prose comes first; author notes follow.

## Unified buffering

IO-Lite [iolite] is the closest work in spirit. It unifies buffering across
the file cache, the network stack and applications, so that all of them share
"a single physical copy of the data safely and concurrently". I/O data lives
in immutable buffers, and subsystems exchange buffer aggregates, lists of
slices into those buffers. SYS shares IO-Lite's goal and its use of
immutability. It differs in how sharing is made safe. IO-Lite, like fbufs
[fbufs] before it, maps buffers read-only into the application, protects them
with access-control lists, and requires network drivers to demultiplex packets
early so that each packet is stored in a buffer its recipient may see. It also
replaces the file cache. SYS never maps payload, so it needs neither the
access control nor the driver support, and it leaves the rest of the kernel
unchanged.

## Network buffers as stored data

PASTE [paste] lets an application keep received packet buffers as its durable
data on non-volatile memory, and its key-value store answers reads by placing
a stored buffer back on the transmit ring. SYS shares the idea that a received
buffer can become the stored object. PASTE exposes the NIC's DMA target to the
application through netmap; its authors note that isolating several
applications requires partitioning by hardware, and that fixed-size packet
buffers leave space utilization as future work. SYS keeps buffers in the
kernel behind handles, works with any driver, and treats the memory cost of
holding buffers as a design problem (§4).

## Zero-copy receive into application memory

`io_uring` zero-copy receive [zcrx] delivers payload into a registered,
user-mapped area. It requires header/data split, flow steering and dedicated
receive queues, and the area is a fixed pool that the application must return
buffers to. Page-remapping receive [tcpmmap] requires page-aligned payload.
MAIO [maio] remodels the kernel for it: it adds I/O-only pages to the memory
manager and new zero-copy paths to TCP, and relies on header split and
per-process hardware rings to keep processes and kernel memory apart. Device-memory TCP [devmem] places
payload in device buffers the CPU does not read, for supporting NICs.
SocksDirect [socksdirect] keeps the socket API over RDMA and shared memory
and remaps pages into the application for zero copy, which requires
page-aligned buffers of page-multiple size and falls back to copying
otherwise. These systems solve a harder problem than pass-through services
pose: they give the application the bytes. SYS gives it a name, and so needs none of their
platform support: a small kernel series with no driver change and no NIC
feature.

## In-kernel forwarding

Splicing two connections inside the kernel is an old idea [tcpsplice], and
proxies have long used it for message bodies after reading headers in
userspace [rosu03, sosplice]. Linux offers `splice` and sockmap [sockmap] for
the same purpose. Rosu and Rosu's proxy support hands the application a copy
of spliced data when the object must be cached [rosu03]. All of these transfer
payload once. SYS's forwarding path uses the same kernel mechanisms; its
contribution is that the payload can also be held and reused.

## Selective copy for unmodified proxies

Libra [libra] is concurrent, unpublished work that reaches the same
observation from the opposite direction. It keeps the proxy unmodified: eBPF
programs in the kernel's copy routines parse HTTP metadata, copy only the
headers to the application, and leave the body anchored in the receive
socket's queue; an eight-byte placeholder stands in for the body in the
application's buffer, and the transmit path resolves it back to the anchored
data. Libra shows that selective copy recovers most of an L7 proxy's copy
cost on commodity hardware, and it argues that zero copy cannot be reconciled
with POSIX receive semantics, which is why SYS changes the API rather than
the kernel. The two designs are duals: Libra modifies the kernel and keeps
the socket API; SYS keeps the kernel and gives the application a handle.
Libra's transparency therefore rests on a modified kernel, a per-protocol
eBPF program and an interposition library; SYS asks the application to
change instead. Because Libra's payload remains in the receive socket's
queue, holding it shrinks the TCP window, so anchored payload is bounded by
the receive buffer, lives only as long as its connection, is forwarded in
stream order, and goes to exactly one socket. A SYS object leaves the queue
when it is captured; it has a lifetime of its own and can be sliced, held,
and sent to any number of sockets at once, and flow control is re-engaged
only as a response to memory pressure. Libra's implementation is not
available, so §7 reports our forwarding results on its axes rather than
against it.

## Kernel buffers named by handle

ublk zero-copy [ublkzc] registers a request's kernel pages in an `io_uring`
buffer table, so that a userspace block server can direct I/O on them by
index without mapping them. This is the same handle model as SYS. Those
buffers live for one block request and originate in the block layer. SYS
applies the model to network receive and gives the object a lifetime of its
own.

## Holding data in the kernel for sending

`sendfile` sends page-cache pages by reference, and servers such as nginx and
Kafka [kafka] rely on it to send one stored copy to many clients. It is the
existing mechanism closest to SYS's hold-and-reuse path, and we compare
against it (§7). Its limits are that network data must be copied into a file
first and that every object is a file. In-kernel web accelerators [tux, nca,
httpsys] also kept content in the kernel and sent it without copying, by
moving the server and its policy into the kernel. SYS leaves the server in
userspace and keeps only the bytes in the kernel. ZCopy [zcopy] sends cached
values from user memory without copying, for the send side only.

## Reducing the cost of copies

zIO [zio] tracks I/O data through unmodified applications and skips copies
they do not need. Copier [copier] performs copies asynchronously as a kernel
service and merges chained copies. Both keep payload in application buffers.
They are complementary to SYS, which removes the payload from the application
altogether.

Valkey 9.0 advertises "zero-copy responses" [valkey9release], implemented as
reply copy avoidance [valkey9reply]. It references stored strings directly
instead of copying them into userspace reply buffers. Its ordinary TCP
`writev` path still copies payload into kernel send buffers [valkey9src].
This removes a redundant userspace copy present in Redis 7.2.4 [redis724reply];
it does not provide zero-copy network transmission or kernel-held storage.
SYS instead captures payload in kernel objects and sends page references. Valkey is a
baseline for separating these benefits from eliminating an avoidable
reply-buffer copy.

## KV-cache systems for LLM inference

A line of work stores and moves the attention KV cache of language models.
CacheGen [cachegen] compresses KV caches and streams them over the network.
Mooncake [mooncake] organizes serving around a distributed KV-cache store.
LMCache [lmcache] layers KV caches across GPU memory, host memory and remote
stores. DirectKV [directkv] removes copies between CPU and GPU memory. These
systems decide what to cache, where, and in what form. SYS is complementary:
it is a substrate for the store itself, which holds each chunk once and sends
it by reference to every node that asks.

## Bypassing the proxy or the kernel

Prism [prism] hands a client connection from an object-storage proxy to the
backend, so that the body never passes through the proxy. Kernel-bypass stacks
[ix, mtcp, demikernel, cornflakes] give applications ownership of packet
buffers. Both change the deployment model: the backend must cooperate, or the
kernel stack and its tooling are given up. SYS keeps the proxy in the path and
the kernel stack in place.

---

## Author notes

### Citation keys used here

`iolite`, `fbufs`, `paste`, `zcrx`, `tcpmmap`, `maio`, `devmem`, `tcpsplice`,
`rosu03`, `sosplice`, `sockmap`, `libra`, `socksdirect`, `ublkzc`, `kafka`, `tux`,
`nca`, `httpsys`, `zcopy`, `zio`, `copier`, `valkey9release`, `valkey9reply`,
`valkey9src`, `redis724reply`, `cachegen`, `mooncake`, `lmcache`,
`directkv`, `prism`, `ix`, `mtcp`, `demikernel`, `cornflakes`.

Reply-path sources, verified on 2026-10-03 (details in `../RELATED_WORK.md`):

- `valkey9release`: [Valkey 9.0 release announcement](https://valkey.io/blog/introducing-valkey-9/).
- `valkey9reply`: [PR #2078, Reply Copy Avoidance](https://github.com/valkey-io/valkey/pull/2078).
- `valkey9src`: Valkey 9.0.0 [`src/networking.c`](https://github.com/valkey-io/valkey/blob/9.0.0/src/networking.c)
  and [`src/socket.c`](https://github.com/valkey-io/valkey/blob/9.0.0/src/socket.c).
- `redis724reply`: Redis 7.2.4 [`src/networking.c`](https://github.com/redis/redis/blob/7.2.4/src/networking.c),
  `addReplyBulk`, `addReply` and the output-buffer copy helpers.

### Still to verify before this goes into LaTeX

- TODO(verify): the SocksDirect sentence rests on Libra's description of it;
  check the alignment requirement against the SocksDirect paper.
- Candidate after a read: Hermes (SIGCOMM '25), L7 load balancing with
  userspace-directed I/O event notification, cited by Libra.
- TODO(verify): the fbufs sentence, against the fbufs paper itself. It
  currently rests on IO-Lite's description.
- TODO(verify): Kafka's use of `sendfile`, with a citable source.
- TODO(verify): page-remapping receive requires page-aligned payload; cite the
  kernel documentation or the original patch description.
- TODO(verify): whether [zio] or [copier] report memory. `problem.md` says
  they do not.
- Candidates to add after a read: AccelTCP (NIC-offloaded splice), StackMap,
  the Exokernel Cheetah server's merged cache and retransmission pool,
  "Disk|Crypt|Net".
- Read on 2026-10-04 and now in Table 2 (`novelty.md`), not yet in the
  prose above: XLB, Beeline and BPF-Broker, as one paragraph on eBPF data
  paths built for one case, potential consumers of the object
  infrastructure (`../refs/systems.md`, Patterns); the encrypted zero-copy
  L7 load balancer [zcl7lb]; flow-based addressing [flowaddr].
- Candidates from `../refs/blogs.md` (read 2026-10-03; vendor documentation
  and blogs, so one sentence at most): cuObject as the hardware counterpart
  of the handle, S3 control over HTTP with the payload moved by RDMA under a
  token; vLLM's tiered offloading as the client architecture, in which the
  inference node is an endpoint and the server behind a tier is the
  pass-through side. Nothing is added to the paper prose above yet.

### Anonymity

- MAIO and the TCP-split work are ours. They stay in the third person.
- Do not cite our patents or the conference talk.
- [libra] is unrefereed and concurrent, and is argued against in a full
  paragraph as if it were published (decided 2026-10-03). If it is accepted
  somewhere before submission, change "concurrent, unpublished" to the venue
  and nothing else. No priority argument: the talk predates it, but the
  claims do not collide on hold-and-reuse. The paragraph is not a candidate
  for cutting.

### Length

This draft is about one page in two-column format, more with the Libra
paragraph. The budget is 0.75 pages; the first cuts are the bypass paragraph
and the web accelerators. The Libra paragraph stays.
