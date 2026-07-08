# Tasks: UAV Operational Layer

**Input**: Design documents from `specs/069-uav-operational-layer/`

**Tests**: Required for each user story because this feature defines reusable
state contracts.

## Phase 1: Setup

- [x] T001 Create Spec069 artifacts in `specs/069-uav-operational-layer/`.
- [x] T002 Review current UAV state models and core/app boundary in
  `NDNSF-UAV-APP/shared/UavProtocol.*` and `docs/ndnsf-core-app-boundary.md`.
- [x] T003 Use CodeGraph and DeepSeek second-pass planning before edits.

## Phase 2: Foundational

- [x] T004 Add operational layer state declarations in
  `NDNSF-UAV-APP/shared/UavProtocol.hpp`.
- [x] T005 Add operational layer serialization, status, and validation helpers
  in `NDNSF-UAV-APP/shared/UavProtocol.cpp`.

## Phase 3: User Story 1 - Persistent Mission Operations (P1)

- [x] T006 [US1] Add `MissionPlanDocument` state and field contract in
  `NDNSF-UAV-APP/shared/UavProtocol.*`.
- [x] T007 [US1] Add mission-plan document round-trip test in
  `tests/unit-tests/uav-protocol-state.t.cpp`.

## Phase 4: User Story 2 - Operational Data Product Catalog (P2)

- [x] T008 [US2] Add `UavDataProductCatalogState` and recording bridge in
  `NDNSF-UAV-APP/shared/UavProtocol.*`.
- [x] T009 [US2] Add catalog summary round-trip test in
  `tests/unit-tests/uav-protocol-state.t.cpp`.

## Phase 5: User Story 3 - Vehicle Capability and Parameter View (P3)

- [x] T010 [US3] Add `VehicleParameterSnapshot` full and compact field views in
  `NDNSF-UAV-APP/shared/UavProtocol.*`.
- [x] T011 [US3] Add parameter/capability view tests in
  `tests/unit-tests/uav-protocol-state.t.cpp`.

## Phase 6: User Story 4 - Operator Authority Lease (P4)

- [x] T012 [US4] Add `OperatorAuthorityLease` validation helper in
  `NDNSF-UAV-APP/shared/UavProtocol.*`.
- [x] T013 [US4] Add lease allow/reject tests in
  `tests/unit-tests/uav-protocol-state.t.cpp`.

## Phase 7: Polish and Validation

- [x] T014 Add field contract and quickstart docs in
  `specs/069-uav-operational-layer/`.
- [x] T015 Update core/app boundary documentation in
  `docs/ndnsf-core-app-boundary.md`.
- [x] T016 Build `unit-tests`.
- [x] T017 Run `UavProtocolState`.
- [x] T018 Run Python app-core envelope migration regression.
- [x] T019 Run `git diff --check`.

## Phase 8: Mission Plan File Persistence

- [x] T020 [US1] Add `MissionPlanDocument` file save/load helpers in
  `NDNSF-UAV-APP/shared/UavProtocol.*`.
- [x] T021 [US1] Extend `UavProtocolState` mission-plan test with a temporary
  file save/load round-trip.
- [x] T022 Update Spec069 plan, quickstart, and field contract to document the
  file format.
- [x] T023 Re-run build, focused C++ unit test, Python envelope regression, and
  whitespace check after the file-persistence slice.

## Phase 9: Ground-Station Save/Load Wiring

- [x] T024 [US1] Add ground-station runtime methods for saving a current or
  preview mission plan and loading a saved mission plan document.
- [x] T025 [US1] Add `mission-plan-file` config/CLI support.
- [x] T026 [US1] Add GUI path entry plus `Save Plan` and `Load Plan` buttons to
  the Map / Mission workflow.
- [x] T027 [US1] Update functionality state/test expectations so mission files
  become available when a mission plan exists.
- [x] T028 Re-run build, focused C++ unit test, Python envelope regression, and
  whitespace check after ground-station wiring.

## Phase 10: Upload Loaded Mission Plan

- [x] T029 [US1] Add a ground-station runtime upload path for an existing
  `MissionPlan`, preserving per-drone mission parts.
- [x] T030 [US1] Change `Upload Mission` to prefer the current loaded or
  preview mission plan before falling back to generated patrol inputs.
- [x] T031 [US1] Update quickstart docs to explain loaded-plan upload behavior.
- [x] T032 Re-run build, focused C++ unit test, Python envelope regression, and
  whitespace check after loaded-plan upload wiring.

## Phase 11: Loaded Mission Plan Smoke Test

- [x] T033 [US1] Add a ground-station auto test that saves, loads, and uploads
  a generated `MissionPlanDocument`.
- [x] T034 [US1] Add MiniNDN harness support for
  `--auto-loaded-mission-plan-test`.
- [x] T035 [US1] Document the loaded mission plan MiniNDN smoke command.
- [x] T036 Run build, focused C++ unit test, Python envelope regression,
  whitespace check, and loaded mission MiniNDN smoke when practical.

## Phase 12: Repo-Backed Data Product Catalog Browsing

- [x] T037 [US2] Add a drone repo catalog service that summarizes the
  recording repo through `UavDataProductCatalogState`.
- [x] T038 [US2] Add ground-station runtime and GUI controls for browsing the
  selected drone's repo catalog.
- [x] T039 [US2] Add MiniNDN harness support for
  `--auto-repo-catalog-browse-test`.
- [x] T040 [US2] Extend catalog state tests so repo chunks collapse into
  object-level recording products.
- [x] T041 Document the repo catalog MiniNDN smoke command.
- [x] T042 Run build, focused C++ unit test, Python envelope regression,
  whitespace check, and repo catalog MiniNDN smoke when practical.

## Phase 13: MAVLink Parameter Cache Service

- [x] T043 [US3] Add per-drone `/UAV/MAVLink/Parameters` service suffix,
  runtime config, and controller policy entries.
- [x] T044 [US3] Add mock and UDP flight-controller parameter snapshots using
  the existing `VehicleParameterSnapshot` state contract.
- [x] T045 [US3] Add ground-station request/cache methods plus a
  vehicle-parameter inspector and `Fetch Params` GUI button.
- [x] T046 [US3] Add MiniNDN harness support for `--auto-parameter-cache-test`.
- [x] T047 [US3] Document the parameter-cache smoke command and validation
  marker.
- [x] T048 Run build, focused C++ unit test, Python envelope regression,
  whitespace check, and parameter-cache MiniNDN smoke.

## Phase 14: Operator Authority Lease Runtime Gate

- [x] T049 [US4] Add a ground-station runtime active authority lease with a
  default local `all/control` lease for existing demos.
- [x] T050 [US4] Gate direct and sync MAVLink command paths with
  `OperatorAuthorityLease::allowsCommand` before readiness, in-flight, or
  network dispatch work.
- [x] T051 [US4] Gate mission assignment entry points so monitor-only or
  expired leases fast-fail before NDNSF mission requests are published.
- [x] T052 [US4] Add a headless runtime smoke path for monitor-only and
  expired-lease rejection.
- [x] T053 [US4] Add MiniNDN harness support for
  `--auto-authority-lease-test`.
- [x] T054 Run build, focused C++ unit test, Python envelope regression,
  whitespace check, and authority-lease MiniNDN smoke.

## Dependencies

- T004-T005 block all user story implementation.
- US1-US4 are independently testable after T004-T005.
- Validation tasks must run after all user story tests are added.

## Next Implementation Stage

Future tasks should connect operator leases to a real multi-operator lease
issuer or controller policy path instead of the current local default lease.
