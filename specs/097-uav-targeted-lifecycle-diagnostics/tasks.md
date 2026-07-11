# Tasks: UAV Targeted Lifecycle Diagnostics

## Phase 1: Reproduce And Audit

- [x] T001 [US1] Preserve Spec 096 abort logs and establish the MiniNDN control-only repro signal.
- [x] T002 [US1] Trace all Ground Station thread creation/join paths and rank falsifiable root-cause hypotheses.
- [x] T003 [US2] Inventory Targeted and UAV command stages, callbacks, sensitive fields, and current observability gaps.
- [x] T004 [US3] Freeze the 0%/5% five-run matrix, no-retry rule, and acceptance contract.
- [x] T005 [US1] Run strict pre-implementation Spec Kit audit and resolve blockers.

## Phase 2: Lifecycle And Diagnostics

- [x] T006 [US1] Reorder Ground Station shutdown to quiesce producer threads before joining workers.
- [x] T007 [US2] Add payload-free generic Targeted phase diagnostics with request correlation and elapsed time.
- [x] T008 [US2] Add UAV command attempt/block/response/timeout diagnostics with reason and elapsed time.
- [x] T009 [US3] Extend isolation parsing/aggregation for lifecycle abort and command-stage evidence.
- [x] T010 [US1] Add focused regression tests and sensitive-data negative scans.

## Phase 3: Verification

- [x] T011 [US1] Build affected UAV targets and run full C++ unit tests.
- [x] T012 [US3] Run focused and full Python regression tests.
- [ ] T013 [US3] Execute five 0% control-only MiniNDN runs once without retries.
- [ ] T014 [US3] Execute five 5% control-only MiniNDN runs once without retries.
- [ ] T015 [US3] Validate zero abort markers and classify every command outcome by stage.

## Phase 4: Closure

- [ ] T016 [US3] Write root-cause, implementation, reproduction, result, and residual-risk evidence.
- [ ] T017 [US3] Run post-implementation audit/converge and complete appended tasks.
- [ ] T018 [US3] Update GSD/agent context/CodeGraph, verify proposal scope, commit, and close.
