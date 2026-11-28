# Errata

Corrections to the paper text, each on the owner's word.

- **Text corrected:** `sections/*.tex` at commit `0258fe0` (2026-11-28). Every `file:line` below is in that commit.
- **Written:** 2026-11-28. Amended the same day with errata 7–12 and the telling of the lifetime argument, and then with erratum 1 as it now stands. Nothing in `sections/` has been edited to apply these.
- **Erratum 1 changed twice** on 2026-11-28. Its first version said to remove every header-copy statement. The decision is now deferred: leave those statements as they are.
- **Order:** erratum 1 is deferred. Errata 2–6 change wording. Errata 7–12 are the story decisions of 2026-11-28, in the order the paper meets them.
- **Whose words:** every correction is the owner's. Wording marked "mine" is a proposal by the agent that wrote this file, not a decision.

| # | Correction | Kind |
| --- | --- | --- |
| 1 | The header copy is an addendum | **Deferred:** leave the text as it is |
| 2 | Say `CAP_NET_ADMIN`, not "trusted" | Underspecified: name it |
| 3 | The results sentence, and "earlier prototype" numbers | Wrong in the text: replace |
| 4 | Hardware independence is a statement of requirements | Wording rule |
| 5 | XDP is not a concern | Do not add |
| 6 | Block I/O is being measured | Pending results |
| 7 | The claim is "faster with fewer prerequisites" | Wrong emphasis: replace |
| 8 | What "zero copy" means in each mode | Missing: define |
| 9 | The opening example is a key-value store | Replace the example |
| 10 | Mapped mode is complete zero copy, for the privilege `RECV_ZC` already requires | Undersold: restate |
| 11 | Named mode: backpressure and reusable buffers | Missing where it is compared: add |
| 12 | Cite DAMN; MAIO is an honorable mention | Citations |
| – | The lifetime argument | Not an error: keep. A better telling is proposed |

## 1. The header copy is an addendum: decision deferred

**Changed twice on 2026-11-28.** The first version of this erratum, pushed that day, said to remove every statement about a header copy. **Do not remove them, and do not build on them.** The owner has deferred the decision.

**The owner, 2026-11-28, in order:**

1. "what header copies? we don't copy headers", then "no header copies its useless", then "we dgaf about the driver and what it might or might not copy".
2. "cap admin already allows modifying dropping and rerouting packets this doesnt expose much more than what a regular net_cap admin already can do with xdp ebpfs for example".
3. "we can say toctou is a new problem, but this we can solve with header copy".
4. Last: "the header copy is an addendum, its not critical we can decide later how we handle it and wether toctou is the new problem".

### What holds now

**The mapped mode does not rest on the header copy.** It rests on two things the text already states:

1. **Segmentation.** The area is dedicated I/O memory, and the kernel's control structures live outside it (`05-mapped.tex:14–19`). The owner's words: "carefully segmented kernel memory may be safely dedicated for user applications" (the week of 2026-11-24); "kernel memory segmentation which allows a net_admin user to access kernel memory safely" (2026-11-14).
2. **Privilege.** Mapping the area requires `CAP_NET_ADMIN` (`05-mapped.tex:41–46`, and erratum 2). That privilege already covers the packets (statement 2 above): a process that holds it can already rewrite, drop and redirect every packet on the device, with tc or netfilter rules, or with an XDP program (which also needs `CAP_BPF`). The text says this once, where it names the capability.

**The header-copy statements stay as they are until the owner decides.** Nobody removes them, expands them, or moves them forward in the argument.

### What is open

1. How the paper handles the header copy: as a mechanism it describes, as a detail of the implementation, or not at all.
2. Whether the paper presents time of check to time of use as the one new problem that mapping adds, with the header copy as its answer (statement 3 above).

### Where the text has the header copy

Leave each as it is.

