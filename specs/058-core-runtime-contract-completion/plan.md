# Implementation Plan: Core Runtime Contract Completion

**Branch**: `058-core-runtime-contract-completion` | **Date**: 2026-07-08 | **Spec**: [spec.md](spec.md)

**Input**: Feature specification from `/specs/058-core-runtime-contract-completion/spec.md`

## Summary

Complete the reusable NDNSF core/app boundary started in Spec 049. The feature
adds C++ helpers that match the Python core runtime envelopes, adds a Python
service-discovery snapshot facade over NDNSD health and provider capability
hints, documents exact-name large-data vs continuous streams, and verifies that
Repo, UAV, and DI keep domain semantics while exposing core-first evidence.

## Technical Context

**Language/Version**: C++17, Python 3.8+

**Primary Dependencies**: ndn-cxx, NDN-SVS/NDNSD, existing NDNSF Python wrapper

**Storage**: N/A for core contracts; existing app storage remains unchanged

**Testing**: Boost unit tests, Python unittest regressions

**Target Platform**: Ubuntu/Linux NDNSF development and MiniNDN environments

**Project Type**: C++/Python framework plus application packages

**Performance Goals**: No new hot-path serialization overhead beyond existing ACK payload helpers; selection/discovery helpers operate on in-memory snapshots

**Constraints**: Preserve NAC-ABE, token, permission, Targeted, stream, and large-data semantics; preserve legacy app payload fields as fallback

**Scale/Scope**: Core helper and bridge completion for Repo, UAV, and DI; no wholesale protocol rewrite

## Constitution Check

- Canonical dynamic runtime: pass; no generated static stubs are introduced.
- Security is part of data path: pass; no NAC-ABE, token, or certificate bypass.
- CodeGraph first: pass; C++ ServiceProvider and app bridge surfaces were explored before edits.
- Spec-driven changes: pass; this feature owns the durable boundary work.
- Verify with right scope: pass; focused C++/Python regressions are sufficient because this feature changes contracts and parsing, not a MiniNDN performance path.

## Project Structure

### Documentation (this feature)

```text
specs/058-core-runtime-contract-completion/
├── spec.md
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/
│   └── runtime-envelope-contract.md
└── tasks.md
```

### Source Code

```text
ndn-service-framework/
├── ServiceProvider.hpp
├── ServiceProvider.cpp
└── Stream.hpp / Stream.cpp

pythonWrapper/ndnsf/
├── runtime_telemetry.py
├── ndnsd_health.py
└── service_discovery.py

NDNSF-DistributedRepo/
NDNSF-UAV-APP/
NDNSF-DistributedInference/

tests/
├── unit-tests/generic-admission-lease.t.cpp
└── python/
    ├── test_ndnsf_core_boundary_envelopes.py
    ├── test_ndnsf_app_core_envelope_migration.py
    └── test_ndnsf_core_service_discovery.py
```

**Structure Decision**: Extend existing core files and focused regression tests.
Do not create a new framework package or move app domain modules.

## Complexity Tracking

No constitution violations.

