# Tasks: Online Replica Repair After Recovery

## Phase 1 - Context and contracts

- [x] T001 Load Spec 079 evidence, current GSD state, and dirty worktree.
- [x] T002 Use CodeGraph to trace catalog summary, durable jobs, repair transfer,
  sidecar lifecycle, and MiniNDN restart orchestration.
- [x] T003 Define the ARS experiment variables, acceptance criteria, and
  interpretation boundaries.
- [x] T004 Activate Spec 080 and GSD Phase 12.
- [x] T005 Add a planner test for RF=3 held by B/C with recovered A eligible.
- [x] T006 Add negative planner tests for stale/already-owning/ineligible targets.

## Phase 2 - Rejoin and repair implementation

- [x] T007 Make MiniNDN restart the recovered Repo's catalog/auto-repair sidecar.
- [x] T008 Preserve sidecar identity, persistent storage, peers, and security config.
- [x] T009 Verify durable repair scan/claim/retry completes against recovered A.
- [x] T010 Fix only catalog or repair-path defects demonstrated by failing tests.
- [x] T011 Run focused planner and repair-job tests.

## Phase 3 - Evidence instrumentation

- [x] T012 Add `objectName` to lifecycle rows and CSV output.
- [x] T013 Correlate outage-window writes with recovered-target repair events.
- [x] T014 Report repair coverage, first/last repair latency, and unrepaired names.
- [x] T015 Keep seed/readiness traffic outside measured control counters.
- [x] T016 Add deterministic summary-contract tests for repair correlation.

## Phase 4 - MiniNDN campaign

- [x] T017 Run the 60-second RF=3/W=QUORUM failure/restart campaign.
- [x] T018 Verify outage writes use at least two validated receipts.
- [x] T019 Verify at least one outage object is repaired to RepoA through NDNSF.
- [x] T020 Record achieved RPS, p50/p95, Targeted counters, repair coverage, and
  negative findings.

## Phase 5 - Acceptance

- [x] T021 Run full Repo Python and focused C++/Targeted/NAC regressions.
- [x] T022 Run Spec Kit consistency and checklist audit.
- [x] T023 Run CodeGraph impact/orphan review and sync if needed.
- [x] T024 Update English/Chinese Repo docs and exact quickstart.
- [x] T025 Record results and residual risks.
- [x] T026 Complete GSD review, UAT, summary, and health.
- [x] T027 Mark all tasks complete and play the completion bell.
