# NDNSF-DistributedRepo Design Slides

`main.tex` is the canonical source for the NDNSF-DistributedRepo design and
mechanism deck. The generated `main.pdf` is kept beside it for presentation.

Build from this directory:

```bash
pdflatex -interaction=nonstopmode -halt-on-error main.tex
pdflatex -interaction=nonstopmode -halt-on-error main.tex
pdfinfo main.pdf
```

Render pages for visual inspection:

```bash
rm -f /tmp/ndnsf-repo-slide-*.png
pdftoppm -png -r 140 main.pdf /tmp/ndnsf-repo-slide
```

The deck is intentionally self-contained: architecture and mechanism diagrams
are drawn in TikZ, so no external image assets are required.
