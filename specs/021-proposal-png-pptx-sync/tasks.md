# Tasks: Proposal PNG PPTX Sync

- [x] S001 Create Feature 021 spec, plan, and task list.
- [x] S002 Rebuild Beamer PDF and speaker notes.
- [x] S003 Regenerate PDF-matched PNG-image PPTX.
- [x] S004 Verify PPTX slide count and notes count.
- [x] S005 Verify embedded image resolution.
- [x] S006 Record accepted sync results.

## Result

Command run:

```bash
python3 generate_pdf_matched_pptx.py
```

Generated:

```text
docs/PAPER/proposal-defense/slides/NDNSF_proposal_google_slides.pptx
```

Historical accepted checks before the slide removal:

- PDF page count: 57.
- PPTX slide count: 57.
- Notes slide count: 57.
- Embedded PNG count: 57.
- Embedded PNG resolution: 3843 x 2162.
- Notes slide 46 included the DI auto layout selection note before that slide
  was removed from the proposal deck.
