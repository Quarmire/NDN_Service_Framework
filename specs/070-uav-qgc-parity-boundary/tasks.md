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
