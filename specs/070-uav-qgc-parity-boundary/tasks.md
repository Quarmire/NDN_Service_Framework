# Tasks: UAV QGC-Parity Boundary Slice

## Phase 1: Design and Boundary

- [x] T001 Review existing UAV operational state contracts and QGC-parity gaps.
- [x] T002 Use CodeGraph before broad code edits.
- [x] T003 Use DeepSeek as a second-pass checklist and keep Codex as final
  architecture authority.
- [x] T004 Update `docs/ndnsf-core-app-boundary.md` with the QGC-parity split.

## Phase 2: Protocol Contracts

- [x] T005 Add `VehicleParameterEditRequest` and validation helper.
- [x] T006 Add `VehicleParameterEditResult` and success helper.
- [x] T007 Add `PreflightCheckItem` and blocking-failure helper.
- [x] T008 Add `MavlinkMessageSummary` and stale/active helper.
- [x] T009 Add `UavAnalyzeSnapshot` with flattened message summaries.

## Phase 3: Tests

- [x] T010 Add parameter-edit request/result round-trip tests.
- [x] T011 Add preflight checklist round-trip and blocking tests.
- [x] T012 Add MAVLink analyze snapshot round-trip and active-message tests.

## Phase 4: Validation

- [x] T013 Build unit tests.
- [x] T014 Run focused `UavProtocolState` tests.
- [x] T015 Run Python app-core envelope migration regression.
- [x] T016 Run `git diff --check`.
- [x] T017 Commit the completed slice.

## Phase 5: Parameter Edit Runtime Slice

- [x] T018 Add `/UAV/MAVLink/ParameterEdit` service suffix and config plumbing.
- [x] T019 Add mock flight-controller parameter write/verify support.
- [x] T020 Register drone provider parameter-edit service.
- [x] T021 Add ground-station parameter-edit async/sync helpers.
- [x] T022 Add headless `--auto-parameter-edit-test` flow.
- [x] T023 Add MiniNDN harness flag and success marker.
- [x] T024 Build UAV apps and unit tests.
- [x] T025 Run focused C++ protocol tests.
- [x] T026 Run Python envelope regression.
- [x] T027 Run parameter-edit MiniNDN smoke when practical.
- [x] T028 Run `git diff --check`.
- [x] T029 Commit the runtime slice.

## Phase 6: Preflight Checklist Runtime Slice

- [x] T030 Add `/UAV/Preflight/Checklist` service suffix and config plumbing.
- [x] T031 Add drone-side checklist generation from telemetry/readiness/camera state.
- [x] T032 Register drone provider preflight checklist service.
- [x] T033 Add ground-station preflight request/cache/sync helpers.
- [x] T034 Add headless `--auto-preflight-checklist-test` flow.
- [x] T035 Add MiniNDN harness flag and success marker.
- [x] T036 Build UAV apps and unit tests.
- [x] T037 Run focused C++ protocol tests.
- [x] T038 Run Python envelope regression.
- [x] T039 Run preflight checklist MiniNDN smoke.
- [x] T040 Run `git diff --check`.
- [x] T041 Commit the preflight runtime slice.

## Phase 7: MAVLink Analyze Snapshot Runtime Slice

- [x] T042 Add `/UAV/MAVLink/AnalyzeSnapshot` service suffix and config plumbing.
- [x] T043 Add drone-side analyze snapshot generation from telemetry, mission, and video state.
- [x] T044 Register drone provider analyze snapshot service.
- [x] T045 Add ground-station analyze snapshot request/cache/sync helpers.
- [x] T046 Add headless `--auto-analyze-snapshot-test` flow.
- [x] T047 Add MiniNDN harness flag and success marker.
- [x] T048 Build UAV apps and unit tests.
- [x] T049 Run focused C++ protocol tests.
- [x] T050 Run Python envelope regression.
- [x] T051 Run analyze snapshot MiniNDN smoke.
- [x] T052 Run `git diff --check`.
- [x] T053 Commit the analyze snapshot runtime slice.

## Phase 8: Operator Dashboard Snapshot Runtime Slice

- [x] T054 Add `UavOperatorDashboardSnapshot` protocol contract.
- [x] T055 Add ground-station dashboard aggregation from telemetry, parameters, preflight, analyze, and action gates.
- [x] T056 Add headless `--auto-dashboard-snapshot-test` flow.
- [x] T057 Add MiniNDN harness flag and success marker.
- [x] T058 Build UAV apps and unit tests.
- [x] T059 Run focused C++ protocol tests.
- [x] T060 Run Python envelope regression.
- [x] T061 Run dashboard snapshot MiniNDN smoke.
- [x] T062 Run `git diff --check`.
- [x] T063 Commit the dashboard snapshot runtime slice.
