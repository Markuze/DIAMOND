# Novelty

Draft for the contributions paragraph of §1 and the framing of §3.
Conventions are in [outline.md](outline.md). Paper prose comes first; author
notes follow.

Brought up on 2026-11-15 to the owner's version of the paper (the owner:
"Yes, update it"). That version is in `outline.md` under "The story", with
its part "Novelty and contributions"; where this file and that part differ,
the outline is the record. Of that version here: the claim, the insight, the
contributions, what is new, the system's two rows of Table 2, the table of
claims and evidence, and the end of "The zcrx test". The claim of
2026-10-06, which covered the named mode alone, is kept as a record at the
end of the file. The system is DIOMND; text recorded before 2026-11-03 says
SYS.

## The claim, in one sentence

DIOMND gives the kernel's network stack zero copy that needs no NIC feature,
on receive as on send: the kernel keeps payload in its own memory and lends
access to it, by name to any application and by mapping to one trusted with
the device, and both modes are as fast as or faster than the existing
zero-copy mechanisms.

The sentence joins two statements of the owner. The claim word, from the
title of 2026-11-14: hardware agnostic, "the common motive between mapped
and named". The claim the evaluation tests, from 2026-11-11
(`../eval/2026-11-11-fixed-objects/EVAL.md` §1): the named and the mapped
mode perform alike, and both are as fast as or faster than the existing
zero-copy mechanisms. The wording of the sentence is proposed; the owner has
not read it. Its half about speed is a claim under test, with no number
behind it yet ("Claims and the evidence each needs").

The three legs of 2026-10-06 stay, because the outline's evaluation tables
are labelled with them. Restated for both modes:

1. **Where.** With no NIC feature, in both modes. The named mode needs
   nothing of the NIC or its driver, and no privilege. The mapped mode needs
   a driver hook and privilege over the device. Against the designs that
   receive into memory mapped or assigned to an application (`RECV_ZC`,
   devmem TCP, MAIO, PASTE): those settle who owns a packet's bytes before
   the packet is stored, which takes something of the NIC: header split and
   flow steering into a queue or a ring of the application's own, or a port
   of its own (`../RELATED_WORK.md`, "What isolation costs it"). State it as
   availability, not convenience: DIOMND is a kernel series that someone
   must apply while `RECV_ZC` is in mainline, and both need application
   changes.
2. **What.** Payload with an owner of its own: an object that outlives its
   connection and is kept, cut and sent to many sockets, and that a mapped
   application also reads and changes in place. Against the kernel's
   unmapped mechanisms (`splice`, `tee`, sockmap, `sendfile`), which express
   it poorly: forward once, a bounded FIFO pipe, or a copy into a file.
3. **How.** The design that keeps holding bounded and cheap
   (`../docs/design_spec.md` §2.5, §2.8, §3.3): an object is a fixed-size
   head and a chain of fixed-size segments from arrays made at registration,
   the memory cgroup is charged once, nothing is allocated while data flows,
   and a full store fails a request at once. For a mapped area, a lazy
   collection returns the buffers of objects that were sent. Background
   compaction, part of this leg on 2026-10-06, is deferred in the design
   (§2.10).

## The insight

Zero copy has been built many times, each time for one case and under a
condition. `sendfile` sends a file and `splice` forwards a stream, inside
the kernel and in the ways the kernel planned for. `MSG_ZEROCOPY` and
`SEND_ZC` send from the application's memory, which is pinned and must stay
unchanged until the kernel reports it free. Zero-copy receive delivers
payload into memory the application registered, and needs a NIC that splits
headers from payload and steers the application's flows into a queue of
their own.

The conditions have one cause. The memory that packets arrive in holds every
application's bytes and the headers the kernel acts on, and the kernel does
not trust an application with it. That distrust is two questions: can the
application harm the kernel, and can it reach another application's bytes.
Each mechanism is an answer, and its condition is what the answer costs.
Some keep the application away from the bytes. Some use the application's
own memory and guard the kernel against it with a pin and a report.
Zero-copy receive has the NIC answer both before a packet is stored: header
split keeps the headers the stack acts on in kernel memory, and flow
steering keeps other flows out of the application's.

So what zero-copy receive asks of the NIC is asked for trust, and trust can
be had without the NIC. DIOMND keeps payload in the kernel's own memory and
lends each application the access it can be trusted with.

Any application may hold payload by name. It declares a range of a TCP
stream an object, and keeps, cuts and sends it by handle, copying out only
the bytes it reads. Nothing is mapped, so neither question arises: a page
may mix flows, headers may share a page with payload, and whatever buffer
the driver used will do. The object is still the application's own data, in
kernel memory, as a file in ramfs is. What a file lacks is a way in without
a copy, and that is what a name gives received payload.

An application trusted with the device may map the memory itself and work on
payload in place. The memory is set apart from everything the kernel relies
on, and the kernel parses its own copies of the headers, so a mapper cannot
harm the kernel. A mapper can reach every flow on the device, so registering
the memory takes `CAP_NET_ADMIN`.

Three things follow from who keeps the memory. Who owns a packet's bytes is
settled after the stack has processed the packet, so nothing is asked of the
NIC. The memory's lifetime is the kernel's, so sending an object pins
nothing and waits for no report. And payload has an owner that is not a
socket, a pipe or a file, so it outlives its connection and can be kept, cut
and sent to many sockets at once. Each of the kernel's own owners fixes the
fate of the bytes: a receive queue, where holding them shrinks the sender's
window; a pipe, where they leave once and in order; a file, which a copy
must fill first.

The two modes divide the applications between them. One that moves or keeps
bytes uses names, and copies out the little it reads. One that reads or
changes them maps, is trusted with the whole device, and needs a driver that
takes its receive buffers from that memory.

