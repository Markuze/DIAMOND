# Errata

Corrections to the paper text, each on the owner's word.

- **Text corrected:** `sections/*.tex` at commit `0258fe0` (2026-11-28). Every `file:line` below is in that commit.
- **Written:** 2026-11-28. Nothing in `sections/` has been edited to apply these.
- **Order:** by weight. Erratum 1 changes what the paper may claim. The others change wording.

| # | Correction | Kind |
| --- | --- | --- |
| 1 | DIAMOND does not copy headers | Wrong in the text: remove |
| 2 | Say `CAP_NET_ADMIN`, not "trusted" | Underspecified: name it |
| 3 | The results sentence, and "earlier prototype" numbers | Wrong in the text: replace |
| 4 | Hardware independence is a statement of requirements | Wording rule |
| 5 | XDP is not a concern | Do not add |
| 6 | Block I/O is being measured | Pending results |
| – | The lifetime argument | Not an error: keep |

## 1. DIAMOND does not copy headers

**The owner, 2026-11-28:** "what header copies? we don't copy headers", and then "no header copies its useless".

**Correction.** Remove every statement that the driver or the kernel copies packet headers into private memory. A header copy is not a mechanism of the design, not the reason the mapped mode is safe, and not a cost of it.

### Where the text says it

| Location | Text | What to do |
| --- | --- | --- |
| `01-introduction.tex:52` | "the kernel parses private copies of packet headers" | Remove the clause. The sentence after it, on segregation working in software, stays and attaches to the dedicated area |
| `03-lending.tex:34` | Figure box: "control structures and header copies" | "control structures" |
| `03-lending.tex:40–41` | Figure arrow from the read/write area to private memory, "network headers for parsing" | Remove the arrow |
| `03-lending.tex:45` | Caption: "On network input, the kernel copies headers into private memory before parsing them." | Remove the sentence |
| `05-mapped.tex:21–28` | The paragraph "Network input poses a further problem": the driver copies an initial prefix, the stack brings further header bytes into private storage, "later writes cannot change the kernel's copy" | Remove those sentences. See the note on line 24–25 below |
| `06-implementation.tex:19–20` | "The driver and stack together keep the headers used for parsing in private memory" | Remove |
| `08-evaluation.tex:198–200` | "An independent protection test changes header bytes in the mapping after the kernel takes its private copy and checks which bytes the stack uses." | Remove |
| `08-evaluation.tex:203` | Note: "Add the private-header and buffer-reclamation checks" | Drop "private-header and" |
| `09-related.tex:20–23` | "the kernel keeps the header copies it parses in private memory. That choice extends writable receive to devices without hardware header splitting. It incurs the work of copying headers" | Remove the copies and the cost. "the mapper is trusted with the whole area" stays |

**Not covered by this erratum.** `05-mapped.tex:24–25` says the driver snapshots device control fields, including how many buffers hold a packet. That is device metadata, not a packet header. It stays unless the owner says otherwise. The same holds for "private control state" in `10-discussion.tex:16` and "Keeping control state private" in `10-discussion.tex:38`: these are about the kernel's own structures.

### What the text says in its place

The owner's own words for why the mapped mode is safe:

- "carefully segmented kernel memory may be safely dedicated for user applications" (the week of 2026-11-24);
- "kernel memory segmentation which allows a net_admin user to access kernel memory safely" (2026-11-14).

So the text rests the mapped mode on two things it already states:

1. **Segmentation.** The area is dedicated I/O memory, and the kernel's control structures live outside it (`05-mapped.tex:14–19`).
2. **Privilege.** Mapping the area requires `CAP_NET_ADMIN` (`05-mapped.tex:41–46`, and erratum 2).

### What not to write instead

- **Not the opposite as a result.** Do not write "DIAMOND parses headers in place" or "no header byte is copied" as a property of the evaluated system. Removing a sentence is safe. Asserting the absence is a claim about the kernel that was measured, and the record below does not yet agree with it.
- **Not a kernel-safety claim for the mapper.** The notes answered "can a mapper harm the kernel?" with "no, the kernel parses its own copies of the headers". With the copy gone, that answer has no stated basis. Until the owner gives the sentence that replaces it, the text says what is segmented and who may map, and does not say that a mapper cannot harm the kernel.

### Where the error came from

The paper took it from the project record in the `zc_proxy` repository, which still describes a private header copy:

- a commit in the kernel series, `5038105`, "virtio_net: copy headers into a private skb head for provider queues" (listed in `maio_handover.md`);
- `maio_handover.md` §2.4, "Private-header requirement and the driver limit";
- `docs/design_spec.md`, the virtio-net paragraph and the flag `PP_FLAG_PRIVATE_HEADERS`;
- `docs/glossary.md`, the entry "Private headers".

For the paper, the owner's statement overrides these. This file does not correct them: the owner reconciles the code and the specification. An agent that reads the specification must not put the header copy back into the paper.

### The same statement in `supporting/`

These notes are copies of older planning documents and carry the same statement. They are superseded on this point and are not edited here.

| File | Lines |
| --- | --- |
| `supporting/novelty.md` | 98, 150, 196–197, 357–360 |
| `supporting/outline.md` | 134, 162, 180, 290, 294, 297, 610, 647, 817, 1290 |

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

## Not an error: keep the lifetime argument

**The owner, 2026-11-28:** "the lifetime clarification is paramount". It stops a reader dismissing mapped objects as the same as `RECV_ZC`, with "who cares where the memory originated, everyone has access".

No shortening pass cuts these:

| Location | Text |
| --- | --- |
| `00-abstract.tex:5` | "a lifetime independent of any single process" and the queued-packet example |
| `01-introduction.tex:23–31` | "A packet can outlive the process that supplied its buffer." |
| `03-lending.tex:49–78` | "One object, three lifetimes" |
| `05-mapped.tex:64–87` | "Returning buffers to the device" |
