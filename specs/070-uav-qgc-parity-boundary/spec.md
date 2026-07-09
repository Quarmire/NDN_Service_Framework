# Feature Specification: UAV QGC-Parity Boundary Slice

**Feature Branch**: `070-uav-qgc-parity-boundary`  
**Created**: 2026-07-09  
**Status**: Implemented foundation slice

## Goal

Clarify which QGroundControl-like mechanisms belong in reusable NDNSF core and
which belong in NDNSF-UAV-APP, then add the first reusable UAV-side protocol
contracts needed for setup, fly, plan, and analyze workflows.

## Boundary Decision

NDNSF core owns app-neutral mechanisms:

- service invocation, ACK/selection/response, Targeted invocation;
- controller security, permissions, certificate bootstrap, tokens;
- core operation status, provider capability hints, stream health, provider
  telemetry, data product references.

NDNSF-UAV-APP owns UAV operational semantics:

- MAVLink command and parameter meaning;
- preflight and arming checks;
- mission, geofence, rally, survey, flight mode, camera, gimbal, H264/FEC/ROI;
- ground-station GUI and QGroundControl-like setup/fly/plan/analyze workflow.

## User Stories

### US1 - Vehicle Parameter Edit Contract

As a ground-station operator, I need a typed request/result contract for editing
vehicle parameters so that a future GUI can implement QGC-like parameter editing
without inventing ad hoc fields.

### US2 - Preflight Checklist Contract

As a ground-station operator, I need a typed preflight item contract so that
safety checks can be displayed, sorted, and block unsafe commands.

### US3 - MAVLink Analyze Snapshot Contract

As a ground-station operator, I need summarized MAVLink message rates and
vehicle state so that an Analyze/Inspector panel can show what the vehicle is
publishing.

## Acceptance Criteria

- `docs/ndnsf-core-app-boundary.md` explains the NDNSF core vs UAV APP split.
- `UavProtocol` declares and implements:
  - `VehicleParameterEditRequest`
  - `VehicleParameterEditResult`
  - `PreflightCheckItem`
  - `MavlinkMessageSummary`
  - `UavAnalyzeSnapshot`
- C++ unit tests round-trip all new contracts and validate key helper logic.
- No proposal slides are changed.

