# Introduction

Draft for §1, version 3 (2026-10-07), revised after the third review,
`../critisizm/fable_intro_review_2026-10-07.md`, which re-checked version 2
(2026-10-06, written after `../critisizm/astra_intro_review_2026-10-06.md`
and `../critisizm/fable_intro_review_2026-10-06.md`) against the mainline
tree of the day. It states the novelty against `io_uring` zero-copy receive
and against the kernel's unmapped mechanisms. Conventions are in
[outline.md](outline.md). Paper prose comes first; notes for the reviewer
follow.

---

Much of what a datacenter service does with network data is not read it. A
load balancer reads a request's header and relays its body. A cache stores a
value and serves it again. A broker delivers one message to many
subscribers. A key-value store keeps a value and returns it. We call these
*pass-through services*. Written against sockets, each of them still copies
every payload byte into the application and back out, and buffers the same
bytes several times over. At 100 Gbps the receive-side copy becomes the
bottleneck of a receiver core once messages reach about 16 KB; below that,
per-request processing costs more [cai21]. In production proxies, copies
across the kernel boundary take up to 55.6% of nginx's and HAProxy's cycles
[libra].

The kernel can already move received payload without copying it, but only
through owners that fix its fate. `splice` and sockmap forward it once, in
order. `tee` duplicates references, but only between pipes, which hold a
bounded FIFO of bytes and cost two descriptors for every held object.
`sendfile` sends one stored copy to any number of sockets, but only from a
file, which a copy must fill first. The goal of holding one copy for every
reader is old. IO-Lite shared a single physical copy among the file cache,
the network and applications [iolite], and the Cheetah web server
transmitted and retransmitted file data straight from its cache [cheetah].
PASTE kept received network buffers as stored values, mapped into the
application through netmap [paste], and `io_uring` already names kernel
buffers by index, for block requests [ublkzc]. None of these turns ordinary
TCP receive buffers into objects that an application can keep for as long
as it needs, slice at any offset and send to many sockets at once, on an
unmodified kernel and NIC.

The newest mechanism, `io_uring`'s zero-copy receive (`RECV_ZC`) [zcrx],
keeps the kernel TCP stack and maps the payload instead. The NIC splits each
packet: headers land in kernel memory and go through TCP, while payload is
written into memory the application registered in advance, where the
application reads it in place. The first cost is the pool itself. The
memory in which an application keeps payload is its receive memory,
registered and pinned up front for the peak, so every byte it holds is a
byte it can no longer receive into. The other costs are costs of placement.
To keep that memory safe, the NIC must keep headers out of it (header/data
split) and keep other flows' bytes out of it (flow steering to a queue of
the application's own), and the driver must fill that queue from the
application's pool. In Linux 7.3, four production drivers qualify:
Broadcom's bnxt, Google's gve, NVIDIA's mlx5 and Meta's fbnic. AWS's ENA,
Azure's netvsc and MANA, Intel's ice and idpf, virtio-net and veth do not
[zcrxdrivers]. virtio-net gained the page pool the mechanism builds on in
2026, with zero-copy receive as the stated goal, and still lacks the queue
API and header split in the driver, in the virtio specification and in
every backend [virtiopp]. Even on a capable NIC, payload that misses the
steered queue is copied into the pool instead [zcrxcopy]. Loopback and
veth, which carry the traffic of sidecars and other same-host proxies, have
no NIC to place anything; netkit can now lease a physical queue to a
container [netkit], which brings `RECV_ZC` to a pod's wire traffic, at the
cost of a queue and steering rules per pod, and brings nothing to traffic
that never leaves the host. These limits come from deciding who owns the
payload before it is stored, in application memory. More software can
extend where that decision is possible, as the netkit and virtio work does;
it cannot remove the decision.

SYS makes the opposite choice on both counts. It decides ownership after the
TCP stack, so there is no placement for the NIC to do, and it never maps the
payload, so pages need no isolation at page granularity. Received payload,
in whatever buffers the driver happened to use, becomes an *opaque data
object*: immutable, owned by the kernel, and named by a handle the
application holds. Pages may mix flows and carry headers. SYS needs no
header split, no flow steering, no dedicated queue and no driver change: it
is a small kernel series, and it works with any NIC, and over loopback and
veth.
Capture takes payload by reference wherever the driver delivers it in page
fragments and copies it otherwise; §7 reports the share for each driver.
Handles still need authorization, range checks and lifetime safety, and §4
describes how SYS provides them.

The object has a lifetime of its own, independent of the socket it arrived
on, so it can be held, sliced and sent by reference to any number of
sockets at once. Making that safe and cheap is the technical core of the
design. Network buffers were never meant to be kept: a held 1,448-byte
segment pins the 2–4 KB buffer the driver received it in, held pages leave
the socket's memory accounting, and one object may be in flight on many
sockets. SYS charges held payload to a memory budget, compacts fragmented
objects in the background, and gives each concurrent send its own cursor,
so that held payload costs bounded memory and the data path makes no copy.

