# Paper outline

Target: OSDI '27. Abstract registration 2026-12-01, full paper 2026-12-08.
12 pages plus references, double-blind.

**Two versions, since 2026-11-14.** This file holds the owner's version of
the paper: its story is "The story" and its title is under "Working titles".
The LaTeX draft in `osdi27/` holds the colleague's revision of 2026-11-13. It
is not changed from here, and "The draft as it stands" describes it. The two
stand in parallel until they are discussed and the outgoing version is
decided (the owner, 2026-11-14: "lets ammend only the outline I would rather
keep Shays work alive in parallel so we could discuss and decide on the
actual outgoing version").

This file is the skeleton. Section drafts live beside it and will become the
LaTeX sources:

| File | Feeds | State |
| --- | --- | --- |
| [intro.md](intro.md) | §1 Introduction | version 3 (2026-10-07), after three reviews; contrasts with `io_uring` zero-copy receive, census dated to Linux 7.3, netkit and virtio_net absorbed |
| [problem.md](problem.md) | §2 Borrowing and its costs (Background and motivation before 2026-10-31) | first draft |
| [novelty.md](novelty.md) | §1 contributions, §3 overview | brought up on 2026-11-15 to the owner's version ("The story", "Novelty and contributions"): the claim, the insight, four contributions, and the evidence each part of the claim needs. The claim of 2026-10-06, for the named mode alone, is kept at its end as a record |
| [prior-work.md](prior-work.md) | §9 Related work | first draft |
| `../RELATED_WORK.md` | evidence and verification notes behind `prior-work.md` | working notes |
| `../KV_CACHE.md` | §4 Design, §7 Applications, §8 Evaluation: the store's design and experiments | proposed design |
| `../LLM.md` | the LLM workload: related work, Stage 0, VM runbook | working notes |
| `../experiments/llm_inference.md` | controlled comparison design for the LLM evaluation | proposed plan |
| `../KPASS.md` | §6 Implementation | code reference for the current prototype |
| `../PROJECT_MAP.md` | §6 Implementation, §8 Evaluation: what exists in v9, what the tests prove, the advantage ledger | the implementation side's map; read-only here |
| `../experiments/evaluation.md` | §7 Applications, §8 Evaluation: what to build and compare, every experiment mapped to ledger rows, what v9 lacks | proposed 2026-10-13 |
| `../apps/uapi/ADDONS.md`, `../maio/ideas.md` §10 | §5 The mapped mode: the ABI of the MAIO mode of `RECV_ZC` and the assessment behind it | built 2026-10-31; no numbers yet |
| [osdi27/](osdi27/README.md) | the LaTeX sources: `main.tex`, one file per section under `sections/`, `macros.tex`, `refs.bib`; USENIX template | started 2026-10-31 from these drafts; cut to the 12-page limit on 2026-11-13 (`ac90ed9`) and revised in full the same day (`fcba7b4`, `9dc23a7`); what it holds now is "The draft as it stands" under Sections; numbers from `../CLAIMS.md`, `\todom` where unmeasured; since 2026-11-14 the colleague's version, kept in parallel with the owner's and not changed from this file |
| [quic-extension/main.tex](quic-extension/main.tex) | no section: a standalone note that recommends extending the paper to QUIC, first with named UDP datagram objects and a connection-ID-aware pass-through relay | added 2026-11-13 with the revision; a proposal with no implementation and no measurement, outside the paper's build; a line in `../FUTURE.md` |

## Conventions

- **DIOMND** (Dedicated I/O Memory for Network Devices) is the system's name
  in the paper (the owner, 2026-11-03: "I said use DIOMND thats better"; KIOM
  was the earlier candidate). OSDI requires a name that differs from any used
  publicly, and the code uses a public one. In LaTeX `\sys` expands to DIOMND
  and `\sysfull` to "dedicated I/O memory for network devices" at first use
  (abstract, §1); both live in `osdi27/macros.tex`.
- **The design, and what exists** (2026-11-11). §3, §4 and §5 describe the
  design of `../docs/design_spec.md`, which is the current design and nothing
  else (the owner, 2026-11-11: "tech speck is only the latest and greatest");
  when it changes, these sections change with it. §6 says what exists and
  keeps it apart from the design: the twelve-patch RFC series for net-next
  with its two-patch liburing companion, the separate four-patch net/core
  splice series, and the evaluation kernel, which alone carries the mapped
  mode and is the source of every measurement. At this date none of the three
  carries the design's object metadata (heads and segments from fixed arrays,
  the magazine allocator, the lazy collection of a mapped area's buffers), and
  every number in §8 predates it. Since the revision of 2026-11-13 the draft
  says so in its text: the second paragraph of §3, the end of §5, §6
  ("Evaluated builds", "Planned changes and evaluation"), the opening of §8
  and §8.7, with one marker in §6 and one in §8.7; §6 no longer gives the
  series' patch counts. `../docs/status-and-lineage.md` has the differences.
  No evaluation number is a measurement of either series, and the paper never
  says the measured kernels carry "the series".
- **The redesign is the system** (the owner, 2026-11-14). Wherever a recorded
  number shows DIOMND behind a baseline, the cause was found to be an
  implementation bug in the object life cycle, and the redesigned build that
  fixes it is being measured: "anywhere where diamond follows we found it to
  be an implementation bug in the object life cycle ant its being being fixed
  and measured now". So the owner's version describes and measures one
  system. Its claim is the one the rerun tests
  (`../eval/2026-11-11-fixed-objects/EVAL.md` §1): the named and the mapped
  mode perform alike, and both are as fast as or faster than the existing
  zero-copy mechanisms. Its numbers come from that rerun through
  `../CLAIMS.md`. It never argues from a number recorded before the redesign
  in which DIOMND trails, and it does not set "earlier prototypes" against a
  "revised design". The repository records the redesign and its target on
  receive (`../eval/2026-11-11-fixed-objects/DESIGN.md` §2), not yet the
  finding itself; until the rerun's record has it, it stands here as the
  owner's statement.
- Citations are written as `[key]` and will become BibTeX keys.
- `TODO(measure)` marks a number we must produce. `TODO(verify)` marks a fact
  not yet checked against its source. `TODO(decide)` marks an open decision.
- Our own earlier work is cited in the third person.
- **Opaque data object** is the paper's term for the abstraction (decided
  2026-10-03): user data that the kernel holds and userspace names, which
  neither side copies or interprets. "Data", not "kernel": the object is
  the application's data, not a kernel resource. In the draft since the
  revision of 2026-11-13 the term appears once, in §1; the abstract, the
  headings and the rest of the text say "named objects" and "the read-only
  named mode".
- **Two modes: named and mapped** (the owner, 2026-11-14: "we don't have read
  only mapped there is no such design or plan"). The named mode maps nothing.
  The mapped mode maps its area read-write. No read-only mapped mode exists
  or is planned, so "read-only" and "read-write" are not part of the modes'
  names in the owner's version. They are in the colleague's draft.
- No result appears in any draft until it has been measured.

## The story (2026-11-14, the owner's version)

This section is the frame of the owner's version: what the abstract and §1
say, and in what order. The title that concludes it is under "Working
titles". What it claims as new and what the paper contributes are in its
last part, "Novelty and contributions".

Where it comes from: the owner's directions of 2026-10-31 (lend, don't
borrow, in two modes on par), of 2026-11-03 (zero copy is widely studied and
never without a price; trust, as security and safety, is the cause; the
novelty is the angle, not a percentage) and of 2026-11-14 (the corrections
and the title's reasoning, quoted below), with the claim the rerun tests
("Conventions"). The arrangement in seven steps and the table are proposed on
those directions (2026-11-14). The owner asked for them to be written here
("update and push") and has not commented on the single steps.

1. **Why it matters.** Copying payload between the kernel and applications
   takes a large part of the CPU that network services spend.
2. **Many answers, each with a condition.** Zero copy has been built for
   sending, for receiving and for forwarding. Each mechanism serves one case
   and asks for something: a use the kernel planned for, a buffer pinned and
   left alone until the kernel reports it free, a NIC that splits headers and
   steers flows.
3. **One cause.** The memory packets live in holds every application's bytes
   and the kernel's own state, and the kernel does not trust an application
   with it. The distrust is two questions: can the application harm the
   kernel (safety), and can it reach other applications' bytes (security).
   Each mechanism has its own answer, and the conditions of step 2 are what
   the answers cost. Some keep the application away from the bytes
   (`sendfile`, `splice`, sockmap). Some use the application's memory and
   guard the kernel against it, with a pin and a report of when the memory is
   free again (`MSG_ZEROCOPY`, `SEND_ZC`). Some have the NIC answer both
   before a packet is stored: header split for the first, flow steering into
   a queue of the application's own for the second (`RECV_ZC`, devmem TCP,
   and MAIO with its hardware rings).
4. **The turn.** DIOMND answers both questions without the NIC, in memory the
   kernel keeps and dedicates to payload, and lends that memory by what each
   application is trusted with. A name, to any process: nothing is mapped, so
   neither question arises. The memory itself, to a process trusted with the
   device: the payload memory is set apart from everything the kernel relies
   on and the kernel parses its own copies of the headers, so the kernel
   cannot be harmed; the mapper can reach every flow on the device, so it
   holds `CAP_NET_ADMIN`.
5. **What follows.** Ownership of the bytes is settled after the stack has
   processed the packet, so nothing is asked of the NIC: hardware
   independence is a consequence of the design, and it is what the title
   names. The memory's lifetime is the kernel's, so sending an object pins
   nothing and waits for no report. Payload has an owner that is not a
   socket, so it outlives its connection and can be kept, cut and sent to
   many. The named mode serves applications that move or keep bytes, the
   mapped mode those that read or change them.
6. **Evidence, last.** Linux, as `io_uring` operations. The five experiments
   of 2026-11-11: send, receive and forwarding by message size,
   request/response, and a key-value store against memcached and Redis, on
   one core and four. They carry the claim under "Conventions", on a NIC
   where mainline zero-copy receive does not register (F5). At most three
   numbers in the abstract, each a `../CLAIMS.md` row from the rerun. The
   sentence "level with hardware zero-copy receive" is fed by campaign 1 of
   `../eval/PAPER-CAMPAIGNS.md`, which is outside the five.
7. **Limits, once.** A named application copies out what it reads. A mapped
   one is trusted with the whole device, and its driver needs a hook.

The two questions of step 3, and who answers them:

| | Can the application harm the kernel? | Can it reach other flows' bytes? |
| --- | --- | --- |
| `RECV_ZC`, devmem TCP, MAIO | No: header split keeps the headers the stack acts on in kernel memory | No: flow steering gives the application a queue of its own (MAIO: a hardware ring per process, or a privileged user where there is none) |
| Named mode | No: nothing is mapped | No: nothing is visible |
| Mapped mode | No: the payload memory is set apart, and the kernel parses its own copies of the headers | Yes, so the mapper must hold `CAP_NET_ADMIN` |

