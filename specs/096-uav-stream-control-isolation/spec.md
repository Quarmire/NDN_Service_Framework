# Feature Specification: UAV Stream And Control Isolation Campaign

**Status**: Planned

## Context

Spec 095 found that at 5% one-way MiniNDN loss, video plus concurrent Targeted
Arm/Takeoff/Land completed 2/3 runs without parity and 1/3 with parity. The
combined workload cannot identify whether failures originate in the stream
path, the Targeted control path, or their interaction. This feature adds only
the experiment modes needed to isolate those paths; it does not tune runtime,
retry, timeout, FEC, bitrate, or Core behavior.

## User Stories

### User Story 1 - Execute isolated workloads (Priority: P1)

An experiment operator can run Targeted control alone, video alone, or video
and Targeted control concurrently through the same MiniNDN launcher.

### User Story 2 - Preserve matched evidence (Priority: P1)

The operator runs three repetitions of each frozen 5% loss workload cell with
the same topology, binaries, configuration, source, and no automatic retries.

### User Story 3 - Locate the observed boundary (Priority: P1)

The campaign separately reports process, video, and control completion so the
result can distinguish path-local failure from a concurrency interaction.

## Functional Requirements

- **FR-001** The campaign MUST support `control-only`, `video-only`, and
  `combined` workload modes through the existing UAV MiniNDN launcher.
- **FR-002** Video modes MUST support parity 0 and 1; control-only MUST have no
  video or FEC treatment and MUST not emit `--auto-video-test`.
- **FR-003** Control modes MUST execute the existing Targeted Arm, Takeoff, and
  Land sequence and require all three accepted markers.
- **FR-004** Video modes MUST use the existing 60-second, 30-fps, 50% usable
  frame gate and fail below 900 decoded frames.
- **FR-005** The primary matrix MUST contain exactly five 5% loss cells:
  control-only, video-only parity 0/1, and combined parity 0/1.
- **FR-006** Each primary cell MUST run three repetitions without automatic
  retry or replacement of failed runs.
- **FR-007** All cells MUST use Memphis GS/controller, UCLA drone, 1 ms delay,
  1000 Mbps, WARN NFD logging, the same software revision, and the same policy.
- **FR-008** Video cells MUST additionally hold file camera, 1200 kbps, width
  320, and 1000 ms startup delay constant.
- **FR-009** Per-run output MUST include workload mode, required components,
  return code, process/video/control completion, acceptance, elapsed time, and
  the Spec 095 video/FEC/buffer/RTT fields when video is present.
- **FR-010** Aggregate output MUST report per-cell completion counts and rates,
  mean decoded frames, FEC recovery, RTT, timeout, and elapsed time where
  applicable, while retaining every failed run.
- **FR-011** Tests MUST cover the exact matrix, mode-specific command flags,
  control-only parsing, video acceptance reuse, aggregation, and invalid mode.
- **FR-012** Interpretation MUST remain descriptive, MUST NOT claim statistical
  significance from n=3, and MUST preserve a negative or inconclusive result.
- **FR-013** Core, UAV runtime protocol, proposal slides, and proposal paper
  MUST NOT be modified.

## Success Criteria

- **SC-001** Strict Spec Kit audit and focused/full Python regressions pass.
- **SC-002** The dry-run contains exactly 15 unique runs in five cells.
- **SC-003** All 15 primary MiniNDN runs execute once and remain in JSON/CSV.
- **SC-004** Control-only evidence reveals whether Targeted control succeeds
  without video at the same 5% loss.
- **SC-005** Video-only versus combined matched cells reveal whether concurrent
  control coincides with lower video completion for either parity treatment.
- **SC-006** Final evidence names confounds and distinguishes implementation
  correctness from measured application success.

## Non-Goals

- Changing retry, timeout, SVS, FEC, bitrate, stream, or Targeted behavior.
- Real radio, camera, PX4, flight-safety, or causal validation.
- Reusing Spec 095 combined logs as if they were randomized Spec 096 runs.