One small interface serves every use: keep a range of the stream, read a
range on request, and send object ranges by reference. Forwarding, caching
and fan-out become uses of it, and the design lets the application load an
eBPF program that calls it in the receive path, so that cache hits over TCP
can be answered without waking the application (TODO(decide): not in v9,
ledger A15). Today each of these uses is
a system of its own, with its own kernel hooks, NIC features or user-space
stack: in-kernel L7 forwarding [libra, xlb, beeline], cache hits served in
eBPF over UDP [bmc], fan-out of single UDP packets [bpfbroker], and
zero-copy receive itself [zcrx].

We do not claim to move bytes faster than `RECV_ZC`. Where it runs, neither
copies payload, but reference counting, fragment layout and transmit
segmentation can still cost differently, and §7 measures them. Nor is the
kernel series the point. SYS is a small `io_uring` feature, and the design
would be the same under any asynchronous interface.

In short, SYS brings zero-copy retention and reuse of received payload to
every NIC, VM and container, with no per-application hardware setup, and
makes holding network buffers safe and bounded. This paper makes three
contributions:

1. **The opaque data object**: ordinary TCP receive buffers, captured after
   the stack and never mapped, turned into immutable, kernel-owned objects
   named by handles. Against designs that map payload, it needs no receive
   placement and no page-granular isolation; against designs that forward
   payload once, it gives the payload an owner with a lifetime of its own.
2. **A design that makes retention safe and cheap**: lifetimes, memory
   accounting, background compaction and concurrent sends for held network
   fragments, implemented as SYS, a five-patch kernel series with an
   `io_uring` interface (TODO(decide): the eBPF attach point is not in v9,
   ledger A15).
3. **An evaluation on commodity hardware** (TODO(measure)): one key-value
   store with its data path switched between copy, zero-copy send, a RAM
   file with `sendfile`, `splice` with `tee`, `RECV_ZC`, and SYS
   (TODO(decide): with and without the eBPF path, which v9 lacks), under
   equal memory budgets; the store against
   Redis, Valkey and memcached, from tens of bytes to megabytes; and the
   share of bytes captured by reference on physical and virtual NICs,
   loopback and veth.

---

## Notes for the reviewer

### What changed from version 2

Item numbers refer to `../critisizm/fable_intro_review_2026-10-07.md` §1.

- **Paragraph 3 opens the costs with the pool** (1.4): held payload is
  receive memory pinned for the peak. It is the delta that no driver or
  device change touches, so it comes before the placement requirements.
- **The census is dated** to Linux 7.3 and names Intel's ice and idpf
  (1.5). It was re-run on 2026-10-07 and stands.
