# Plan: Proposal PPTX LibreOffice Check

## Approach

Use a temporary output directory:

```bash
libreoffice --headless --convert-to pdf --outdir /tmp/ndnsf-pptx-lo-check \
  docs/PAPER/proposal-defense/slides/NDNSF_proposal_google_slides.pptx
```

Then inspect page count and render page 46 for a visual spot check.

## Observed Result

LibreOffice 6.4.7.2 successfully exported the generated PPTX before the DI auto
slide removal. The post-removal deck validation belongs to Feature 023.
