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

## Dependencies

- T004-T005 block all user story implementation.
- US1-US4 are independently testable after T004-T005.
- Validation tasks must run after all user story tests are added.

## Next Implementation Stage

Future tasks should wire these models into:

- repo-backed catalog browsing;
- MAVLink parameter fetch/cache service;
- optional command lease checks in the MAVLink execute and mission assign paths.
