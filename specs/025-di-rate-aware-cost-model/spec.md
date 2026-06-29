# Feature Specification: DI Rate-Aware Cost Model

**Status**: Accepted
**Created**: 2026-06-28

## User Story

As an NDNSF-DI experiment runner, I want the NativeTracer planner to account for
target request rate, provider capacity, and dependency byte rate so auto layout
selection can be evaluated under high-concurrency workloads without changing the
smallest Qwen NativeTracer artifacts.

## Problem

The current planner evidence can distinguish low concurrency from higher
concurrency using provider ready-queue pressure, but it still treats workload
pressure mostly as an outstanding-request count. A high-concurrency plan should
also expose whether a candidate overloads a provider's single worker and whether
cross-provider dependencies create network byte-rate pressure.

## Requirements

- **FR-001**: Planner evidence must accept an optional target request rate in
  requests per second.
- **FR-002**: Candidate cost must report provider utilization and provider
  capacity queue pressure when a target request rate is supplied.
- **FR-003**: Candidate cost must report dependency byte rate and dependency
  link utilization for cross-provider dependency edges.
- **FR-004**: Default behavior must remain compatible: if no target request
  rate is supplied, existing c1/c4/c8 auto recommendations must not change.
- **FR-005**: MiniNDN/campaign helper summaries must propagate the new fields so
  experiments can compare estimated pressure with measured provider queue wait.

## Success Criteria

- **SC-001**: Python planner/campaign scripts compile.
- **SC-002**: Planner-only validation shows target RPS fields in
  `planner-optimization.json` and CSV.
- **SC-003**: With `targetRps=0`, c1 recommends `single-provider-serial` while
  c4/c8 recommend `shared-backbone-current`.
- **SC-004**: With a positive target RPS, candidate costs include non-zero
  provider utilization and dependency byte-rate fields where applicable.

## Scope

In scope:

- NativeTracer planner evidence and campaign summaries.
- CLI parameter propagation through `plan_tracer.py`, the MiniNDN harness, and
  campaign helpers.
- Planner-only validation and a small smoke if needed.

Out of scope:

- Changing the C++ provider worker.
- Changing the smallest Qwen NativeTracer artifacts.
- Proposal slides.
