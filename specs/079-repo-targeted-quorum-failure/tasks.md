# Tasks: Targeted Quorum Provider Failure

## Phase 1 - Context and contracts

- [x] T001 Load Spec 078 results, current GSD state, and dirty worktree.
- [x] T002 Use CodeGraph to trace reservation, store, receipt, cooldown, and failure injection.
- [x] T003 Define matched ARS experiment variables and interpretation limits.
- [x] T004 Activate Spec 079 and GSD Phase 11.
- [x] T005 Add failing RF=3/W=QUORUM partial-reservation contract test.
- [x] T006 Add W=ALL negative regression and cooldown-selection test.

## Phase 2 - Failure-aware quorum implementation

- [x] T007 Pass required acknowledgements into reservation coordination.
- [x] T008 Permit partial reservations only when successful count reaches W.
- [x] T009 Store only to successfully reserved providers.
- [x] T010 Preserve desired replicationFactor and confirmed receipt owners.
- [x] T011 Record Targeted outcomes in provider health/cooldown state.
- [x] T012 Exclude active-cooldown explicit replicas when W remains satisfiable.
- [x] T013 Preserve W=ALL and release successful reservations on failure.
- [x] T014 Run focused Repo unit tests and fix regressions.

## Phase 3 - Failure instrumentation and campaign

- [x] T015 Add request start/completion epoch timestamps to lifecycle CSV.
- [x] T016 Add MiniNDN pre/overlap/post failure phase aggregation.
- [x] T017 Add request-timeout plumbing and bounded recorded seed readiness retries.
- [x] T018 Run 60-second no-failure RF=3/W=QUORUM baseline.
- [x] T019 Run matched RepoA-loss campaign at 20 seconds.
- [x] T020 Validate every successful write has at least two receipts.
- [x] T021 Compare latency, fallback, timeout, failure, and achieved-RPS evidence.

## Phase 4 - Acceptance

- [x] T022 Run build and all focused C++/Python/NAC regressions.
- [x] T023 Run Spec Kit consistency and checklist audit.
- [x] T024 Run CodeGraph impact/orphan review.
- [x] T025 Update English/Chinese Repo docs and exact quickstart.
- [x] T026 Record results, negative findings, and residual risks.
- [x] T027 Complete GSD review, UAT, summary, and health.
- [x] T028 Mark all tasks complete and play the completion bell.
