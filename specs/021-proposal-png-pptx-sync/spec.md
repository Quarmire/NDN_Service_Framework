# Feature 021: Proposal PNG PPTX Sync

Status: Superseded

## Goal

Regenerate the Google-Slides-friendly PNG-image PPTX from the current Beamer
proposal-defense slides and speaker notes.

## Scope

- Reuse the PDF-matched PPTX generator.
- Keep Beamer `main.pdf` as the canonical visual source.
- Keep `speaker_notes.tex` as the speaker-note source.
- Verify PPTX slide count, image resolution, and embedded notes.

## Non-Goals

- Editable PPT layout recreation.
- New slide content.
- Changing speaker-note prose.

## Acceptance

- [x] `main.pdf` is current and has the expected page count.
- [x] `NDNSF_proposal_google_slides.pptx` has one slide per PDF page.
- [x] Embedded slide images are at least 2560x1440.
- [x] Speaker notes are embedded for every slide.
- [x] Notes count equals slide count.

## Accepted Evidence

Generated file:

```text
docs/PAPER/proposal-defense/slides/NDNSF_proposal_google_slides.pptx
```

Validation:

```text
main.pdf pages: 57
speaker_notes.pdf pages: 14
PPTX slides: 57
PPTX notes slides: 57
PPTX PNG images: 57
PNG resolution: 3843 x 2162
PPTX size: 16 MB
```

Superseded: the generated PPTX included the `DI Auto Layout Selection` slide at
the time, but that slide has now been removed from the proposal deck. Current
PPTX sync belongs to Feature 023.
