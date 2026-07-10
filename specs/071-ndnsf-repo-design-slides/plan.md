# Implementation Plan: NDNSF-Repo Design Slides

**Branch**: `Experimental` | **Date**: 2026-07-09 | **Spec**: [spec.md](spec.md)

**Input**: Feature specification from `specs/071-ndnsf-repo-design-slides/spec.md`

## Summary

Create a self-contained 16:9 Beamer deck under `docs/NDNSF-REPO/slides/` that explains the implemented NDNSF-DistributedRepo architecture and mechanisms. The deck uses current source, README design notes, and existing MiniNDN regressions as evidence. TikZ diagrams carry the architecture and flow explanations; text remains concise and unmeasured behavior is labelled experimental or future work.

## Technical Context

**Language/Version**: LaTeX2e with pdfLaTeX-compatible source

**Primary Dependencies**: Beamer, TikZ, booktabs, tabularx, listings, graphicx

**Storage**: Source-controlled `.tex` and `.md`; generated PDF beside the source

**Testing**: Two `pdflatex` passes, `pdfinfo`, `pdftotext`, `pdftoppm`, contact-sheet and selected-page visual inspection

**Target Platform**: Linux development environment and ordinary PDF presentation software

**Project Type**: Academic technical presentation artifact

**Performance Goals**: Build in under one minute on the current workstation; render without external network access

**Constraints**: At most 20 frames; 16:9; one primary point per content frame; no unsupported production guarantees; no changes to proposal slides or application code

**Scale/Scope**: One English design deck of approximately 18 frames, including title and summary

## Constitution Check

- **Canonical Dynamic Runtime**: PASS. The deck presents the current generic NDNSF service API and does not revive generated stubs or legacy Direct terminology.
- **Security Is Part Of The Data Path**: PASS. Remote repo calls retain NDNSF permission, NAC-ABE, token, and replay protections; local invocation is labelled trusted same-process only.
- **CodeGraph First, Source Verified**: PASS. CodeGraph identified RepoClient, RepoNode, RepoCore, RepoTypes, and placement paths before README and regression details were read.
- **Spec-Driven Durable Work**: PASS. This feature has a specification, plan, task list, content contract, and reproducible quickstart.
- **Right-Scope Verification**: PASS. The deliverable is documentation, so build and visual checks are required; existing MiniNDN results are cited rather than rerun as though the deck changed runtime behavior.

## Project Structure

### Documentation (this feature)

```text
specs/071-ndnsf-repo-design-slides/
├── spec.md
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/
│   └── deck-content-contract.md
├── checklists/
│   └── requirements.md
└── tasks.md
```

### Slide Artifact

```text
docs/NDNSF-REPO/slides/
├── main.tex
├── main.pdf
└── README.md
```

**Structure Decision**: Keep the presentation self-contained in the requested directory. Reuse the visual language of `docs/NDNSFDI/slides/main.tex`, but redraw all Repo-specific figures as local TikZ so the canonical LaTeX source has no fragile asset dependency.

## Content Architecture

The deck follows a system-design narrative:

1. Problem and responsibility boundary.
2. Implementation layering and deployment roles.
3. Names, manifests, and data references.
4. INSERT and fetch/verification data paths.
5. Capability discovery and replica placement.
6. Catalog synchronization, tombstones, retention, and repair.
7. Security boundary and application integrations.
8. Existing MiniNDN validation, limitations, and design takeaways.

Each frame uses one of four visual forms: boundary stack, component flow, state/control-plane diagram, or concise comparison table. No slide is an API inventory dump.

## Complexity Tracking

No constitution violations or exceptional complexity are required.