The policy stays where the protocol state already is. An application parses
its own headers; with a handle it also says what happens to each byte range,
so no second parser is needed in the kernel. Transparent designs that keep
the socket API must instead mirror the application's protocol state in the
kernel, in a restricted language, once per protocol [libra]. DIOMND asks the
application to change, and in exchange keeps policy out of the kernel.

This is the story of `outline.md` in prose, its steps 2 to 5 and 7. The
steps are proposed on the owner's directions and await the owner's comments,
and so does the reading of header split and flow steering as the two halves
of trust.

## Contributions

Four, as `outline.md` has them under "Novelty and contributions": told back
to the owner on 2026-11-14 and written on the owner's word, with the single
items awaiting comments. They follow the list that the plan of 2026-10-31
gives for §1.

1. **The argument.** The conditions of today's zero-copy paths have one
   cause, trust, and trust is two questions: whether the application can
   harm the kernel, and whether it can reach another application's bytes.
   Each existing mechanism is placed by how it answers them and by what its
   answer costs.
2. **The named mode: opaque data objects.** An application with no
   privilege, on any NIC and with no driver change, declares a range of a
   TCP stream an object and keeps, cuts and sends it by handle. Nothing is
   mapped, so nothing has to be isolated. The object is the application's
   own data in kernel memory, with an owner that is not a socket, and
   holding it is bounded by a store whose size is fixed when it is
   registered.
3. **The mapped mode.** A process trusted with the device maps the memory
   that supplies the device's receive queues and works on payload in place.
   The kernel acts on its own copies of the headers, so the mode needs a
   driver hook and no header split, no flow steering and no hardware ring.
   Its objects are held and sent by handle, as named objects are.
4. **The evidence** (TODO(measure)). The implementation in Linux, as
   `io_uring` operations, and five experiments at full speed, on one core
   and on four: send, receive and forwarding by message size,
   request/response, and a key-value store against memcached and Redis.
   They carry the claim that the two modes perform alike and are as fast as
   or faster than the existing zero-copy mechanisms.

The three contributions of 2026-10-06 map onto these. The opaque data object
and the retention design are both in 2, without background compaction, which
the design defers. The evidence is 4.

## What is new and what is not

Not claimed as new:

- Forwarding payload between sockets inside the kernel.
- Sharing one physical copy between a cache and the network as a goal.
  IO-Lite states it [iolite].
- Keeping a received buffer as stored data. PASTE does it with mapped netmap
  buffers [paste].
- Sending kernel pages by reference. The kernel's own splice path does it.
- Copying only the headers to the application and keeping the body in the
  kernel for forwarding. Rosu and Rosu [rosu03], SO_SPLICE [sosplice] and
  Libra [libra] do it.
- Memory dedicated to I/O and mapped into the application. MAIO does it
  [maio], with header split and a hardware receive ring for each process.
- Trusting one privileged process with a device's receive memory. MAIO
  without such rings is limited to a privileged user.

Claimed as new:

- The reading of zero copy's conditions as the price of trust, and of what
  zero-copy receive asks of the NIC as two answers to it: header split for
  the kernel's safety, flow steering to keep flows apart.
- Zero copy through the kernel's stack with no NIC feature, by lending
  kernel memory in two modes that share one object: a name for any
  application, the memory itself for one trusted with the device.
- Received payload with an owner of its own: an object that outlives its
  connection, is cut and sent concurrently, and whose holding is decoupled
  from the connection's flow control until its store is full.
- Holding and reusing received payload *without mapping it*, and the
  observation that this removes the isolation requirement that every design
  which maps received payload carries.
- Mapping a device's receive memory without header split: the kernel acts on
  private copies of the headers.
- Treating the memory cost of holding network buffers as a design problem:
  holding is bounded by construction, and what it costs in capacity is
  measured.

## Table 2: property matrix

The last two rows describe the design the paper presents, one for each
mode.

| System | Stock kernel | Stock drivers, any NIC | Payload never mapped | Held and reused | Copies to hold network data | Unmodified applications |
| --- | --- | --- | --- | --- | --- | --- |
| IO-Lite [iolite] | no | no | no | yes | 0 | no |
| PASTE [paste] | yes | no | no | yes, whole packet buffers | 0 | no |
| io_uring zcrx + send_zc [zcrx] | yes (in-tree) | no | no | limited by a fixed pool | 0 | no |
| MAIO [maio] | no | no | no | not its goal | 0 | no |
| zIO [zio] | yes | yes | no | no | n/a | yes, with an interposition library |
| Copier [copier] | no | yes | no | no | n/a | TODO(verify) |
| splice, sockmap | yes | yes | yes | no | n/a | no |
| Page cache + sendfile | yes | yes | yes | yes | 1 | no |
| Libra [libra] | no | yes | yes | no, forward once within the connection | n/a | yes, with an interposition library and a kernel program per protocol |
| XLB [xlb] | no (patched Linux 5.10) | yes | yes | no, forward once | n/a | yes |
| Beeline [beeline] | yes, plus a 172-line module | yes | yes, for policies enforced in the kernel | no, forward once | n/a | yes, and unmodified proxies |
| BPF-Broker [bpfbroker] | yes | yes | yes | no; one UDP packet cloned per subscriber, then dropped | n/a | no, its own protocol |
| Encrypted zero-copy L7 LB [zcl7lb] | bypassed: userspace TCP stack | no: NVIDIA NICs with TLS offload | no | no, forward once | n/a | no |
| MSG_ZEROCOPY [msgzerocopy] | yes (in-tree) | yes | no: user memory, send side only | no | n/a | no: a send flag and completion notifications |
| Packets are not pages [flowaddr] | n/a: runs on the NIC | no: custom SmartNIC | n/a | not its goal; reads payload in place | n/a | n/a |
| **DIOMND, named** | **no: a kernel series, with one receive claim in `net/`** | **yes** | **yes** | **yes** | **0 for payload taken by reference; shared or cloned fragments are copied at capture** | **no** |
| **DIOMND, mapped** | **no: a kernel change, with a receive-buffer provider** | **a driver hook; no NIC feature** | **no: mapped read-write into a process trusted with the device** | **yes, as objects in the mapped area** | **0 for payload that arrived in the area; other input is copied into it** | **no** |

