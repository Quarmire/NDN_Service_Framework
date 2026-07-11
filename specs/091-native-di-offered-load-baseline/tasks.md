# Tasks: Native DI Offered-Load Baseline

## Phase 1: Freeze And Preflight

- [x] T001 [US1] Record the git commit, fixed controls, hypotheses, thresholds, and exact commands.
- [x] T002 [US1] Run dry-run expansion for child, threaded, and process-pool modes and confirm only the treatment variable changes.
- [x] T003 [US1] Confirm no stale MiniNDN/NFD process or dirty source change can confound the runs.

## Phase 2: Matched Screening

- [x] T004 [US1] Run child mode at 1 RPS, concurrency 4, for 60 seconds.
- [x] T005 [US1] Run threaded mode with the same controls.
- [x] T006 [US1] Run process-pool mode with the same controls.

## Phase 3: Validation And Decision

- [x] T007 [US1] Extract the required scheduling, completion, latency, provider, and dependency metrics from all summaries.
- [x] T008 [US1] Check command/config equivalence and classify anomalies or invalid runs.
- [x] T009 [US1] Identify the first limiting layer without making an unsupported maximum-RPS claim.
- [x] T010 [US1] Select either a three-run replication campaign, a higher-rate search, or a narrowly scoped implementation fix.
- [x] T011 [US1] Run Spec Kit structure/analyze/audit/converge and update the DI workflow with the accepted baseline.
- [x] T012 [US1] Mark Spec 091 complete only when all evidence and the next-decision gate are reproducible.
