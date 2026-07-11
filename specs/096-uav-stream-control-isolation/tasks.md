# Tasks: UAV Stream And Control Isolation Campaign

## Phase 1: Design And Audit

- [x] T001 [US1] Inventory launcher branch order, video/control automation lifetime, reusable Spec 095 helpers, and current evidence.
- [x] T002 [US2] Freeze the five-cell 5% matrix, constants, no-retry rule, output contract, and acceptance gates.
- [x] T003 [US3] Apply ARS experiment-design checks for hypotheses, confounds, descriptive-only inference, and failure retention.
- [x] T004 [US1] Run strict Spec Kit pre-implementation audit and resolve every blocker.

## Phase 2: Workload Modes

- [x] T005 [US1] Add default-preserving `include_video` command construction and parsing to the canonical campaign helpers.
- [x] T006 [US1] Implement the thin five-cell isolation campaign with deterministic run IDs and no retry.
- [x] T007 [US3] Add stable per-run and per-cell JSON/CSV output with component completion fields.
- [x] T008 [US1] Add focused tests for matrix, flags, control-only acceptance, video thresholds, aggregation, and invalid modes.
- [x] T009 [US1] Run dry-run, focused tests, and strict structural audit.

## Phase 3: MiniNDN Evidence

- [ ] T010 [US2] Execute the 15-run primary MiniNDN campaign once at 5% loss.
- [ ] T011 [US3] Validate all run directories, command modes, component markers, metrics, and failure retention.
- [ ] T012 [US3] Compare control-only, video-only, and combined cells descriptively by parity.
- [ ] T013 [US3] Run the full Python regression suite and any affected C++ checks.

## Phase 4: Closure

- [ ] T014 [US3] Write reproducibility, result, limitations, negative-result, and residual-risk evidence.
- [ ] T015 [US3] Run post-implementation audit/converge and execute appended tasks.
- [ ] T016 [US3] Update GSD/agent context/CodeGraph, verify no runtime/proposal changes, commit, and close.