The last two rows lose their first and last columns by design: DIOMND is a
kernel change, and it asks the application to change. In exchange the named
mode needs nothing from drivers, NICs or per-application setup, and the
mapped mode needs a driver hook and no NIC feature. The copies in their
sixth column are those of `../docs/design_spec.md` §2.3 and §3.2; the row of
2026-10-06, with its deferred copy for data that stays, is in the record at
the end of the file. "n/a" means the system does
not hold network data. The rows from XLB to Packets are not pages were added
on 2026-10-04 from the requirements tally in `../refs/systems.md`. Packets
are not pages runs on the NIC, so the kernel columns do not apply to it; XLB
and Beeline are preprints, cited as evidence. TODO(decide): add "reclaimable by the kernel" and
"application reads payload in place" as columns, or pair this table with a
two-axis figure (who can address the bytes, against lifetime) and the
ownership table (`../critisizm/review_2026-10-02.md` §1.8).

## What was hard

The series builds on existing kernel paths (`tcp_read_sock`,
`MSG_SPLICE_PAGES`), so the paper must say what those paths do not give.
Each item below is a design subsection in §4. v9 settles the rows marked
"settled" (`../PROJECT_MAP.md` §7 item 2).

The table is as of v9 and covers the named mode. The design of 2026-11-11
(`../docs/design_spec.md`) changes it in three places:

- **Accounting.** The memory cgroup is charged once, at registration, for
  the store's whole limit and for its arrays (§2.8).
- **Budget and refusal.** The limit is the size of the store's arrays. A
  request that finds no free head or segment fails at once with `-ENOMEM`; a
  capture pass stops, and the bytes it did not take stay in the socket, so
  TCP's flow control slows the sender (§2.8).
- **Fragment slack and transmit granularity.** Compaction, which both rows
  lean on, is deferred (§2.10).

The mapped mode's problems are not in the table: what a driver must do for
the provider, the quiet test, and the lazy collection of a sent object's
buffers (§3.3 to §3.5).

| Problem | Why existing interfaces do not solve it |
| --- | --- |
| Many sends on one object | The by-reference send path takes pages for one send. Concurrent sends need objects separate from operations, with a cursor per send. Settled in v9: ledger A7. |
| Fragment slack | A held 1448-byte segment pins the 2–4 kB buffer the driver received it in. A cache of raw fragments can use more memory than one that copies. |
| Accounting | Once consumed from the receive queue, pages are no longer charged to the socket. Held payload would be invisible to memory limits. Settled in v9: held pages are charged to the store owner's memory cgroup (A3, A17). |
| Budget and refusal | Capture consumes from the receive queue, so holding never throttles the sender; that is the point, and it means the holder must bound memory itself. When the budget is reached the store stops capturing and TCP's own flow control slows the writer. Settled in v9: a per-store budget reserved before bytes leave TCP, a nonterminal `ENOBUFS` CQE, automatic resume, collection held on undecided bytes so TCP flow control applies, and NORMAL/LOWMEM watermarks with an advisory forwarding-only class (A17, A18, A19). |
| Same-host senders | Over loopback and veth the received packet references the sender's pages: page-cache pages under `sendfile`, user pages under `MSG_ZEROCOPY`, which the sender may still change. Immutability needs those fragments copied at capture. Settled in v9: cloned and shared fragments are copied at capture (`opaque_capture_clone_copy`, `opaque_capture_shared_copy`; A1). |
| Two producers of replies on one byte stream | If the kernel answers cache hits while userspace answers misses and writes on the same connection, replies must stay contiguous on the wire and, for unmodified RESP clients, in request order. A per-connection reply queue with slots reserved in arrival order in the receive path gives both for free: the kernel fills hits, userspace fills the rest by slot ID, the drain is in order for RESP and any order for protocols that carry request IDs. TODO(decide), with the hit path (`../KV_CACHE.md` §8.1). |
| Reading on demand | The application needs headers, and sometimes a range inside the payload, without mapping the object. Settled in v9: INSPECT and READ_OBJECT copy bounded ranges of at most 64 KiB (A2). |
| Transmit granularity | A transmit packet holds at most 17 fragments on a distribution kernel, so MSS-sized received fragments make two or three packets where a copying sender makes one. Compaction, jumbo frames and a larger fragment limit each recover it at a different cost; measured three ways (Q6). |

## Claims and the evidence each needs

For the claim above. The status words are those of `../CLAIMS.md`: measured,
argued (a design consequence, said "by design" and never with a number), not
measured. No number is given here; a number comes from its row. The rows
that exist were measured on kernels that predate the design's object
metadata (`../docs/status-and-lineage.md`), and the rerun of 2026-11-11
restates them (`../eval/2026-11-11-fixed-objects/EVAL.md` §9). Row numbers
marked "proposed" are proposals of that plan or of
`../eval/PAPER-CAMPAIGNS.md`.

