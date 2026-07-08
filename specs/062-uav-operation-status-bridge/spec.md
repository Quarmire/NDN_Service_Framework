# Feature Specification: UAV Operation Status Bridge

**Feature Branch**: `062-uav-operation-status-bridge`

**Created**: 2026-07-08

**Status**: Draft

## User Story

As an NDNSF application developer, I need UAV command, mission, and recording
states to map into the reusable core `ServiceOperationStatus` and
`DataProductReference` shapes, so UAV can share generic operation evidence
without moving MAVLink, camera, codec, or mission policy into NDNSF core.

## Requirements

- **FR-001**: UAV flight command state MUST map to core
  `ServiceOperationStatus` using the core lifecycle vocabulary.
- **FR-002**: UAV mission part state and mission progress MUST map to core
  `ServiceOperationStatus` while preserving UAV phase/detail text in the
  status message and reason code.
- **FR-003**: UAV recording data products MUST map to core
  `DataProductReference` and attach that reference to a completed recording
  operation status.
- **FR-004**: The bridge MUST stay in `NDNSF-UAV-APP/shared` and MUST NOT move
  MAVLink, camera, H264/FEC/ROI, or mission-planning policy into core.
- **FR-005**: Existing UAV state behavior MUST remain compatible with current
  tests.

## Non-Goals

- Do not change UAV GUI behavior.
- Do not change stream/FEC/video bitrate policy.
- Do not add metadata fields to the C++ core operation-status schema in this
  migration.
- Do not change wire protocol names.
