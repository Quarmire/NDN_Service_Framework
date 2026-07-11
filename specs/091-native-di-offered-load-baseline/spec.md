# Feature Specification: Native DI Offered-Load Baseline

**Feature**: `091-native-di-offered-load-baseline`
**Created**: 2026-07-11
**Status**: Complete - User Driver Boundary Identified

## Purpose

Establish a reproducible Native DI full-network offered-load baseline before
changing scheduler or provider code. Determine whether the first throughput
limit is the user driver, NDNSF invocation, provider admission, dependency
exchange, or model execution.

## User Scenarios & Testing

### User Story 1 - Locate The First Throughput Boundary (Priority: P1)

An NDNSF-DI developer can run the same Qwen proportional workload through each
existing open-loop driver and identify the first limiting layer from measured
schedule slip, backpressure, completion, latency, throughput, provider state,
and dependency counters.

**Independent Test**: Run the three 60-second treatment commands and compare
only their machine-readable summaries against FR-005 and FR-006.

## Functional Requirements

- **FR-001**: All screening runs MUST use the same git commit, runtime profile,
  topology, assignment, model artifacts, request cap, concurrency, target RPS,
  duration, timeouts, logging, and telemetry settings.
- **FR-002**: The only screening treatment variable MUST be
  `open_loop_driver_mode` in `child`, `threaded`, or `process-pool`.
- **FR-003**: Every measured run MUST use a 60-second open-loop window and the
  real full-network Qwen NativeTracer path.
- **FR-004**: Every run MUST record scheduled/submitted/success/failure counts,
  offered and achieved RPS, p50/p95, maximum schedule slip, local backpressure,
  timeout/rejection reasons, provider utilization, and dependency completion.
- **FR-005**: A driver is scheduling-capable at the screening point only if it
  submits at least 95% of scheduled requests, has maximum schedule slip below
  one request interval, and does not report local backpressure failures.
- **FR-006**: A system point is stable only if success rate is at least 99%,
  achieved throughput is at least 95% of offered load, all dependency roles
  execute, and no malformed/security bypass is observed.
- **FR-007**: No production scheduler/provider change may be made before the
  screening result identifies a specific limiting layer.
- **FR-008**: Negative or inconclusive results MUST be preserved without being
  described as an optimization.

## Success Criteria

- **SC-001**: Three matched screening summaries exist and are machine-readable.
- **SC-002**: The report names the first observed limiting layer with direct
  counters, or explicitly reports that 1 RPS is below all observed limits.
- **SC-003**: The report selects the next experiment or implementation task
  without conflating user-driver backpressure with provider capacity.
- **SC-004**: Exact commands, commit, environment, result paths, anomalies, and
  skipped checks are recorded.

## Non-Goals

- Claiming maximum stable RPS from one screening point.
- Comparing model layouts or changing Qwen artifacts.
- Modifying proposal slides.
- Optimizing NDN-SVS or weakening NDNSF security.