| Claim | Evidence it needs | Where it stands, 2026-11-15 |
| --- | --- | --- |
| No NIC feature is needed: the named mode runs on any NIC with no driver change, the mapped mode with a driver hook | The availability table: per environment, whether `RECV_ZC`, the mapped mode and the named mode register (campaign 4, proposed row F15) | Measured on virtio-net only: `RECV_ZC` refuses it and the named mode runs there (F5); the mapped mode gives a `CAP_NET_ADMIN` process zero-copy receive there (F6, a functional fact). Other environments: not measured |
| The two modes perform alike | `rx` and `fwd` of the rerun: named capture against capture into the mapped area and raw mapped receive; the named relay against the bound and the raw mapped relay (proposed F22, F25) | Not measured on the redesigned build |
| Both are as fast as or faster than the existing zero-copy mechanisms | `tx` against registered `SEND_ZC`, like for like, and against `send()`; `fwd` against `splice`; `rr` by size (S1–S6, S11 and S12 restated; proposed S21, F25, F28) | Not measured on the redesigned build |
| Receive costs less than a copy where hardware zero-copy receive does not run | `rx` against `recv()` and an `io_uring` copying receiver (proposed F22 with F21, restating F24) | Not measured on the redesigned build |
| Level with hardware zero-copy receive where a NIC provides it | Campaign 1 (GCP, gve): the mapped mode against hardware `RECV_ZC`, and the named relay against a `RECV_ZC` into `SEND_ZC` relay (proposed F13) | Not measured |
| Sending an object pins nothing and waits for no notification | A design consequence (S10). What a pin costs a sender that borrows: S2 and S26 | Argued. The pin's cost is measured; the cost of waiting for a notification is a design argument |
| Payload outlives its connection: kept, cut, sent to many | `kv`, whose reads serve one held object to many connections (proposed K2, K3). Retention and fan-out as experiments of their own are campaign 2 (proposed R1–R4) | Not measured |
| Holding is bounded, and nothing is allocated while data flows | Gates 1 and 2 of the rerun (`EVAL.md` §7): no allocation and no cgroup charge under capture, publish, send and `FREE`; a store with too few segments returns `-ENOMEM`, loses no byte and resumes after a `FREE`. Capacity at equal memory: proposed K4, restating K1 | By design. The gates have not run. Capacity is measured (K1) |
| A key-value store on named objects against memcached and Redis | `kv` (proposed K2) | Not measured |

## The zcrx test, per feature

The owner (2026-10-06): every feature must answer what its delta is from an
application that uses `IORING_OP_RECV_ZC` and holds the payload in its own
memory. The table assumes a NIC that supports zero-copy receive. Without
one (most virtual NICs, and loopback and veth always), every feature's delta
is the whole mechanism: commodity hardware.

The table is of 2026-10-06. It compares the named mode, as v9 had it, with a
`RECV_ZC` application, and the in-kernel features it names (a hit path, a
program that sends on arrival) are not built. The end of the section, from
"What software cannot remove", was brought up on 2026-11-15; its version of
2026-10-31 is in the record at the end of the file.

