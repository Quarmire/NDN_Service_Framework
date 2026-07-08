# Feature Specification: DI Execution Operation Status

**Feature Branch**: `063-di-execution-operation-status`

**Created**: 2026-07-08

**Status**: Draft

## User Story

As an NDNSF-DI experiment user, I need NativeTracer `userExecution` and
`dependencyExecution` summaries to carry core `ServiceOperationStatus` while
preserving the existing summary fields, so GUI, campaign, and paper evidence
can use the same app-neutral lifecycle vocabulary as Repo, UAV, and deployment
status.

## Requirements

- **FR-001**: NativeTracer summary `userExecution` dictionaries MUST preserve
  legacy fields and add `operationStatus`.
- **FR-002**: NativeTracer summary `dependencyExecution` dictionaries MUST
  preserve legacy fields and add `operationStatus`.
- **FR-003**: Status mapping MUST be explicit and defensive:
  `executed`/success-like statuses map to `DONE`, failed statuses map to
  `FAILED`, timeout-like statuses map to `EXPIRED`, active statuses map to
  `RUNNING`, and unknown/missing statuses map to `QUEUED`.
- **FR-004**: The helper MUST keep model/dependency details in
  `ServiceOperationStatus.metadata` without moving model, tensor, cache, or
  dependency-exchange policy into core.
- **FR-005**: Existing GUI/RPS/campaign consumers MUST remain compatible with
  legacy top-level summary fields.

## Non-Goals

- Do not change the NativeTracer request execution path.
- Do not change MiniNDN workload behavior.
- Do not move DI model-planning or tensor semantics into core.
- Do not require GUI or campaign consumers to read only the new envelope.
