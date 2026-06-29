# Feature 023: Remove DI Auto Slide From Proposal Deck

Status: Accepted

## Goal

Remove the recently added NDNSF-DI auto-assignment evidence slide from the
proposal defense deck because it is too detailed for proposal slides.

## Scope

- Remove `DI Auto Layout Selection` from Beamer slides.
- Remove the matching speaker note.
- Regenerate PDF, speaker notes PDF, and PNG-image PPTX.
- Verify page counts and PPTX notes after removal.

## Non-Goals

- Removing NDNSF-DI experiment evidence from project documentation.
- Changing the core DI implementation or campaign results.

## Acceptance

- [x] No `DI Auto Layout Selection` slide remains in proposal slides.
- [x] No matching speaker note remains.
- [x] Rebuilt Beamer PDF has 56 pages.
- [x] Regenerated PPTX has 56 slides, 56 notes, and high-resolution images.
- [x] LibreOffice can export the regenerated PPTX with 56 pages.

## Accepted Evidence

Current proposal-defense deck:

```text
main.pdf: 56 pages
speaker_notes.tex: 56 slide entries
NDNSF_proposal_google_slides.pptx: 56 slides, 56 notes, 56 PNG images
PNG resolution: 3843 x 2162
LibreOffice export: 56 pages
```

Rendered LibreOffice page 46 is now `DI Validation`, confirming that the removed
auto-selection slide is no longer in the presentation flow.
