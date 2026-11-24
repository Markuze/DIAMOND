# DIOMND: layout of the Overleaf project

The paper's LaTeX sources and the notes that go with them, and nothing else.
Copied unchanged on 2026-11-24 from the `zc_proxy` repository at commit
`a98de85`.

## Layout

| Path | What it holds |
|---|---|
| `main.tex` | The paper. The file to compile, with pdfLaTeX. |
| `macros.tex` | Shared commands, the system's name (`\sys`) and the draft markers. |
| `refs.bib` | The references. |
| `usenix2019_v3.sty` | The USENIX template. |
| `sections/` | One file per section, `00-abstract.tex` to `10-discussion.tex`. `08-evaluation-long.tex` is not built. |
| `figures/` | The figures, as LaTeX. `rx.tex`, `tx.tex` and `fwd.tex` are placeholders, `style.tex` holds the plot styles, and `rrkv.tex` is not built. |
| `eval/` | Empty. Graphs for the evaluation go here. |
| `supporting/` | The Markdown notes below. Not part of the build. |

## The notes in `supporting/`

| File | Copied from | What it is |
|---|---|---|
| `outline.md` | `paper/outline.md` | The paper's outline: conventions, the story, titles, the section plan, figures and tables, open decisions. Its first table gives the status of the other notes. |
| `novelty.md` | `paper/novelty.md` | The claim, the insight and the contributions. |
| `intro.md` | `paper/intro.md` | Draft of the introduction in Markdown. |
| `problem.md` | `paper/problem.md` | Draft of the problem statement in Markdown. |
| `prior-work.md` | `paper/prior-work.md` | Draft of the related work in Markdown. |
| `osdi27-README.md` | `paper/osdi27/README.md` | Notes on the LaTeX sources: format, length, figures, names and markers. |

## Paths inside the copies

The copies are unchanged, so the paths they mention are the repository's.

- `osdi27/` is this project's root.
- `../outline.md`, in the comments of `main.tex` and `macros.tex`, is
  `supporting/outline.md`.
- `CLAIMS.md`, `RELATED_WORK.md`, `eval/...` and `docs/...` are in the
  repository and are not copied here. Every number in the paper comes from
  the repository's `CLAIMS.md`.
- `make`, `make pages` and `make data` belong to the repository's Makefile.

## Not copied

- `Makefile`, `figures/mkdata.py` and `figures/data/*.csv` from
  `paper/osdi27/`. They are not LaTeX, and no figure reads the CSVs.
- `paper/quic-extension/`, a separate note with its own `main.tex`.
- The reference PDFs under `paper/`.