| Location | Text |
| --- | --- |
| `01-introduction.tex:51–54` | "the kernel parses private copies of packet headers. This segregation works in software, including on devices without header splitting or per-application flow steering." |
| `03-lending.tex:34`, `:40–41`, `:45` | Figure box "control structures and header copies", the arrow "network headers for parsing", and the caption sentence |
| `05-mapped.tex:21–28` | "Network input poses a further problem…": the driver copies a prefix, the stack brings further header bytes into private storage, "later writes cannot change the kernel's copy" |
| `06-implementation.tex:19–20` | "The driver and stack together keep the headers used for parsing in private memory" |
| `08-evaluation.tex:198–200`, `:203` | The protection test that changes header bytes in the mapping after the kernel takes its copy |
| `09-related.tex:20–23` | "the kernel keeps the header copies it parses in private memory. That choice extends writable receive to devices without hardware header splitting. It incurs the work of copying headers" |

### Background: why `RECV_ZC` requires header split

The owner asked on 2026-11-28 why `RECV_ZC` needs header split when its user already holds `CAP_NET_ADMIN`.

- **`RECV_ZC`.** It receives payload into the application's memory, and the stack does not read that memory: zcrx marks it unreadable "by policy, not necessity" (`supporting/novelty.md:327`). So headers have to land in ordinary kernel memory to be parsed at all. The kernel documentation gives this placement as the reason: headers are "received into kernel memory as normal", and payload goes directly into userspace memory (`docs.kernel.org/networking/iou-zcrx.html`, read 2026-11-28). Steering and RSS keep every other flow off that queue.
- **Mapped mode.** Its area is kernel memory, and the stack reads it like any other receive buffer, so the device does not have to place anything elsewhere.

**Still needed.** The statement in erratum 10, that `RECV_ZC` requires `CAP_NET_ADMIN`, wants a citation to the kernel source. The documentation page names no capability.

### The project record

The record in the `zc_proxy` repository describes a header copy. Whether the paper relies on it is part of the open decision.

- a commit in the kernel series, `5038105`, "virtio_net: copy headers into a private skb head for provider queues" (listed in `maio_handover.md`);
- `maio_handover.md` §2.4, "Private-header requirement and the driver limit";
- `docs/design_spec.md`, the virtio-net paragraph and the flag `PP_FLAG_PRIVATE_HEADERS`;
- `docs/glossary.md`, the entry "Private headers".

The notes in `supporting/` describe the copy as well: `novelty.md` (98, 150, 196–197, 331–346, 357–360) and `outline.md` (134, 162, 180, 290, 294, 297, 610, 647, 817, 1290). They are neither confirmed nor superseded on this point until the decision is made.

## 2. Say `CAP_NET_ADMIN`, not "trusted"

**The owner, the week of 2026-11-24:** a bare "trusted application" is a red flag to a reviewer. "We should say NET_ADMIN", to show that no trust barrier was dropped or simplified, and that the mapped mode uses "a legitimate trust mode" that "existed before".

**Correction.** Where the abstract and the introduction first say who may map, name the capability and give its reason: the mapped area carries every flow on the device. Say once that neither mode lowers a trust barrier the kernel has today. After that first naming, "trusted" may stand.

| Location | Text |
| --- | --- |
| `00-abstract.tex:6` | "mapped mode gives trusted applications direct read/write access" |
| `01-introduction.tex:29` | "A trusted process may map the pool" |
| `01-introduction.tex:49` | "A trusted application can also read and edit data in place" |
| `03-lending.tex:27`, `:44` | Figure box and caption: "trusted application", "A trusted process" |
| `05-mapped.tex:5` | "maps it into a trusted process" |
| `10-discussion.tex:37` | Conclusion: "a trusted service" |

Only `05-mapped.tex:41–42` names the capability today.

## 3. The results sentence, and "earlier prototype" numbers

**The owner, the week of 2026-11-24, confirmed 2026-11-28 ("its the correct framing"):** the results are "on par for small objects" and "a clear lead over copied or dynamically mapped payloads on RX and TX".

### The abstract claims more than that

