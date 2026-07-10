# Tasks: Repo Repair Fast Path and Observability

## Phase 1 - Context and Design

- [x] T001 Load Spec 081 code, logs, and accepted evidence.
- [x] T002 Trace merge, scan, claim, transfer, and control executor with CodeGraph.
- [x] T003 Identify the serialized target-miss preflight root cause.
- [x] T004 Define the ARS matched experiment and negative-result rule.
- [x] T005 Activate Spec 082 and GSD Phase 14.

## Phase 2 - Repair Visibility

- [x] T006 Add SQL-derived repair state/claimability diagnostics.
- [x] T007 Add scan counter/backoff/target tests.
- [x] T008 Return structured sidecar cycle metrics.
- [x] T009 Log peer merge duration and batches.
- [x] T010 Parse cycle/merge telemetry into campaign summaries.
- [x] T011 Add evidence parser tests.

## Phase 3 - Fast Path

- [x] T012 Remove catalog-known target `FETCH_PREPARE` preflight.
- [x] T013 Preserve source prepare, exact retrieval, hash, and authorization checks.
- [x] T014 Add no-target-probe repair-flow regression.
- [x] T015 Verify idempotent replay and existing-object safety.

## Phase 4 - Verification and Campaign

- [x] T016 Run Python compile and Repo tests.
- [x] T017 Run build and focused C++/Targeted/security/worker tests.
- [x] T018 Run the matched workers=3 MiniNDN campaign once.
- [x] T019 Verify success, W floor, invalid-repair count, and telemetry.
- [x] T020 Compare against Spec 081 workers=3 evidence.

## Phase 5 - Acceptance

- [x] T021 Update English/Chinese docs and Spec results.
- [x] T022 Run Spec Kit consistency/checklist audit.
- [x] T023 Run CodeGraph impact/orphan review.
- [x] T024 Complete GSD review/UAT/summary/health.
- [x] T025 Mark complete and play the completion bell.
