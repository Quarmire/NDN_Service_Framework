# Implementation Plan: UAV Operational Layer

**Branch**: `069-uav-operational-layer` | **Date**: 2026-07-08 |
**Spec**: [spec.md](spec.md)

## Summary

Extend `NDNSF-UAV-APP` with a small operational layer that closes the highest
value gap against QGroundControl-like workflows without trying to clone QGC.
The layer adds persistent mission-plan state, data-product catalog state,
vehicle parameter/capability state, and operator authority leases. These are
typed UAV application models that can later be connected to GUI panels and
NDNSF services.

## Technical Context

**Language/Version**: C++17 for UAV shared models and Boost unit tests.

**Primary Dependencies**: Existing `NDNSF-UAV-APP/shared/UavProtocol.*`,
`ndn-service-framework` core envelopes, Boost unit-test target.

**Storage**: State round-trips through existing `Fields`.
`MissionPlanDocument` also has lightweight line-oriented file save/load helpers
for ground-station workflows. Repo-backed storage remains a later service
wiring step.

**Testing**: `./waf build --targets=unit-tests` and
`./build/unit-tests --run_test=UavProtocolState`.

**Target Platform**: Linux NDNSF-UAV app and MiniNDN/PX4-jMAVSim experiments.

**Project Type**: C++ library/application shared state extension.

**Performance Goals**: Negligible overhead; all helpers are local state
conversion and validation.

**Constraints**: Do not change existing UAV service names or NDNSF core wire
protocol. Keep MAVLink semantics in UAV app, not core.

**Scale/Scope**: One ground station, multiple drones, future multi-operator
support through operator leases.

## Constitution Check

- Canonical dynamic runtime: Pass. No generated service/stub path added.
- Security/data path: Pass. No auth bypass or NAC-ABE routing change.
- CodeGraph first: Pass. CodeGraph was used before edits.
- Spec-driven durable work: Pass. This spec/plan/tasks set records scope.
- Verify right scope: Pass. Focused C++ unit tests cover new state models; no
  network path changed in this iteration.

## Boundary Decision

NDNSF core continues to own service-neutral mechanisms: stream health,
operation status, data product references, provider capability hints, and
future generic coordination/lease envelopes.

NDNSF-UAV-APP owns this feature's semantics:

- mission plan document fields, geofence/rally interpretation, and mission
  upload policy;
- data-product classes such as recording, telemetry log, detection event, and
  mission log;
- MAVLink parameter/capability interpretation;
- operator authority scope names for UAV flight operations.

## Project Structure

```text
NDNSF-UAV-APP/shared/UavProtocol.hpp
NDNSF-UAV-APP/shared/UavProtocol.cpp
tests/unit-tests/uav-protocol-state.t.cpp
docs/ndnsf-core-app-boundary.md
specs/069-uav-operational-layer/
├── spec.md
├── plan.md
├── data-model.md
├── quickstart.md
├── contracts/
│   └── uav-operational-fields.md
└── tasks.md
```

**Structure Decision**: Keep the first iteration in the existing shared UAV
state layer so GUI, MiniNDN smoke tests, and future NDNSF service handlers can
consume the same contracts.

## Validation

```bash
./waf build --targets=unit-tests
./build/unit-tests --run_test=UavProtocolState
PYTHONPATH=.:pythonWrapper:NDNSF-DistributedInference:NDNSF-DistributedRepo/pythonWrapper \
  python3 tests/python/test_ndnsf_app_core_envelope_migration.py
git diff --check
```

## Next Wiring Step

After this state-contract slice, wire the models into:

1. ground-station mission save/load UI using the shared file helpers;
2. repo-backed catalog browsing for recordings/logs/detections;
3. MAVLink parameter fetch/cache service;
4. command execution checks that require a control lease when multi-operator
   mode is enabled.