| Feature | The RECV_ZC application | Delta on a capable NIC |
| --- | --- | --- |
| Cache hits answered in the kernel over TCP | Cannot today: its payload is marked unreadable to the kernel and its values are its own memory, so every request wakes userspace | **Today's design.** No wakeup and no ring round trip per hit. An io_uring extension could give kernel code access to zcrx buffers, so this is not fundamental |
| One payload to many receivers | Sends the same area buffer N times with `send_zc` and tracks N completions before reusing it | **Ownership.** The kernel tracks lifetime and keeps the object immutable; copies and memory are equal. Structural only when the fan-out runs in the receive path, as an eBPF program sending on arrival |
| Keeping what is forwarded, to retry, mirror or replay it | Keeps the area buffers and sends them again | **Sizing.** Held buffers come out of the receive pool, so the area is sized for peak holding. Under TLS, see the next row |
| A proxy that terminates TLS | Kernel TLS cannot decrypt payload marked unreadable (`TODO(verify)` in the kernel's TLS receive path) | **Not in v9.** v9 refuses a socket whose protocol operations were replaced, which kernel TLS does: `io_opaque_attach()` returns `EBUSY` (ledger A16). Capture after kernel TLS decryption through the TLS `read_sock` path is a design change, not a feature of the current series |
| Rewriting headers around an untouched body | Writes new headers in its own memory and sends them with the area body | **In-kernel, today's design.** An in-kernel L7 program, as in Beeline, cannot read zcrx payload today; in userspace the two are equal |
| Holding without starving receive | Keeps area buffers, so the area doubles as cache and receive pool, or copies out lazily | **Memory management.** The area is registered and pinned up front for the peak; SYS charges held pages to a budget as they are kept. Both can defer one copy for what stays |
| Many applications on one NIC | Needs a queue and steering rules per application; a software mode without steering (`../maio/ideas.md` §10) gives the whole device to one privileged process instead | **Density** |

The owner (2026-10-06): zcrx buffers are host pages with kernel addresses,
so anything added to the object infrastructure could be added to io_uring
for the same effect. For the features this is true. Kernel access to the
payload, kernel TLS, eBPF hooks and kernel-tracked lifetimes are kernel
software, and zcrx marks its payload unreadable by policy, not necessity
(device memory under devmem TCP is the exception). The rows above marked
"today's design" are deltas against zcrx as it is, not as it could become.

What software cannot remove is the choice that mapping forces. Payload
mapped into an application has to be kept apart from every other flow's
payload and from the headers the kernel acts on, and there are two ways to
do it. Hardware: the NIC places only the chosen flows into the registered
memory, which needs header split (headers must not land in application
memory), flow steering and a queue per application (other flows must not
either); loopback and veth have no such queue. Privilege: the receive
buffers of the device are mapped into one process that is trusted to see all
of them.

Behind the two ways are the two questions of the story (`outline.md`, "The
story"): header split answers whether the application can harm the kernel,
and steering or privilege answers whether it may see other flows' bytes. The
way of privilege settles only the second, so a design that takes it still
has to keep the kernel's headers out of the application's reach. MAIO [maio]
dedicated memory to I/O and mapped it once into the process. It kept headers
off the shared pages by assuming header/data split, and kept processes apart
with a dedicated hardware receive ring for each; without such rings it was
limited to a privileged user (`../refs/systems.md` §35, from §3.2 and §3.3
of the paper; the owner, 2026-11-14: MAIO "also relied on HW based RX
rings"). AF_XDP in its zero-copy mode is one privileged process over a
queue's frames, with the frames it passes to the stack copied
(`TODO(verify)`: the drivers' `*_construct_skb_zc`).

DIOMND's mapped mode takes the way of privilege and needs no hardware for
either question: a kernel-allocated area supplies every receive queue of the
device, with no header split and no steering rule, because the kernel parses
its own copies of the headers (`../docs/design_spec.md` §3.4, §3.5; built
2026-10-31, `../apps/uapi/ADDONS.md`). It runs on a driver that takes its
receive buffers from the area and keeps the headers it parses private; a
page pool alone is not enough (§3.5). It adds no privilege, since every zcrx
registration already requires `CAP_NET_ADMIN`, the NODEV mode included,
because a queue can observe data destined for other tasks' sockets
(`../talks/advisor_deck.md` slide 8, checked against master 2026-10-14). In
every mapped design the mapped memory is the receive pool, which holding
drains; the queue or the device belongs to one application, or to those it
passes its trust to; and nothing crosses a VM boundary, because no guest may
see the host's receive memory (`../maio/ideas.md` §10).

The named mode needs neither way. Named objects settle ownership after the
TCP stack, in software, on whatever buffers the driver used, and never map
them, so they need neither the hardware nor the privilege.

The two modes are one answer: trust settled in software, in memory the
kernel keeps. That is the novelty, and the features show that it is general.
The refusal of zcrx on virtio-net, measured on our VMs (F5 of
`../CLAIMS.md`; `../FINDINGS-LEDGER.md` entry 13), is the state of mainline:
an instance of the argument, not its basis.

---

## Author notes

### Wording to hold to

Later decisions changed some of the rules below (noted 2026-11-15; each
decision is in `outline.md`):

- The name is DIOMND (2026-11-03). The modes are named and mapped, with no
  "read-only" or "read-write" in their names (2026-11-14).
- The claim word is "hardware agnostic", from the title of 2026-11-14, and
  "zero copy" names the genre (2026-10-31). The rule below that the claim
  word is the object is superseded.
- The redesign is the system (2026-11-14). No number recorded before it in
  which DIOMND trails is argued from, and the paper never sets earlier
  prototypes against a revised design.
- Compaction is deferred in the design (2026-11-11). The rule below on
  Table 2's "0 copies to hold" describes v9.
- The five-patch series is v9; what exists now is in
  `../docs/status-and-lineage.md`. "No driver, NIC or per-application
  hardware setup" is said of the named mode. The mapped mode needs a driver
  hook and privilege over the device, and no NIC feature.
- "Pins nothing and waits for no notification" is said of sending an
  object. The mapped mode's raw path, `RECV_ZC` into `SEND_ZC`, keeps the
  notification and pins per send (`../docs/design_spec.md` §3.3).
- Open, for the owner (`outline.md`, "The story"): "kernel memory dedicated
  to the user" and not "user memory"; "dedicated" or "set apart" against
  "segmentation".

The rules as recorded:

- Say "no driver, NIC or per-application hardware setup". Never "stock
  kernel", "no kernel patch" or "loadable module": v9 is a five-patch
  kernel series, one patch in `net/` (the owner, 2026-10-10;
  `../PROJECT_MAP.md` §1).
- Raising the per-packet fragment limit for
  transmit is a kernel build option, acceptable as a measured optimization;
  the text must say which configuration each result used.
- Say "not bounded by a pre-registered pool", not "unlimited caching".
- "Fastest" is stated against stores that use the kernel TCP stack (Redis,
  Valkey, memcached). The eBPF path removes the userspace round trip, not
  the stack's own per-packet work, and pipelining lets a conventional store
  spread its system calls over many requests, so report both settings.
- Do not claim "without duplicating memory" in numbers until Q5 is measured.
- Small payloads: say where the break-even is rather than claiming a win
  everywhere.
- The "0 copies to hold" entry for SYS in Table 2 means: nothing in transit,
  and at most one copy for data that stays, made in the kernel after the
  reply, only when the object's slack makes it worth it. Compaction is a
  dial, not a requirement; without it the premium is about 1.4 bytes pinned
  per payload byte on drivers that pack two segments per page (TODO(measure)
  per driver). `../LLM.md` §4 says compaction is required and should be
  softened to match.
- Never write "no backpressure". Write "holding is decoupled from flow
  control; flow control is re-engaged under memory pressure".
- Never call the server a KV cache; it is the chunk store or the tensor
  store. "KV cache" is the tensors.
- Say who owns the retained bytes (a receive socket, a pipe, a file, an
  object) before saying what SYS can do; the capabilities follow from the
  owner.
- "Zero copy" is the genre, not the claim. The claim word is the object.
  Use "zero copy" for the family of prior work and for the data path's
  property, never as the contribution.
- RDMA is out of scope. Say once, in §2, that the target is commodity
  hardware and cloud infrastructure without RDMA, cite a published RDMA
  transfer figure as the ceiling if a reader needs one, and never argue
  against RDMA.
- Never "zero latency" for the hit path. Say "served in the receive path
  with no system call and no process wakeup", and report p50 and p99
  against the userspace path. If a kernel worker performs the send, say so.
- The hit path's BPF program executes a lookup the store decided; it holds
  no policy and no protocol state of the store's. Say that wherever it
  appears, so the paper's argument against policy mirrors in the kernel
  (Libra) stays intact.

### Sources for the rows added to Table 2 on 2026-10-04

Each row follows the requirements tally in `../refs/systems.md`, where the
entries give sections and reading status.

- [xlb] Wang, Shou, Qian, Liu, arXiv 2602.09473 (2026-02-10), unrefereed.
  `../refs/xlb-arxiv2602.09473v1.pdf`, systems.md §24.
- [beeline] Brandner et al., "Enforcing Application-Layer Policies in eBPF",
  arXiv 2605.31084, v4 of 2026-08-13, unrefereed.
  `../refs/beeline-arxiv2605.31084v4.pdf`, systems.md §33.
- [bpfbroker] Zhou, Al-Kiswany, Tahmasbi Arashloo, "Toward eBPF-Accelerated
  Pub-Sub Systems", eBPF '25. `../refs/bpfbroker-ebpf25.pdf`, systems.md §31.
- [zcl7lb] Podleś et al., "Towards encrypted zero-copy L7 load balancing",
  ANRW '26. `../refs/zcl7lb-anrw26.pdf`, systems.md §25.
- [msgzerocopy] de Bruijn, Dumazet, "sendmsg copy avoidance with
  MSG_ZEROCOPY", Netdev 2.1, 2017. `../refs/msgzerocopy-netdev17.pdf`,
  systems.md §11.
- [flowaddr] Adams et al., "Packets are Not Pages: Flow-Based Addressing
  Conserves Memory Bandwidth", CoNEXT '26. `../refs/flowaddr-conext26.pdf`,
  systems.md §34.

### Where v9 differs from this design

A record of v9: "the design above" is the claim and the design of
2026-10-06, now in the record at the end of this file. How each kernel that
exists differs from the current design is in
`../docs/status-and-lineage.md`.

From `../PROJECT_MAP.md` §3 and §5: v9 implements the design above on
ordinary application descriptors, with a per-store byte budget in place of
a fixed pool, a cursor per concurrent send and background compaction. Not
in v9: the BPF hit path (A15), capture after kernel TLS (A16), crc (A13),
FILL (A14), a slice that yields a new handle, multi-object gather and
WRITE. The design text must not claim these as built.

### Kernel-series claim, checked

A record of v9. What exists now is in `../docs/status-and-lineage.md`.

Superseded on 2026-10-10 (the owner; `../README.md` "Decisions taken"):
the loadable module is historical. v9 is five patches on Linux 7.3-rc6
(`af32da41b032`): patch 1 adds an optional exclusive TCP receive claim in
`net/` (`CONFIG_SOCK_RX_OWNER`); patches 2-5 add the `io_uring` feature,
its KUnit tests, its selftest and its ABI documentation
(`../PROJECT_MAP.md` §1). The data path uses `tcp_read_sock` and
`sock_sendmsg` with `MSG_SPLICE_PAGES`. Registration needs no capability
and charges locked memory (ledger A3).

### Reviewer notes (2026-10-03)

Pointers into `../critisizm/review_2026-10-02.md` for items that belong in
this file's prose once decided:

- The cost model (§2.13): fixed per message, per fragment, nothing per
  byte; SYS pays a page reference per fragment at capture and at each send,
  which compaction and jumbo frames reduce. Predict the break-even in §4.
- The safety case (§2.5): the four invariants (immutable, confined,
  conserved, live) and "a handle never confers a mapping"; range checks are
  the whole guarantee.
- Encryption (§2.6): kernel TLS has had a `read_sock` path since 6.6, so
  capture can run after decryption with one crypto pass and no copy; TLS
  halves the gain rather than erasing it. Not in v9, which refuses kernel
  TLS sockets (A16).
- Transforms (§2.12): compaction, compression, hashing and encryption are
  one mechanism, a kernel operation that reads an object and produces a new
  one; claim only the one built.
- The transmit-granularity decision and the stock-kernel wording (§8.2):
  headline numbers on the stock configuration; the build option as an
  ablation.

---

## Record: the claim of 2026-10-06

Kept as it stood before 2026-11-15, for the reviews and notes that refer to
it (`../critisizm/`, `intro.md`, `../talks/advisor_deck.md`). It covers the
named mode alone, calls the system SYS, counts background compaction in the
design, and puts MAIO with the designs that isolate by privilege. It does
not say where the paper stands; the sections above do.

### The claim, in one sentence (2026-10-06)

SYS brings zero-copy retention and reuse of received payload to every NIC,
VM and container, with no per-application hardware setup, and makes holding
network buffers safe and bounded.

Decided 2026-10-06 (the owner: "yes I like the framing"). The claim has
three legs, each against a different line of prior work:

1. **Where.** Any NIC, in VMs and containers, with no per-application
   hardware setup. Against zero-copy receive into application memory (zcrx,
   devmem TCP, MAIO, PASTE): they decide ownership before DMA, so they need
   header split, flow steering and a queue per application, and do not run
   on AWS's ENA, Azure's netvsc and MANA, virtio-net, veth or loopback. SYS
   captures after the stack and never maps. State it as availability, not
   convenience: SYS is an out-of-tree kernel series someone must apply
   while zcrx is in mainline, and both need application changes, so the
   management claim is "no per-application hardware configuration", not
   "simpler overall".
2. **What.** Received payload that can be kept, sliced, sent to many sockets
   or served from the kernel. Against the kernel's unmapped mechanisms
   (`splice`, sockmap, `tee`, `sendfile`), which express it poorly: forward
   once, a bounded FIFO pipe, or a copy into a file.
3. **How.** A retention design that keeps held network buffers bounded and
   safe: lifetimes, memory accounting for fragments, background compaction,
   concurrent sends. This is the technical depth, and it holds whatever zcrx
   becomes.

Leg 1 alone would be a deployment paper; legs 2 and 3 make it a systems
paper.

The object itself, in the 2026-10-03 wording: SYS makes the data a service
does not read an *opaque data object*: the kernel holds the bytes, the
application holds a name, and neither copies nor interprets them. One
physical copy can then be forwarded, held and sent to many sockets at once,
with stock drivers, any NIC and no per-application hardware setup.

Framing decided 2026-10-03: zero copy via shared opaque data objects, across
a breadth of network-bound uses. An object, unlike a stream, is something
you hold: a socket's stream can only be consumed, while an object can be
kept, sliced and sent again. Compaction of held objects runs in the
background, so the system chooses when to spend memory bandwidth and CPU and
the heavy cost moves off the data path. The exact claim is therefore zero
copy on the data path, and at most one deferred copy for what is kept. "Opaque" means opaque by default: the
application sees a fixed-size prefix and can ask to read a range. The object
is opaque to the kernel as well, which applies no policy of its own; policy
comes from userspace, directly or as programs it loads.

### The insight (2026-10-06)

The kernel can already move received payload without copying it: `splice`
forwards page references from one socket to another, and `sendfile` sends
page-cache pages to any number of sockets. What it cannot do is let an
application name received payload, keep it, slice it, or send it to many
sockets at once. Each existing path ties the bytes to an owner that fixes
their fate: a receive queue, where holding them shrinks the sender's window;
a pipe, where they leave once and in order; a file, which a copy must fill
first. SYS gives received payload an owner of its own, an immutable,
reference-counted object with a lifetime independent of any socket, and
gives the application a handle to it.

Not mapping that object is what makes the design cheap. Zero-copy receive
has always meant giving the application the bytes. Once payload is mapped
into a process, the system has to guarantee that the page
holds only that process's data and no kernel headers. It must therefore know
where a packet is going before it stores the packet. Every prior design pays
for that guarantee: early demultiplexing in drivers, header/data split, flow
steering to dedicated queues, per-application hardware rings, or a new class
of memory in the kernel.

A pass-through service does not read the payload. If the application is given
a name instead of the bytes, the guarantee is no longer needed. Pages may mix
data from different flows, headers may share a page with payload, and any
buffer the driver happens to use is acceptable. The cost of isolation
disappears, and with it every hardware and kernel requirement.

Not mapping has a second consequence. A mapped buffer belongs to a
pre-registered pool, so an application that holds data starves receive. A
named kernel object has no such pool. It can be held for as long as memory
allows and referred to by any number of concurrent sends.

The policy stays where the protocol state already is. A pass-through service
parses its own headers; with a handle it also says what happens to each byte
range, so no second parser is needed in the kernel. Transparent designs that
keep the socket API must instead mirror the application's protocol state in
the kernel, in a restricted language, once per protocol [libra]. SYS asks the
application to change, and in exchange needs no kernel change and no policy
outside the application.

### Contributions (2026-10-06)

Decided 2026-10-06 with the claim above: three contributions, as the reviews
of 2026-10-02 and 2026-10-03 and both introduction reviews of 2026-10-06
recommended.

1. **The opaque data object** (legs 1 and 2). Ordinary TCP receive buffers,
   captured after the stack and never mapped, turned into immutable,
   kernel-owned objects named by handles. Against designs that map payload,
   it needs no receive placement and no page-granular isolation; against
   designs that forward payload once, it gives the payload an owner with a
   lifetime of its own. It is a small kernel series inside `io_uring`, with
   no driver change, NIC feature or per-application hardware setup.
2. **A design that makes retention safe and cheap** (leg 3): lifetimes,
   memory accounting for held fragments, background compaction and
   concurrent sends. The memory result is stated as copy placement: nothing
   copied in transit, and at most one deferred copy for data that stays.
   TODO(decide): the compaction policy.
3. **The evidence** (all three legs; TODO(measure)). One store with its data
   path switched between copy, zero-copy send, a RAM file with `sendfile`,
   `splice` with `tee`, `RECV_ZC`, and SYS (TODO(decide): "with and without
   the eBPF path" needs a hit path that v9 lacks, ledger A15), under equal
   memory budgets; the store against Redis, Valkey and memcached; the
   applications of §6; and the portability count across physical and
   virtual NICs, loopback and veth.

The five contributions of 2026-10-03 map onto these: "a design that needs
nothing from the platform" is part of 1, memory-efficient holding is 2, and
the applications and the evaluation are 3.

### What is new and what is not (2026-10-06)

Not claimed as new:

- Forwarding payload between sockets inside the kernel.
- Sharing one physical copy between a cache and the network as a goal. IO-Lite
  states it [iolite].
- Keeping a received buffer as stored data. PASTE does it with mapped netmap
  buffers [paste].
- Sending kernel pages by reference. The kernel's own splice path does it.
- Copying only the headers to the application and keeping the body in the
  kernel for forwarding. Rosu and Rosu [rosu03], SO_SPLICE [sosplice] and
  Libra [libra] do it.

Claimed as new:

- Received payload with an owner of its own: an object that outlives its
  socket, is sliced and sent concurrently, and whose holding does not
  throttle the sender.

- Holding and reusing received payload *without mapping it*, and the
  observation that this removes the isolation requirement that every
  zero-copy receive design carries.
- Doing so with no driver, NIC or per-application hardware setup.
- Treating the memory cost of holding network buffers as a design problem and
  measuring it.

### The system's row of Table 2 (2026-10-06)

| System | Stock kernel | Stock drivers, any NIC | Payload never mapped | Held and reused | Copies to hold network data | Unmodified applications |
| --- | --- | --- | --- | --- | --- | --- |
| **SYS** | **no: a five-patch series, one patch in `net/`** | **yes** | **yes** | **yes** | **0 in transit; at most 1 for data that stays** | **no** |

The last row loses its first and last columns by design: SYS is a kernel
series, and it asks the application to change; in exchange it needs nothing
from drivers, NICs or per-application setup.

### Claims and the evidence each needs (2026-10-06)

| Claim | Evidence |
| --- | --- |
| No copy on receive or send | TODO(measure): cycles/byte against copy, splice, sockmap (Q1) |
| Forwarding at parity with in-kernel forwarding, with control it lacks | TODO(measure): the TCP-split relay built with copy, splice, sockmap and SYS; capability table |
| No hardware needed | TODO(measure): portability table across virtual and physical NIC drivers, and the comparison with zcrx + send_zc on a header-split NIC, required (the owner, 2026-10-06; `outline.md` §7 Q2) |
| Hardware zero-copy receive is not enough for pass-through services (the owner, 2026-10-06). It runs on few drivers and never on loopback or veth; on any NIC, holding drains its receive pool and each application needs its own queue (`problem.md` §2.4) | Availability and measurement. The portability table's availability columns (header/data split, flow steering, zcrx registers) with loopback and veth rows, plus a driver census from the kernel tree; the count goes in the introduction (bar-raiser R5). On a header-split NIC, `outline.md` §7 Q2: parity on the relay, and the holding, fan-out and density cases |
| Faster than hardware zero-copy receive on small values (2026-10-06). Stock eBPF has no general way to answer on a TCP connection with a stored value: sockmap programs redirect the packets they are handed, and BMC answers GETs only over UDP and passes TCP to userspace. SYS's hit path would exist because the series adds it (TODO(decide): v9 has no BPF hook, ledger A15 is T): a struct_ops attach point called from the collector's data-ready path, and kfuncs that read a window of the stream and queue an object or bytes on the socket (`../OBJECTS.md` §9), (the first version is `TODO(verify)` in `../KV_CACHE.md` §8.1). With zcrx as designed, kernel code would have nothing to work on: the payload is marked unreadable (`skb_frags_readable()` false, `../OBJECTS.md` §5) and the values are application memory, not kernel objects a kfunc can send, so it pays a completion, a wakeup and a send per request. An io_uring extension could give kernel code access to zcrx buffers, but not remove zcrx's placement requirements ("The zcrx test, per feature"). The claim is that the object infrastructure gives eBPF a TCP fast path on any NIC, not that eBPF alone can, nor that zcrx never could | `outline.md` §7 Q2 (e): GETs per second per core, p50 and p99, SYS with the hit path against the zcrx store; on large values the claim is parity per byte where zcrx runs (Q2 a) |
| One copy for N receivers | TODO(measure): memory and CPU against N (Q3) |
| Lower memory per connection | TODO(measure): bytes per connection at scale (Q4) |
| Holding is memory-efficient | TODO(measure): bytes pinned per payload byte, with and without compaction (Q5) |
| The send path matches a copying sender's packet rate | TODO(measure): transmit packets and descriptors per MB for compacted objects, the stock scatter-gather of received fragments, and a larger fragment limit (Q6) |
| Faster at every object size. Small objects mean more operations, and the in-kernel (eBPF) hit path removes the per-request cost. Large objects mean more bytes, and opaque objects remove the per-byte cost. (2026-10-03) | TODO(measure): CPU per request and per GB against Redis, Valkey and memcached from 64 bytes to 64 MB, pipelining off and on, with the eBPF path off and on (`KV_CACHE.md` §10.2–10.3). One figure with object size on the x-axis. |
| No driver, NIC or per-application hardware setup | Implementation section: the five-patch series (one patch adds an optional exclusive TCP receive claim in `net/`), the kernel paths it uses, the kernel version built against (Linux 7.3-rc6, `af32da41b032`, at v9) |
| All of the performance of a remodeled kernel, for most of its use cases, as an add-on (the owner, 2026-10-04). MAIO [maio] needs a modified memory manager, new TCP paths, header/data split and per-process rings; SYS needs no NIC feature and no driver change; its kernel change is five patches, one of them an optional exclusive TCP receive claim in `net/`. The use cases it leaves out are those that read every byte, which MAIO maps into the application | TODO(measure): the relay's cycles per byte against splice and sockmap on our testbed (Q1). MAIO reports about 10% fewer cycles per byte than both for TCP splicing, on its own testbed, because it skips the splice abstraction and eBPF; SYS skips both too (`tcp_read_sock` to `MSG_SPLICE_PAGES`), so the relay should show the same margin. That margin goes beyond the parity row above, so the two rows must be reconciled once Q1 is measured |
| Useful to real applications | TODO(measure): the applications of §6 (Q7) |

### The end of "The zcrx test", as restated on 2026-10-31

What software cannot remove is the choice that zcrx's placement model
forces. Payload mapped into an application has to be kept apart from every
other flow's payload, and there are two ways to do it. Hardware: the NIC
places only the chosen flows into the registered memory, which needs header
split (headers must not land in application memory), flow steering and a
queue per application (other flows must not either); loopback and veth have
no such queue. Privilege: every receive buffer of the device is mapped into
one process that is trusted to see all of it. MAIO [maio] takes the second
way, a kernel pool mapped once into a `CAP_NET_ADMIN` process, with the
kernel deciding after the headers are parsed whether a page goes to the
process by reference or stays an ordinary packet; so does AF_XDP in its
zero-copy mode, one privileged process over a queue's frames, with the
frames it passes to the stack copied (`TODO(verify)`: the drivers'
`*_construct_skb_zc`). SYS's mapped mode is of the same kind: a
kernel-owned area on every receive queue of the device, no header split,
no steering rule, zero-copy receive on any driver with a page pool
(`../maio/ideas.md` §10; built 2026-10-31, `../apps/uapi/ADDONS.md`). It
adds no privilege, since every zcrx registration already requires
`CAP_NET_ADMIN`, the NODEV
mode included, because a queue can observe data destined for other tasks'
sockets (`../talks/advisor_deck.md` slide 8, checked against master
2026-10-14). Either way the mapped memory is the receive pool, which holding
drains; one application owns the queue or the device; and nothing crosses a
VM boundary, because no guest may see the host's receive memory
(`../maio/ideas.md` §10). Opaque objects decide ownership after the TCP
stack, in software, on whatever buffers the driver used, and never map them,
so they need neither the hardware nor the privilege. That model is the
novelty; the features show that it is general. The refusal of zcrx on
virtio-net, measured on our VMs (`../FINDINGS-LEDGER.md` entry 13), is the
state of mainline: an instance of the argument, not its basis. If io_uring
took the model in, capturing ordinary receive buffers as kernel-owned
objects named by an index (as it already names ublk's kernel buffers), that
would be this design inside io_uring, whose transport SYS already uses.
