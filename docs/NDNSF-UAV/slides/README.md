# NDNSF-UAV-APP Design Slides

This directory contains the standalone LaTeX/Beamer deck for the design and
mechanisms of `NDNSF-UAV-APP`.

Canonical files:

- `main.tex`: editable Beamer source.
- `main.pdf`: compiled 16:9 PDF.

Build with:

```bash
cd /home/tianxing/NDN/ndn-service-framework/docs/NDNSF-UAV/slides
pdflatex -interaction=nonstopmode -halt-on-error main.tex
pdflatex -interaction=nonstopmode -halt-on-error main.tex
```

The deck is implementation-grounded. Its main sources are:

- `NDNSF-UAV-APP/README.md`
- `NDNSF-UAV-APP/shared/UavNames.hpp`
- `NDNSF-UAV-APP/shared/UavProtocol.*`
- `NDNSF-UAV-APP/drone/DroneServiceContainer.inc.hpp`
- `NDNSF-UAV-APP/ground-station/GroundStationServiceContainer.inc.hpp`
- `Experiments/NDNSF_UAV_GUI_Minindn.py`
- `specs/069-uav-operational-layer/`
- `specs/070-uav-qgc-parity-boundary/`

This deck is independent from the PhD proposal-defense slides. Updating it must
not modify `docs/PAPER/proposal-defense/slides` unless explicitly requested.
