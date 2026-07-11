# Feature Specification: Native DI User Driver Correctness

**Feature**: `092-native-di-user-driver-correctness`
**Created**: 2026-07-11
**Status**: Active - Test First

## Purpose

Remove the user-driver lifecycle and measurement defects identified by Spec 091
so a Native DI offered-load experiment measures NDNSF execution rather than
driver setup or missing scope-key service.

## User Scenarios & Testing

### User Story 1 - Run A Valid Open-Loop Native DI Workload (Priority: P1)

An NDNSF-DI developer can use the threaded or process-pool driver and obtain a
machine-readable summary whose scope-key producer remains available, schedule
slip is measured from the actual target time, and throughput excludes the
intentional process startup lead.

**Independent Test**: Run focused unit tests, then rerun threaded and
process-pool at 1 RPS for 60 seconds with concurrency 4 on the same Qwen
full-network fixture used by Spec 091.

## Functional Requirements

- **FR-001**: The base `ServiceUser` that publishes scope-key large Data MUST
  remain started for the complete threaded workload and MUST stop in a
  `finally`-equivalent cleanup path.
- **FR-002**: Process-pool workers MUST record `scheduleSlipMs` for every
  emitted request relative to its preassigned wall-clock target.
- **FR-003**: Open-loop workload metadata MUST report measurement elapsed time
  and maximum observed schedule slip; process-pool MUST also report its
  scheduled measurement start.
- **FR-004**: Achieved throughput MUST use the measured interval from the
  scheduled workload start through request completion, excluding intentional
  worker startup lead and post-workload user teardown.
- **FR-005**: Existing child, closed-loop, security, model, topology, placement,
  and provider execution behavior MUST remain unchanged.
- **FR-006**: Missing worker results and failed requests MUST remain visible and
  MUST NOT be excluded from completion or latency evidence.
- **FR-007**: Unit tests MUST fail before the implementation change for the
  lifecycle and measurement contracts, then pass after the change.
- **FR-008**: Final MiniNDN validation MUST use the same Qwen full-network
  controls and Spec 091 scheduling/stability gates.
- **FR-009**: A paper-facing or maximum-stable-RPS claim MUST NOT be made from a
  single post-fix run; a passing candidate MUST receive at least three matched
  repetitions first.

## Success Criteria

- **SC-001**: Focused user-driver tests pass and exercise lifecycle cleanup,
  worker slip calculation, and measurement-interval throughput.
- **SC-002**: The threaded rerun emits a complete workload summary without the
  prior scope-key fetch failure, or preserves a newly diagnosed failure with
  direct evidence.
- **SC-003**: The process-pool rerun reports non-placeholder slip and a measured
  throughput interval that excludes the startup lead.
- **SC-004**: At 1 RPS, the selected driver is classified using the predeclared
  Spec 091 gates; any passing candidate has three matched repetitions before a
  performance claim.
- **SC-005**: Exact commands, commits, raw result paths, and negative outcomes
  are recorded.

## Non-Goals

- Raising provider concurrency or changing admission policy.
- Optimizing NDN-SVS, model execution, placement, or dependency exchange.
- Claiming maximum stable RPS.
- Modifying proposal slides.
