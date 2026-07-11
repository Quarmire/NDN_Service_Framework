# Feature Specification: Native DI Threaded RPS Boundary

**Feature**: `093-native-di-threaded-rps-boundary`
**Created**: 2026-07-11
**Status**: Active - Measurement First

## Purpose

Find a bounded, reproducible offered-load boundary for the validated threaded
Qwen Native DI user driver without confusing user scheduling, NDNSF control,
provider capacity, dependency exchange, or model execution.

## User Scenarios & Testing

### User Story 1 - Locate The Highest Tested Stable Rate (Priority: P1)

An NDNSF-DI developer can start from the validated 1 RPS result, increase real
offered load, stop at the first unstable point, refine the boundary, and name
the first limiting layer from machine-readable counters.

**Independent Test**: Run the frozen 60-second MiniNDN commands and apply the
same scheduling and stability gates to every point.

## Functional Requirements

- **FR-001**: Every treatment MUST use commit `855b6ac` or a later docs-only
  commit with identical runtime code, the real Qwen ONNX NativeTracer path,
  AI_Lab topology, proportional 2/4/8 GB placement, threaded driver,
  concurrency 4, and a 60-second measured interval.
- **FR-002**: Request cap MUST equal `ceil(targetRps * 60)` so the complete
  offered schedule can be emitted.
- **FR-003**: The coarse search MUST test 2, 4, and 8 RPS in ascending order and
  MUST stop after the first unstable point; the validated Spec 092 1 RPS runs
  serve as the initial stable anchor.
- **FR-004**: A point is scheduling-capable only if submitted/scheduled is at
  least 95%, maximum slip is below one request interval, and local
  backpressure failures are zero.
- **FR-005**: A point is stable only if success is at least 99%, achieved
  throughput is at least 95% of target, all expected dependencies complete,
  and no malformed/security bypass is observed.
- **FR-006**: After the first unstable point, midpoint treatments MUST refine
  the interval until its width is at most 0.25 RPS or a resource/time stop is
  recorded.
- **FR-007**: The highest tested stable point MUST have three matched runs in
  total before being reported as the highest tested stable rate.
- **FR-008**: Every point MUST retain success/failure, throughput, p50/p95,
  measurement elapsed, schedule slip, backpressure, negative ACK/timeout,
  provider queue/utilization, and dependency counters.
- **FR-009**: The report MUST distinguish highest tested stable RPS from a
  theoretical or universal maximum and MUST preserve all negative results.
- **FR-010**: No production runtime, provider, NDNSF core, NDN-SVS, model, or
  proposal-slide change may be made during this measurement feature.

## Success Criteria

- **SC-001**: At least one rate above 1 RPS is classified with complete
  machine-readable evidence.
- **SC-002**: A stable/unstable bracket no wider than 0.25 RPS is measured, or
  the exact reason refinement could not safely finish is recorded.
- **SC-003**: Three matched runs support the highest tested stable point.
- **SC-004**: The first limiting layer is named using direct counters rather
  than inferred from aggregate throughput alone.
- **SC-005**: Exact commands, commits, paths, anomalies, and skipped checks are
  recorded with an ARS validation report.

## Non-Goals

- Changing the driver or runtime after observing the boundary.
- Comparing layouts, leases, cache policy, or provider counts.
- Claiming a production SLA or universal maximum RPS.
- Modifying proposal slides.