The first row follows the kernel's documentation of `RECV_ZC`
(`networking/iou-zcrx`, read 2026-11-14: headers are received into kernel
memory and payloads into the application's; flow steering keeps other flows
off the queue) and, for MAIO, `../RELATED_WORK.md` ("What isolation costs
it"). Reading the two features as the two halves of trust is proposed here.

What the owner said on 2026-11-14, and where it sits in the story:

- **MAIO.** "MAIO is an early predecessor but it also relied on HW based RX
  rings." MAIO is the mapped mode's predecessor: it dedicated memory to I/O
  and mapped it into the process. It kept headers off the shared pages by
  assuming header split, and kept processes apart with a hardware receive
  ring for each; without such rings it was limited to a privileged user
  (`../refs/systems.md` §35, from §3.2 and §3.3 of the paper). It is one of
  the designs of step 3, never a design in which receive was already free of
  the NIC. The mapped mode keeps the dedicated memory and the privileged
  user, and puts the kernel's own copies of the headers in the place of
  header split. The other designs that share I/O memory
  with an application also isolate before a packet is stored
  (`../RELATED_WORK.md`: IO-Lite demultiplexes in the driver; PASTE needs a
  port of its own, or hardware partitioning for more than one application).
- **The common motive.** Of the title "HW agnostic ZC for the network stack":
  "thats the common motive between mapped and named and we achieve this via
  kernel memory segmentation which allows a net_admin user to access kernel
  memory safely and we can claim that the named objects are also a form of
  user memory. There is already a precedent of dedicating kernel memory to
  the user in the form of ramfs for example." So:
  - The claim word is hardware agnostic, the one both modes share.
  - The mapped mode is safe because the memory it maps is set apart, and it
    is given to a `CAP_NET_ADMIN` process (step 4).
  - A named object is kernel memory dedicated to the user: the user's data,
    charged to the user's memory cgroup once, the user's to keep, cut, send
    and free, as a file in ramfs is. That is what makes the named mode
    zero-copy receive and not only forwarding. It agrees with the convention
    of 2026-10-03: the object is the application's data, not a kernel
    resource.
  - The file precedent has both forms of access, by descriptor and by
    `mmap`, and `sendfile` sends such a file by reference with no pin, no
    notification and no NIC feature. What a file lacks is a way in without a
    copy (`../RELATED_WORK.md`, "Page cache plus sendfile": the baseline for
    holding and re-sending, measured on send in S12).
- **The redesign** and **the two modes**: under "Conventions".

Proposed with the story on 2026-11-14, each the owner's to settle:

- TODO(decide): cite tmpfs beside ramfs, or in its place. The kernel's
  documentation (`filesystems/ramfs-rootfs-initramfs`, read 2026-11-14) says
  that ramfs has no limit, so only root or a trusted user should write to it,
  and that tmpfs added size limits so that ordinary users can. A named store
  is the tmpfs case: bounded by its `hard_limit` and open to unprivileged
  users.
- TODO(decide): name the mapped mode's precedent too: the packet socket's
  ring (`networking/packet_mmap`), memory that the kernel allocates, fills
  with a device's packets and maps into a privileged process. It is filled by
  a copy, as a file is.
- TODO(decide): two words. "Kernel memory dedicated to the user" and not
  "user memory", which in the paper is what the existing mechanisms borrow.
  "Dedicated" or "set apart" and not "segmentation". The idea is sound and
  the word describes it; the trouble is the root. "Segment" is a defined
  term of the design (the fixed-size entry for one piece of an object,
  `../docs/design_spec.md`, glossary and §2.5) and runs through §3 to §5, a
  TCP paper also has TCP segments and segmentation offload, and in operating
  systems "memory segmentation" names the classical addressing scheme. The
  owner asked what is wrong with the word on 2026-11-14; not settled.
- TODO(decide): what the story keeps from the colleague's draft. Proposed:
  its statements of scope (TCP payload, the fallback copies, the device-wide
  trust of a mapper, a send's completion is not permission to rewrite). The
  draft sets two access contracts side by side and states their combination
  as the contribution; in the story one cause produces both modes.

### Novelty and contributions (2026-11-14)

What the story claims as new and what the paper contributes, as the
contributions paragraph of §1 will state them. Told back at the owner's
request ("tell me back whats the contribution and the novelty of the paper")
and written here on the owner's word ("Yes, write it"), both on 2026-11-14.
The owner has not commented on the single items.

**The novelty** is an angle on a problem that has many solutions, and not
one more mechanism for one more case. The owner, 2026-11-03 (quoted in full
under "Abstract"): the paper is novel "because we looked at something that
has been analyzed and engineered for decades and found a new angle that gets
more done with less". It has three parts:

- **The diagnosis: trust.** The conditions of the existing zero-copy paths
  have one cause, and it is two questions (steps 2 and 3). Read this way,
  what zero-copy receive asks of the NIC is asked for trust: header split
  answers the first question and flow steering the second. The pairing is
  the proposal noted under the table; the owner has not yet confirmed it.
- **The rule: lend, don't borrow.** The kernel keeps payload in its own
  memory and lends access by what each application is trusted with (step 4):
  a name to any process, and the memory itself, set apart from everything
  the kernel relies on, to a process trusted with the device.
- **The result: zero copy that is hardware agnostic.** Neither mode needs a
  NIC feature, which is the title's claim. Because the memory's lifetime is
  the kernel's, sending an object pins nothing and waits for no report, and
  payload outlives its connection (step 5). The mapped mode's raw path,
  `RECV_ZC` into `SEND_ZC` with no object, keeps the notification and pins
  per send (`../docs/design_spec.md` §3.3).

**The contributions** are four. They follow the list that the plan of
2026-10-31 gives for §1 ("Sections": the thesis; the named mode and its
retention design; the mapped mode; the evidence), and the thesis is now the
argument from trust.

1. **The argument.** Trust is the one cause of the conditions, and each
   existing mechanism is placed by how it answers the two questions and by
   what its answer costs (the table above).
2. **The named mode.** An application with no privilege, on any NIC and with
   no driver change, declares a range of a TCP stream an object and keeps,
   cuts and sends it by handle, copying out only what it reads
   (`../docs/design_spec.md` §2.8, "Security": the named API "requires no
   `CAP_NET_ADMIN` and no NIC configuration").
   - Nothing is mapped, so neither question arises.
   - The object is the application's own data in kernel memory, as a file in
     ramfs is, with what a file lacks: a way in without a copy. That makes a
     name zero-copy receive, and not only forwarding.
   - Holding is bounded (§2.8 of the design): a store's limit is fixed at
     registration and charged to its memory cgroup once, the metadata of its
     objects comes from arrays made then, and a full store fails a request
     at once, so the bytes stay in the socket and TCP's flow control slows
     the sender.
3. **The mapped mode.** A process trusted with the device maps the memory
   that supplies the device's receive queues, read-write, and works on
   payload in place (registration requires `CAP_NET_ADMIN`).
   - That memory is set apart from everything the kernel relies on, and the
     kernel parses its own copies of the headers, so a mapper cannot harm
     it. The privilege answers for the mapper's reach over every flow on the
     device.
   - The driver needs a hook: it takes its receive buffers from that memory
     and keeps the headers the kernel parses private
     (`../docs/design_spec.md` §3.5). The NIC needs no feature.
   - It keeps MAIO's dedicated I/O memory and needs neither its header split
     nor its hardware ring per process. The kernel's own header copies take
     the place of the first. Privilege, which MAIO fell back to without such
     rings, takes the place of the second.
4. **The evidence.** The implementation in Linux, as `io_uring` operations,
   and the five experiments of 2026-11-11. They carry the claim under
   "Conventions": the two modes perform alike, and both are as fast as or
   faster than the existing zero-copy mechanisms.

**Not claimed as new** (`novelty.md`, "What is new and what is not"):
forwarding payload between sockets inside the kernel; sending kernel pages
by reference, which the kernel's splice path does; memory dedicated to I/O
and mapped into the application (MAIO); trusting one privileged process with
a device's receive memory (MAIO without per-process rings); keeping a
received buffer as stored data (PASTE); one physical copy shared by a cache
and the network, as a goal (IO-Lite); copying only the headers to the
application while the kernel forwards the body (Rosu and Rosu, SO_SPLICE,
Libra).

**Where the evidence stands** (2026-11-14):

- Measured: mainline zero-copy receive refuses virtio-net, where the named
  mode runs (F5).
- Not measured yet: the numbers of the claim. They come from the rerun on
  the redesigned build (`../eval/2026-11-11-fixed-objects/EVAL.md`), which
  has no row in `../CLAIMS.md` yet.
- Not measured yet: the comparison with hardware zero-copy receive on a NIC
  that has it. It is campaign 1 of `../eval/PAPER-CAMPAIGNS.md`, outside the
  rerun.

**`novelty.md`** was brought up to this section on 2026-11-15 (the owner:
"Yes, update it"). It holds the prose form: a one-sentence claim, the
insight in the order of the story, the four contributions, what is and is
not new, the system's two rows of its Table 2, and a table of the claim's
parts with the evidence each needs and where it stands. Its one-sentence
claim is proposed wording that the owner has not read. The claim and the
three contributions of 2026-10-06, for the named mode alone, are kept at its
end as a record. Where that file and this section differ, this section is
the record.

Two statements here were made exact on the same day, from the sources read
for that update. Sending *an object* pins nothing and waits for no report
(step 5 and "the result" above): the mapped mode's raw path keeps
`SEND_ZC`'s notification. And MAIO kept processes apart with hardware rings
or, without them, by privilege (the bullet on MAIO, and contribution 3).

## Framing (decided 2026-10-03; the LLM tier's weight tempered the same day)

Update (2026-11-14): "The story" above is the frame of the owner's version,
and the title's claim word is now "hardware agnostic" ("Working titles"). The
records below are the decisions as they were taken, kept as history. Where
one differs from the story, the story is later.

Update (2026-11-13): the draft as revised that day (`fcba7b4`, `9dc23a7`) is
framed differently from the records below. It presents two peer access
contracts, read-only by name and read-write through a segregated I/O area,
and states the contribution as their combination with TCP, receive-buffer
providers and `io_uring`, crediting IO-Lite, PASTE and MAIO with shared
buffers, retention and I/O-memory segregation. Its title and abstract no
longer call the system zero copy or ubiquitous. The argument of 2026-11-03,
that trust is what keeps the cases of zero copy apart, is not in its abstract
or introduction; trust appears as the mapped mode's device-wide requirement.
The records below are the owner's decisions as taken. Which framing the paper
keeps is under "Open decisions".

Update (2026-10-31): the premise under "Working titles" supersedes the
claim-word discussion below (zero copy names the genre, kernel-owned I/O
memory is the idea, ubiquitous is the claim) and puts the mapped mode of
`RECV_ZC` on par with the opaque data object. The section plan is being
revised to match; the calibration rules and the four-part comparison stand.

A zero-copy systems paper for OSDI. The contribution is the OS mechanism:
received payload as a kernel object with an owner of its own, never mapped,
with any NIC and no per-application hardware setup. The paper's claim word is the object, not
"zero copy": zero copy is the genre every reviewer has rejected a paper in,
and Libra claims it too. The store is the application that separates the
work from Libra, and the evaluation's backbone is a strong improvement over
classical KV caches (Redis, memcached) on commodity Ethernet. The LLM KV
tier is a motivating application where RDMA is absent, not the lead:
state-of-the-art providers move KV over RDMA or CXL (`../refs/papers.md`,
"Transport tally"). The relay and the other use cases are extensions of the
object model. RDMA is out of scope as a baseline: the target is commodity
hardware and cloud infrastructure without it. RDMA figures are cited as the
ceiling on the same axes (bandwidth, operations per second, latency), never
measured; they show how far the store can try to reach.

The owner, 2026-10-03, after the paper survey (relayed by the bar-raiser
session): "we cant lean to heavily on LLM KV use case ... SOTA uses RDMA or
CXL. we do need to see a strong improvement over classical KV caches and
while RDMA is our ceeling we know [what] we can try an reach." On the count:
"at least one was an sdwan paper not data center tcp. but we need to be
carefull about it and mindfull of RDMA." Before this, the paragraph above
made the LLM KV tier the lead application and the motivation that elevated
the work.

The owner, the same day, in the refs session: "we should show usefullness
for LLM KV cache usecase but our main claim to fame is zero copy
functionality where opaque objects can be used. this is strong but not as
fashionable as improving LLM perforrmance." So the LLM KV tier is shown to
be useful, and the claim is the zero-copy functionality that opaque objects
make possible.

The owner, 2026-10-06, after reading how `io_uring` zero-copy receive works:
"no we need a zcrx comparrissson no way around it. our only claim to fame
that actually has any potential is the comodity HW". So the claim to fame is
commodity hardware: SYS gets zero copy without the NIC features that
hardware-assisted zero-copy receive needs, and the evaluation must show it
against that path (§7, Q2). The object is what makes it possible: payload
that is never mapped needs no header split, no flow steering and no
dedicated queue. This reverses the 2026-10-02 decision that no comparison
with zero-copy receive was required.

The owner, later the same day: "having a generic interface that can also add
non trivial features that are needed then we have a paper. a wide and
growing capability list for ZC is barely a reason to publish." So the thesis
is the interface: one abstraction for payload a service does not read, under
which needed features that today each require their own kernel hooks, NIC
features or user-space stack become a few lines of application code or a
small eBPF program. Zero copy is what the interface provides for free, not
the claim.

The owner, the same day: "with RECV_ZC most of the functionality is already
there. the buffers have kernel addresses and anything we add to opaque
objects infra can potentialy be added to io_uring infra for the same
effect." True of the features, so they cannot be the novelty on their own.
What cannot be added to zero-copy receive in software is a different
placement model: zcrx decides who owns the payload before DMA, which needs
header split, flow steering and a queue per application; opaque objects
decide it after the TCP stack, on whatever buffers the driver used, and
never map them. The model is the novelty and the features show it is
general (`novelty.md`, "The zcrx test, per feature").

The claim, approved by the owner the same day ("yes I like the framing"):
SYS brings zero-copy retention and reuse of received payload to every NIC,
VM and container, with no per-application hardware setup, and makes holding
network buffers safe and bounded. Its three legs, where, what and how, and
the three contributions are in `novelty.md`.

Calibration rules, to apply as results arrive:

1. The §1 test. Every claim in §1 must be true of a pass-through service
   that is not an LLM store. The store appears in §1 as the headline
   application and the LLM KV tier as one motivating instance where RDMA is
   absent; neither is the claim.
2. The result test. A number that needs a model name to be stated is an
   application result and lives in the store's part of §7. Contributions
   are stated in bytes, cores and copies.
3. The design test. Each design decision names the workload fact that
   forced it (two lifetimes, whole-prefix reuse, a DRAM budget, chunk sizes,
   the SSD tier), and at least one non-LLM application exercises it.
4. The baseline test. Every baseline runs on commodity Ethernet; for the
   store, the baselines are classical KV caches (Redis, memcached). A
   comparison that needs hardware the design does not require is out of
   scope; cite its ceiling.
5. The Libra test. The store is the distance from Libra; the relay is the
   parity with it. The relay never becomes the headline.
6. The setting test. A source is evidence only for its own setting: a result
   from an RDMA cluster, from one machine, or from a link between datacenters
   says nothing about cloud Ethernet inside a datacenter
   (`../refs/papers.md`, "Transport tally"; the owner's correction above).
7. The zcrx test (the owner, 2026-10-06). Every feature states its delta
   from an application that uses `IORING_OP_RECV_ZC` and holds the payload
   in its own memory, on a NIC that supports it (`novelty.md`, "The zcrx
   test, per feature"). A feature whose only delta is availability claims
   commodity hardware, not performance.
8. The overhead test (the owner, 2026-10-14). The challenge every related
   system answers is zero-overhead I/O for general purpose, in MAIO's own
   definition: zero copy plus exception-less system calls. Each group is
   scored on what it removes (the copy across the boundary, system calls,
   the remap or pin per operation) and what it gives up (the kernel stack,
   any NIC, retention). Valkey 9's "zero-copy responses" remove an internal
   reply-buffer copy, not the boundary copy: it is a baseline in §7, never
   related work. BSD socket splicing is a footnote; we run Linux and no
   version of it matters.

The comparison against `RECV_ZC` + `SEND_ZC`, approved by the owner on
2026-10-14 ("yes lets go with this framing"), in four parts, each with its
qualification. It is the spine of §2.4, §7 and the zcrx rows of §8.

1. **Simpler to manage.** No queue, steering rule, header split or
   registered area sized for the peak per application; the kernel tracks
   object lifetimes instead of the application tracking N completions per
   buffer; no capability, where every `RECV_ZC` registration needs
   `CAP_NET_ADMIN`. The wording that survives review is "no per-application
   hardware setup", never "simpler overall": both change the application,
   ours is a twelve-patch RFC series and zcrx is mainline.
2. **Less overhead on transmit.** No pin and unpin per page sent, a cost
   that scales with the pages and never amortizes (MAIO measured about 22
   cycles per byte in `get_user_pages_fast` at 16 KB) and no notification
   per send. Against `SEND_ZC` with fixed buffers the delta is one CQE and
   the registration bookkeeping. Our caveat: an uncompacted object is a
   chain of MSS-sized fragments and may cost more descriptors per MB; the
   three send configurations of §4.4 measure it and background compaction
   is the remedy.
3. **Receive at parity.** Both pass through the full TCP stack and neither
   copies to the application; we add object assembly and claim bookkeeping,
   zcrx adds refill-ring bookkeeping. Ahead where traffic misses the steered
   queue (copied into the pool there, never here) and where holding would
   drain the pool. Behind for an application that reads every byte: we hand
   out a prefix and copy on `READ_OBJECT`.
4. **Hardware agnostic.** Any NIC, virtio-net, veth, loopback, VMs and
   containers; netkit's leased queues bring `RECV_ZC` to a pod's wire
   traffic on a capable NIC and nothing to traffic that stays on the host.
   Not always copy free: v9 copies when the backing is slab-allocated or
   unsafe to hold, and fallback copies are counted. Hardware agnostic, not
   kernel agnostic.

All three performance parts are TODO(measure). The slide version is
`../talks/advisor_deck.md` 19b; the kernel facts behind them were checked
against master on 2026-10-14 (fact-check log in the same file).

## Working titles

The owner's choice (2026-11-14): **Lend, Don't Borrow: Hardware-Agnostic Zero
Copy for the Network Stack**.

The owner proposed "HW agnostic ZC for the network stack" as "the common
motive between mapped and named"; the reasoning is quoted in full under "The
story". Asked whether *lend, don't borrow* stays in front of it, the owner
kept it ("Keep the hook"). What each part carries, as discussed that day:

- *Lend, don't borrow* is the design's rule, and the one part the colleague's
  title shares. It also covers the send side, where the existing paths are
  hardware agnostic already and what DIOMND removes is the pin and the wait:
  both are costs of borrowed memory.
- "Hardware agnostic" is the claim: the property both modes share. The named
  mode needs nothing of the NIC or its driver; the mapped mode needs a driver
  hook and no NIC feature. "Ubiquitous", the claim word of 2026-10-31, fits
  the named mode only, since the mapped mode needs privilege and that hook.
  In the story hardware independence is a consequence of how trust is
  handled, not a feature beside others.
- "Zero copy" names the genre, as on 2026-10-31.
- "For the network stack" says the kernel's stack stays, which separates the
  work from bypass designs in the title itself.

The title asks for two exhibits before any other: the table of where DIOMND
runs (the portability table), and the comparison with hardware zero-copy
receive on a NIC that has it (campaign 1 of `../eval/PAPER-CAMPAIGNS.md`).

In the draft since the revision of 2026-11-13 (`fcba7b4`): **Lend, Don't
Borrow: Reusable TCP Payload in Kernel-Owned Memory**. The QUIC note
(`quic-extension/main.tex`) proposes "Reusable Transport Payload in
Kernel-Owned Memory" for the time a datagram path is built and measured. The
draft keeps its title while the two versions stand in parallel; which title
the outgoing version carries is decided with that version. The revised draft
keeps *lend, don't borrow* and the two modes of the premise below, and no
longer argues the four costs as the price of borrowing (§2 under "The draft
as it stands").

Previous choice (2026-10-31): **Lend, Don't Borrow: Ubiquitous Zero Copy
from Kernel-Owned I/O Memory**. Decided by the owner on 2026-10-31 with the
premise below, and superseded in its claim word on 2026-11-14; the premise is
kept as recorded. The 2026-10-03 title stays as history.

Premise (2026-10-31). Every zero-copy path in mainline lends the
application's memory to the kernel, which must then pin it per operation
or for the peak, tell the application when it may reuse it, place payload
into it by hardware before the stack runs, and treat the result as a
special case in the stack. SYS lends kernel-owned I/O memory to the
application instead, in two modes that are on par: by name (the opaque
data object, which needs no isolation) and by mapping (the mapped mode of
`RECV_ZC`: a kernel area on every receive queue, which costs privilege).
Both decide ownership after the stack, send by reference through ordinary
pages, and keep lifetime in the kernel's reference counts: no pin, no
notification, no placement, no special case. "Zero copy" names the genre
the reviewer knows; "no per-operation overhead" is the explanation one
sentence later; "ubiquitous" is the claim (any NIC, VM and container).
The owner, 2026-10-31: the mapped mode "closes and extends the utility
gap" against zcrx; the paper trusts the design, on par or faster than
every other path, and the text carries the pinned campaign's numbers.
Reasoning: `../maio/ideas.md` §10 (the assessment and the decision),
`novelty.md` "The zcrx test" (hardware or privilege).

Previous choice (2026-10-03): **Zero Copy via Shared Opaque Data Objects**.

"Shared" carries the holding and sharing; "object" carries the contrast
with a stream: a stream can only be consumed, an object is something you
hold.

Scope (2026-10-03): the paper's weight is a breadth of network-bound uses.
Objects can also be written to and read from storage (NVMe, disk); that is
an extension, not the headline.

Merge note (2026-10-03): the Framing section above and this title were
recorded in different sessions, the title later the same afternoon. They
differ on two points. The Framing section says the claim word is the object
and not "zero copy"; the title leads with "Zero Copy". The Framing section
makes the LLM KV tier the lead application and the other uses extensions;
the scope here puts the weight on a breadth of network-bound uses. Both are
kept as recorded, for the owner to reconcile. Update, later the same day: the
owner's conclusion from the paper survey, quoted in the Framing section,
answers the second point as far as the LLM tier goes: it is a motivating
application, not the lead, and the store against classical KV caches
carries the evaluation. The first point is still open, though the owner's
later sentence in the Framing section names zero copy through opaque objects
as the main claim, which reads like the title; whether the claim's wording
leads with the object or with zero copy is the owner's to settle.

Reviewer note (2026-10-03), on the earlier candidate titles, since replaced:
the one that led with proxies named the application where concurrent work
has the better forwarding numbers. Prefer a title that leads with the
object and the store.

## Abstract (in `osdi27/sections/00-abstract.tex`)

**In the draft since 2026-11-13 (`9dc23a7`).** One paragraph, 169 words by its commit's count,
under the USENIX template's guidance of at most 200. In its order:
- network services retain, forward and sometimes modify received payload;
- DIOMND applies *lend, don't borrow*: the kernel owns network storage and lends two forms of
  access;
- the read-only named mode: handles to received plaintext TCP ranges for unprivileged
  applications, for copied inspection, retention and concurrent range sends, with no mappings
  and no driver changes;
- the read-write mapped mode: a segregated I/O area, with kernel control structures and the
  header copies used for parsing kept private; supporting drivers but no hardware header
  split; device-wide trust, and edits coordinated with outstanding uses;
- both modes support objects with kernel-managed lifetimes; fallback copies and unused buffer
  space remain costs;
- two numbers, each a `CLAIMS.md` row from the earlier prototypes: F24's 42% for named capture
  against `recv()` with 64 KiB source writes, and F10's 16% for a mapped-area object relay
  against `splice` at comparable mean throughput;
- payload-edit performance is unmeasured, and the revised metadata and reclamation still
  require implementation and evaluation.

It describes the two modes. It does not tell the story of the owner's instructions below (zero
copy solved case by case, the cases kept apart by trust), it is about half the length they
name, and S1's send number is no longer in it. TODO(decide): which abstract stands ("Open
decisions").

**For the owner's version (2026-11-14).** The abstract tells "The story" in its order, in about
300 words, with at most three numbers in its last sentences, each a row of the rerun. Draft 1
follows. The owner asked for it ("Yes, draft it") and has not read it. It is text for this file
only: `osdi27/sections/00-abstract.tex` keeps the colleague's abstract.

> Network services spend much of their CPU copying payload between the kernel and applications.
> Decades of work have removed that copy one case at a time, each under a condition: a use the
> kernel planned for, such as sending a file; buffers pinned and untouched until the kernel
> reports them free; or a NIC that splits off headers and steers each application's flows to its
> own queue.
>
> We argue that the conditions have one cause: trust. Driver buffers mix every application's
> flows and carry the headers the kernel acts on, so an application with access could read
> others' data or corrupt what the kernel relies on, endangering security and safety. Each
> mechanism is a way around that danger, and its condition is the price.
>
> DIOMND (dedicated I/O memory for network devices) keeps payload in the kernel's own memory and
> lends each application the access it can be trusted with. Any application may hold payload by
> name: it declares a range of a TCP stream an object, then keeps, slices and sends it by
> handle, copying out only what it reads. Nothing is mapped, so neither concern arises, yet the
> object is the application's own, as a file in the page cache is. An application trusted with
> the device may map the memory and work on payload in place: privilege settles security, and
> private header copies keep the kernel safe. Ownership is decided after the network stack has
> run, so no NIC feature is needed; sending an object pins nothing and awaits no notification;
> and payload outlives its connection, to be kept and sent to many.
>
> Implemented in Linux as `io_uring` operations, the two modes perform alike: both send and
> forward at or below the CPU cost of the kernel's zero-copy paths, and on a virtual NIC that the
> kernel's zero-copy receive rejects, both receive with ⟨rx⟩% less CPU than a copy. They match
> hardware-assisted receive where a NIC provides it, and a key-value store built on named
> objects ⟨kv⟩.

About 330 words in four paragraphs. The owner's direction is about 300; the template's guidance
is 200 at most, and the draft's abstract has 169. `⟨rx⟩` and `⟨kv⟩` are TODO(measure), and there
is room for a third number.

What the last paragraph rests on. Only the first line below is measured; the others are claims
that the named campaigns test, and each sentence stands or is rewritten with its row:
- "a virtual NIC that the kernel's zero-copy receive rejects": measured, F5 (virtio-net).
- "the two modes perform alike": `rx` and `fwd` of the rerun, named against bound and mapped
  (proposed rows F22, F25).
- "send and forward at or below the CPU cost of the kernel's zero-copy paths": `tx` against
  registered `SEND_ZC` (S3 restated) and `fwd` against `splice` (F25).
- `⟨rx⟩`: `rx` against `recv()` (F22 with F21, restating F24).
- "match hardware-assisted receive": campaign 1 of `../eval/PAPER-CAMPAIGNS.md`, outside the
  rerun.
- `⟨kv⟩`: `kv` against memcached and Redis (proposed row K2).

Choices made in draft 1, each the owner's to correct:
- It opens on the cost of copying and on the conditions, and names trust in the second
  paragraph, with security and safety as the two concerns.
- Each mode is introduced by what its application is trusted with. The limits sit inside those
  sentences (a named application copies out what it reads; a mapper is trusted with the device)
  and have no sentence of their own.
- The file precedent is one clause, "as a file in the page cache is". Its wording follows the
  proposals under "The story".
- No earlier design is named. *Lend, don't borrow* appears as one verb, since the title carries
  it.
- The memory is "the kernel's own memory": the abstract uses neither "segmentation" nor
  "segregation".

Its nearest ancestor is the version of `ac90ed9` below, which the story corrects in
two places:
- the sentence that every design takes the kernel's I/O memory to be off limits to applications.
  IO-Lite, PASTE and MAIO share that memory with applications; what each still does is isolate
  before a packet is stored (step 3 of the story);
- the closing sentence that parity with hardware zero-copy receive is not claimed. The owner's
  direction for the abstract has receive on par with the hardware-assisted path, and campaign 1
  of `../eval/PAPER-CAMPAIGNS.md` measures it.

**The owner's instructions**, from four critiques of the drafts of 2026-11-03:
- "This paper is novel not because it has +XX% performance gains but because
  we looked at something that has been analyzed and engineered for decades and
  found a new angle that gets more done with less."
- "Every word counted and was true. That's your standard: not saying something
  that sounds blatantly wrong, and every word counts and adds something new."
  Never "no zero-copy solutions exist" or "has not removed the copy"; not
  confined to proxies (with the mapped mode the scope is every network I/O
  process).
- "It's an abstract not an intro, easy with the numbers, ~300 words to tell a
  story. Zero copy is important, it's been studied but not solved; we claim
  it's all about the trust (security and safety) mechanism, and thus by
  revisiting the premise from first principles we get something faster and
  less constrained."
- "Zero copy is a subject of intense scrutiny with numerous solutions being
  proposed. No one cares about 100 Gb/s and 16K sends in the abstract! Story,
  not eval cherry-picking; that comes at the very end of the abstract, not the
  beginning."
- The name is DIOMND (said twice; KIOM was the first candidate).

**The version written on those instructions** (2026-11-13, `ac90ed9`; replaced the same day
by the abstract above, and recoverable from that revision). It followed the owner's verdict on
the version of 2026-11-03 ("the previous model took my instructions way too literal"): that
version repeated the instructions' own sentences ("a subject of intense scrutiny", "None has
solved it", "revisits it from first principles", "faster and less constrained"). The
instructions say what the story is; this version told it in the paper's words. Three
paragraphs, about 320 words, numbers only in the last sentence (three, each a CLAIMS.md row at
full speed: F24, S1, F10; all three predate the design's object metadata):
1. Zero-copy networking is studied for three decades and solved many times over, each time
   for one case: a file sent, a stream forwarded once, a buffer sent if it is pinned and its
   reuse waits, payload received if the NIC splits and steers. No one mechanism covers the
   cases, and none is free.
2. The claim: the cases are kept apart by trust, not by engineering. Each design takes it as
   given that the kernel's I/O memory is off limits to applications, so zero copy stays inside
   the kernel, limited to uses the kernel anticipated, or moves into the application's memory,
   which the kernel must guard and the NIC must fill in isolation. The restrictions and the
   overheads are the price of that boundary.
3. DIOMND redraws it: the kernel keeps its I/O memory and lends it by what each application
   is trusted with. By name to any process (the opaque data object: never mapped, nothing to
   isolate, kept, sliced, sent to many); by mapping to a process trusted with the device.
   Ownership after TCP; an object sent as ordinary pages, nothing to pin, wait for or ask of
   the NIC. One interface for forwarding, caching, fan-out and replay. Then the numbers.
   Parity with hardware zero-copy receive is not measured and is not claimed.

The introduction of `ac90ed9` told the same story, so that the two agreed: copies and why they
matter; solved case by case; one cause, trust, with the kernel's three owners and the four
costs of a loan; lend, don't borrow, by what the borrower is trusted with; what a name adds and
what makes holding cheap; the limits; four contributions, with the numbers in the last. The
introduction in the draft now is the row for §1 under "The draft as it stands".

The draft abstract of 2026-10-03 is superseded.

## Sections (2026-10-31, on the premise under Working titles)

The owner approved this plan on 2026-10-31 ("Write it"). It supersedes the
section table of 2026-10-02, kept below as history. Twelve pages plus
references; the rows sum to 12. The draft was revised in full on 2026-11-13
and no longer follows every row: what it holds is "The draft as it stands"
below, with its departures from this plan.

| § | Section | Pages | Must contain |
| --- | --- | --- | --- |
| 1 | Introduction | 1.25 | The ownership gap: a socket, a pipe or a file is the only owner the kernel has for received payload; copies are the symptom. Borrowing and its four costs, one measured number each: the pin, the notification, hardware placement, the special-cased skb. Lend, don't borrow, in two modes, by name and by mapping, and what each needs (nothing; privilege). What a name adds: hold, slice, fan out, bounded. Contributions (the thesis; the named mode and its retention design; the mapped mode; the evidence) and the headline numbers from the pinned campaign. The 4 KiB break-even stated in §1, not hidden. |
| 2 | Borrowing and its costs | 1.0 | `problem.md` §2.1 (pass-through services) and the ownership gap. The four costs, each with our own motivating measurement from §8. The design space by who lends: borrow per operation (`MSG_ZEROCOPY`, default `SEND_ZC`, `tcp_mmap`, `vmsplice`); borrow for the peak (registered `SEND_ZC`, zcrx, devmem TCP, AF_XDP, PASTE, netkit's leased queues); the kernel keeps it but fixes its fate (`splice`, sockmap, `tee`, the page cache with `sendfile`); lend (the 2021 I/O-memory design, in the third person; SYS). Table 1, 2026 edition, with a lend/borrow column (figure 2 below). Requirements R1–R6 of `problem.md` §2.5 plus R7: no privilege where the application does not read the bytes. |
| 3 | Lending kernel memory | 0.75 | The model both modes share: ownership decided after the stack; ordinary pages; lifetime in the kernel's reference counts; no handshake in either direction (no pin, no notification, no placement, no special case). Name against mapping, and the isolation each needs: none; privilege over the device. Composition: a mapping's pages can be named. The cost model: fixed per name and per operation, nothing per byte; copy below the break-even. |
| 4 | The named mode: opaque data objects | 2.25 | The items under "§4 Design" below, as `../docs/design_spec.md` §2 has them: capture into segments, the object as a fixed-size head and a chain of fixed-size segments with three lifetimes (name, metadata, pages), the magazine allocator, header access and the whole-response send, the send path (vectored, ordered per socket, the socket-lock window), memory (slack, the limit as the size of the arrays, a full store that fails at once and TCP flow control, what is deferred), safety invariants, the hit path if built. This is the paper's depth. |
| 5 | The mapped mode | 1.0 | `../docs/design_spec.md` §3; in the evaluation kernel only. The readable-page provider on every receive queue, and what the driver must do (accept a provider through the queue API, keep parsed headers private; virtio-net was changed); the kernel-allocated area and its mapping; exhaustion to ordinary pages, copies, then an error, with counters; the quiet test and buffers handed out through per-CPU magazines; mapped objects (a bound store) with the heads and segments of any object, sent by handle with no pin or notification, and the lazy collection that returns the buffers of sent objects (a zone of the pool, scanned after 1,000 frees or 100 ms, no callback at the ACK); raw buffers through `SEND_ZC` with its notification (the measured raw relay pins per send); a mapped object is never moved; privilege and its limits (the process reads and writes every flow on the device, unbound stores' objects and in-flight sends included; the trust passes with the ring FD; one instance per device; the VM boundary). One table against hardware zcrx and NODEV: what each needs, what the process sees, what each copies. Sources: the design spec, `../apps/uapi/ADDONS.md`, `../maio/ideas.md` §10. |
| 6 | Implementation | 0.5 | What exists, kept apart from the design (`../docs/status-and-lineage.md`, `../code/upstream/README.md`): the twelve-patch RFC (one `net/` receive claim; store, sharing, framed and vectored sends, collector, CREATE, tests) with its liburing companion; the independent four-patch `skb_splice_from_iter` series, measured only in an earlier version; the evaluation kernel as a separate lineage carrying the mapped mode and the evaluation's own changes, the source of every number; the magazine allocator as one utility with three users. Lines of code from the cover letters' diffstats and the RFC's bounded verification (92/92 UML on the development tree), both of the series as prepared before the design's object metadata and marked to be restated. The marker that every number predates that metadata. |
| 7 | Applications | 0.75 | The relay five ways: copy, `splice`, sockmap, named, mapped plus named. The store with its data paths as flags. Fan-out. Lines changed per application (X6). The table under "§7 Applications" below. |
| 8 | Evaluation | 3.25 | Under "§8 Evaluation, in the order of the title" below. |
| 9 | Related work | 0.75 | One paragraph per group of §2's design space; the 2021 I/O-memory paper in the third person; IO-Lite and Cheetah as the historic goal; Libra; netkit; devmem TCP and the NIC-placement family (`../virtualization/gvnic.md` §10.3). `prior-work.md`, regrouped. |
| 10 | Discussion, limitations, conclusion | 0.5 | Small messages and data-touching protocols; kernel TLS; the privilege of the mapped mode; the VM boundary as the pointer to lending across VMs by name (the HotOS position, `../virtualization/shared/hotos-2027-framing.md`); storage and GPUs as extensions (`../FUTURE.md`). |

Decisions taken with the plan (owner, 2026-10-31), each open to being
overturned:

- §2 tells the story along lend/borrow; the nine groups of 2026-10-14
  survive as the rows of the §9 table, not as §2's structure.
- The mapped mode has its own section: on par needs a visible home, and
  it keeps §4 about retention.
- §8 runs by workload in the title's order: ubiquity first, overhead
  second, capabilities third. The three legs of `novelty.md` label each
  figure instead of structuring the section.
- The modes are called *named* and *mapped* in the text; "opaque data
  object" is the proper name of the named one. TODO(decide): the mapped
  mode's proper name.
- Table 1, 2026 edition, sits in §2 as the map of the design space, not
  in §9.

What the plan demands by 2026-12-08, beyond the pinned campaign and the
MAIO pass: the retention rows of §8.5, the store of §8.6 on the final
kernel, and the gVNIC row of §8.2. The abstract (2026-12-01) needs none
of them.

### The draft as it stands (2026-11-13, `9dc23a7`)

Since 2026-11-14 this section describes the colleague's version, which stands in parallel with
the owner's (the top of this file). It is a description of `osdi27/`, not the plan.

The draft changed twice on 2026-11-13. In the morning it was cut to the limit (`ac90ed9`): the
body ended on page 12, at 11.9 pages, and the markers became numbered tags with their text in a
list after the references (`osdi27/macros.tex`). Cut that morning, each recoverable from
`1668040`, which holds the longer form of every section: §1's driver census and second
literature paragraph; §2's requirements R1 to R7 and most of the eBPF paragraph; §4's
subsection on serving from the kernel; §7's table of applications and baselines; every paced
number of §8; about a third of §9. In the evening the owner's colleague revised every section
(`fcba7b4`) and shortened the abstract (`9dc23a7`).

Since the revision the body measures 12.64 pages, 0.64 over the limit, with the markers shown
or hidden (a local pdfTeX build of `9dc23a7`; `make pages` prints page 13, and 15 pages with
the references and the draft notes). Six draft notes remain: four to measure (§4.3, §6, §7,
§8.7), one to verify (§8.1) and one to decide (§6, the statement on coding assistance).

Pages run from one section's start to the next, with the floats where LaTeX put them.

| § | Title in the draft | Pages | What it holds |
| --- | --- | --- | --- |
| – | Title and abstract | 0.33 | "Working titles" and "Abstract" above. |
| 1 | Introduction | 1.10 | What services do with payload (forward it unchanged, retain it, filter or transform it) and what socket reads and writes cost them. The existing interfaces each address part of it: `sendfile`, `splice`, `MSG_ZEROCOPY` and `SEND_ZC`, hardware zero-copy receive. Two questions remain: how to retain and reuse ordinary received data without exposing its pages, and how to expose writable I/O buffers while kernel control state stays private. *Lend, don't borrow* answers both through "two peer interfaces": read-only lending by name, with a proxy as its example, and read-write lending through segregated memory. What the modes share: object retention and range sends, three lifetimes, preallocated metadata, mapped recycling delayed until a page's users are gone. IO-Lite, PASTE and MAIO as the precedents; "this combination is the contribution". Where each mode fits and what both pay. Three contributions, with no number in them: read-only lending through named objects; read-write lending through segregated I/O memory; shared lifetime and retention management. A closing paragraph of evidence from the earlier prototypes: F24's 42%, S1's 15–27% from 4 KiB, F10's 16%; then what is not measured (payload edits, the revised design, hardware receive, direct writes). |
| 2 | What Zero-Copy Interfaces Require | 1.42 | Opens on the distinction the section is built on: avoiding copies, retaining data and permitting writes are different capabilities. Table 1 ("Figures and tables" below). §2.1 Access and retention: mapped designs retain and share (IO-Lite, PASTE); pipes and files reuse (`tee`, `sendfile`); writable access asks which memory may be exposed (MAIO). §2.2 Costs depend on the interface: resident memory, safe reuse, receive placement and stack handling. They are the plan's four costs, written as obligations that "arise in different combinations", each qualified and none with a number. §2.3 The interface gap: what the two modes combine and what each requires. |
| 3 | Lending Kernel Memory | 1.05 | The two forms of access and what each keeps private, and a paragraph saying that the revised metadata, capacity handling and mapped reclamation still require implementation and testing. §3.1 Two access contracts, with Figure 1. §3.2 Common objects and three lifetimes: the head and its chain of segments, arrays made at registration, an *admitted* send, the lifetimes of the handle, the metadata and the pages, the magazine allocator, and what each mode asks of the application. §3.3 Costs and capacity. |
| 4 | Read-Only Lending: Named Objects | 1.63 | §4.1 Interface: the store, the stream token, absolute stream offsets, a table of eight operations (`INSPECT`, `AVAILABLE`, `RECV_OBJECT`, `DISCARD`, `SEND`, `FREE`, `READ_OBJECT`, `CREATE`), what a failed request leaves behind, closing a stream, a store shared between rings. §4.2 Capturing received bytes: the collector, references transferred or taken, fallback copies and refused input, undecided bytes, collection on demand. §4.3 Sending object ranges: the splice path, generated framing, vectored sends, order per store and socket, what a completion reports, the last-use flag, fragment granularity (a marker, Q6). §4.4 Memory limits and a full store: segments bound the buffers held, the byte limit and slack, the charge at registration, a request that fails at once, slack as a tradeoff. §4.5 Safety and shared authority. |
| 5 | Read-Write Lending: Segregated I/O Memory | 1.53 | §5.1 The memory boundary: what the area holds, the private header copies, the virtio-net changes, MAIO as the principle it builds on. §5.2 Setup and driver requirements. §5.3 Read/write access and trust: the mapper reads and changes every flow's bytes; references keep storage alive, not its contents; edits are coordinated with outstanding uses; the capability, and its delegation with the ring descriptor. §5.4 Raw buffers and retained objects, with Table 2. §5.5 When storage becomes reusable: quiet pages, the three stages of reclaiming a sent object, the background reclaimer. §5.6 Capacity and fallback; its last paragraph says the object-level reclaimer is not implemented. |
| 6 | Implementation | 0.66 | By paragraph: the read-only named mode (a prepared patch series with liburing helpers; no driver change); the read-write mapped mode (development kernels only); shared support (the splice-helper series; user space); the recorded checks and their bounds; the evaluated builds (Linux 7.3-rc6, one per campaign, none the prepared series) and how their object internals differ from the design; planned changes and evaluation. Two markers: implement and test the revised design; the statement on coding assistance. |
| 7 | Applications | 0.62 | Read-only forwarding: the framed benchmark relay, and the byte-stream relay the forwarding experiments use. Read-write processing: a trusted relay that replaces a fixed-width field before its first send, "a design example, not an implemented or measured editing workload"; the implemented mapped relays, bound and raw, forward unchanged bytes. Read-only storage: the memcached-protocol store, one binary with copying or named storage, single-threaded, mapped stores not exposed by the server. Further workloads: the benchmark tool's retention and fan-out modes; keep, mirror and replay planned. |
| 8 | Evaluation | 2.88 | "§8 Evaluation, as it stands" below. |
| 9 | Related Work | 0.83 | Six paragraphs: shared buffers and stored network data (fbufs, IO-Lite, PASTE); segregated writable I/O memory (MAIO, as the foundation of the mapped mode); kernel-buffer names (ublk); forwarding and transparent proxies (kernel-assisted forwarding, pipes and files, Libra); specialized kernel data paths (XLB, Beeline, BMC, BPF-Broker); copying and other deployment choices (zIO, Copier, IX, Demikernel, Prism). |
| 10 | Discussion and Limitations; Conclusion | 0.59 | Workload fit; writable access and trust; retention costs; deployment and processing (fallback copies, driver support, kernel TLS excluded); remaining work. The conclusion: two ways to use kernel-owned network memory, and what the earlier prototypes' results show and do not show. |

Where the draft departs from the plan's table and from decisions recorded in this file. Each
departure came with the revision; none is recorded here as the owner's decision, and each has a
line under "Open decisions".

- **Title, abstract and the story of §1.** "Working titles" and "Abstract" above. §1 has no
  ownership gap, no four costs of a loan and no thesis among its contributions: the two modes
  are "two peer interfaces", and the contribution is their combination.
- **§2 no longer argues that the costs follow from borrowing.** The plan gives each of the four
  costs a measurement of ours. The draft lists four obligations that depend on the interface and
  qualifies each: registration amortizes the pin; reuse notifications "can be batched"; hardware
  placement is "one solution to isolation, not an inherent requirement of zero-copy receive"; a
  tap's copy is "not a universal property of application-owned memory". The design space is not
  told by who lends, and Table 1 is a different table ("Figures and tables").
- **The modes are named by access.** The draft says the *read-only named mode* and the
  *read-write mapped mode*; §5 is titled on "segregated I/O memory" and credits MAIO, in the
  third person, as the principle it builds on; §7 carries an editing example that is not built.
  The design specification already maps the area read-write (its §3.2, with the trust that
  follows in §3.4), so the design is the same and the emphasis is new. "Opaque data object"
  appears once.
- **The object model is §3.2**, shared by both modes; the plan has it in §4, as the paper's
  depth.
- **§8 reports the recorded results and no longer plots by size and by core** ("§8 Evaluation,
  as it stands").
- **Not in the draft:** the eBPF hit path (item 9 under "§4 Design"; one decision marker in §10
  at `ac90ed9`), the findings that opened §8, and `osdi27/figures/rrkv.tex`, which is in the
  tree and not included.
- **Length.** 0.64 page over. By section, with the plan's pages in brackets: §1 1.10 (1.25),
  §2 1.42 (1.0), §3 1.05 (0.75), §4 1.63 (2.25), §5 1.53 (1.0), §6 0.66 (0.5), §7 0.62 (0.75),
  §8 2.88 (3.25), §9 0.83 (0.75), §10 0.59 (0.5).

### Previous section plan (2026-10-02, superseded 2026-10-31)

| § | Section | Pages | Must contain |
| --- | --- | --- | --- |
| 1 | Introduction | 1.25 | The problem in two paragraphs (CPU and memory). The insight: do not map. What SYS is. Contributions from `novelty.md`. Headline numbers. |
| 2 | Background and motivation | 1.5 | `problem.md`: pass-through services, the two costs, requirements; then §2.4 as the solutions by category in the order of the deck (dynamic remapping; static mapping; hardware-assisted on generally available NICs, where MAIO and `RECV_ZC` sit together; specialized hardware; bypass; kernel transfer; eBPF data paths; copy as a service; held after a copy in), each in one or two sentences with what it removes and what it gives up. One motivating measurement of our own. |
| 3 | The opaque data object | 1 | The abstraction: capture, hold, slice, send, release. Lifetime and immutability. A ten-line proxy loop and a ten-line cache as examples. |
| 4 | Design | 3 | See below. |
| 5 | Implementation | 0.75 | The five-patch series (one patch adds an optional exclusive TCP receive claim in `net/`), the kernel paths it uses, the control path, lines of code, kernel versions tested. |
| 6 | Applications | 1 | What each application needed and how many lines changed. |
| 7 | Evaluation | 3 | See below. |
| 8 | Related work | 0.75 | `prior-work.md`, regrouped into the nine groups of §2.4 (TODO: its ten headings predate the grouping; MAIO moves beside `RECV_ZC` in the hardware-assisted group; Valkey leaves the section). The 2026 edition of MAIO's Table 1 is the section's table (figure 7). |
| 9 | Discussion and limitations | 0.25 | Small payloads, data-touching protocols, what is left to the application. Future work: the §6 candidates that were not built, first the storage gateway on the TCP data path of a KV object tier. |
| 10 | Conclusion | 0.25 | |

### §4 Design (the named mode): what has to be explained

Each item is a problem that gluing existing kernel paths does not solve
by itself. These carry the paper's technical depth.

Where each is in the draft since the revision of 2026-11-13: item 1 is §4.2;
item 2, the object model, is §3.2, shared by both modes; item 3 is §4.1
(`INSPECT`, `READ_OBJECT`) and the generated framing of §4.3; item 4 is §4.3,
where the three send configurations are one marker; item 5 is §4.4; item 6 is
one sentence of §10; item 7 is §4.5; item 8 is §3.3 and the first paragraph of
§10, without a break-even size; item 9 is not in the draft.

1. **Capture.** Taking payload out of the TCP receive queue by reference:
   fragments, linear data, chained buffers.
2. **Object model.** An object is a fixed-size head and a chain of fixed-size
   segments, from arrays made when the store is registered and recycled by
   index through a magazine allocator: nothing is allocated for an object
   while data flows. Three lifetimes: the name, the metadata (kept while a
   send is still queuing; the head counts such sends), and the pages (each
   packet holds its own references; packets hold pages, never metadata).
   Immutable; slicing; many sends on one object, each with its cursor.
3. **Header access.** Copying out only the byte ranges the application asks
   for; composing a new header with a held body in one send.
4. **Send.** By-reference transmit to one or many sockets; partial sends;
   interaction with segmentation offload. Fragment granularity on transmit:
   small received fragments fill a packet's fragment array early, so a 64 KB
   segmentation-offload packet built from MSS-sized fragments becomes two or
   three packets, each with one NIC descriptor per fragment. Measure three
   configurations against each other (decided 2026-10-03): (a) compacted
   objects, sent as dense 64 KB packets; (b) the stock kernel sending the
   scatter-gather list of received fragments as it is; (c) a kernel built
   with a larger per-packet fragment limit, sending received fragments.
   Report transmit packets per MB, descriptors per MB, cycles per byte and
   Gbit/s per core against a copying sender. Changing the kernel build for
   (c) is acceptable; the text says which configuration each result used.
   Jumbo frames are a fourth row where the link allows them.
5. **Memory.** Fragment slack; the limit, which is the size of the store's
   arrays (a segment holds at most one receive buffer), reserved against
   locked memory and charged to the memory cgroup once, at registration, with
   nothing charged or counted while data flows; a full store: the request
   fails at once (`-ENOMEM`), nothing waits, the bytes stay in TCP and its
   flow control slows the writer; what is deferred: growing the arrays, and
   slack, with its three remedies (smaller buffers in the pool, a NIC that
   fills the page, compaction). Compaction is a side story (the owner,
   2026-11-11): the design defers it, and §8.4 reports what the measured
   builds' dense copy cost (F3).
6. **Data-touching operations.** Transforms that read an object in the
   kernel and produce a new one: a dense copy, checksums, compression and
   hashing for dedup; encryption through kernel TLS, whose `read_sock` path
   (6.6 and later) lets capture run after decryption. None is in the design
   today, and kernel TLS sockets are refused. TODO(decide): which are design
   subsections and which are discussion.
7. **Safety.** The threat model and the invariants: immutable, confined,
   conserved, live. A handle never confers a mapping, so range checks are the
   whole guarantee. Shared and zero-copy fragments from same-host senders are
   copied at capture (TODO(decide)).
8. **The cost model.** Fixed per message, per fragment, nothing per byte; the
   predicted break-even and the per-fragment floor that compaction and jumbo
   frames lower.
9. **The hit path.** TODO(decide); not in v9 (ledger A15 is T). A BPF program in the receive path answers
   a GET for a cached key by handle, from a map the store owns; misses and
   writes go to userspace. Policy stays in userspace; the program executes a
   lookup. Reply ordering on pipelined connections, the hook point, and
   sending from the receive path are the problems (`../KV_CACHE.md` §8.1).

### §7 Applications (§6 before 2026-10-31)

TODO(decide) which of these are built; ordered by how long payload is held.

In the draft since the revision of 2026-11-13, §7 describes what is built and
says what is not. Built: the relay of the first row, framed and as a byte
stream, by name and over the mapped area (bound and raw); a memcached-protocol
store, the classical counterpart of the third row, with copying or named
storage in one binary; retention and fan-out as modes of the benchmark tool.
Not built: the mapped mode's editing relay, which §7 gives as a design example,
and the relay that keeps, mirrors and replays. The other rows are not in the
draft.

| Application | Shows | Compared against |
| --- | --- | --- |
| L7 proxy / TCP split (1→1) | parity with in-kernel forwarding, break-even payload size | copy proxy, splice, sockmap |
| Fan-out relay or pub/sub broker (1→N) | one copy for N receivers | copy per receiver, sendfile from a RAM file |
| LLM KV-cache store (large-value KV) | hold and re-serve multi-megabyte values to many inference nodes; GB/s per core; one copy for N readers | Redis, memcached, RAM file plus sendfile |
| Caching proxy | hold and re-serve, memory per object | nginx proxy cache |
| Relay over several WAN tunnels | one object sent on N paths, striped or duplicated; replay on another path after a failure. TODO(decide) | copy per path |
| Storage gateway on the TCP data path of a KV object tier (added 2026-10-03) | a gateway that relays KV chunks it never reads from an object store to many GPU nodes: hot chunks held once, layer-ordered gathers of ranges from many objects sent by reference. The TCP counterpart of ObjectCache, which keeps its S3 gateway on control only by moving data over RDMA (`../refs/papers.md` §17). It shows both regimes in one application: a miss is forward-once from the backend, a hit is hold-and-serve. It needs a send that gathers ranges from many objects, which the store does not. Backend `TODO(decide)`: a caching gateway in front of any S3 endpoint, with HTTP on both sides (the caching-proxy row above, pointed at an object store), or one that speaks the object store's own protocol. The S3 front end is item C2 of `../refs/blogs.md`; GETs pass through untouched, but S3 PUTs may require hashing the body (signed payloads, ETag), `TODO(verify)` in the gateway. Bar-raiser notes in `../critisizm/bar_raiser_tierd_KV.md` §7. A bar-raiser scenario: if not built, it is named in §6 as a candidate and in §9 as future work. TODO(decide) | the gateway over TCP as shipped (Ceph RGW's S3 path); ObjectCache's RDMA numbers as the ceiling |

### §8 Evaluation, as it stands (2026-11-13, `fcba7b4`)

The revision reports the recorded results of the earlier prototypes and collects everything
that is not measured in a closing subsection. Every number is a `CLAIMS.md` row, quoted with its
bound in place: the throughput group, pooled means, one run, another build. The figures show
only what is measured, so they no longer have the form of "by direction" below (CPU against
message size, a panel for one core and one for several). That form is the owner's direction of
2026-11-10 and stays the plan for the rerun's figures
(`../eval/2026-11-11-fixed-objects/EVAL.md`).

| # | Subsection | Figure | CLAIMS rows | What it says |
| --- | --- | --- | --- | --- |
| – | Opening | a table of recorded coverage by access mode | – | Named, read-only: capture and discard with 64 KiB source writes; prepared-data sends from 1 KiB to 1 MiB; forwarding; store capacity. Mapped, read-write: raw receive, bound capture, raw and bound forwarding with 64 KiB source writes; targeted correctness checks. Both: relay request/response at 64 bytes. The mapped tests receive and forward unchanged payload; there is no result for application writes, for the revised metadata or reclamation, or on four cores. |
| 8.1 | Setup and comparison rules | – | F24, F7, S13, S16, M2, F6 | Testbed and metric: cloud VMs with virtio-net, a separate interface for each leg, an experimental Linux 7.3-rc6 kernel on the machine under test, no source pacing; CPU-seconds per GB over all CPUs of the machine under test, threads and interrupts pinned, one connection. Throughput variation: full-speed receive runs fall into two groups whose cause is unresolved; the receive figure uses the faster group, the send and forwarding figures average all runs. Marks are means with one sample standard deviation, and a percentage uses its own experiment's baseline. Controls: the packet tap removed, notification chains bounded, interrupt time accounted, bytes verified separately. One verify marker: machine shapes, link allocations and kernel identities. |
| 8.2 | Receive through both modes | `figures/rx.tex`: five receivers with 64 KiB source writes, one column | F5, F6, F24 | Availability first: hardware zero-copy receive refuses the testbed's virtio-net (`EOPNOTSUPP`); the named path works there, and the mapped path with the driver changes; "not performance parity with hardware-assisted receive on a supported NIC". Five receivers of one stream, in CPU-s per GB in the faster group: `recv()` 0.335; named capture 0.194, 42% less; bound capture 0.186; raw mapped receive 0.160; discard 0.128, 62% less, "a lower-work reference". The slower group's capture estimate rests on one run. |
| 8.3 | Sending prepared objects | `figures/tx.tex`: CPU against buffer or range size, 1 KiB to 1 MiB; the left panel one range per request (`send()`, `SEND_ZC` pinned per send, `SEND_ZC` registered, named), the right panel eight ranges per request (`SEND_ZC` registered, named) | S1, S2, S10, S3, S5, S6, S11, S12, N2, N3 | Objects are made from application bytes before the timed loop: the reuse of prepared data, not the sending of small received fragments. Against a per-send pin: 15% less at 4 KiB, 22–27% from 16 KiB, no comparison at 1 KiB (S1); the pin itself, `SEND_ZC` against its own registered buffer, 17–90% from 4 KiB (S2). Against registered buffers with eight ranges per request: 6–12% more at 1 and 4 KiB, from 21% less to 4% more from 16 KiB, "without asserting statistical equivalence" (S3); the absent reuse tracking is by design and carries no number (S10). Batching: 3.6× at 1 KiB (S5). Against `send()`: 40–57% less from 16 KiB, 4% and 5% more at 1 and 4 KiB (S6). `MSG_ZEROCOPY` (12–48% less) and `sendfile` (28–39% less, 64 KiB to 1 MiB), each against its own campaign's baseline (S11, S12). The splice-helper study in two sentences (N2, N3). |
| 8.4 | Forwarding through both modes | `figures/fwd.tex`: `splice`, the named relay, the bound relay and the raw mapped relay with 64 KiB source writes, one column | F10, F1, F2, F3 | The bound relay's mean cost is 16% below `splice` at comparable mean throughput (F10). The means pool both throughput groups; the raw mapped relay's runs are all in the slower group, so its lower plotted cost "cannot establish a matched-throughput advantage"; the bound relay's lead over the named relay is not isolated. From another build, as a separate comparison: the named relay 4% above `splice` and 23% below the copying relay (F1); the sockmap redirect delivered 0.81 GB/s with a queue that grew without effective backpressure (F1, F2). Compaction: 17% above the relay without it at full speed (F3), and deferred in the design. |
| 8.5 | Request/response | – | F8, F10 | One connection, 64-byte requests and responses. Median latency 59 µs without a relay and 146–159 µs through the forwarders; the named relay 37.0 CPU-seconds per million transactions against 32.9 for `splice` and 30.8 for the copying relay, 12–20% more (F8). In the mapped-forwarding experiment the bound relay used 19% more CPU per transaction than `splice` (F10). |
| 8.6 | Store capacity | – | K1 | A capacity measurement, not a throughput or latency comparison: with the same configured 1 GiB budget the named store holds about a fifth less payload than the copying store from 64 KiB values and half as much at 4 KiB, one run per size; the same nominal budget is not the same physical footprint. |
| 8.7 | Planned experiments | – | – | First implement and test the revised design, then compare it with the earlier prototype on the same testbed: receive, send, forwarding, request/response and the store by size, on one core and four, reporting delivered throughput and keeping the throughput groups apart. The gaps beyond that: a copying `io_uring` receiver; sending captured, not generated, objects; all forwarding paths on one build; hardware-assisted receive on a supported NIC; the store against its own copying path, memcached and Redis; retention, concurrent fan-out, slow readers and recovery from a full store; keep, mirror and replay. For mapped access: direct reads and same-length edits before sending, with byte verification, and private-header parsing stressed under mapper writes. One measure marker. |

Against the rules of "by direction" below:
- **Full speed only** holds: no paced number is in the draft.
- **A figure per direction against message size, with a one-core and a several-core panel:**
  not in this draft. Receive and forwarding are one-column figures of the receivers and the
  relays with 64 KiB source writes, the only size measured. Send keeps the size axis, and its
  two panels are split by ranges per request, not by cores. `osdi27/figures/mkdata.py` still
  emits the size grid, and its header says no unmeasured panel is shown.
- **The summary first:** not in this draft. The section opens with the coverage table.
- **The claim §8 tests** (the owner, 2026-11-11: both modes as fast as or faster than the
  existing zero-copy mechanisms): the draft does not state it. Its conclusion says the earlier
  prototypes "show lower CPU cost in selected bulk receive, send and relay comparisons,
  alongside higher small-transaction cost and reduced capacity in the named store".
- **The raw mapped relay's 30%** (F10) is not quoted; the row for §8.4 has the draft's reason.
- **Request/response and the store have no figure.** `osdi27/figures/rrkv.tex`, the placeholder
  of `ac90ed9`, is in the tree and not included.
- **The five experiments** of `../eval/2026-11-11-fixed-objects/EVAL.md` are the first paragraph
  of §8.7, as the plan for the revised build.

### §8 Evaluation, by direction (2026-11-10)

The owner, 2026-11-10: "we are demonstrating that opaque buffers perform well for RX TX and
the combined forwarding we should show and plot it against different message sizes on single
core and multiple cores. all the rest is for your consideration". This supersedes the order of
the table below ("in the order of the title"); that table's rows stay as the record of what each
measurement answers. The draft (`osdi27/sections/08-evaluation.tex`) had this shape from that
day until the revision of 2026-11-13 ("as it stands", above). The long form it replaced is kept
beside it, unbuilt, in
`osdi27/sections/08-evaluation-long.tex`: the eight methodology traps, the paced cells, the
transmit table and the send path's mechanism rows.

| # | Subsection | Figure | Drawn from | Measured today |
| --- | --- | --- | --- | --- |
| 8.1 | Setup | – | – | The testbed; one core and several; the metric (full speed, CPU per GB, whole host); the baselines; five precautions (the long form's traps 1–6, condensed). |
| 8.2 | Receive | `figures/rx.tex` | `capt2-ab`, new-kernel passes (dev6); F24 | Full speed at 64 KiB only. Nothing on several cores. |
| 8.3 | Send | `figures/tx.tex` | `tx-eor/final`, `final/{zcopy,bprime}`; S1–S6, S11, S12 | 1 KiB to 1 MiB. Nothing on several cores. |
| 8.4 | Forwarding | `figures/fwd.tex` | `maio/campaign2/F`; F10 | 64 KiB only, and four of the six forwarders on that kernel (the copying relay and sockmap are F1, another kernel, in the text). Nothing on several cores. |
| 8.5 | Retention and the store | – | – | Nothing: the markers of the old 8.5 and 8.6. |

Rules of this form:
- **Full speed only.** No paced number is in the draft since 2026-11-13: the 1 KiB paced
  break-even (S6) and F10's two paced deltas went with the cut of `ac90ed9`.
- **Each figure:** CPU-s per GB of the machine under test against message size (1 KiB to 1 MiB),
  log scales; left panel one connection on one data-path core, right panel N connections on N
  cores; one campaign per figure. `osdi27/figures/mkdata.py` emits the data (`make data`); a cell
  that was not measured is `nan` and is skipped, so a figure fills in when its campaign lands.
  Since the revision of 2026-11-13 the draft's figures show only measured cells and do not have
  this form ("as it stands", above).
- **The summary comes first:** four findings (receive, send, forwarding, the price of a name)
  before the setup. Not in the draft since the revision of 2026-11-13.
- **What the figures still need:** campaign 9 of `../eval/PAPER-CAMPAIGNS.md` (added
  2026-11-10): receive by size (its campaign 5, F22); forwarding by size with the six forwarders
  on one kernel (F25); all three on several cores (S27, F26, F27); and the three experiments on
  one build of the evaluation kernel.
- **What a several-core panel can show on kc1–kc3:** two and four connections deliver about
  22.8 Gbit/s in total, and on send one core fills that path from 4 KiB. N connections therefore
  share the path's rate, and a mark is read against the other mechanisms at the same N and
  against one connection paced at the same rate per connection, not against the one-core panel
  (campaign 9, "What the several-core runs meet on this testbed").
- **What the evaluation is advised to run first, and why** (2026-11-11):
  `../eval/EVAL-RECOMMENDATIONS.md`, R1–R10. On receive and in forwarding the size is the unit of
  the machine under test, with the source always writing 64 KiB (R4).
- **The forwarders' full-speed rates are not resolved** (2026-11-11). F1's "delivering 21% more"
  and F10's "delivering 11% less" compare means of three runs that span the path's two
  throughput regimes; `splice` reached 22.1 Gbit/s in F10's campaign and never passed 16.5 in
  F1's. Both rows are annotated in `../CLAIMS.md`. The draft says it in §8.1 ("Throughput
  variation"), in §8.4 and in the send and forwarding captions (R1).
- **The claim §8 tests** (the owner, 2026-11-11): both modes are hardware agnostic (the mapped
  mode needs a driver change, the named mode none), the two perform alike, and both are as fast
  as or faster than the existing zero-copy mechanisms. Where the record stands against it, cell
  by cell, measured or not measured: `../eval/EVAL-RECOMMENDATIONS.md`, R9.
- **The named relay copies nothing** (corrected 2026-11-11). CLAIMS F1, two results files and
  the draft said it copied every byte once on receive: its passes set the buffer-cache knobs,
  which act only on a `COMPACT` request, and it sends none. The draft was corrected that day.
  Since 2026-11-13 only §8.4 quotes F1, as 4% above `splice` and 23% below the copying relay;
  of F10's comparison it says that neither relay requests compaction.
- **Compaction is a side story** (the owner, 2026-11-11): one paragraph of §8.4, with the cost
  of a relay that compacts every object (F3).
- **§8's numbers predate the design's object metadata** (2026-11-11). The design keeps an object
  in fixed-size heads and segments from arrays, recycles them through a magazine allocator,
  allocates nothing while data flows and collects a mapped area's buffers lazily
  (`../docs/design_spec.md`; the decisions are recorded in
  `../eval/2026-11-11-fixed-objects/DESIGN.md`). §3 to §6 of the draft follow it since
  2026-11-11. Every named and mapped number of §8 is measured again on it
  (`../eval/2026-11-11-fixed-objects/EVAL.md`): send, receive, forwarding, request/response and the
  key-value store, each by size, at full speed, on one core and on four. Deferred with compaction:
  H2 (its latency while it runs) and the compacted row of the three send configurations (Q6).
- **On the five experiments** (2026-11-13, `ac90ed9`; replaced by the revision the same day).
  §8 followed the set the owner fixed on 2026-11-11
  (`../eval/2026-11-11-fixed-objects/EVAL.md`): Setup, Receive, Send, Forwarding,
  Request/response (F8 and F10's 64-byte row, until it is measured by size) and The key-value
  store (K1, capacity, until K2 to K4), on one core and on four, with
  `osdi27/figures/rrkv.tex` holding the place of the last two figures and five findings opening
  the section. The paced numbers left §8, §1 and the abstract with that cut: F10 quoted as 16%
  at equal throughput, F1 as 4%, F3 as 17%, and S6 in its full-speed form (one object per send
  costs what `send()` costs at 1 and 4 KiB). The revision keeps those four as they are.
- **The mapped relay's 30%** (F10) was written with its caveat at `ac90ed9`: its three runs all
  fell in the path's slower regime while `splice`'s span both. The revision does not quote it.

### §8 Evaluation, in the order of the title (2026-10-31)

Superseded in its order by the section above (2026-11-10); kept as the record of what each
measurement answers.

Approved with the plan. Each subsection names the claim it answers, the
legs, questions and micro-suite tests below that it absorbs, and what is
measured today (`../FINDINGS-LEDGER.md`, `../eval/`). CPU is kc2
CPU-seconds per GB, total minus idle, on the accounting-fixed kernels with
the NIC interrupts pinned (`../eval/2026-11-01-pinned/`).

| # | Subsection | Answers | Absorbs | Measured today |
| --- | --- | --- | --- | --- |
| 8.1 | Measuring zero copy honestly | the method | the methodology rows of the ledger | Testbed and accounting: ledger 7 and 17 (interrupt time booked as idle on 7.3; fixed and validated), 15 (a packet tap turns `SEND_ZC` into a copy), 19 (the usage report over-counts copies), 20 (concurrent sends interleave; ordered senders), 27 (interrupt placement moves CPU per GB 10–25%; pinned since 2026-10-31). A half-column box with the four traps. |
| 8.2 | Ubiquitous | leg 1, where | W1–W4; Q2 (a); T1, T8 | Ledger 13: zcrx refused on virtio-net (the probe). The mapped mode registers on virtio-net (`fb_uring zcsink`; F6). Discard against `recv()` has two values, one per `recv()` copy cost (M6, trap 8): 1.61–1.65× with the dear copy, 1.16–1.26× with the cheap one (F4, `../eval/2026-11-01-pinned/capture-ab`). The mapped mode 24.2–26.5% below named capture paced and 35% at full speed (F9); the cheaper-capture series K1–K4 cuts capture 11% and bound capture 14% at full speed and narrows that lead to 23% (F12, before the review fixes). Missing: the availability table across drivers, VMs, loopback and veth; the gVNIC row (hardware zcrx, parity; Phase B on GCP). |
| 8.3 | Zero copy without overhead | the four costs | Q6; T2 | The pinned transmit table with every `SEND_ZC` sender's notification chains bounded (`../eval/2026-11-01-pinned/tx-eor/final/`; S16 is the sixth methodology trap, M5 the seventh): against per-send pinning 6–27% cheaper, unresolved at 1 KiB full speed (S1, S2); against a registered, never-rewritten buffer a tie from 16 KiB and when paced, a win on contiguous ranges, 6–12% more at 1–4 KiB saturation (S3, S4), with the remainder profiled to the request path (S15, S17–S19); the tap experiment (F7, S13, S14); `MSG_ZEROCOPY` 7–48% dearer (S11); the break-even: at 1 KiB and 0.5 GB/s a copying `send()` is the cheapest sender (S6). Missing: mapped-mode pages sent through the binding. |
| 8.4 | Forwarding | leg 2, parity | X1; Q1, Q4; T3, T4 | Relay, `splice`, sockmap, named, named with compaction: ledger 21 (parity with `splice` at 1 GB/s, +8–20% at saturation by front node, below the copying relay), 2 and 3 (sockmap's unbounded, invisible queue), 4 and 16 (compaction after the buffer-cache fix), 11 (latency). The mapped relay is F10 (the bound forwarder and the `RECV_ZC` → `SEND_ZC` relay). Missing: the named relay without compaction on the final kernel; compaction's remaining cost and its latency while it runs (H2); the keep, mirror and replay demonstration (X4). |
| 8.5 | Retention | leg 3, how; leg 2, hold and fan out | X3, X4, H1–H4; Q3, Q5; T5–T7 | Nothing beyond compaction's cost (ledger 4, 16). To measure: holding at equal memory, fan-out against N, density as applications per NIC (named unlimited, mapped one per device, zcrx a queue each), behaviour at the budget. |
| 8.6 | The store | leg 2, serve; the application | X2, X5; Q7; `../experiments/kv_store.md` E1–E9 | The 2026-10-26 qualification only (COPY GET 830 k operations/s), on the old kernel. To measure: against Redis, Valkey and memcached on the final kernel; the continuum figure; capacity at equal memory. |

The campaigns still needed, in the order the story needs them, are in
`../eval/PAPER-CAMPAIGNS.md` (2026-11-03); the rows above keep the per-section
detail.

Order rule: 8.2 before 8.3, because the title's claim word is ubiquitous
and its explanation is the absent overhead; 8.4 to 8.6 are what lent
memory can do. Every figure carries the leg it answers as a label. The
legs, questions and coverage below are kept as the record of what each
figure must answer.

### §8 Evaluation, organized by the claim (proposed 2026-10-06; §7 before 2026-10-31)

The owner (2026-10-06), after approving the claim: "next we need to focus our
evaluation on this". Every experiment answers one leg of the claim
(`novelty.md`, "The claim, in one sentence"): where SYS runs, what received
payload can do, and how retention stays bounded. Experiments that answer no
leg are demoted to applications or ablations. One rule holds throughout:
every comparison in legs 2 and 3 runs at equal total memory
(`../critisizm/astra_intro_review_2026-10-06.md` §3). Proposed for the
owner's review; it reorganizes the questions below and
`../experiments/kv_store.md` without replacing them.

**Leg 1, where: any NIC, VMs and containers, no per-application hardware setup.**

| # | Question | Experiment | Baselines | Metric | Planned in | State |
| --- | --- | --- | --- | --- | --- | --- |
| W1 | In how many environments does each mechanism give zero copy? | the relay and the store's GET path on: a physical NIC with header split (bnxt or mlx5), an Intel NIC, AWS (ENA), Azure (MANA), GCP (gVNIC and virtio-net), two containers over veth, loopback | `RECV_ZC`: registers or not, header split and steering available | share of bytes captured by reference; throughput against the copy relay | portability table (below); bar-raiser R5 | **missing** |
| W2 | Where `RECV_ZC` runs, how does SYS compare? | Q2 (a)–(e) on a header-split NIC | `RECV_ZC` + `send_zc` | as in Q2 | Q2 | **missing; required** |
| W3 | What does each need per application? | N applications on one NIC (Q2 d) | `RECV_ZC`: a queue and steering rules each | applications per NIC; queues and rules configured | Q2 (d) | **missing** |
| W4 | Does it hold where there is no NIC? | a proxy between two containers over veth, and between processes over loopback | copy, `splice`, sockmap | cycles/byte, Gbps/core | — | **missing** |

**Leg 2, what: received payload kept, sliced, sent to many, served from the kernel.**

| # | Question | Experiment | Baselines | Metric | Planned in | State |
| --- | --- | --- | --- | --- | --- | --- |
| X1 | Does forwarding stay at least as cheap? | the TCP-split relay, 1 KB–1 MB | copy, `splice`, sockmap | cycles/byte, Gbps/core | Q1 | **missing** |
| X2 | Keep and serve | the store's GET and SET across value sizes | its own data paths: copy, zero-copy send, RAM file + `sendfile`, `splice` + `tee`, `RECV_ZC`; Redis, Valkey, memcached | CPU per request and per GB, throughput, memory | `../experiments/kv_store.md` E1, E2, §2.6 | planned |
| X3 | One payload to many | one object to N receivers | copy per receiver, RAM file + `sendfile`, `splice` + `tee` (a pipe per object, a `tee` per send), `RECV_ZC` + `send_zc` | memory and CPU against N | Q3; `kv_store.md` E4 | planned in part |
| X4 | Keep what you forward | a relay that keeps the body, mirrors it to a second backend and replays it after a backend failure | a copying proxy that buffers for retry; `splice` (cannot); `tee` (mirror only, within a pipe's limits) | capability table; CPU and memory per mirrored or retried byte | coverage table below ("Control"); bar-raiser R3 | **missing** |
| X5 | Serve from the kernel | small GETs, the hit path on and off | Redis, Valkey, memcached, BMC (UDP), the `RECV_ZC` store | GETs per second per core, p50, p99 | `kv_store.md` E1, E5, §2.3; Q2 (e) | planned |
| X6 | One interface for all of it | lines of code per feature: relay, cache, fan-out, mirror and replay, hit path | the sizes published for the special-purpose systems (Beeline's policies, XLB, BPF-Broker) | lines of code | — | **missing** |

**Leg 3, how: holding network buffers safe and bounded.**

| # | Question | Experiment | Baselines | Metric | Planned in | State |
| --- | --- | --- | --- | --- | --- | --- |
| H1 | What does a held byte cost? | bytes pinned per payload byte by driver and value size, compaction off and on | the copy store at equal memory | pinned bytes per payload byte; capacity at equal physical memory | Q5; `kv_store.md` E4 | planned |
| H2 | Does compaction stay off the data path? | background compaction's CPU and its effect on data-path latency; policies compared | always compact, never compact | CPU; p99 while compaction runs | Q5 | partial |
| H3 | Is memory bounded under pressure? | holding past the budget; slow and hostile readers; teardown | — | memory against the budget; behavior at the limit; leaks | robustness row below; `kv_store.md` E9 | partial |
| H4 | Do concurrent sends stay bounded? | one object on many sockets, some receivers slow | copy per receiver | memory bound; throughput of the fast receivers | — | **missing** |

**Headline figures.**

1. Availability (W1): the environments where SYS gives zero copy against
   those where `RECV_ZC` can register. Its count is the first number in the
   introduction (bar-raiser R5).
2. The continuum (X2, X5; `kv_store.md` E1): CPU per request and per GB
   against value size, all of the store's data paths and the external
   stores, both regimes in one figure.
3. Holding and fan-out at equal memory (X3, H1): memory against N readers,
   and capacity at equal physical memory.
4. `RECV_ZC` head to head (W2): parity on the relay; holding, density and
   small GETs.
5. Memory amplification and compaction (H1, H2).

Tables: the capability table (X4) and lines of code per feature (X6).

**Demoted**, because they answer no leg directly: LLM inference against a
remote KV-cache service (a motivating application in §6, not a leg); KV
files on CephFS delivered to GPUs (feasibility open); Q4, memory per
connection at a million connections (secondary to leg 3); Q6, transmit
granularity (an ablation under leg 3); `kv_store.md` E3 and E8 (core scaling
and traces, kept for realism); Redis over DPDK (the small-value ceiling, a
line on the continuum, not a leg).

**Micro suite (2026-10-11):** `../experiments/microbench.md` gives W1–W4,
X1, X3, X5, H1, H3 and H4 their experiments as one netperf-like tool over
every data path (T1–T8), with hardware and software `NODEV` `RECV_ZC` as
required rows, and proposes the macro set (store, HAProxy as relay and as
sidecar, HTTP cache; the LLM tier one figure).

**v9 plan (2026-10-13):** `../experiments/evaluation.md` maps every
experiment above to the advantage ledger of `../PROJECT_MAP.md`, sets the
three comparison tiers, and orders the work in phases (the owner: no cuts):
Phase 1 on IBM Cloud with any NIC, the micro tool, the store against
memcached then Redis and Valkey, the relay against HAProxy, one LLM figure;
Phase 2 the eBPF hook, the store's hit path first (X5, Q2 e); Phase B the
hardware `RECV_ZC` rows and the DPDK line on GCP; Phase 3 the HAProxy port,
the HTTP cache, kernel TLS, crc, FILL and the storage gateway. It carries
the test plan: machines, hardware and kernel builds per test.

**Gaps to close before the figures exist:** the `splice` + `tee` variant
(added to `kv_store.md` §2.6 the same day); W1's environments (one VM type
per cloud, a container pair); W3, W4, X4, X6 and H4, which no plan has yet;
the equal-memory rule in `kv_store.md` §6.

**Testbed this implies:** a physical pair at 100 GbE with a header-split NIC
(bnxt, or mlx5 on Linux 6.17 or later) for W2 and the continuum; one VM type
in each of AWS, Azure and GCP for W1; local containers for W4.

### §8 Evaluation: questions

| Q | Question | Experiment | Baselines | Metric |
| --- | --- | --- | --- | --- |
| 1 | What does forwarding cost per byte, from what payload size does SYS beat copying, and does it match in-kernel forwarding? | TCP-split relay, 1 KB–1 MB payloads, single core | read/write, io_uring, splice, sockmap | cycles/byte, Gbps/core |
| 2 | Where hardware-assisted zero-copy receive runs, does SYS match it without the hardware, and what can SYS do there that it cannot? Required (the owner, 2026-10-06) | on a header-split NIC: (a) the relay, 1 KB–1 MB; (b) a cache holding a growing share of what it receives; (c) one object to N receivers; (d) N applications on one NIC; (e) small GETs: SYS with the eBPF hit path against a zcrx store, which must serve every request in userspace | io_uring zcrx + send_zc; for (b), zcrx holding area buffers and zcrx copying out | (a) cycles/byte, Gbps/core; (b) receive throughput and drops against the share held, CPU; (c) memory and CPU against N; (d) applications per NIC; (e) GETs per second per core, p50 and p99 |
| 3 | What does one object cost when N connections receive it? | hot object, N concurrent receivers | copy per receiver, RAM file + sendfile | memory vs N, cycles/byte |
| 4 | How much memory does a connection cost? | 10k–1M mostly idle proxied connections | copy proxy | bytes per connection |
| 5 | What does holding cost, and what does compaction buy? | cache fill at varying object sizes | no compaction, always compact | bytes held per payload byte, CPU |
| 6 | What do small received fragments cost on transmit, and what do compaction and a larger fragment limit recover? | send 1 MB objects three ways: compacted (64 KB packets), the stock scatter-gather of received fragments, received fragments on a kernel built with a larger fragment limit | the copy sender's dense packets | transmit packets and descriptors per MB, cycles/byte, Gbps/core |
| 7 | Do real applications benefit? | the applications in §6 | their stock versions | throughput, tail latency, memory |

### §8 Evaluation: coverage (2026-10-02)

Implementation priority (2026-10-03): the opaque object interface and the
userspace + eBPF KV store come first. Forwarding initially provides a small
smoke/parity reference; broader relay and application work follows the first
object/KV result. Evaluate SYS's payload mechanism and the eBPF request path
separately, including the userspace-only store as an ablation. Execution gates
are in [`../workplan.md`](../workplan.md).

Three evaluation plans exist: the store (`KV_CACHE.md` §10), LLM inference
with a remote KV-cache service, and KV files on CephFS delivered to GPUs (both
in `experiments/llm_inference.md`).

| Need | Backs which claim | Planned in | State |
| --- | --- | --- | --- |
| Q3, Q5: one copy for N readers; cost of holding | hold and reuse; memory | `KV_CACHE.md` §10.3–10.4 | planned |
| Q6: transmit granularity, three send configurations | the send path against a copying sender's packet rate; what compaction and the build option recover | `KV_CACHE.md` §10.8 | planned |
| Q7: real applications | usefulness | `KV_CACHE.md` §10.2, `experiments/llm_inference.md` | planned, with our own server |
| Q1, Q4: TCP-split relay. One relay built four ways: copy, splice, sockmap, kpass. Cost per byte and per call, break-even payload size, memory per connection, scaling with cores and connections. | Forwarding: ahead of copy, **parity** with in-kernel forwarding. Superiority is not claimed. | — | **missing** |
| What only kpass can express on that relay: keep the payload while forwarding it, send it to a second destination, replay it to another backend after a failure | Control: the reason to prefer it at parity | — | **missing**; a capability table plus one demonstration each |
| Portability table: the relay run on each NIC driver and kernel available (see below) | "any NIC, no per-application hardware setup"; this is the evidence for software-only | — | **missing**; cheap, one row per machine |
| Q2: against `io_uring` zero-copy receive and send on a header-split NIC | commodity hardware, the claim to fame (the owner, 2026-10-06): parity where zero-copy receive runs, and the holding, fan-out and density cases it cannot serve without copies or queues | `experiments/kv_store.md` §2.6 (a zcrx variant of the store) | **missing; required** |
| Copier and zIO | none: they speed up applications that keep the bytes in userspace, a different goal | related work only | not planned. Both did evaluate Redis, so "Redis with zIO" could be one extra bar if a reviewer asks. |
| Robustness: memory pressure, budget enforcement, slow or hostile peers, leak-free teardown | safe to deploy | partly in `KV_CACHE.md` §10.1 | partial |
| Ablations: inline against deferred execution, compaction, size threshold | design choices matter | partly in `KV_CACHE.md` §10.4 | partial |
| A port of one existing server or proxy, with lines changed | adoption cost | — | optional, strong |
| Kernel TLS | one figure as a limit: capture through the TLS layer's `read_sock`, hardware kTLS by reference, software kTLS one crypto pass | — | cut for v9, which refuses kernel TLS sockets (ledger A16); a limitation sentence in §9 |

**The portability table.** kpass works on any driver, but how well depends on
how the driver lays out received data. One row per NIC driver, from the same
relay benchmark:

| Column | Why it matters |
| --- | --- |
| Driver, and virtual or physical | The claim is "any NIC"; virtual NICs are where hardware-assisted paths are unavailable |
| Share of bytes captured by reference | Fragments and page-fragment heads can be held by reference; slab-backed heads and cloned packets have to be copied. A driver that delivers everything in slab-backed linear buffers turns kpass into a copy |
| Bytes pinned per payload byte | Receive buffer size differs per driver (half a page, a page, more with jumbo frames), and it decides the memory cost of holding |
| Throughput and CPU against the copy relay | Whether the benefit survives on that driver |
| Kernel versions the series applies to and runs on | the series is small and its base recent; never "stock kernel" |
| Zero-copy receive available: header/data split, flow steering, zcrx registers | The count of environments where hardware-assisted zero-copy receive runs at all (bar-raiser R5; the owner, 2026-10-06). Rows for loopback and veth, which have no queue to register |

Testbed: TODO(decide). At least one virtual NIC and one physical NIC.

### The LLM angle

Related work, what to build and the staged experiment plan are in
[`../LLM.md`](../LLM.md).

The large-value store is evaluated first as a KV cache against classical
ones (Framing above). LLM inference is one of its workloads, the motivating
one where RDMA is absent, described in `problem.md` §2.1. It fits the design
on every axis:
values are 0.5–40 MB, immutable, written once, and fetched by many nodes that
share a prompt prefix; the metric is GB/s, not requests per second.

- **Store-level evaluation needs no GPU.** Drive the store with the value
  sizes and access pattern of the workload and report GB/s per core, server
  CPU, and memory against the number of concurrent readers. Baseline: Redis,
  which the LMCache project already uses as a remote backend.
- **End-to-end evaluation needs one GPU node.** Run LMCache with vLLM against
  the store and report time to first token. TODO(decide): in scope.
- **Positioning.** Mooncake moves KV caches with a transfer engine for RDMA
  that also offers TCP (`../refs/papers.md` §20); in its successor TENT,
  TCP is the fallback, 322 lines of TCP RPC against 6,975 for RDMA, and one
  misrouted fallback to TCP degraded service severely (`../refs/papers.md`
  §24; TENT §2.1, §5.3). SYS makes TCP on commodity NICs the main path, not
  the fallback.

## Figures and tables (2026-10-31)

**In the draft since the revision of 2026-11-13 (`fcba7b4`):**

- **Table 1** (§2, `tab:space`) is a new table, not the one decided below. Four columns:
  mechanism; access and reuse; setup requirements; remaining costs or limits. Three groups.
  Existing Linux interfaces: `MSG_ZEROCOPY` and default `SEND_ZC`; registered `SEND_ZC`;
  hardware `RECV_ZC`; devmem TCP; TCP page remapping; `vmsplice`; AF_XDP zero copy; `splice`
  and `tee`; page cache + `sendfile`; sockmap. Research designs: IO-Lite; PASTE; MAIO. The two
  modes, as "kernel extensions". It has no grouping by who lends and no columns for the four
  costs; MAIO is a row; netkit's lease, fbufs, Libra, XLB, Beeline and BMC are in the text of
  §2.2 and §9. The decided table is in `ac90ed9`.
- **Figure 1** (§3, `fig:modes`), drawn in TikZ: the two access contracts across the boundary
  between application and kernel. Named access leaves ordinary receive pages unmapped behind
  checked operations; mapped access exposes the segregated I/O area read/write; private kernel
  memory holds the control structures and the header copies. It takes the place of items 1 and
  2 of the list below.
- **Figures 2 to 4** (§8): `osdi27/figures/rx.tex`, `tx.tex` and `fwd.tex`, as "§8 Evaluation,
  as it stands" describes them.
- **Table 2** (§5, `tab:mapped`): the two lifetime interfaces over the area, raw buffers and
  mapped objects, by receive result, return of receive references, forwarding, and what
  holding uses. It replaces the table of the three receive modes.
- **Two tables without a number:** the eight operations (§4.1) and the recorded coverage by
  access mode (the opening of §8).

**Table 1, decided** (the owner, 2026-10-31: "looks good lets add this to the paper"). In
§2 of the LaTeX (`osdi27/sections/02-borrowing.tex`, `tab:space`). Rows grouped by who
lends: the application per operation (`MSG_ZEROCOPY` and default `SEND_ZC`, `tcp_mmap`,
`vmsplice`); the application for the peak (registered `SEND_ZC`; `RECV_ZC` and netkit's
lease; devmem TCP; AF_XDP zero copy and PASTE; IO-Lite and fbufs); the kernel keeping the
bytes but fixing their fate (`splice` and `tee`; page cache + `sendfile`; sockmap; Libra, XLB,
Beeline; BMC); the kernel lending (kernel I/O memory, mapped; kernel I/O memory, named).
Columns: copy-free, TCP kept, the four costs (no pin, no notification, any NIC, ordinary
pages), held and sent to many, general usability. Only the two lend rows are full; the named
row's usability mark carries an asterisk (pass-through services; a byte reader uses the
mapped mode or copies a range). Out of the table: the 2021 design, the predecessor, cited in
the caption and the text; ublk's registered kernel buffers, one sentence in §9, not a network
mechanism. The mapped mode's privilege and its area are discussed in §5, not columns. The
eBPF rows get the paragraph beside the table: what each is and why it is not infrastructure;
sockmap's missing backpressure is our measurement.

**Figures, proposed 2026-10-31, to decide.** Seven figures and five further tables.
Since 2026-11-10 figures 3 to 5 are the three figures of "§8 Evaluation, by direction":
receive, send and forwarding, each against message size at full speed with a panel for one core
and one for several (`osdi27/figures/{rx,tx,fwd}.tex`, shared styles in `figures/style.tex`). The
paced panel of figure 3 and the transmit table are in the long form only.

1. Borrow against lend (§1): three boxes, application, kernel, NIC, in two panels; the four
   handshakes as arrows on the borrow side, none on the lend side.
2. One pool, two ways to borrow (§3): the driver's pages through TCP; a handle to a head and
   its chain of segments in the named mode, the area in the mapped mode; both send through the ordinary page
   path; the reference counts drawn on the pages. Absorbs "life of a payload".
3. The send side (§8.3): CPU per GB against size, paced and full speed, every sender including
   `MSG_ZEROCOPY` and `sendfile`; the headline figure. **Drawn 2026-10-31**: pgfplots in
   `osdi27/figures/tx.tex` from `osdi27/figures/data/tx_{paced,full}.csv`, which
   `osdi27/figures/mkdata.py` emits from `../eval/2026-11-01-pinned/tx-eor/final` (the transmit senders, every `SEND_ZC` sender's notification chains bounded) and `final/{zcopy,bprime}`
   (`make data`). The large transmit table stays beside it until the figure is accepted; then it
   becomes a five-row table of the deltas as `CLAIMS.md` states them.
4. Forwarding (§8.4): three rates for relay, `splice`, sockmap, named, named with compaction,
   with throughput labels; the sockmap queue growing over time as an inset. Data
   `final/fwd` and `../eval/2026-10-29-fwdbase`; drawable today.
5. Receive on any NIC (§8.2): sink CPU per GB for `recv()`, NODEV, the mapped mode and the
   named sink. Needs one pinned receive pass (the mapped mode's campaign script exists).
6. Fan-out (§8.5): memory and CPU against N receivers, named against a copy per receiver and
   `sendfile`. Not measured; the signature figure of the object.
7. The store continuum (§8.6): CPU per request and per GB against value size, all data paths
   and the external stores. Not measured.

Tables besides Table 1: the three receive modes (§5, in the draft); capability with lines of
code per feature (§7); the transmit deltas (§8.3); request/response latency (§8.4, F8);
availability across drivers, VMs, loopback and veth (§8.2, pending); holding cost against
object size (§8.5, pending). Tooling TODO(decide): pgfplots reading the committed CSVs inside
the build, since the development Mac has no matplotlib, or a Python environment.

### Previous list (2026-10-14)

1. Where the bytes sit today: receive queue, application buffer, send queue,
   cache (motivation figure).
2. Property matrix (from `novelty.md`).
3. The payload object: extents, handles, operations.
4. Life of a payload: forward, fan-out, hold and re-serve.
5. Who owns retained bytes (receive socket, pipe, file, object) against what
   follows: lifetime, order, bound, readers, coupling to flow control. The
   table that answers `tee`, Libra and `sendfile` at once.
6. Evaluation: one figure per question above.
7. Table 1, 2026 edition (`../talks/advisor_deck.md` slide 13): MAIO's
   Netdev 0x15 Table 1 reproduced with its five columns (zero-copy,
   exceptionless, statically mapped, network stack, generic use; the marks
   were read from the PDF on 2026-10-14) and extended with the rows that
   appeared since (`SEND_ZC`, `RECV_ZC`, devmem TCP, io_uring kernel
   buffers, PASTE, IO-Lite, Libra, the eBPF trio, zIO and Copier, tmpfs
   plus `sendfile`, SYS) and four columns (any NIC, never mapped, held,
   sent to many). Read in two passes: the 2021 columns are all met by
   mainline io_uring today; the four new columns are where only the last
   row is full. A second table scores the nine groups on the overheads
   each removes (calibration rule 8).
8. The Venn diagram (`../talks/advisor_deck.md` slide 14): Z, zero copy on
   the data path; C, commodity (stock stack, any NIC, no dedicated cores);
   R, retention (held, sliced, sent to many). Forward-once systems in Z∩C,
   mapped and hardware designs in Z∩R, copy-in stores in C∩R, SYS alone in
   the centre. Candidate for §1 or §2.4; the property matrix (figure 2)
   stays as the table behind it.

## Open decisions

Raised by the revision of 2026-11-13 (`fcba7b4`, `9dc23a7`). Each is the owner's to settle;
where two readings are given, the draft carries the second. Since 2026-11-14 the owner's
version is worked out in this file while the draft stands in parallel. A line marked "owner's
version" is settled for this file, and open only as the choice of the outgoing version.

- Owner's version, 2026-11-14: the title is "Lend, Don't Borrow: Hardware-Agnostic Zero Copy
  for the Network Stack" ("Working titles"). The draft has "Reusable TCP Payload in
  Kernel-Owned Memory".
- Owner's version, 2026-11-14: the story of the abstract and §1 is "The story": zero copy
  solved case by case, trust as the one cause, hardware independence as its consequence. Its
  seven steps are proposed on the owner's directions and await the owner's comments. The draft
  has two peer access contracts whose combination is the contribution.
- TODO(decide): whether §2 argues that the four costs follow from borrowing (the premise of
  2026-10-31), or lists them as obligations that depend on the interface (the draft); and with
  it the form of Table 1.
- Owner's version, 2026-11-14: the modes are "named" and "mapped", and no read-only mapped mode
  exists or is planned ("Conventions"). The draft has the "read-only named mode" and the
  "read-write mapped mode", with "segregated I/O memory". Still open: the mapped mode's proper
  name, and the word for its memory ("dedicated" is proposed under "The story").
- TODO(decide), 2026-11-14: the four proposals in "The story": tmpfs beside ramfs, the packet
  socket's ring as the mapped mode's precedent, two words, and what the story keeps from the
  draft.
- Owner's version, 2026-11-14: the novelty (trust as the diagnosis, *lend, don't borrow* as the
  rule, hardware-agnostic zero copy as the result) and the four contributions (the argument,
  the named mode, the mapped mode, the evidence) are in "The story", under "Novelty and
  contributions". Written on the owner's word; the single items await the owner's comments.
  The draft has three contributions and no thesis among them. `novelty.md` was brought up to
  them on 2026-11-15 on the owner's word; its one-sentence claim is proposed wording that the
  owner has not read.
- TODO(decide): where the object model lives. §4, as the named mode's depth (the plan), or
  §3.2, shared by both modes (the draft).
- TODO(decide): §8's figures until the rerun reports. The by-size panels for one core and
  four, with the unmeasured cells marked (the direction of 2026-11-10, as in `ac90ed9`), or
  only what is measured (the draft).
- TODO(decide): what is cut. The body is 0.64 page over the limit.
- TODO(decide): the QUIC extension that `quic-extension/main.tex` recommends (`../FUTURE.md`).

Earlier:

- Settled 2026-11-03: the system's name is DIOMND (Conventions).
- TODO(decide): which applications are in the paper.
- TODO(decide): whether data-touching operations (checksum, encryption,
  transforms) are design sections or discussion.
- Settled: the API surface is ordinary application socket descriptors and a
  per-store memory limit; the ring is native `io_uring` opcodes; shared and
  zero-copy fragments from same-host senders are copied at capture
  (`../PROJECT_MAP.md` §3, ledger A1, A4, A17). Settled by the owner on
  2026-11-11: the limit is the size of the store's arrays, which are made at
  registration and do not grow, so payload is still not pre-registered but the
  object metadata is pre-allocated.
- TODO(decide): a capture that fits in the prefix holds nothing.
- TODO(decide): a per-connection in-flight cap. What a full store does is
  settled by the owner on 2026-11-11: the request fails at once with
  `-ENOMEM`, the collector reports it once without ending, nothing waits, and
  collection still holds on undecided bytes.
- Deferred with compaction (2026-11-11): the compaction target; both reviews
  recommended page-cache pages, which also make a file a sink at no cost.
- Table 1, 2026 edition, sits in §2 (2026-10-31). TODO(decide): whether
  the Venn diagram goes in §1 or §2, and whether Table 1 replaces or sits
  beside the property matrix.
- TODO(decide): the mapped mode's proper name in the paper; the text says
  "named" and "mapped" until then (2026-10-31). Since the revision of
  2026-11-13 the draft says "read-write mapped mode" and titles §5 on
  "segregated I/O memory" (the modes' names, above).
- Reviewer notes on the sections above are in
  `../critisizm/review_2026-10-02.md` §8.
