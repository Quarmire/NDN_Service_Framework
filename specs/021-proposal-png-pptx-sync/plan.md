# Plan: Proposal PNG PPTX Sync

## Approach

Use:

```bash
python3 generate_pdf_matched_pptx.py
```

The script renders each Beamer PDF page with `pdftoppm` and inserts each page as
a full-slide image. It then calls `add_speaker_notes_to_pptx.py` to inject
speaker notes directly into the PPTX package.

## Validation

- Rebuild Beamer PDF and speaker notes PDF.
- Regenerate PPTX.
- Inspect PPTX internals:
  - slide XML count
  - notes slide count
  - embedded media image dimensions
- Confirm generated PNGs are high resolution.

## Observed Result

The regenerated PPTX was PDF-matched and notes-enabled before the DI auto slide
was removed:

```text
57 slides before removal
57 notes slides
57 embedded PNG images
3843 x 2162 pixels per slide image
```

The current post-removal PPTX validation belongs to Feature 023.