| Location | Text | Why it is wrong |
| --- | --- | --- |
| `00-abstract.tex:8` | "consistently reducing CPU time per delivered byte compared to alternatives" | It claims a win at every size and on block I/O. `08-evaluation.tex:68–70` reports named sends costing more at 1 and 4 KiB against a registered buffer, and the note at `01-introduction.tex:77–79` already asks for this claim to be checked |

**Correction.** The abstract's last sentence states the owner's two clauses: parity for small objects, a lead over copied and dynamically mapped payloads on receive and send. "Dynamically mapped" is the owner's phrase. The text has so far read it as pages pinned on each operation.

### Numbers from an earlier prototype

The paper does not set an earlier prototype against a revised design (the owner, 2026-11-14; the convention is recorded at `supporting/outline.md:67–72`). Every number in the paper comes from a row of `CLAIMS.md` in `zc_proxy`, measured on the final build.

| Location | Text |
| --- | --- |
| `01-introduction.tex:72–76` | "On an earlier TCP prototype, named sends used 15--27\% less…" and the 15% relay figure |
| `06-implementation.tex:48–50` | "Early fixed-pool builds…", "The earlier TCP comparisons in §8" |
| `08-evaluation.tex:2` | Comment: "Earlier TCP observations below…" |
| `08-evaluation.tex:13–14` | "The measurements quoted below come from earlier TCP prototypes." |
| `08-evaluation.tex:20` | "The earlier network testbed…" |
| `08-evaluation.tex:65–70` | "On an earlier prototype, full-speed named sends…" |
| `08-evaluation.tex:77–79` | "On an earlier build, the mapped-object relay…" |

**Correction.** Remove these numbers and the words "earlier prototype", "earlier build" and "early builds". The marked result slots take the final-build rows when they exist.

## 4. Hardware independence is a statement of requirements

**The owner, 2026-11-28:** "recv_zc needs specialized HW the subset is much more narrow than ours."

**Rule.** Compare what each design requires of the device. Never compare how many drivers each supports today: the mapped mode runs on one.

- `RECV_ZC` requires a NIC that splits headers and steers flows in hardware.
- Named mode requires nothing of the device or its driver.
- Mapped mode requires a driver that draws its receive pages from the area, and no hardware feature.

| Location | Text | What to do |
| --- | --- | --- |
| `00-abstract.tex:6` | "Both modes operate without specialized hardware support." | A bare sentence. Tie it to the reason, or state the requirement comparison |
| `10-discussion.tex:13–19` | "How far does hardware independence extend?" | Keep the content. State the driver hook in one line, as a requirement, not as a limit of the claim |

The measured anchor: registering a zero-copy receive queue on virtio-net is refused. It is recorded at `08-evaluation-long.tex:105`, a file the build does not include. The built section states the requirement without the measurement (`08-evaluation.tex:104`).

Proposed wording, not the owner's: "`RECV_ZC` runs only on NICs that split headers and steer flows in hardware. DIAMOND asks nothing of the device: named mode runs on any NIC unchanged, and mapped mode on any NIC whose driver draws its receive pages from the area."

## 5. XDP is not a concern

**The owner, 2026-11-28:** "we dont care about XDP its an ebpf raw packet inspector".

The sections do not mention XDP today. Do not add it as a limitation, an exclusion to discuss, or a threat to the mapped mode.

## 6. Block I/O is being measured

**The owner, 2026-11-28:** "we are running experiment for block io as well".

Nothing to change yet. When the results are in `CLAIMS.md`:

- the conditional sentences become statements of what was built and measured (`05-mapped.tex:17`, `05-mapped.tex:45`, `07-applications.tex:63`);
- the abstract's performance sentence may name block I/O. Until then it names the network only.

The owner's description of the block paths, for whoever rewrites that text (the week of 2026-11-24):

