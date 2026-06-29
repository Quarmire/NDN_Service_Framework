# Feature Specification: Dynamic Network Telemetry

**Status**: Accepted
**Created**: 2026-06-28

## User Story

As an NDNSF application or DI planner, I want runtime RTT and bandwidth/goodput
statistics so custom provider-selection policies and collaboration layout
decisions can use observed network conditions instead of only static profiles.

## Problem

NDNSF currently has several useful but separate signals:

- ACK arrival timing in `ServiceUser`;
- provider ACK payload metadata;
- collaboration large-data fetch timing in `ServiceProvider`;
- experiment-side ndnping/NFD monitor diagnostics;
- static planner network profiles.

These are not yet exposed as one runtime telemetry surface. As a result, custom
selection strategies and DI collaboration planning cannot directly ask: which
provider has lower recent service-path RTT, which dependency edge has enough
observed goodput, or which measurements are stale.

## Requirements

- **FR-001**: NDNSF must define a runtime `NetworkTelemetrySnapshot` model with
  observed RTT, first-byte latency, goodput/bandwidth, bytes, segment counts,
  timeout/NACK counts, sample count, timestamp, and confidence/staleness fields.
- **FR-002**: NDNSF must collect user-side service control telemetry:
  request-to-ACK RTT, selection-to-response latency, response payload bytes, and
  provider/service identity.
- **FR-003**: NDNSF must collect collaboration large-data telemetry:
  producer/consumer provider, key scope, Data name, elapsed time, first-segment
  time, received wire bytes, encoded bytes, segment counts, timeout/NACK counts,
  and derived goodput.
- **FR-004**: Selection policies must be able to receive a telemetry snapshot
  alongside each `AckSelectionCandidate` without breaking existing strategies.
- **FR-005**: NDNSF-DI planner/campaign tools must be able to export telemetry
  as a dynamic network profile that can override or annotate static RTT and
  bandwidth estimates.
- **FR-006**: Telemetry must be optional and low overhead by default. Expensive
  active probes are diagnostic tools, not the default runtime path.
- **FR-007**: Telemetry must not expose payload contents or weaken NAC-ABE,
  token, permission, or signature checks.

## Success Criteria

- **SC-001**: Unit tests can update and query an EWMA telemetry store by
  provider/service and by dependency edge.
- **SC-002**: Existing ACK selection tests continue to pass without requiring
  telemetry.
- **SC-003**: A custom selection policy can prefer a lower-RTT candidate using
  only candidate telemetry.
- **SC-004**: A DI experiment can write a dynamic network profile JSON with
  observed RTT/goodput fields from full-network logs.
- **SC-005**: Documentation clearly distinguishes observed service-path
  goodput from physical link bandwidth.

## Scope

In scope:

- Runtime telemetry data model and EWMA store.
- Passive telemetry from ACK/response and collaboration large-data fetches.
- Selection-policy read access to telemetry.
- DI export path from observed telemetry to planner network profile.

Out of scope:

- Physical-layer bandwidth measurement.
- Mandatory active probing for every request.
- Changing NDN-SVS reliability behavior.
- Proposal slides.

## Design Notes

Terminology:

- **RTT** means observed service-path round trip, not IP ping RTT.
- **Bandwidth** should be reported as observed goodput unless an active probe
  explicitly measures a synthetic path.
- **Confidence** should increase with sample count and decrease with staleness,
  timeout/NACK ratio, or high variance.

Initial passive sources:

- User request publication time to ACK receive time.
- Selection publication time to final response receive time.
- Collaboration large-data fetch complete events already logged by
  `NDNSF_COLLAB_LARGE_FETCH_TIMING`.

Future active sources:

- Low-rate ndnping-style probes for idle provider prefixes.
- Optional dependency-edge probes from consumer provider to producer provider.
