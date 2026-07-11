# Ismail's Glossary
*A Complete Navigation Index for Mathlib4*

[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.21192789.svg)](https://doi.org/10.5281/zenodo.21192789) ![modules](https://img.shields.io/badge/modules-9%2C150-brightgreen)

**[Website](https://m-ismail-za.github.io/IsmailsGlossary/) · [Ismail's Primitives ↗](https://github.com/M-Ismail-ZA/IsmailsPrimitives) · [Paper (DOI)](https://doi.org/10.5281/zenodo.21192789)**

Ismail's Glossary is a complete, human-readable navigation index for every
one of Mathlib4's 9,150 modules — the mathematics library of the Lean 4
proof assistant, and one of the largest libraries of formalized mathematics
in existence. Every existing way of searching Mathlib — a declaration-name
lookup, an in-editor tactic, a semantic search engine — presupposes that you
already know, at least approximately, what you're looking for. None of them
answer the question a newcomer actually asks: *I want to work with spectral
sequences — where do I even begin?* This glossary answers exactly that
question, one module at a time, in plain mathematical English rather than
Lean's internal vocabulary, explicitly marking how each module differs from
its similarly named neighbors.

The full methodology, data design, and versioning rationale are documented
in the accompanying paper (see [Paper](#paper) below).

> **Independence note.** Ismail's Glossary is an independent,
> community-created resource. It is not an official product of the Lean
> Focused Research Organization (Lean FRO) or the Mathlib community, and it
> is not maintained by either — nor has its content been formally reviewed
> by Mathlib's maintainers. The project was originally developed and
> privately documented under the working title "The Mathlib Glossary
> Project"; it has since been renamed and released publicly as an
> open-source resource, to make explicit what was always true of the
> underlying work. The data, descriptions, and infrastructure are unchanged
> by the rename — only the framing is corrected.

## Contents

- [Coverage & Versioning](#coverage--versioning)
- [Repository Structure](#repository-structure)
- [The Glossary (JSON)](#the-glossary-json)
- [RAG-Ready Export](#rag-ready-export)
- [Claude Skill](#claude-skill)
- [Master Spreadsheet](#master-spreadsheet)
- [Interactive Website](#interactive-website)
- [Paper](#paper)
- [Related work](#related-work)
- [Contributing](#contributing)
- [Citation](#citation)
- [Author](#author)

## Coverage & Versioning

| | |
|---|---|
| Modules described | 9,150 (1,129 directories + 8,021 files) |
| Coverage | Complete — every module, every depth, verified against the JSON, RAG JSON, and spreadsheet, which agree exactly |
| Top-level domains | 32 |
| Glossary version | v4.30 |
| Mathlib reference snapshot | `master` @ commit `1ad783f9bf` (2026-05-09) |
| Lean toolchain | 4.29.1 |

Three version numbers run through this project, and they are **not
interchangeable**: the Lean toolchain version, the specific Mathlib commit
this glossary describes, and the glossary's own release counter. In
particular, **glossary v4.30 does not correspond to Mathlib's own `v4.30.0`
tag** — Mathlib's tag tracks Lean 4.30.0, released 2026-05-26, a distinct
and later point than this glossary's reference snapshot. Module paths may
differ in earlier or later Mathlib versions than the one pinned here.

## Repository Structure

```
IsmailsGlossary/
├── README.md
├── Ismails_Glossary_JSON.json      -- primary data (9,150 entries)
├── Ismails_Glossary_RAG.json       -- flat, embedding-ready export
├── ismails-glossary.skill          -- Claude Skill bundle
├── Ismails_Glossary_vMASTER.xlsx   -- static spreadsheet snapshot
├── index.html                      -- interactive website [live]
└── paper/
    ├── PAPER.md                    -- Markdown edition
    └── Ismails_Glossary_Paper.pdf  -- PDF edition
```

## The Glossary (JSON)

`Ismails_Glossary_JSON.json` is the primary deliverable: a single JSON array
of 9,150 entries, one per Mathlib module — every directory and every `.lean`
file. Each entry carries six fields:

| Field | Description |
|---|---|
| `Path` | Full module path from the Mathlib root, e.g. `Mathlib/Algebra/Homology/DerivedCategory` |
| `Name` | File or directory name alone |
| `Type` | `directory` or `file` |
| `Parent Path` | The containing folder |
| `Depth` | Nesting level — 0 = Mathlib root, 1 = top-level domain, 2 = subdomain, and so on |
| `Description` | 1–3 sentences: what the module contains, who uses it, how it differs from adjacent modules |

Nothing about the format is Lean-specific — it's deliberately just a JSON
array, so any retrieval pipeline, vector database, or AI platform can load
it directly: index `Description` for semantic search, and use `Path`,
`Type`, and `Depth` as metadata filters. This file exists so the glossary
has one canonical, machine-readable source of truth that every other
deliverable in this repository (the RAG export, the Skill, the spreadsheet)
is generated from.

## RAG-Ready Export

`Ismails_Glossary_RAG.json` is a flat, pre-processed export for anyone
building a retrieval-augmented-generation pipeline. The primary JSON above
already works for RAG as-is — its entries are short, self-contained, and
consistently structured — but this export removes the last preprocessing
step: each entry merges the path, type, and description into a single text
field ready for direct embedding, plus a metadata block recording the Lean
version, Mathlib snapshot, and glossary version it was generated against.
It exists for the "batteries-included" case: a Mathlib-aware retrieval
system running with minimal setup friction.

## Claude Skill

`ismails-glossary.skill` installs the glossary as an active reference inside
Claude. It bundles three components: a `SKILL.md` that tells Claude when to
activate and how to query the data, the full glossary JSON embedded in the
bundle (so the skill needs no external network access), and a Python search
script that retrieves relevant entries by path, keyword, name, depth, or
type — rather than loading all 9,150 entries into context at once.

The bundled design exists for reliability: an external reference introduces
version drift, network failure, and link rot as live risks; a
self-contained skill keeps working at inference time regardless of
connectivity or later changes to this repository. Installing it is a
one-time upload through Claude's Skills interface — after that, it
activates automatically for Mathlib navigation questions ("where is the
Hahn–Banach theorem?", "what's in `Mathlib/Analysis/SpecialFunctions`?",
"what's the difference between `HomotopyCategory` and `DerivedCategory`?").

## Master Spreadsheet

The master spreadsheet is the authoritative contributor workbook and the
source from which the JSON, RAG JSON, and Skill are all generated. It
exists so the community can propose new descriptions or corrections without
needing to touch the JSON or the repository directly; contributions are
moderated before merging, since an inaccurate entry actively misleads the
next reader, which is worse than leaving the entry blank.

- **Live, editable version:** [Google Sheets — Ismail's Glossary Master](https://docs.google.com/spreadsheets/d/1al-8A-JPK9JGD1zGWSBAixwIvG3wbWh7JFJuhCyhK_o/edit?usp=sharing) — open for anyone with Lean/Mathlib knowledge to comment on or propose changes.
- **Static snapshot:** `Ismails_Glossary_vMASTER.xlsx` — a fixed, offline, citable copy of the spreadsheet as of the reference snapshot, for anyone who needs a stable copy rather than whatever the live sheet currently says.

## Interactive Website

A GitHub Pages site is live at
[m-ismail-za.github.io/IsmailsGlossary](https://m-ismail-za.github.io/IsmailsGlossary/),
with six sections: **Home** (project overview), **Glossary** (a searchable,
hoverable tree of the full Mathlib hierarchy with quick-copy import paths),
**Atlas** (a visual map of all 32 top-level domains, built for orientation
rather than lookup), **Lean 4 References** (a beginner-friendly, 16-section
progressive syntax guide), **Getting Started** (environment setup for
Windows/macOS/Linux), and **About Ismail** (the author's background, and
how this project relates to Ismail's Primitives — see
[Related work](#related-work) below).

## Paper

**Ismail's Glossary: A Complete Navigation Index for Mathlib4**
Muhammed Ismail, Zenodo, V1.0 (2026).
DOI: [10.5281/zenodo.21192789](https://doi.org/10.5281/zenodo.21192789) · CC BY 4.0

The full paper documents the methodology, data design, and versioning
rationale behind the glossary. It's also included in this repository,
independent of Zenodo's continued availability:
[Markdown edition](paper/PAPER.md) · [PDF edition](paper/Ismails_Glossary_Paper.pdf)

## Related work

This glossary is a byproduct of a larger project, not the main event.
Building it meant learning Mathlib well enough to write a ~12,700-line,
zero-`sorry`, zero-`axiom` Lean 4 formalization — published separately as
*Ismail's Primitives*. That project, and the papers extending it, are
below; the fuller story is in [Author](#author).

### Ismail's Primitives

A Lean 4 / Mathlib formalization proving that six structural primitives are
necessary, mutually independent, and sequentially sufficient for adaptive
decision-making under uncertainty — ~12,700 lines, zero `sorry`, zero
`axiom`, zero `opaque`, checked by Lean's kernel.

- Website: [m-ismail-za.github.io/IsmailsPrimitives](https://m-ismail-za.github.io/IsmailsPrimitives/)
- Repository: [github.com/M-Ismail-ZA/IsmailsPrimitives](https://github.com/M-Ismail-ZA/IsmailsPrimitives)
- Paper: *Ismail's Primitives: A Unified Functional Theory of Necessity,
  Independence, and Sequential Dependence in Adaptive Decision Systems*,
  Zenodo, V6.1 (2026).
  DOI: [10.5281/zenodo.21177368](https://doi.org/10.5281/zenodo.21177368)

### Extending Ismail's Primitives

Companion papers apply the same six primitives, under a fixed relabelling,
to other domains:

- **Ismail's Primitives and Human Development: A Functional Isomorphism
  Between a Lean-Verified Computational Theory and Developmental
  Psychology.** Erikson, Maslow, and Bowlby's stage theories, read as three
  independent confirmations of the same six-stage structure.
  DOI: [10.5281/zenodo.21257553](https://doi.org/10.5281/zenodo.21257553) (V3.0)
- **Ismail's Primitives: An Emotional Adaptation Theory for Therapy
  Discovered through Functional Isomorphism.** The same primitives applied
  to psychotherapy and emotional adaptation in the clinical setting.
  DOI: [10.5281/zenodo.21289914](https://doi.org/10.5281/zenodo.21289914) (V2.0)
- **Ismail's Primitives: Lean-verified Economic Adaptation Theory.**
  Extending the primitives to economic coordination under uncertainty.
  DOI: [10.5281/zenodo.21289756](https://doi.org/10.5281/zenodo.21289756) (V2.0)

Further papers extending the theory are in preparation. All four papers
above are CC BY 4.0.

## Contributing

Contributions run through the master spreadsheet linked above. Propose a
new description, or an improvement to an existing one, as a comment or
edit; a moderator reviews it before it's folded into the next glossary
release. This is quality control, not gatekeeping — the precision bar has
to be high, because a description that's imprecise or conflates two
adjacent modules is actively worse than no entry at all.

## Citation

Please cite the Zenodo record for the version you used, so citations stay
tied to an immutable snapshot rather than whatever the live repository
happens to contain when read:

```
Ismail, M. (2026). Ismail's Glossary: A Complete Navigation Index for
Mathlib4 (v4.30) [Data set]. Zenodo. https://doi.org/10.5281/zenodo.21192789
```

```bibtex
@misc{ismail2026glossary,
  author    = {Ismail, Muhammed},
  title     = {Ismail's Glossary: A Complete Navigation Index for Mathlib4},
  year      = {2026},
  publisher = {Zenodo},
  version   = {v4.30},
  doi       = {10.5281/zenodo.21192789},
  url       = {https://doi.org/10.5281/zenodo.21192789}
}
```

## Author

Muhammed Ismail is an independent researcher based in South Africa,
self-taught in mathematics, economics, and enough psychology to notice a
pattern worth proving. This glossary isn't the main project — it's a
byproduct of one.

In September 2025, while trying to understand his own decision-making,
Ismail noticed a six-step pattern recurring no matter the decision's
subject, and set out to prove it held generally. As an unaffiliated
researcher, a result that size has no realistic route through traditional
peer review — every claim would need its own domain specialist, and
credentials aren't something an independent researcher can offer — so the
proof was built in Lean 4 instead: a claim a kernel checks deterministically,
without first needing to check the author. Learning Mathlib well enough to
write that proof — ~12,700 lines, published separately as *Ismail's
Primitives* (see [Related work](#related-work)) — is what produced this
glossary: a byproduct of the learning, released as a resource in its own
right.

The full background is on the website's
[About Ismail](https://m-ismail-za.github.io/IsmailsGlossary/) page.

Email: <literacity@outlook.com> · ORCID: [0009-0000-3713-7105](https://orcid.org/0009-0000-3713-7105) · [LinkedIn](https://www.linkedin.com/in/researcher-adaptationtheory/) · [Zulip](https://leanprover.zulipchat.com/#user/1070455)
