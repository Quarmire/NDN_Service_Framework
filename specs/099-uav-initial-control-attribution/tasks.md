# Tasks: UAV Initial Control Attribution

## Phase 1: Setup And Pre-Audit

- [x] T001 Record Spec 098 run 03/04 diagnosis in `specs/099-uav-initial-control-attribution/evidence/baseline.md`.
- [x] T002 Run strict pre-implementation audit and resolve blocking findings.

## Phase 2: User Story 1 - Trust Command Terminal State

**Goal**: Correct command time fields and observer termination.

**Independent Test**: Factory/observer tests fail before implementation and pass afterward.

- [x] T003 [P] [US1] Add Spec 098 run 03/04 attribution fixtures in `tests/python/test_ndnsf_uav_stream_control_isolation_campaign.py`.
- [x] T004 [US1] Implement baseline attribution parsing in `Experiments/NDNSF_UAV_Stream_Control_Isolation_Campaign.py`.
- [x] T005 [P] [US1] Add failing time-invariant tests in `tests/unit-tests/uav-protocol-state.t.cpp`.
- [x] T006 [US1] Add named pending/timeout factories in `NDNSF-UAV-APP/shared/UavProtocol.hpp` and `.cpp`.
- [x] T007 [US1] Replace ambiguous initializers in `NDNSF-UAV-APP/ground-station/GroundStationServiceContainer.inc.hpp`.
- [x] T008 [US1] Verify automation observes corrected terminal state without weakening stale-state rejection.

## Phase 3: User Story 2 - Attribute Initial Control Failures

**Goal**: Produce bounded sender-side telemetry/Arm attribution.

**Independent Test**: Fixtures distinguish every attribution category.

- [x] T009 [P] [US2] Add parser tests for request correlation, overlap, precedence, and sensitive-field exclusion.
- [x] T010 [US2] Complete per-run/aggregate attribution output in the campaign parser.
- [x] T011 [US2] Build Ground Station and run focused plus full regressions.

## Phase 4: User Story 3 - Frozen Diagnostic

**Goal**: Run and interpret one five-run 5% MiniNDN cell.

- [x] T012 [US3] Run once into `results/spec099-uav-initial-control-attribution-loss05-final`.
- [x] T013 [US3] Verify attribution, dispatch, terminal, and lifecycle invariants.
- [x] T014 [US3] Write ARS-compatible `evidence/final-validation.md`.

## Phase 5: Closure

- [x] T015 Run post-implementation audit and converge.
- [x] T016 Verify proposal/paper unchanged, diff check, CodeGraph and context sync.
- [x] T017 Update `.planning/STATE.md` with result and next decision.
- [x] T018 Commit verified Spec 099 and play completion bell.

## Dependencies

- T001–T002 gate implementation; T003→T004; T005→T006–T008;
  T009→T010; T011→T012→T013–T014; closure is sequential.

## Parallel Opportunities

- T003 and T005 touch independent test files.

## Implementation Strategy

Correct evidence first, classify boundaries, then run one diagnostic cell. A
flat or worse completion rate is valid because this is not a reliability treatment.
