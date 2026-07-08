# Feature Specification: UAV Operational Layer

**Feature Branch**: `069-uav-operational-layer`

**Created**: 2026-07-08

**Status**: Draft

**Input**: User request to move NDNSF-UAV-APP toward a QGroundControl-like
operational workflow while preserving the NDNSF core/application boundary.

## User Scenarios & Testing

### User Story 1 - Persistent Mission Operations (Priority: P1)

As a ground-station operator, I need a mission plan object that can be saved,
loaded, reviewed per drone, and later extended with geofence and rally points,
so the current patrol mission prototype can become a repeatable operating
workflow rather than a one-off GUI action.

**Why this priority**: Mission planning is the largest gap between the current
UAV app and a real ground station.

**Independent Test**: A mission plan with drone parts, geofence points, rally
points, metadata, and operator information can round-trip through the shared
state representation and report that it is saveable.

**Acceptance Scenarios**:

1. **Given** a patrol mission plan with assigned drone parts, **When** the plan
   is wrapped as a mission document, **Then** it reports a stable plan id,
   operator id, part count, and saveable status.
2. **Given** a mission document with fence and rally points, **When** it is
   serialized and parsed again, **Then** the operational plan details are
   preserved.

---

### User Story 2 - Operational Data Product Catalog (Priority: P2)

As an operator or post-mission analyst, I need a common catalog summary for
recordings, telemetry logs, detection events, and mission logs, so UAV data
products can be discovered without each UI panel inventing its own status
format.

**Why this priority**: QGroundControl-like workflows depend on logs and
recordings, and NDNSF should expose named data products consistently.

**Independent Test**: A completed recording can produce a catalog summary that
counts available products, bytes, and latest named object prefix.

**Acceptance Scenarios**:

1. **Given** a completed camera recording, **When** the catalog summary is
   created, **Then** it reports a queryable recording product and latest object
   prefix.
2. **Given** telemetry-log and detection counts, **When** the catalog summary is
   serialized and parsed again, **Then** all product counts remain stable.

---

### User Story 3 - Vehicle Capability and Parameter View (Priority: P3)

As an operator, I need a compact view of vehicle firmware, modes, and parameter
snapshot completeness, so NDNSF-UAV can show setup/configuration state without
putting MAVLink parameter semantics in the NDNSF core.

**Why this priority**: Parameter/setup inspection is a major QGroundControl
capability and the current app marks it as limited.

**Independent Test**: A parameter snapshot can carry firmware, vehicle type,
flight modes, parameter count, completeness, and optionally raw parameter
values.

**Acceptance Scenarios**:

1. **Given** a parameter snapshot with raw parameter values, **When** it
   round-trips through fields, **Then** count, firmware, modes, and values are
   preserved.
2. **Given** a compact snapshot without raw values, **When** it is parsed,
   **Then** it still reports a usable capability view.

---

### User Story 4 - Operator Authority Lease (Priority: P4)

As a multi-operator or multi-ground-station deployment, I need a small control
lease model, so two users do not accidentally command the same drone at the
same time.

**Why this priority**: NDNSF's service model can support distributed operators,
but UAV flight control needs a clear authority boundary.

**Independent Test**: A lease allows commands only for the matching drone,
scope, and time window, while monitor-only leases allow telemetry but reject
control.

**Acceptance Scenarios**:

1. **Given** a valid control lease for Drone A, **When** the operator sends
   takeoff to Drone A, **Then** the lease allows the command.
2. **Given** the same lease, **When** the operator sends takeoff to Drone B or
   after expiry, **Then** the lease rejects the command with a reason.
3. **Given** a monitor-only lease, **When** the operator requests telemetry,
   **Then** it is allowed; **When** the operator sends land, **Then** it is
   rejected.

### Edge Cases

- A mission document without a stable id or parts must not be considered
  saveable.
- A catalog with zero products must report that no queryable product is
  available.
- Parameter snapshots may omit raw parameter values for compact telemetry.
- Revoked, expired, wrong-drone, and monitor-scope leases must all return
  explicit reasons.

## Requirements

### Functional Requirements

- **FR-001**: UAV shared state MUST include a mission-plan document that wraps
  an existing `MissionPlan` with save/load metadata, operator id, geofence
  points, rally points, and opaque metadata.
- **FR-002**: Mission-plan documents MUST serialize to and from the existing
  `Fields` representation without changing current NDNSF wire names.
- **FR-003**: UAV shared state MUST include a data-product catalog summary for
  recordings, telemetry logs, detection events, and mission logs.
- **FR-004**: The catalog summary MUST interoperate with the existing
  `RecordingDataProductState`.
- **FR-005**: UAV shared state MUST include a vehicle parameter/capability
  snapshot that can carry both compact metadata and optional raw parameters.
- **FR-006**: UAV shared state MUST include an operator authority lease that
  can validate control versus monitor actions by drone, scope, expiry, and
  revocation.
- **FR-007**: All new state models MUST stay in `NDNSF-UAV-APP/shared` and MUST
  NOT move MAVLink, QGC GUI policy, mission semantics, H264/FEC/ROI, or flight
  control policy into NDNSF core.
- **FR-008**: Existing UAV unit tests and core envelope tests MUST remain
  compatible.

### Key Entities

- **MissionPlanDocument**: Saveable operational wrapper around `MissionPlan`.
- **UavDataProductCatalogState**: Summary of named UAV data products.
- **VehicleParameterSnapshot**: Compact vehicle setup/capability view.
- **OperatorAuthorityLease**: Time-scoped authority token for monitor/control.

## Success Criteria

### Measurable Outcomes

- **SC-001**: The UAV shared-state unit test suite verifies all four new models
  independently.
- **SC-002**: Mission plan, catalog, parameter snapshot, and lease fields
  round-trip through `Fields` without losing required operational information.
- **SC-003**: The implementation requires no changes to NDNSF core wire names
  or UAV service names.
- **SC-004**: The core/app boundary document states which parts belong to UAV
  and which generic mechanisms remain core-owned.

## Assumptions

- This iteration is a state-contract and testable helper layer, not a full
  QGroundControl replacement.
- GTK UI panels and real MAVLink parameter download services will be wired in a
  later iteration using these models.
- The current `Fields` encoding remains the compatibility path for C++ UAV
  state exchange.
