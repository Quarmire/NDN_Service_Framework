# Implementation Plan: UAV QGC-Parity Boundary Slice

## Technical Approach

Use the existing `NDNSF-UAV-APP/shared/UavProtocol.*` field-map contract style.
Do not introduce protobuf or a new wire format in this slice. The new contracts
are application payloads carried over existing NDNSF Request/Response and status
services.

## What Goes In NDNSF Core

- Generic `ServiceOperationStatus` and data product references.
- Generic provider capability hints and rejection reasons.
- Generic stream health and stream chunk metadata.
- Generic provider-pair telemetry.

## What Stays In NDNSF-UAV-APP

- MAVLink target system/component, parameter ids, parameter value type, and
  parameter-name length validation.
- Preflight labels, categories, blocking policy, and safety meaning.
- MAVLink message ids, rates, system/component ids, and Analyze panel state.
- QGC-like GUI workflows and flight-controller translation.

## Implementation Slices

1. Document the boundary in `docs/ndnsf-core-app-boundary.md`.
2. Add UAV protocol declarations.
3. Add UAV protocol serialization, status, and validation helpers.
4. Add focused round-trip tests.
5. Run build, unit tests, Python envelope regression, and whitespace checks.

## Follow-Up Runtime Work

- Wire `VehicleParameterEditRequest/Result` to real MAVLink `PARAM_SET` and
  verification reads.
- Add a GUI preflight panel that consumes `PreflightCheckItem`.
- Add a MAVLink inspector/analyze panel that consumes `UavAnalyzeSnapshot`.
- Add MiniNDN smoke tests for parameter edit and analyze snapshot services.