- **Read.** A block read goes into kernel memory instead of user pages: like `sendfile`, without the VFS overhead.
- **Write.** A write to a block device needs compaction. After it the write is simple, and needs no `O_DIRECT`, which still pins the user's pages.
- **Mapped.** A similar path, with memory from the pool that the `CAP_NET_ADMIN` process has access to.

## 7. The claim is "faster with fewer prerequisites"

**The owner, 2026-11-28:** "the story is faster with fewer prerequisites".

**Correction.** The abstract and the introduction state this as the paper's claim. Today they state two other things: that a unified mechanism is missing, and that the contribution is a common interface.

| Location | Text | What to do |
| --- | --- | --- |
| `00-abstract.tex:1` | "What is missing is a unified mechanism for reusing kernel-resident I/O data across I/O paths and application trust levels." | Replace. The problem is what zero copy asks for today: on send, pinned pages that stay unchanged until a notification; on receive, a device that splits headers and steers flows. The same paragraph already names both, as "buffer-management obligations" and "specialized hardware" |
| `01-introduction.tex:58` | "The contribution is a common object interface for these two levels of trust." | Replace with the claim: zero copy that is faster and asks less of the application and of the device. The common interface is how, not what |
| `08-evaluation.tex:6–7` | "The central question is how much CPU a service spends delivering a byte that is already in memory." | Keep, and add the second question: what each path requires of the application and of the device. Today that is one late paragraph, "Device support" (`08-evaluation.tex:100–105`) |

The title already says it and stays.

## 8. What "zero copy" means in each mode

**The owner, 2026-11-28:** "mapped is always ZC but needs driver support". Named mode "is ZC in the sence that it doesnt force you copy anything you dont need" and "asks nothing from you totally HW agnostic". (The owner said "opaque" for named mode.)

**Correction.** Define zero copy once for each mode, where the mode is introduced. A fallback copy inside the kernel is described after the rule, as the exception.

- **Named.** The application is never made to copy bytes it does not need: it copies out only what it chooses to read. The mode asks nothing of the device or its driver.
- **Mapped.** Always zero copy. It needs driver support.

| Location | Text | What to do |
| --- | --- | --- |
| `01-introduction.tex:42–47` | The named-mode paragraph. It lists the operations and never says in what sense the mode is zero copy | Add the definition |
| `04-named.tex:50–52` | "Where possible, network capture retains the pages that already hold the payload. It copies readable fragments into store-owned buffers when sharing the original pages would be unsafe" | The section's first statement about copying is the fallback. State the rule first |
| `05-mapped.tex:53–54` | "It retains eligible pages by reference and copies other readable input into area pages." | The rule and the exception carry equal weight. State the rule first |
| `00-abstract.tex:6` | "Both modes operate without specialized hardware support." | Say what each mode asks: named nothing, mapped driver support and no hardware feature (with erratum 4) |

## 9. The opening example is a key-value store

**The owner, 2026-11-28:** "The opening example should be KVS."

**Correction.** The introduction opens on an in-memory key-value store: a client sets a value, and many clients read it. The storage server and the relay are no longer the opening.

| Location | Text | What to do |
| --- | --- | --- |
| `01-introduction.tex:4–11` | "Consider a storage server that reads a block once and sends it to several clients…", then the relay | Replace with the key-value store |
| `02-borrowing.tex:6–7` | "Two kinds of reuse arise in the storage server from §1." | Follows the new example |
| `03-lending.tex:7–9`, `:53` | "a server can read a range from a block device, send it to two clients…"; "Consider the server's last send." | Follow the new example |
| `07-applications.tex:31–36` | The key-value server: one paragraph inside the TCP subsection | Becomes the section's lead example |
| `08-evaluation.tex:185` | The key-value server: last in the evaluation | Its result answers the opening |

Draft opening, mine, for discussion:

> Consider an in-memory key-value store whose values are large. A client sets a value, and many clients then read it, some of them slowly. The store needs a few bytes of each request to find the key. It needs none of the value's bytes: it only has to keep them and send them again. Today it either copies each value in from the socket and out again on every read, or it accepts the conditions of zero copy. Its memory is pinned, and it must leave a value untouched for as long as any reader is still being sent it. When a value is replaced while slow readers are being served, the store itself must work out when the old memory is free.

