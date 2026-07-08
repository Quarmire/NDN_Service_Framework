# Data Model: UAV Operational Layer

## MissionPlanDocument

Wraps the existing `MissionPlan` so it can be saved, loaded, displayed, and
later expanded with QGroundControl-like plan elements.

Fields:

- `schema`: current value `ndnsf-uav-mission-plan-v2`
- `planId`: stable local identifier
- `displayName`: operator-facing name
- `operatorId`: creating or owning operator
- `createdMs`, `updatedMs`: timestamps
- `plan`: existing `MissionPlan`
- `geofence`: optional polygon points
- `rallyPoints`: optional rally/return points
- `metadata`: opaque UAV-app metadata

Rules:

- Saveable requires a non-empty id, task id, and at least one mission part.
- Geofence and rally are UAV semantics; core should not interpret them.

## UavDataProductCatalogState

Summarizes named UAV products for browsing and post-mission analysis.

Fields:

- product counts for camera recordings, telemetry logs, detection products,
  and mission logs
- `totalBytes`
- latest product type, object prefix, and mission id
- `updatedMs`

Rules:

- Queryable when total product count is greater than zero.
- Existing `RecordingDataProductState` can create a one-recording catalog
  summary.

## VehicleParameterSnapshot

Compact vehicle configuration/capability view.

Fields:

- `droneId`
- `source`
- `firmware`
- `vehicleType`
- `flightModes`
- `parameterCount`
- `completePercent`
- `updatedMs`
- optional raw parameter map

Rules:

- Usable when it has either raw parameters or high-level vehicle metadata.
- MAVLink parameter ids and values remain UAV-app semantics.

## OperatorAuthorityLease

Time-scoped control authority for multi-operator deployments.

Fields:

- `leaseId`
- `operatorId`
- `droneId` or `all`
- `scope`: `monitor`, `control`, `mission`, or `admin`
- `issuedMs`, `expiresMs`
- `revoked`

Rules:

- Revoked or expired leases reject commands.
- Monitor scope allows only status/telemetry style actions.
- Control, mission, and admin scopes allow UAV commands for the matching drone.
