# Tasks: Bounded Parallel Replica Repair

## Phase 1 - Context and contracts

- [x] T001 Load Spec 080 evidence and dirty worktree.
- [x] T002 Use CodeGraph to trace durable scheduling, leases, transfer, and
  `ServiceUser` thread boundaries.
- [x] T003 Define the ARS matched experiment and negative-result rule.
- [x] T004 Activate Spec 081 and GSD Phase 13.
- [x] T005 Add schema-v8 in-place migration tests.
- [x] T006 Add risk/priority/age/backoff ordering tests.
- [x] T007 Add bounded parallel sidecar transfer test.

## Phase 2 - Durable scheduling

- [x] T008 Add repair scheduling columns and schema migration.
- [x] T009 Populate/refresh scheduling metadata during scan.
- [x] T010 Order atomic claims by risk, priority, age, retry, and ID.
- [x] T011 Expose scheduling fields in claimed-job diagnostics.
- [x] T012 Run focused schema/catalog tests.

## Phase 2A - Quorum finalization correctness

- [x] T012A Add staged-versus-finalized catalog and repair tests.
- [x] T012B Persist multi-replica local writes as staged catalog entries.
- [x] T012C Add protected receipt-backed `FINALIZE_WRITE` handling.
- [x] T012D Finalize confirmed replicas after user-side quorum validation.
- [x] T012E Exclude staged-only generations from repair planning.
- [x] T012F Preserve direct commit for single-replica and authorized repair writes.

## Phase 3 - Bounded transfer workers

- [x] T013 Add repair-workers and repair-max-jobs sidecar options.
- [x] T014 Claim jobs serially and execute only transfers in the worker pool.
- [x] T015 Serialize complete/fail control operations on the main thread.
- [x] T016 Add duration/worker scheduling logs and safe exception handling.
- [x] T017 Run focused sidecar parallelism tests.

## Phase 4 - Campaign

- [x] T018 Add MiniNDN repair-worker/max-job options and metadata.
- [x] T019 Run matched workers=3/max-jobs=6 campaign.
- [x] T020 Compare against same-code workers=1 and historical Spec 080 evidence.
- [x] T021 Verify request success and receipt floor remain correct.
- [x] T022 Record positive or negative throughput evidence without rerun bias.

## Phase 5 - Acceptance

- [x] T023 Run build, full Repo Python, C++ Repo, Targeted, security, and worker tests.
- [x] T024 Run Spec Kit and checklist consistency audit.
- [x] T025 Run CodeGraph impact/orphan review.
- [x] T026 Update English/Chinese docs, quickstart, and results.
- [x] T027 Complete GSD review/UAT/summary/health.
- [x] T028 Mark complete and play the completion bell.
