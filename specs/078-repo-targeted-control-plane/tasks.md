# Tasks: Targeted Parallel Repo Control Plane

## Phase 1 - Context and baseline

- [x] T001 Read constitution, Spec 077 evidence, current GSD state, and dirty worktree.
- [x] T002 Use CodeGraph to trace Targeted, async, provider registration, and Repo control calls.
- [x] T003 Confirm baseline read/write latency from canonical lifecycle CSV.
- [x] T004 Define matched experiment variables and interpretation rules.
- [x] T005 Activate Spec 078 and GSD Phase 10.

## Phase 2 - NDNSF core Targeted Python API

- [x] T006 Add failing wrapper tests for sync/async Targeted calls.
- [x] T007 Implement native synchronous Targeted request binding.
- [x] T008 Implement native asynchronous Targeted request binding with safe Python callbacks.
- [x] T009 Expose both APIs through `ndnsf.ServiceUser`.
- [x] T010 Register Python provider handlers as `NormalAndTargeted`.
- [x] T011 Preserve authenticated request context in both modes.
- [x] T012 Make provider Targeted token batch size bounded/configurable.
- [x] T013 Add normal/Targeted coexistence and security regressions.
- [x] T014 Rebuild pybind and run core wrapper tests.

## Phase 3 - Repo parallel replica coordinator

- [x] T015 Add failing tests for parallel submission, total deadline, partial failure, and receipt ordering.
- [x] T016 Add Repo control metrics and phase timers.
- [x] T017 Implement one-dispatcher async Targeted fan-out.
- [x] T018 Add bounded NormalOnly fallback and counters.
- [x] T019 Parallelize capacity reservations.
- [x] T020 Parallelize reservation release after failure.
- [x] T021 Parallelize replicated STORE/STORE_PACKETS receipt collection.
- [x] T022 Preserve operation ID, W validation, CAS, and incomplete-write evidence.
- [x] T023 Apply the coordinator to generic object and versioned write paths.
- [x] T024 Keep catalog/repair calls compatible with Targeted and Normal providers.
- [x] T025 Add close/cancellation behavior with no callback-after-destroy race.
- [x] T026 Run all focused Repo tests and fix regressions.

## Phase 4 - Instrumented campaign

- [x] T027 Extend lifecycle CSV with reserve/store/control phase timings.
- [x] T028 Add Targeted and replica-concurrency counters to summary JSON.
- [x] T029 Add campaign switch for targeted versus Spec-077 normal control.
- [x] T030 Run Targeted bootstrap/steady-state MiniNDN smoke.
- [x] T031 Run matched c16/2-RPS 60-second campaign.
- [x] T032 Run matched write-heavy c4/0.5-RPS 60-second campaign.
- [x] T033 Aggregate baseline and optimized results.
- [x] T034 Report improvement or negative result honestly.

## Phase 5 - Documentation and acceptance

- [x] T035 Update English and Chinese Repo documentation.
- [x] T036 Update quickstart with exact commands and result paths.
- [x] T037 Run full build and all C++ Repo tests.
- [x] T038 Run focused Python and relevant security/runtime regressions.
- [x] T039 Run Spec Kit consistency and checklist audit.
- [x] T040 Run CodeGraph orphan/impact review.
- [x] T041 Run GSD review, UAT, and health checks.
- [x] T042 Record residual risks and next optimization boundary.
- [x] T043 Mark Spec/GSD/tasks complete only after all gates pass.
- [x] T044 Play completion bell and report results.
