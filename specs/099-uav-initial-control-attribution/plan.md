# Implementation Plan: UAV Initial Control Attribution

**Branch**: `Experimental` | **Date**: 2026-07-11 | **Spec**: [spec.md](spec.md)

## Summary

Replace ambiguous positional pending/timeout `FlightCommandState` construction
with named factories, extend the existing campaign parser to correlate telemetry
and Arm Targeted phases with automation outcomes, then run one frozen five-run
5% MiniNDN diagnostic. No timeout, retry, polling, security, or safety changes.

## Technical Context

**Language/Version**: C++17 and Python 3.8+

**Primary Dependencies**: ndn-cxx, GTK/Glib, NDNSF Targeted runtime, Python
standard library

**Storage**: Markdown/JSON/CSV evidence plus local `results/`

**Testing**: Boost.Test, Python `unittest`, MiniNDN

**Target Platform**: Linux MiniNDN with PX4 SITL UAV example

**Project Type**: C++ desktop/runtime example plus Python campaign parser

**Performance Goals**: preserve 60-second runs and existing timeout budgets;
linear-time log parsing

**Constraints**: no retry, timeout tuning, host-NFD final validation, sensitive
logging, safety bypass, or replacement runs

**Scale/Scope**: shared command state, Ground Station construction, one parser,
focused tests, five runs

## Constitution Check

- Canonical Dynamic Runtime: PASS; existing unified Targeted services only.
- Security In Data Path: PASS; no wire/security behavior changes.
- CodeGraph First: PASS; command/telemetry/observer paths traced.
- Spec-Driven Durable Work: PASS.
- Right-Scope Verification: PASS; unit/parser tests plus MiniNDN.
- Post-design recheck: PASS; shared invariants stay shared, evidence stays in campaign.

## Design

1. Add named pending/timeout factories assigning all time fields explicitly.
2. Use them at the Ground Station MAVLink seam without behavioral changes.
3. Parse Targeted records into request-ID keyed attempts.
4. Derive bounded initial telemetry/Arm attribution.
5. Freeze one diagnostic cell after tests pass.

## Project Structure

```text
NDNSF-UAV-APP/shared/UavProtocol.{hpp,cpp}
NDNSF-UAV-APP/ground-station/GroundStationServiceContainer.inc.hpp
Experiments/NDNSF_UAV_Stream_Control_Isolation_Campaign.py
tests/unit-tests/uav-protocol-state.t.cpp
tests/python/test_ndnsf_uav_stream_control_isolation_campaign.py
specs/099-uav-initial-control-attribution/
```

**Structure Decision**: state invariants live in the shared UAV model;
Ground Station consumes them; attribution remains evidence-only.

## Complexity Tracking

No constitution violation or compatibility layer.

## Rollback

Source revert only; no wire, configuration, or persisted-data migration.
