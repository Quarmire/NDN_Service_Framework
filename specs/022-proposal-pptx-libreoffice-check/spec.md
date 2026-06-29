# Feature 022: Proposal PPTX LibreOffice Check

Status: Superseded

## Goal

Verify that the regenerated proposal-defense PNG-image PPTX can be opened by
LibreOffice and exported without corrupting slide count or the new DI auto
selection slide.

## Scope

- Export `NDNSF_proposal_google_slides.pptx` to PDF with LibreOffice headless.
- Check exported PDF page count.
- Render and inspect the exported DI auto selection slide.
- Keep source Beamer PDF and PPTX unchanged.

## Acceptance

- [x] LibreOffice export completes.
- [x] Exported PDF has 57 pages.
- [x] Exported page 46 visually contained `DI Auto Layout Selection` before
  that slide was removed.

## Accepted Evidence

LibreOffice exported:

```text
/tmp/ndnsf-pptx-lo-check/NDNSF_proposal_google_slides.pdf
```

Validation:

```text
Pages: 57
Rendered check page: /tmp/ndnsf-pptx-lo-check/page-46.png
```

Superseded: the DI auto layout selection slide has been removed from the
proposal deck. Current LibreOffice validation belongs to Feature 023.
