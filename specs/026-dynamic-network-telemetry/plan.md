# Implementation Plan: Dynamic Network Telemetry

**Branch**: current worktree | **Date**: 2026-06-28 |
**Spec**: [spec.md](spec.md)

## Summary

Introduce a first-class NDNSF runtime telemetry surface. The implementation
should start with passive measurements already available in the service path:
ACK RTT and response latency on the user side, plus collaboration large-data
fetch timing on the provider side. These samples feed an EWMA telemetry store
that selection policies and DI planner exporters can query.

## Technical Context

**Language/Version**: C++17 runtime, Python 3.8 experiment tooling

**Primary Dependencies**: NDNSF dynamic runtime, ndn-cxx, existing MiniNDN
NativeTracer and YOLO experiment parsers

**Storage**: In-memory runtime telemetry store; optional JSON export from
experiment tooling

**Testing**: Boost unit tests, focused selection-policy tests, Python parser
syntax/fixture tests, MiniNDN smoke after runtime integration

**Target Platform**: Ubuntu/Linux development host

**Project Type**: C++ runtime library plus Python experiment tooling

**Performance Goals**: Passive telemetry should add negligible per-message
overhead and be optional for existing call sites

**Constraints**: Preserve existing ACK strategy APIs and behavior; do not modify
proposal slides; do not weaken security checks

## Constitution Check

- **Canonical Dynamic Runtime**: Pass. This extends the current dynamic
  `RequestService`/ACK selection surface.
- **Security Is Part Of The Data Path**: Pass. Telemetry records metadata only
  and does not bypass NAC-ABE, permissions, tokens, or signature validation.
- **CodeGraph First, Source Verified**: Pass. `ServiceUser`, `ServiceProvider`,
  ACK selection, and large-data fetch timing were inspected before design.
- **Spec-Driven Changes**: Pass. This feature has a dedicated spec/plan/tasks.
- **Verify With The Right Scope**: Pass. Start with unit tests and use MiniNDN
  after runtime integration.

## Architecture

### Runtime Model

Add a telemetry model in the NDNSF runtime:

```text
NetworkTelemetrySnapshot
  providerName
  serviceName
  peerName / edgeName
  sampleKind: ack-rtt | response-rtt | large-data-fetch | active-probe
  rttMs
  firstByteMs
  elapsedMs
  encodedBytes
  wireBytes
  goodputMbps
  receivedSegments
  timeoutCount
  nackCount
  sampleCount
  lastUpdatedMs
  confidence
```

Use an EWMA store keyed by:

```text
service path:   <provider, service>
dependency edge:<consumer-provider, producer-provider, key-scope>
```

### User-Side Control Telemetry

`ServiceUser` already tracks pending calls and ACK candidates. Add timestamps to
pending calls and update telemetry on:

- request publish -> ACK receive;
- selection publish -> response receive;
- response bytes received.

Attach the latest `<provider, service>` snapshot to `AckSelectionCandidate` so
custom selection policies can rank by observed RTT or confidence.

### Provider-Side Large-Data Telemetry

`ServiceProvider::fetchLarge` already has `CollaborationLargeFetchTiming` and
logs elapsed time, first segment, wire bytes, segment counts, timeouts, and
NACKs. Convert those completion events into telemetry samples and optionally
export them to logs/JSON for DI planner input.

### DI Planner Integration

The planner should accept a dynamic network profile that can override or annotate
static profile fields:

```json
{
  "edges": [
    {
      "consumerProvider": "/provider/merge",
      "producerProvider": "/provider/head0",
      "keyScope": "head0-to-merge",
      "rttMs": 18.4,
      "goodputMbps": 3.2,
      "confidence": 0.8,
      "sampleCount": 32
    }
  ]
}
```

Static topology defaults remain the fallback when dynamic confidence is low or
samples are stale.

## Project Structure

```text
specs/026-dynamic-network-telemetry/
├── spec.md
├── plan.md
└── tasks.md

ndn-service-framework/
├── NetworkTelemetry.hpp
├── NetworkTelemetry.cpp
├── ServiceUser.hpp/.cpp
└── ServiceProvider.hpp/.cpp

examples/python/NDNSF-DistributedInference/
└── native_di_tracer or yolo diagnostics telemetry exporters

tests/unit-tests/
└── network-telemetry.t.cpp
```

## Complexity Tracking

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| None | N/A | N/A |