**Related work for this example.** PASTE's key-value store keeps the received packet buffer as the value and sends it again without a copy (`RELATED_WORK.md` in `zc_proxy`). The existing citations stay (`01-introduction.tex:59–60`, `02-borrowing.tex:66`, `09-related.tex:7–8`). The owner regards the differences from it as "minor details" (2026-11-28): it gets no more room than it has.

## 10. Mapped mode is complete zero copy, for the privilege `RECV_ZC` already requires

**The owner, 2026-11-28:** "its a true e2e zc its the zc panacea it has two limitations that are lower than any other no special HW first in breed and it needs net_cap_admin that RECV_ZC already claims".

**Correction.** Mapped mode is introduced as zero copy from receive to send. Its two requirements are stated as lower than those of any existing interface:

1. driver support, and no special hardware;
2. `CAP_NET_ADMIN`, which `RECV_ZC` requires as well.

The text introduces the mode by a narrower need, and illustrates it with an edit to one field.

| Location | Text | What to do |
| --- | --- | --- |
| `01-introduction.tex:49` | "A trusted application can also read and edit data in place." | Lead with the case: the application receives, works on and sends payload where it lies |
| `05-mapped.tex:4` | "A service that transforms payload in place needs write access." | The same |
| `05-mapped.tex:57–58` | "a relay can edit a fixed-width field at a returned offset and then send the object" | May stay as an illustration. It is not the motivation |

**Source needed.** That `RECV_ZC` registration requires `CAP_NET_ADMIN` is not stated on the kernel's documentation page for it (read 2026-11-28). Table 1 says "administrative registration" (`02-borrowing.tex:34`). Cite the kernel source for the capability check where the text leans on it.

Proposed wording for the abstract, mine:

> A *mapped* object lies in a dedicated I/O area that a process with `CAP_NET_ADMIN` maps. The process receives, works on and sends payload where it lies, zero copy from end to end. Zero-copy receive already requires that privilege today; what it no longer requires is special hardware.

## 11. Named mode: backpressure and reusable buffers

**The owner, 2026-11-28:** "Named mode actually handles backpressure and the buffers are reusable".

**Correction.** Where named mode is set against the kernel's existing paths, name both properties together.

| Location | Text | What to do |
| --- | --- | --- |
| `09-related.tex:25–33` | "What survives an I/O request?" compares named mode with `sendfile`, `splice`, `tee`, a registered pool and ublk. It says only that an object outlives its request | Add the pair: an object is sent many times, and each send keeps its own place under backpressure |
| `03-lending.tex:64–65`, `07-applications.tex:21`, `:24` | Where the text has the two properties today, apart from each other | Keep |

A note on the comparison, mine. Backpressure alone does not separate named mode from `splice`: the text says all the relays "preserve byte order and backpressure" (`08-evaluation.tex:75`). The pair does, for data that arrived from the network:

- `splice` keeps its data in a bounded pipe with no named ranges (`02-borrowing.tex:44–47`);
- sockmap has no retained range (`02-borrowing.tex:56–59`) and, in the record, no backpressure (`CLAIMS.md` F2 in `zc_proxy`);
- `sendfile` has both, once the data is in a file.

Do not write that a pipe cannot retain or duplicate data: `tee` duplicates it.

## 12. Cite DAMN; MAIO is an honorable mention

**The owner, 2026-11-28:**

- "the mapped work should cited as a successor of DAMN that used memory segmentation for device-kernel security";
- on MAIO: "maio is the actual prior work but I also used HW support just like RECV_ZC and its not peer reviewed so it deserves just an honor mention";
- on weight: "I wouldn't press on damn too much I would quote it but Im not confident it needs a full paragraph".

