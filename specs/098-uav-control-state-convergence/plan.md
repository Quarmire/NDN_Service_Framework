# Implementation Plan: UAV Control State Convergence

**Branch**: `Experimental` | **Date**: 2026-07-11 | **Spec**: [spec.md](spec.md)

**Input**: Feature specification from
`specs/098-uav-control-state-convergence/spec.md`

## Summary

Replace the Ground Station's fixed-clock auto-MAVLink sequence with a bounded,
observed-state sequence. Each flight command remains single-attempt and still
passes through the existing production lease and safety gates. Add
machine-readable automation convergence events, parse them into the existing
control-only campaign, and compare one five-run 5% treatment against the
immutable Spec 097 baseline.

## Technical Context

**Language/Version**: C++17 and Python 3.8+

**Primary Dependencies**: ndn-cxx/Boost.Asio, GTKmm/Glib, MiniNDN, existing
NDNSF Targeted runtime

**Storage**: Local experiment JSON/CSV/log artifacts only; no persistence or
wire-format changes

**Testing**: Boost.Test C++ unit tests, Python unittest, MiniNDN control-only
campaign

**Target Platform**: Linux desktop Ground Station and MiniNDN namespaces

**Project Type**: C++ framework/application with Python experiment harness

**Performance Goals**: Preserve one flight-command attempt per step; bound each
state convergence wait; keep the control-only run within its existing outer
timeout

**Constraints**: No command retry, timeout tuning, safety bypass, Targeted wire
change, security change, host-NFD acceptance, proposal, or paper edits

**Scale/Scope**: One drone, four-command automated sequence, preserved five-run
baseline, one five-run 5% treatment

## Constitution Check

- **I Canonical Dynamic Runtime — PASS**: existing Targeted APIs and unified
  service names remain unchanged.
- **II Security Is Part Of The Data Path — PASS**: every command uses the
  current permission, token, replay, and provider checks.
- **III CodeGraph First — PASS**: CodeGraph and source/log verification traced
  the command response, telemetry cache, and Takeoff safety gate.
- **IV Spec-Driven Changes — PASS**: Spec 098 owns requirements, design,
  tasks, audit, and evidence.
- **V Verify With The Right Scope — PASS**: unit, parser, and MiniNDN evidence
  cover the state-machine and network risks.

Post-design re-check: **PASS**. The plan changes only test automation sequencing
and observability; no protocol, authority, or persistent state is introduced.

## Design

1. Keep the application-owned auto-MAVLink worker and GTK-thread command
   dispatch introduced by Spec 097.
2. Add a bounded sequence helper that observes command terminal state and
   telemetry-derived flight readiness before advancing.
3. Poll telemetry through the existing authenticated Targeted telemetry path;
   telemetry polling may repeat, but each flight command is posted once.
4. Emit `UAV_AUTO_CONTROL_PHASE` events for wait begin, satisfied, expired,
   command dispatch, and sequence terminal states.
5. Treat expiry, blocked, busy, rejected, or timeout as terminal measured
   outcomes. Never auto-retry a flight command.
6. Extend the campaign parser and aggregate to retain convergence stages and to
   reject unterminated waits.
7. Freeze one 5% five-run treatment; retain all outcomes and report exact
   binomial intervals without a general reliability claim.

## Project Structure

### Documentation

```text
specs/098-uav-control-state-convergence/
├── spec.md
├── plan.md
├── research.md
├── experiment-plan.md
├── data-model.md
├── quickstart.md
├── contracts/automation-events.md
├── checklists/requirements.md
└── tasks.md
```

### Source And Tests

```text
NDNSF-UAV-APP/ground-station/
├── GroundStationWindow.inc.hpp
└── GroundStationServiceContainer.inc.hpp

NDNSF-UAV-APP/shared/
├── UavProtocol.hpp
└── UavProtocol.cpp

Experiments/
└── NDNSF_UAV_Stream_Control_Isolation_Campaign.py

tests/
├── python/test_ndnsf_uav_stream_control_isolation_campaign.py
└── unit-tests/uav-protocol-state.t.cpp
```

**Structure Decision**: Keep sequencing in the Ground Station application,
reuse the runtime's existing command/telemetry snapshots, and keep evidence
parsing in the existing control-only campaign. No new framework abstraction is
needed.

## Rollback

Revert the state-driven automation helper, event parser fields, and focused
tests. There is no protocol, persistence, migration, or mixed-version state to
undo. The immutable Spec 097 baseline remains available.
