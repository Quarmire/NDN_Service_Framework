# Tasks: Native DI User Driver Correctness

## Phase 1: Test-First Contracts

- [x] T001 [US1] Add a failing lifecycle test proving start-before-work and stop-on-success/failure for the base scope-key publisher.
- [x] T002 [US1] Add failing deterministic tests for worker `scheduleSlipMs`, process-pool measurement metadata, and measured-interval throughput.
- [x] T003 [US1] Run the focused tests and record the expected RED evidence.

## Phase 2: Minimal Driver Fix

- [x] T004 [US1] Add the base-user lifecycle helper and wire threaded open-loop execution through it.
- [x] T005 [US1] Record per-request process-pool target/start/slip timing in worker results.
- [x] T006 [US1] Aggregate open-loop measurement elapsed interval and maximum slip, record process-pool measurement start, and compute measured throughput without hiding failed/missing requests or including startup/teardown.
- [x] T007 [US1] Run focused and full Python regression tests and record results.

## Phase 3: MiniNDN Validation

- [x] T008 [US1] Preflight matched threaded and process-pool commands against Spec 091 controls.
- [x] T009 [US1] Run the 60-second threaded validation and preserve its complete or negative result.
- [x] T010 [US1] Run the 60-second process-pool validation and classify it with Spec 091 gates.
- [x] T011 [US1] For the selected passing driver, run three matched repetitions and summarize them without claiming maximum stable RPS.

## Phase 4: Closure

- [x] T012 [US1] Update traceability, experiment validation, and the canonical DI runtime workflow.
- [x] T013 [US1] Run Spec Kit analyze/audit/converge, GSD health, diff checks, and mark the feature complete only when evidence is reproducible.