**Correction.** DAMN is cited once, in a clause, where the dedicated area is introduced: the mapped mode continues DAMN's segmentation of I/O memory, which protected the kernel from a device. MAIO keeps one mention beside `RECV_ZC` and loses its other three places.

| Location | Text | What to do |
| --- | --- | --- |
| `refs.bib` | No entry for DAMN | Add it: Markuze, Smolyar, Morrison and Tsafrir, "DAMN: Overhead-Free IOMMU Protection for Networking", ASPLOS 2018. This entry is written from memory: the owner confirms it |
| `05-mapped.tex:14–19` | The paragraph on what the area contains and what stays private | Add the clause that cites DAMN |
| `01-introduction.tex:21` | "dedicated queues~\cite{zcrx,maio}" | Stays. This is the mention |
| `01-introduction.tex:59–61` | "MAIO separates writable I/O memory from kernel state" | Remove MAIO from the sentence. It credits the mapped mode's own idea to a citation |
| `02-borrowing.tex:70–73` | The MAIO row under "Research designs" in Table 1 | Remove the row |
| `09-related.tex:15–18` | "MAIO preserves kernel TCP and segregates writable I/O memory…" | Reduce to a clause, or remove. ZeroNIC stays |

Proposed clause, mine: "…kept apart from kernel state, the segregation DAMN introduced between a device and the kernel."

## Not an error: keep the lifetime argument

**The owner, 2026-11-28:** "the lifetime clarification is paramount". It stops a reader dismissing mapped objects as the same as `RECV_ZC`, with "who cares where the memory originated, everyone has access".

No shortening pass cuts these:

| Location | Text |
| --- | --- |
| `00-abstract.tex:5` | "a lifetime independent of any single process" and the queued-packet example |
| `01-introduction.tex:23–31` | "A packet can outlive the process that supplied its buffer." |
| `03-lending.tex:49–78` | "One object, three lifetimes" |
| `05-mapped.tex:64–87` | "Returning buffers to the device" |

### How to tell it

The owner asked on 2026-11-28 how the argument could be told better. The proposal below is mine. The owner's reply to it was the question about `RECV_ZC` (answered in erratum 1), and then "otherwise ammend errate.md". The draft paragraph was revised afterwards to give header split and steering the same cause.

The dismissal to block says that access is the same as with `RECV_ZC`, so the origin of the memory does not matter. The answer: origin decides who is responsible for a buffer, and who else can use it. That has three consequences, and the text leads with the rarest.

| Order | Consequence | Where the text has it today |
| --- | --- | --- |
| 1. Forward | With `RECV_ZC` and `SEND_ZC` the application holds each receive buffer until the send side reports, then returns it. With a mapped object it sends and forgets, and the kernel reclaims | `05-mapped.tex:82–85`, as a closing detail |
| 2. Share | The stack sorts packets after they arrive, so other sockets are served from the same area and no steering is needed | `05-mapped.tex:37–38`, as a caveat |
| 3. Exit | The mapper can exit, and nothing in flight is lost | `00-abstract.tex:5` and `01-introduction.tex:23–31`: the lead today |

Draft paragraph, mine, in place of `01-introduction.tex:23–31`. It also carries the case for mapped mode (erratum 10):

> Mapped mode is zero copy from receive to send. The device receives into a dedicated area of kernel memory. A process with `CAP_NET_ADMIN` maps the area, works on the payload where it lies, and sends it from there. io_uring's zero-copy receive asks for the same privilege, and also for a device that splits headers and steers flows. The difference is whose memory it is. There the buffers are the application's. The stack does not read them, so headers must land elsewhere and only the application's own flows may land in them. The application must hand each buffer back, and to forward one it must hold it until the send side reports. Here the buffers are the kernel's. Everything the device receives lands in them and the stack reads and sorts it as it always has, so sockets that know nothing of DIAMOND are served from the same area. The application sends an object and forgets it, and the kernel returns each page to the device when its last user is done, retransmissions included. The process that mapped the area can exit, and nothing in flight is lost.
