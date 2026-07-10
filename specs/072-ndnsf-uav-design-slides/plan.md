# Implementation Plan: NDNSF-UAV Design Slides

## Technical Approach

Use a standalone Beamer source derived from the visual conventions of the
NDNSF-DistributedRepo technical deck, not from the proposal deck. Diagrams are
implemented in TikZ so the architecture remains reviewable and text stays
sharp in PDF.

## Source-of-Truth Order

1. `NDNSF-UAV-APP` implementation and shared protocol types.
2. `NDNSF-UAV-APP/README.md` for deployment and validation recipes.
3. Specs 062, 064, 069, and 070 for completed design boundaries.
4. Proposal text only for concise motivation; it must not override current
   implementation.

## Deck Narrative

1. Motivation and responsibility boundary.
2. Runtime roles, service containers, names, security, and invocation modes.
3. Flight-control state, safety, authority, MAVLink, and mission mechanisms.
4. Live-video control/data split, session safety, adaptation, and FEC.
5. Durable recording/data products, object-detection callback, validation,
   and current limitations.

## Accuracy Guardrails

- Telemetry is currently requested through normal/Targeted NDNSF invocation;
  it is not presented as a `StreamChunk` subscription.
- MAVLink semantics, mission logic, H264, XOR FEC, and bitrate policy remain in
  UAV-APP, while core supplies invocation, security, operation status, stream
  envelopes, and data references.
- Mission compensation means reassigning missing mission parts after an
  invocation deadline, not autonomous in-flight route optimization.
- Stream session guards reject stale sessions; they are not exclusive-consumer
  leases.
- Adaptive video logic changes fetch pressure and recommends bitrate. Producer
  bitrate changes use an explicit Stop-then-Start control path.
- XOR parity recovers at most one missing data shard per frame in the current
  implementation.
- Recorded media is an encrypted repo object fetched by exact chunk names, not
  a continuation of the live stream.

## Verification

```bash
cd docs/NDNSF-UAV/slides
pdflatex -interaction=nonstopmode -halt-on-error main.tex
pdflatex -interaction=nonstopmode -halt-on-error main.tex
pdfinfo main.pdf
rg -n "Overfull|Underfull|LaTeX Error|Undefined control sequence" main.log
pdftotext main.pdf /tmp/ndnsf-uav-slides.txt
pdftoppm -png -r 120 main.pdf /tmp/ndnsf-uav-slide
```

The rendered montage and representative full-resolution pages are inspected
after compilation. Auxiliary files are removed from the tracked slide folder.
