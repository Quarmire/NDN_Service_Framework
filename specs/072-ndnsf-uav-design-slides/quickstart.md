# Quickstart

Build the deck:

```bash
cd /home/tianxing/NDN/ndn-service-framework/docs/NDNSF-UAV/slides
pdflatex -interaction=nonstopmode -halt-on-error main.tex
pdflatex -interaction=nonstopmode -halt-on-error main.tex
pdfinfo main.pdf | rg 'Pages|Page size'
```

Render it for visual review:

```bash
rm -f /tmp/ndnsf-uav-slide-*.png
pdftoppm -png -r 120 main.pdf /tmp/ndnsf-uav-slide
```

Tracked deliverables:

```text
docs/NDNSF-UAV/slides/main.tex
docs/NDNSF-UAV/slides/main.pdf
docs/NDNSF-UAV/slides/README.md
```
