# Feature Specification: NDNSF-UAV Design Slides

**Feature Branch**: `072-ndnsf-uav-design-slides`
**Created**: 2026-07-09
**Status**: Complete

## Goal

Create a standalone LaTeX/Beamer PDF deck under `docs/NDNSF-UAV/slides` that
explains the implemented design and mechanisms of `NDNSF-UAV-APP`. The deck is
not a proposal-defense derivative and must not modify proposal slide sources.

## Audience

The primary audience is an NDN or distributed-systems researcher who needs to
understand how the UAV application uses NDNSF for secure service invocation,
low-latency flight control, multi-drone workflows, live video, and durable data
products.

## Content Requirements

- **CR-001**: Explain why a UAV workload stresses mobility, authorization,
  low-latency control, continuous media, and multi-provider coordination.
- **CR-002**: Separate NDNSF Core, UAV application, and DistributedRepo
  responsibilities without moving MAVLink, mission, H264, FEC, or operator
  policy into the core.
- **CR-003**: Show Controller, Ground Station, and Drone process roles and the
  service-container boundary.
- **CR-004**: Show the current named service catalog and distinguish normal,
  Targeted, and provider-specific invocation.
- **CR-005**: Explain certificate/permission bootstrap, NAC-ABE authorization,
  one-time tokens, and replay protection at a system level.
- **CR-006**: Explain typed telemetry/readiness, command lifecycle, safety
  gates, and operator authority leases.
- **CR-007**: Explain the flight-controller backend boundary and the current
  mock, UDP, serial, and MAVLink-router-compatible paths.
- **CR-008**: Explain mission planning, deterministic per-drone parts, MAVLink
  upload, missing-part compensation, and persistent mission documents.
- **CR-009**: Explain vehicle parameter compare-and-set, preflight checklist,
  and MAVLink analyze snapshot services without claiming QGroundControl parity.
- **CR-010**: Explain the live-video control/data split, stream/session metadata,
  adaptive fetch loop, and one-loss XOR FEC recovery.
- **CR-011**: Clearly distinguish continuous live streaming from encrypted,
  repo-backed recording objects fetched by exact names.
- **CR-012**: Explain the ground-station object-detection callback and state
  that service requests carry metadata rather than image bytes.
- **CR-013**: Summarize GUI/headless deployment and MiniNDN/SITL validation
  paths without inventing benchmark results.
- **CR-014**: End with an honest implementation boundary and next engineering
  priorities.

## Presentation Requirements

- **PR-001**: Use 16:9 Beamer, white background, dark-blue titles, and the
  established NDNSF technical-deck visual language.
- **PR-002**: Keep each slide centered on one mechanism and favor diagrams over
  paragraphs.
- **PR-003**: Use implementation evidence labels on technical slides.
- **PR-004**: Keep diagrams and text within the frame with no overfull boxes.
- **PR-005**: Build with `pdflatex` and keep only the canonical source, PDF, and
  README in the slide directory.

## Non-Goals

- Rewriting or synchronizing proposal-defense slides.
- Claiming production certification or full QGroundControl replacement.
- Moving UAV-specific semantics into NDNSF Core.
- Adding new UAV runtime functionality or benchmark measurements.
- Creating an editable PowerPoint version.

## Acceptance Criteria

- The deck compiles twice with `pdflatex` without errors or overfull boxes.
- The resulting PDF has the same frame count reported by `pdfinfo` and the
  Beamer footer.
- Rendered page images are visually inspected for clipping, overlap, and
  unreadable type.
- The deck states the live-stream versus exact-name object-transfer boundary.
- Every implementation claim can be traced to current code, README, specs, or
  test harnesses.
- `docs/PAPER/proposal-defense/slides` remains unchanged.