- **virtio-net's trajectory is stated** (1.2): the page pool landed in 2026
  with zero-copy receive as its stated purpose; the queue API, header split
  and a steering control are missing in the driver, the specification and
  every backend. The sentence is there so that a reviewer who knows the
  series finds it acknowledged, and so that the structural point (the host
  must split and steer into the guest's buffers) carries the argument.
- **netkit is named** (1.1): a container on a capable NIC can now lease a
  physical queue, which gives `RECV_ZC` to its wire traffic at the cost of
  per-pod setup, and nothing to same-host traffic. The summary sentence is
  unchanged because its qualifier, "with no per-application hardware
  setup", is exactly what leasing does not remove.
- **"More software would not remove them"** became "can extend where that
  decision is possible ... cannot remove the decision" (1.3).
- **"A receiver core"** in the Cai sentence (1.7).

### What changed from version 1

Item numbers refer to the two 2026-10-06 reviews (Fable §1.x, Astra §x).

- **The unmapped relatives and the predecessors** are now in paragraph 2:
  `splice`, sockmap, `tee`, `sendfile`, IO-Lite, Cheetah, PASTE and ublk's
  kernel buffers (Fable 1.2, 1.11; Astra 1, 2). The claim against them is
  that they express retention poorly (FIFO, bounded, two descriptors per
  object, no slicing, a copy into a file), not that they cannot express it.
- **`RECV_ZC`** is no longer called the wrong tool. "Runs only on" became
  "qualify" and a driver census, the copy fallback is cited, and the cloud
  sentence names the NICs that lack it (Fable 1.1, 1.5; Astra 3). One
  correction to Fable: its verdict says zero-copy receive is unavailable on
  every cloud virtual NIC, but its own census lists gve, which is Google
  Cloud's virtual NIC. The intro therefore names AWS's and Azure's NICs and
  virtio-net, not "every cloud".
- **The pool limit** is a separate cost from placement (Fable 1.3;
  Astra 3); version 3 moved it first.
- **The principle has two clauses**: after the stack (no placement for the
  NIC) and never mapped (no page-granular isolation). `tcp_mmap` shows why
  both are needed: it decides after the stack but maps, so it still needs
  page-aligned payload (Fable 1.6). "Nothing needs isolating" is gone;
  handles still need authorization, range checks and lifetime safety
  (Astra 4).
- **"Any NIC" is qualified**: capture copies where the driver does not
  deliver page fragments, and §7 reports the share (Fable 1.7; Astra 5).
- **Parity with `RECV_ZC` is no longer asserted** (Astra 5). The eBPF hit
  path is stated as what the design allows, not as a result (Fable 1.4).
- **The copy sentence carries [cai21]'s 16 KB bound** (Fable 1.8).
- **Retention is contribution 2**: lifetimes, accounting, compaction and
  concurrent sends, the depth both reviews asked for (Fable 1.10; Astra,
  "What would strengthen the contribution").
- **The evaluation** adds `splice` with `tee` and equal memory budgets
  (Astra 2, 3), and leads with the controlled variants of one store, since
  beating Redis alone would not isolate the contribution (Astra). The `tee`
  variant is not yet in `../experiments/kv_store.md` §2.6.
- **The summary sentence before the contributions** is the claim the owner
  approved on 2026-10-06; its three legs are in `novelty.md`, "The claim,
  in one sentence".

### Status

Nothing in this section is measured. The prototype differs from the design:
it owns its sockets and uses a fixed pool (`novelty.md`, "Where the current
prototype differs from this design"). The eBPF hit path is not built, and
the module-defined `struct_ops` attach point it needs is `TODO(verify)`
(`../KV_CACHE.md` §8.1).

### Facts and sources

- [zcrx] Kernel documentation, "io_uring zero copy Rx": header/data split,
  flow steering and RSS are required; buffers return through a refill ring.
  Verified.
- [zcrxdrivers] Census of `torvalds/linux` master, first taken 2026-10-06
  (`../critisizm/fable_intro_review_2026-10-06.md` §2) and re-run
  2026-10-07 at `VERSION = 7 PATCHLEVEL = 3`
  (`../critisizm/fable_intro_review_2026-10-07.md` §2): `queue_mgmt_ops`
  present in bnxt, gve, mlx5, fbnic, netdevsim and netkit; absent in ice,
  idpf, virtio_net, ENA, netvsc, MANA and veth. Verified against raw files;
  `TODO(verify)` on a checked-out tree at the submission kernel, since the
  `../ceph-client` tree was not available.
- [zcrxcopy] `io_uring/zcrx.c` on master: `io_zcrx_copy_chunk` and
  `io_zcrx_copy_frag`, called for fragments outside the zero-copy area.
  Verified 2026-10-07. The notification series ("io_uring/zcrx: add CQE
  based notifications", Léger and Begunkov, v2, 2026-05-18) adds counters
  for the same fallback.
- [virtiopp] "virtio_net: add page_pool support", Vishwanath Seshagiri,
  v5 2026-02-06 to v11 2026-03, in mainline master. Cover letter: "a
  prerequisite for enabling memory provider-based zero-copy features in
  virtio_net, specifically devmem TCP and io_uring ZCRX". Verified. No
  split-header feature in `include/uapi/linux/virtio_net.h` or in the
  virtio specification's network device chapter on master; the
  `VIRTIO_NET_F_SPLIT_HEADER` proposal (v7, 2023) was not merged. Verified
  by grep.
- [netkit] "netkit: Support for io_uring zero-copy and AF_XDP", Daniel
  Borkmann, v11 2026-04-02, in mainline master; tested on ConnectX-6 and
  BCM957504. LWN, 2026-03-27. Verified. Framing talk: "The case for
  zero-copy in containers", LPC 2025-12-13.
- [cheetah] Kaashoek et al., "Application Performance and Flexibility on
  Exokernel Systems", SOSP '97, p. 10, the merged file cache and
  retransmission pool. The PDF landed as `paper/268998.266644.pdf` and is
  not yet filed.
- [libra] An unrefereed preprint, Figure 1a; to be replaced by our own
  Stage 0 numbers (bar-raiser R9).
- [cai21] §3.7, verified 2026-10-07: "with just 16KB RPCs, data copy
  becomes the dominant factor and with 64KB RPCs, the CPU breakdown
  becomes very similar to the case of long flows." Setting: an incast of
  16 ping-pong RPC clients onto one receiver core at 100 Gbps; the 4 KB
  breakdown is TCP/IP processing, scheduling and skb work.
- The 1,448-byte segment pinning a 2–4 KB buffer is from `novelty.md`, "What
  was hard"; `TODO(measure)` per driver.
- [bmc], [ublkzc], [iolite], [paste], [xlb], [beeline], [bpfbroker] as in
  `prior-work.md` and `problem.md`. [bmc] is not yet in `../refs/`.

### Claims deliberately not made

- That SYS is faster than `RECV_ZC` per byte, or at parity before §7
  measures it.
- That only SYS could provide these features: they are kernel software, and
  `io_uring` could add them to its own buffers (`novelty.md`, "The zcrx
  test, per feature").
- That existing mechanisms cannot hold or fan out payload: `tee` can, within
  a pipe.
- That SYS beats in-kernel forwarding (`splice`, sockmap); against those the
  claim is parity.
- That `RECV_ZC` is unavailable in containers: netkit brings it to a pod's
  wire traffic on a capable NIC, with per-pod setup.
- That virtio-net will never qualify: the page pool is in, and the rest is
  work someone may do. The claim is that the host must then do the
  placement, which is the same decision one level down
  (`../maio/ideas.md` §8).
