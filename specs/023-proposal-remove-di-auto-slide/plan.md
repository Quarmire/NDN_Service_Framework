# Plan: Remove DI Auto Slide From Proposal Deck

## Approach

Keep the broader DI layout discussion slide, but remove the auto-assignment
campaign page and presenter note. Then rebuild all deck artifacts from Beamer:

```bash
pdflatex -interaction=nonstopmode main.tex
pdflatex -interaction=nonstopmode main.tex
pdflatex -interaction=nonstopmode speaker_notes.tex
pdflatex -interaction=nonstopmode speaker_notes.tex
python3 generate_pdf_matched_pptx.py
```

Finally, verify PPTX internals and LibreOffice export.

## Observed Result

The post-removal deck has 56 pages. The generated PPTX has 56 slides and 56
speaker-note parts. LibreOffice exported the PPTX to a 56-page PDF, and page 46
renders as `DI Validation`.
