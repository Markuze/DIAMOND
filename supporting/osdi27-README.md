# The OSDI '27 submission, in LaTeX

One file per section under `sections/`, in the order of the section plan in
[`../outline.md`](../outline.md) ("Sections (2026-10-31)"). `main.tex` assembles them;
`macros.tex` holds the shared commands and the draft markers; `refs.bib` the references.

- **Format.** The USENIX template: `usenix2019_v3.sty`, fetched from
  <https://www.usenix.org/sites/default/files/usenix2019_v3.sty> on 2026-10-31,
  with `\documentclass[letterpaper,twocolumn,10pt]{article}` as in the template file
  `usenix2019_v3.2.tex`. The local style adds engine compatibility guards for microtype and
  breakurl. OSDI '27: 12 pages for the body including figures and tables,
  references extra; 10-point type on 12-point leading; double-blind.
- **Build.** `make` (latexmk). `make clean`. `make pages` prints the page on which the body ends
  (the limit is 12) and the total with the references and the draft notes.
- **Length** (2026-11-16). The revised draft, including result placeholders, has a 12-page body
  and 13 pages including references and draft notes. Tectonic checks with draft
  markers shown and with `\draftfalse` both end the body on page 12. The desktop editor
  also compiles the saved source. Type size, leading, margins and figure sizes are unchanged;
  the cuts remove repeated explanations and peripheral comparisons. Use `make pages` to
  check a local build. `\draftfalse` removes the notes and tags for submission.
- **Figures and result slots** (2026-11-16). At the owner's request, receive, send and forwarding
  now have explicit figure placeholders while Alex updates qualified results. Section 8 keeps
  the workloads and comparison rules, with quantitative slots rather than superseded CPU
  percentages. The old compact CSVs remain as historical material and are not plotted. Future
  data generation must use `python3 apps/fwdbase/make_figure_data.py paper/osdi27/figures/mkdata.py`
  from the repository root, as CLAIMS requires; `make data` delegates to that wrapper. The guard
  requires Linux and CPython 3.14. Restore plots only with campaign-specific data, provenance
  and matching CLAIMS updates; provisional fixed-object tables are not qualified results.
- **The evaluation's two forms.** `sections/08-evaluation.tex` is the one `main.tex` builds:
  workloads, evidence boundaries and result placeholders for receive, send, forwarding,
  request/response and capacity. Historical observations remain explicitly qualified.
  The revised-build plan is `../../eval/2026-11-11-fixed-objects/EVAL.md`; its provisional TX/RX
  tables do not supply final-design results. `figures/rrkv.tex` is not included in the build.
  `sections/08-evaluation-long.tex` is the long form replaced on 2026-11-10, kept unbuilt as the
  source of the paragraphs the short form leaves out.
- **Names and argument.** The system is `\sys`, which prints DIOMND. Its two peer modes are
  *named* (read-only operations on unmapped objects) and *mapped* (read-write I/O memory
  segregated from private kernel control state and parsing copies). The opening separates
  protection of kernel state from authority over other flows, then derives both modes and
  their independence from specialized NIC features. The four contributions are that argument,
  named mode, mapped mode, and implementation/evidence. Shared object lifetimes are
  described in Section 3. The authors' earlier work
  is cited in the third person; branch names, tool names and host names stay out of the text.
- **IRQ and block-I/O audit** (2026-11-16). Sections 3--6 distinguish the design's per-CPU
  task/softirq magazine pairs, their bounded extra stock and the mapped receive pool's different
  cache discipline. The new allocator passed source review with caveats; follow-up fixes and
  integration remain, and it is not installed or measured. Fixed-pool
  implementation and provisional measurements are a separate version. Block I/O appears only
  as a proposed storage-backed-object use case in Discussion. Its alignment, tail padding,
  device support and lifetime contracts remain unresolved; the audit at the top of
  [`../../docs/bio_design_spec.md`](../../docs/bio_design_spec.md) records why existing storage
  paths and network-only results do not establish its benefit.
- **Numbers.** Every result comes from [`../../CLAIMS.md`](../../CLAIMS.md), with its complete
  qualification and bounds. Pinned placement alone does not establish current qualification;
  preserve historical and unmatched labels. Do not pool CPU estimators, builds or campaigns.
  Argued claims say "by design"; pending and unmeasured results remain placeholders.
- **QUIC recommendation.** [`../quic-extension/main.tex`](../quic-extension/main.tex) is a
  standalone design and evaluation note with its own bibliography. It proposes named UDP
  datagram lending and a QUIC pass-through relay first, with terminating QUIC applications
  as later work. It reports no QUIC implementation or measurements and is not part of the
  main paper build.
- **Markers.** `\todom{}` measure, `\todod{}` decide, `\todov{}` verify, `\todo{}` other;
  `\draftfalse` in `macros.tex` hides them. Since 2026-11-13 a marker prints a numbered tag
  where it stands (`[M7]`, `[D3]`, `[V2]`) and its text goes to the list "Draft notes" that
  `\printdraftnotes` sets after the references, so the notes take no room in the twelve pages.
  The forms `\todomi{}`, `\tododi{}`, `\todovi{}` and `\todoi{}` print the note in place: use
  them in figures and captions. `\figplaceholder{height}{text}` stands where a figure will go.
