# Tasks: UAV Armed Visibility Boundary

## Phase 1: Baseline And Audit
- [x] T001 Record run 02/05 baseline in `evidence/baseline.md`.
- [x] T002 Run strict pre-implementation audit.

## Phase 2: Cross-Log Attribution
- [x] T003 [P] [US1] Add drone/GS timeline fixtures in Python tests.
- [x] T004 [US1] Parse armed visibility in campaign script.
- [x] T005 [US1] Reparse Spec 099 baseline and verify both classifications.

## Phase 3: Final Cached Observation
- [x] T006 [P] [US2] Add source/contract tests for final cached-only evaluation.
- [x] T007 [US2] Extract shared predicate and add final read in GroundStation window.
- [x] T008 [US2] Build and run focused/full regressions.

## Phase 4: Frozen Treatment
- [x] T009 [US3] Run one five-run 5% MiniNDN cell.
- [x] T010 [US3] Verify attribution/lifecycle/single-attempt invariants.
- [x] T011 [US3] Write ARS final validation evidence.

## Phase 5: Closure
- [x] T012 Run post-implementation audit and converge.
- [x] T013 Sync CodeGraph/context/GSD and verify proposal/paper unchanged.
- [x] T014 Commit verified Spec 100 and play bell.

## Dependencies
T001-T002 gate work; T003→T005; T006→T008; T008→T009→T011; closure sequential.
