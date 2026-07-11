# Feature Specification: UAV Video FEC And Control Campaign

**Status**: Planned

## Context

Spec 089 proved that the Core stream engine and UAV XOR parity path can carry an
eight-second H264 stream at 5% one-way loss. It did not include a no-FEC control,
a 60-second measured window, or concurrent flight-control traffic. This feature
adds the minimum experiment controls and evidence needed to measure those cases
without moving H264, FEC, ROI, MAVLink, mission, or safety policy into Core.

## User Stories

### User Story 1 - Select FEC treatment (Priority: P1)

An experiment operator can request zero or one XOR parity shard for a UAV video
stream. The accepted value is visible in the video-control response and logs.

### User Story 2 - Run matched MiniNDN treatments (Priority: P1)

An operator runs one campaign that holds topology, bitrate, width, duration,
camera input, and control actions constant while varying link loss and parity.

### User Story 3 - Obtain falsifiable evidence (Priority: P1)

The campaign reports completion, decoded frames, timeout/gap/buffer/FEC metrics,
RTT, and Arm/Takeoff/Land outcomes per run and aggregates matched treatments.

## Functional Requirements

- **FR-001** Ground Station MUST accept `video-fec-parity-shards` in the range
  0..1 and include it as `fec_parity_shards` in every video start request.
- **FR-002** Drone MUST accept `fec_parity_shards` in the range 0..1, default to
  one when absent, and report the accepted value. Zero MUST publish only data
  symbols while preserving ordinary StreamChunk framing.
- **FR-003** FEC configuration MUST remain UAV-owned; Core StreamFecInfo remains
  codec-neutral and MUST NOT implement XOR policy.
- **FR-004** The canonical UAV parity campaign MUST support a matrix of loss
  percentages, parity treatments, repetitions, and a 60-second default stream.
- **FR-005** Every treatment MUST use the same Memphis GS/controller, UCLA
  drone, link delay/bandwidth, H264 source, bitrate, width, and startup timing.
- **FR-006** The campaign MUST optionally run Arm/Takeoff/Land through Targeted
  NDNSF while video is active and fail if any accepted response is absent.
- **FR-007** Per-run evidence MUST include return code, completion, treatment,
  decoded-frame count, FEC recovery, timeout, Nack, duplicate, frame-gap,
  pending-buffer, and RTT measurements.
- **FR-008** Aggregate evidence MUST include completion rate and p50/p95/mean or
  maxima appropriate to each metric, without claiming causality from one run.
- **FR-009** A run MUST fail acceptance on process error, missing 60-second
  stream completion, missing control markers, unbounded buffering, stale stream
  acceptance, or malformed metrics.
- **FR-010** Unit tests MUST cover parity validation, request propagation,
  command construction, parsing, aggregation, and failure classification.
- **FR-011** MiniNDN final evidence MUST include three repetitions for the 0%
  and 5% matched FEC-off/FEC-on cells. A 15% one-way-loss pair MAY be retained
  as boundary evidence but MUST be labeled exploratory unless replicated.
- **FR-012** Proposal files MUST NOT be modified.

## Success Criteria

- **SC-001** Focused C++ and Python tests and affected UAV targets pass.
- **SC-002** The campaign produces machine-readable JSON and CSV with no missing
  required field for successful runs.
- **SC-003** All four primary cells (0/5% x parity 0/1) execute three 60-second
  runs; failures remain recorded as results rather than silently retried.
- **SC-004** Concurrent Arm/Takeoff/Land succeeds in every accepted primary run.
- **SC-005** Buffering remains below 48 chunks and 16 MiB in accepted runs.
- **SC-006** Final interpretation separates measured effect, uncertainty,
  boundary observations, and implementation correctness.

## Non-Goals

- Moving XOR FEC or H264 policy into NDNSF Core.
- Claiming real-radio, real-camera, PX4 hardware, or flight-safety validation.
- Replacing SegmentFetcher for finite named objects.
- Tuning bitrate, window, or retry policy during a matched run.
