# The OSDI '27 submission, in LaTeX

One file per section under `sections/`. `main.tex` assembles the current manuscript;
`macros.tex` holds the shared commands and the draft markers; `refs.bib` the references.

- **Format.** The USENIX template: `usenix2019_v3.sty`, fetched from
  <https://www.usenix.org/sites/default/files/usenix2019_v3.sty> on 2026-10-31,
  with `\documentclass[letterpaper,twocolumn,10pt]{article}` as in the template file
  `usenix2019_v3.2.tex`. The local style adds engine compatibility guards for microtype and
  breakurl. OSDI '27: 12 pages for the body including figures and tables,
  references extra; 10-point type on 12-point leading; double-blind.
- **Build.** `make` (latexmk). `make clean`. `make pages` prints the page on which the body ends
  (the limit is 12) and the total with the references and the draft notes.
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
- **Design and examples.** DIOMND has two access modes for retained kernel I/O objects.
  Named mode lends checked access by handle. Mapped mode also lends writable access to
  an eligible I/O area, with control state and network parsing copies kept private.
  Sections 3–5 describe the shared lifetimes, IRQ-aware recycling and access rules.
  Section 7 uses TCP streams, UDP datagrams and block I/O to illustrate both modes.
  Section 6 describes the combinations exercised by the Linux prototypes.
- **Endpoint sources.** The TCP design follows
  [the design specification](../../docs/design_spec.md). The datagram example draws on
  [the UDP relay notes](../../apps/fwdbase/README-UDP.md) and the vendored UDP interface.
  The block example draws on [blockbench](../../apps/blockbench/README.md) and the later
  dense-capture branch. [The block proposal and audit](../../docs/bio_design_spec.md)
  record the original design questions. The prototype uses a bounded raw-device
  endpoint and a more restricted write interface than the proposal.
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
