# Tasks: UAV Control State Convergence

## Phase 1: Setup And Baseline

- [x] T001 Confirm Spec 097 baseline integrity and record Arm-to-telemetry-to-Takeoff timing in `specs/098-uav-control-state-convergence/evidence/baseline.md`.
- [x] T002 Run strict pre-implementation Spec Kit audit against `specs/098-uav-control-state-convergence/` and resolve every blocking finding.

## Phase 2: User Story 1 - Explain The Blocked Transition

**Goal**: Correlate accepted Arm, armed telemetry, and Takeoff decision without conflating transport and local safety outcomes.

**Independent Test**: Reparse the five Spec 097 runs and reproduce the 1/5 correlation between armed telemetry before Takeoff and Takeoff dispatch.

- [x] T003 [P] [US1] Add parser fixtures for Arm response, telemetry convergence, and Takeoff decision ordering in `tests/python/test_ndnsf_uav_stream_control_isolation_campaign.py`.
- [x] T004 [US1] Extend per-run state-convergence correlation in `Experiments/NDNSF_UAV_Stream_Control_Isolation_Campaign.py`.
- [x] T005 [US1] Reparse the immutable Spec 097 5% baseline and write correlation evidence to `specs/098-uav-control-state-convergence/evidence/baseline.md`.

## Phase 3: User Story 2 - Sequence Commands By Observed State

**Goal**: Advance automated flight commands only after observed prerequisites, with one attempt per command.

**Independent Test**: Controlled delayed/missing-state tests prove dispatch-once and bounded expiry behavior.

- [x] T006 [P] [US2] Add unit tests for monotonic automation phases, dispatch-once, convergence satisfaction, expiry, and shutdown in `tests/unit-tests/uav-protocol-state.t.cpp`.
- [x] T007 [P] [US2] Add Python source/contract tests for `UAV_AUTO_CONTROL_PHASE`, no command retry, and sensitive-field exclusion in `tests/python/test_ndnsf_uav_stream_control_isolation_campaign.py`.
- [x] T008 [US2] Implement the bounded state-driven auto-MAVLink sequence and diagnostics in `NDNSF-UAV-APP/ground-station/GroundStationWindow.inc.hpp`.
- [x] T009 [US2] Expose only the minimal thread-safe command/telemetry readiness observations needed by the sequence in `NDNSF-UAV-APP/ground-station/GroundStationServiceContainer.inc.hpp` and shared UAV state files if required.
- [x] T010 [US2] Extend parser aggregation and unterminated-wait rejection in `Experiments/NDNSF_UAV_Stream_Control_Isolation_Campaign.py`.

## Phase 4: User Story 3 - Measure The Treatment

**Goal**: Execute and report one frozen five-run 5% treatment without retries or overclaiming.

**Independent Test**: Five retained runs have terminal command/wait stages and zero lifecycle aborts.

- [x] T011 [US3] Build `UavGroundStationApp` and run focused C++ and Python regressions from `specs/098-uav-control-state-convergence/quickstart.md`.
- [x] T012 [US3] Run the five-repetition 5% MiniNDN treatment once into `results/spec098-uav-control-state-loss05-current-final`.
- [x] T013 [US3] Validate lifecycle markers, command attempts, convergence terminals, and no-retry invariants from the treatment artifacts.
- [x] T014 [US3] Compute exact binomial intervals and bounded baseline/treatment comparison in `specs/098-uav-control-state-convergence/evidence/final-validation.md`.

## Phase 5: Closure

- [x] T015 Run post-implementation code-aware audit and converge for `specs/098-uav-control-state-convergence/`.
- [x] T016 Verify proposal/paper paths are unchanged, run `git diff --check`, and synchronize CodeGraph and agent context.
- [x] T017 Update `.planning/STATE.md` with the measured result and next research boundary.
- [x] T018 Commit the verified Spec 098 change set and play the completion bell.

## Dependencies

- T001-T002 gate all implementation.
- T003 precedes T004-T005.
- T006-T007 precede T008-T010.
- T011 gates T012; T012 gates T013-T014.
- T015-T018 run sequentially after all evidence tasks.

## Parallel Opportunities

- T003 and T006-T007 touch independent test files and may be prepared in parallel after the audit.
- Documentation evidence can be updated independently only after its producing command completes.

## Implementation Strategy

The MVP is US1: establish the causal correlation. US2 then changes only the
automation seam. US3 accepts or rejects that treatment with one frozen MiniNDN
cell. A flat or negative treatment is a valid completion outcome.
