# Baseline And Pre-Implementation Audit

## Context gate

- CodeGraph index: current, 2,149 files and 47,479 nodes before edits.
- Git worktree: clean before Spec 095 artifacts were created.
- Disk: workspace 7.0 GiB; filesystem 8.0 GiB free.
- Existing focused campaign tests: 2 passed.
- Existing Spec 089 evidence: three 8-second runs at 5% one-way loss completed,
  recovered 7 chunks total, stayed within buffering limits, and reported zero
  decoded-frame gap. It has no FEC-off control or concurrent command traffic.

## Disposition and boundary audit

- KEEP Core StreamInfo/StreamChunk/StreamFecInfo and native stream state.
- KEEP UAV-owned H264, XOR FEC, ROI, MAVLink, mission, and safety policy.
- ADD only a UAV request parameter and experiment orchestration/measurement.
- DO NOT add a second stream engine, transport, retry layer, or finite-object
  streaming path.
- Security impact is bounded: normal video-control and Targeted MAVLink calls
  retain existing permissions, tokens, replay checks, and provider handlers.
- Persistence impact is none; recording Repo is not part of the primary matrix.
- Rollback is one focused implementation revert; default parity remains one.

## Experiment freeze

- Primary cells: loss 0/5% x parity 0/1 x three repetitions.
- Duration: 60 seconds; no automatic retries.
- Fixed: Memphis GS/controller, UCLA drone, 1 ms, 1000 Mbps, file camera,
  1200 kbps, width 320, 1000 ms start delay, WARN NFD logging.
- Every run also requires accepted Arm, Takeoff, and Land responses while the
  video automation is active.
- Acceptance thresholds: all required markers, no stale acceptance, pending
  chunks <=48, pending bytes <=16 MiB.

## Audit verdict

**PASS**. The feature has a falsifiable control/treatment design, clear Core/UAV
ownership, no security bypass, bounded parameter domain, failure-preserving
evidence, and executable validation. No implementation blocker remains.

