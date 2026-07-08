# Feature Specification: Deployment Operation Status

**Feature Branch**: `061-deployment-operation-status`

**Created**: 2026-07-08

**Status**: Draft

## User Story

As an NDNSF application developer, I need deployment lifecycle records to carry
core `ServiceOperationStatus` while preserving the existing deployment `status`
field, so deployment discovery, sorting, waiting, and diagnostics share the same
operation-status vocabulary used by Repo and DI provisioning.

## Requirements

- **FR-001**: Deployment dictionaries produced by `deploy_service()` MUST include
  a core `operationStatus` object.
- **FR-002**: Deployment discovery sorting MUST prefer `operationStatus.state`
  when present and fall back to legacy `status` otherwise.
- **FR-003**: Helpers MUST map legacy deployment statuses into
  `ServiceOperationStatus` without removing the legacy field.
- **FR-004**: Eviction and rejected/not-found responses SHOULD include
  operation status where practical.
- **FR-005**: Tests MUST cover active, provisioning, evicted, rejected, and
  legacy-only deployments.

## Non-Goals

- Do not change NDNSD publication protocol.
- Do not remove the existing `status` field.
- Do not move DI deployment policy into core.

