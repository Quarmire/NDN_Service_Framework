# Tasks: Native DI User Driver Correctness

## Phase 1: Test-First Contracts

- [ ] T001 [US1] Add a failing lifecycle test proving start-before-work and stop-on-success/failure for the base scope-key publisher.
- [ ] T002 [US1] Add failing deterministic tests for worker `scheduleSlipMs`, process-pool measurement metadata, and measured-interval throughput.
- [ ] T003 [US1] Run the focused tests and record the expected RED evidence.

## Phase 2: Minimal Driver Fix

- [ ] T004 [US1] Add the base-user lifecycle helper and wire threaded open-loop execution through it.
- [ ] T005 [US1] Record per-request process-pool target/start/slip timing in worker results.
- [ ] T006 [US1] Aggregate process-pool measurement start, elapsed interval, maximum slip, and measured throughput without hiding failed/missing requests.
- [ ] T007 [US1] Run focused and full Python regression tests and record results.

## Phase 3: MiniNDN Validation

- [ ] T008 [US1] Preflight matched threaded and process-pool commands against Spec 091 controls.
- [ ] T009 [US1] Run the 60-second threaded validation and preserve its complete or negative result.
- [ ] T010 [US1] Run the 60-second process-pool validation and classify it with Spec 091 gates.
- [ ] T011 [US1] If process-pool passes, run two additional matched repetitions and summarize all three without claiming maximum stable RPS.

## Phase 4: Closure

- [ ] T012 [US1] Update traceability, experiment validation, and the canonical DI runtime workflow.
- [ ] T013 [US1] Run Spec Kit analyze/audit/converge, GSD health, diff checks, and mark the feature complete only when evidence is reproducible.
