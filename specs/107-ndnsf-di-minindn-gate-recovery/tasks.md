# Tasks: NDNSF-DI MiniNDN Gate Recovery

**Input**: Design artifacts in `specs/107-ndnsf-di-minindn-gate-recovery/`

**Execution rule**: Spec 105 is immutable. Every campaign is once-only, keeps
failures, and writes to a new exclusive path. Tests precede implementation.
Performance/fault/soak commands MUST NOT run until their named preflight passes.

## Phase 1: Freeze and Identity Setup

**Purpose**: Make predecessor immutability, successor identity, and disk safety
machine-verifiable before any runtime change.

- [X] T001 Add RED lineage-lock tests covering every pinned Spec 105 digest and write rejection in `tests/python/test_ndnsf_di_spec107_lineage.py`
- [X] T002 Implement read-only lineage verification and Spec 105 path denial in `tools/ndnsf-di/spec107_lineage.py` (depends on T001)
- [X] T003 [P] Add RED canonical candidate/campaign identity tests, including Spec 105 ID/path rejection, in `tests/python/test_ndnsf_di_spec107_identity.py`
- [X] T004 Implement digest-derived `spec107-c1` candidate and disjoint campaign identities in `tools/ndnsf-di/spec107_identity.py` (depends on T003)
- [X] T005 [P] Add RED output exclusivity, stale-writer, ownership, artifact-hash, projected-growth, and 1 GiB reserve tests in `tests/python/test_ndnsf_di_spec107_preflight.py`
- [X] T006 Implement fail-before-role-start campaign preflight and `INVALID_PREFLIGHT` record in `tools/ndnsf-di/spec107_preflight.py` (depends on T005)
- [X] T007 [P] Add RED content-addressed hardlink/reflink/copy fallback, read-only sealing, and `.pt` cleanup tests in `tests/python/test_ndnsf_di_spec107_artifacts.py`
- [X] T008 Implement one-time artifact materialization in `tools/ndnsf-di/spec107_artifacts.py` and expose lineage/artifact/campaign/gate subcommands through `tools/ndnsf-di/spec107_candidate.py` (depends on T007)
- [X] T009 Validate `lineage-lock.json` against current frozen files and record the verification command/output in `specs/107-ndnsf-di-minindn-gate-recovery/evidence/lineage-baseline.md` (depends on T002)

**Checkpoint**: Spec 107 can prove it is a new candidate and can refuse unsafe
campaign startup without modifying Spec 105.

---

## Phase 2: Foundational Contracts

**Purpose**: Establish bounded state, evidence schemas, and build targets shared
by attribution, performance, recovery, and operations.

- [X] T010 Add RED codec/state-machine tests for generation/session/token-epoch/candidate/plan/attempt bindings in `tests/unit-tests/di-qwen-generation-session.t.cpp`
- [X] T011 [P] Add RED tests for bounded generation, wait, tensor, callback, metrics, and token-pair queues in `tests/unit-tests/distributed-inference-async-runtime.t.cpp`
- [X] T012 [P] Add RED timing reconciliation and sampler tests in `tests/python/test_ndnsf_di_spec107_timing.py`
- [X] T013 [P] Add RED fault-record/process-ownership/cleanup schema tests in `tests/python/test_ndnsf_di_spec107_faults.py`
- [X] T014 [P] Add RED release-gate tests for missing, failed, tampered, mixed-candidate, diagnostic, and physical-PASS evidence in `tests/python/test_ndnsf_di_spec107_release_gate.py`
- [X] T015 Define `QwenGenerationSessionSpec`, token-epoch state, bounds, and terminal enums in `NDNSF-DistributedInference/cpp/ndnsf-di/QwenGenerationSession.hpp` (depends on T010)
- [X] T016 Implement DI-application payload codec and fail-closed validation in `NDNSF-DistributedInference/cpp/ndnsf-di/QwenGenerationSession.cpp` (depends on T015)
- [X] T017 Register only the normal provider/user generation-session build targets and tests in `examples/wscript` (depends on T016; fault target remains separate in T044)
- [X] T018 Implement the stable sampled timing schema and critical-path reconciler in `tools/ndnsf-di/spec107_timing.py` (depends on T012)
- [X] T019 Extend candidate/evidence models without changing Core wire names in `NDNSF-DistributedInference/ndnsf_distributed_inference/runtime_v1_evidence.py` (depends on T004, T018)
- [X] T020 Run focused unit/Python tests for T001-T019 and retain exact commands/results in `specs/107-ndnsf-di-minindn-gate-recovery/evidence/foundation-tests.md`

**Checkpoint**: Shared contracts are executable and fail closed. No campaign has
run and no optimization has been selected.

---

## Phase 3: User Story 1 — Attribute Dominant Delay (P1)

**Goal**: Reconcile the token-step critical path and select exactly one measured
optimization branch.

**Independent test**: One warm single-generation diagnostic and one four-worker
diagnostic reconcile at least 99% of completed token steps and yield exactly one
branch at or above 25%; otherwise this phase stops for replanning.

- [X] T021 [US1] Add RED tests that diagnostics are never acceptance-eligible and that dominance selects exactly one branch or `REPLAN_REQUIRED` in `tests/python/test_ndnsf_di_spec107_attribution.py`
- [X] T022 [P] [US1] Add only missing sampled admission/ACK/selection/response events to existing `TimelineTrace` calls in `ndn-service-framework/ServiceUser.cpp` and `ndn-service-framework/ServiceProvider.cpp`, plus validation, queue, compute, codec, and dependency events in `NDNSF-DistributedInference/cpp/ndnsf-di/NativeProviderRuntime.cpp` and `NDNSF-DistributedInference/cpp/ndnsf-di/ProviderRoleWorker.cpp`; do not change wire behavior
- [X] T023 [P] [US1] Add corresponding request/session timing capture in `examples/python/NDNSF-DistributedInference/llm_pipeline/user.py`
- [X] T024 [US1] Implement attribution, reconciliation, rejected-hypothesis retention, and single-branch lock in `tools/ndnsf-di/run_spec107_attribution.py`, integrated by `Experiments/NDNSF_DI_LlmPipeline_Minindn.py` (depends on T018, T021-T023)
- [ ] T025 [US1] Preflight and execute exactly once the warm 32-token diagnostic into a unique `results/spec107-attribution-*/warm-single/` path; preserve success, failure, or invalidity (depends on T020, T024)
- [ ] T026 [US1] Preflight and execute exactly once the four-worker diagnostic into a unique `results/spec107-attribution-*/four-worker/` path; preserve success, failure, or invalidity (depends on T025)
- [ ] T027 [US1] Materialize digest-bound `bottleneck-decision.json` and `specs/107-ndnsf-di-minindn-gate-recovery/evidence/bottleneck-decision.md`; STOP and revise the plan if no unique branch reaches 25% (depends on T025-T026)

**Checkpoint**: Only the locked branch may proceed. The preliminary
generation-scoped design is not authority if T027 falsifies it.

---

## Phase 4: User Story 2 — Pass Fixed-Load Qwen Candidate (P1)

**Goal**: Use one collaboration session per generation, preserve security and
exact tokens, then run three unreplaced acceptance repetitions.

**Independent test**: Exact 1/2/32-token correctness plus all security negatives,
then each of exactly three 60-second 1 RPS repetitions independently passes.

- [ ] T028 [US2] Add RED token-epoch feedback, provider-local KV, exact-final-once, bound, cancellation, deadline, and stale-attempt tests in `tests/unit-tests/di-qwen-generation-session.t.cpp` (depends on T027 selecting generation-scoped collaboration)
- [ ] T029 [P] [US2] Add RED user orchestration and 1/2/32-token baseline equality tests in `tests/python/test_ndnsf_di_spec107_qwen_session.py`
- [ ] T030 [P] [US2] Add RED permission, NAC-ABE, UserToken, ProviderToken, replay, provider-permission, lease, attempt, digest, deadline, and mixed-candidate negative tests in `tests/python/test_ndnsf_di_spec107_security.py`
- [ ] T031 [US2] Implement bounded provider token loop, token-epoch dependencies, feedback, KV lifecycle, and one terminal response in `NDNSF-DistributedInference/cpp/ndnsf-di/QwenGenerationSession.cpp` (depends on T028)
- [ ] T032 [US2] Integrate the session with collaboration dependency I/O and attempt/lease authority in `NDNSF-DistributedInference/cpp/ndnsf-di/NdnsfCollaborationDependencyIo.cpp` and `NDNSF-DistributedInference/cpp/ndnsf-di/NativeProviderRuntime.cpp` (depends on T031)
- [ ] T033 [US2] Replace acceptance-mode per-token requests with one generation request while retaining an explicitly diagnostic-only legacy loop in `examples/python/NDNSF-DistributedInference/llm_pipeline/user.py` (depends on T029, T032)
- [ ] T034 [US2] Wire generation-session provider handling, immutable artifact references, and digest-bound `qwen-generation-session-v1` readiness capability into the real CPU ONNX provider in `examples/DI_NativeProviderExecutable.cpp` (depends on T032)
- [ ] T035 [US2] Extend `Experiments/NDNSF_DI_LlmPipeline_Minindn.py` with candidate identity, preflight, exact-token evidence, stable sampling, and the no-retry repetition manifest from `tools/ndnsf-di/run_spec107_performance.py` (depends on T006, T008, T019, T033-T034)
- [ ] T036 [US2] Run focused generation-session, async-runtime, security, lease, attempt, dependency, and sanitizer checks; retain exact commands/results in `specs/107-ndnsf-di-minindn-gate-recovery/evidence/session-tests.md` (depends on T030-T035)
- [ ] T037 [US2] Materialize the final read-only Qwen artifact set once and lock candidate/performance manifests before any measured role starts (depends on T008, T036)
- [ ] T038 [US2] Execute and retain 1-, 2-, and 32-token correctness cells; STOP performance on any mismatch or incomplete provider evidence (depends on T037)
- [ ] T039 [US2] Preflight and execute performance repetition 1 exactly once for 60 seconds at 1 RPS in its locked unique output directory (depends on T038)
- [ ] T040 [US2] Preflight and execute performance repetition 2 exactly once for 60 seconds at 1 RPS in its locked unique output directory; do not replace T039 (depends on T039)
- [ ] T041 [US2] Preflight and execute performance repetition 3 exactly once for 60 seconds at 1 RPS in its locked unique output directory; do not replace earlier cells (depends on T040)
- [ ] T042 [US2] Derive per-repetition completion/throughput/p50/p95/p99/TTFT/inter-token/resource verdicts and retain negative results in `specs/107-ndnsf-di-minindn-gate-recovery/evidence/performance-verdict.md` (depends on T039-T041)

**Checkpoint**: Performance is PASS only if all three original cells pass. Failure
does not block the independent recovery story but blocks soak and release.

---

## Phase 5: User Story 3 — Live MiniNDN Fault Recovery (P1)

**Goal**: Inject proven live faults only into campaign-owned MiniNDN processes
or data and prove bounded authority/cleanup.

**Independent test**: Positive control plus all eight preregistered cells run
once, set `networkInjection=true`, and end in one authoritative outcome with no
second replacement or leaked owned state.

- [X] T043 [US3] Extend RED tests for `/proc` start-time/PID/PGID/boot/identity/command matching, trigger proof, one replacement, original deadline, old-authority rejection, and cleanup stop rules in `tests/python/test_ndnsf_di_spec107_faults.py`
- [X] T044 [US3] Create an experiment-only fault adapter and binary in `NDNSF-DistributedInference/cpp/ndnsf-di/NativeFaultInjection.cpp`, `NDNSF-DistributedInference/cpp/ndnsf-di/NativeFaultInjection.hpp`, `examples/DI_NativeFaultProviderExecutable.cpp`, and `examples/wscript`; prove `DI_NativeProviderExecutable.cpp` exposes no fault flag (depends on T043)
- [X] T045 [US3] Implement owned child-process registry, guarded kill/restart, trigger/control observation, and cleanup in `tools/ndnsf-di/spec107_fault_controller.py` (depends on T043)
- [X] T046 [US3] Implement immutable once-only live-cell orchestration and evidence emission in `tools/ndnsf-di/run_spec107_live_faults.py`, integrated by `Experiments/NDNSF_DI_LlmPipeline_Minindn.py` (depends on T006, T019, T044-T045)
- [X] T047 [US3] Build normal/fault providers and run focused process-ownership, authority, cleanup, and security tests; retain results in `specs/107-ndnsf-di-minindn-gate-recovery/evidence/fault-tests.md` (depends on T046)
- [ ] T048 [US3] Preflight and execute the no-fault positive control once; STOP if intended-stage arrival and cleanup are not proven (depends on T047)
- [ ] T049 [US3] Execute once each provider kill/restart and straggler cells; preserve both original results and STOP later cells on cleanup failure (depends on T048)
- [ ] T050 [US3] Execute once each missing-segment and dependency-digest-mismatch cells under the same stop rule (depends on T049)
- [ ] T051 [US3] Execute once each stale-telemetry and KV-eviction cells under the same stop rule (depends on T050)
- [ ] T052 [US3] Execute once each provider-boot-change and late-old-output cells under the same stop rule (depends on T051)
- [ ] T053 [US3] Derive cell-by-cell injection, recovery/terminal, authority, and cleanup verdicts in `specs/107-ndnsf-di-minindn-gate-recovery/evidence/live-fault-verdict.md` (depends on T048-T052)

**Checkpoint**: Recovery evidence is real MiniNDN-path evidence, not a contract
fixture. Any failed cell remains failed and blocks the recovery dimension.

---

## Phase 6: User Story 4 — Local Operations and Soak (P2)

**Goal**: Operate the candidate through real packaged child processes while
truthfully retaining physical systemd deferral.

**Independent test**: Two clean canaries, restart, N/N+1 upgrade/rollback with
Repo preservation, then a gated unreplaced 24-hour soak with one restart.

- [X] T054 [US4] Add RED local supervisor ownership, readiness, status, restart, stop, cleanup, and `local-process-supervision` classification tests in `tests/python/test_ndnsf_di_spec107_operations.py`
- [X] T055 [P] [US4] Add RED upgrade/rollback/Repo-preservation and incompatible-cache-discard tests in `tests/python/test_ndnsf_di_spec107_upgrade.py`
- [X] T056 [US4] Implement packaged-command child supervision and structured process registry in `packaging/ndnsf-di-systemd/run-local-supervised.sh` and `tools/ndnsf-di/spec107_local_supervisor.py` (depends on T054)
- [X] T057 [US4] Extend staging validation/status/metrics evidence with release, plan, candidate, boot, queue, request, terminal, Repo, and supervision identity in `packaging/ndnsf-di-systemd/validate-staging.sh` (depends on T056)
- [X] T058 [US4] Implement isolated N/N+1 activation and rollback drill with Repo preservation in `tools/ndnsf-di/run_spec107_operations.py` (depends on T055-T057)
- [X] T059 [P] [US4] Update exact local-only operator commands and claim boundaries in `packaging/ndnsf-di-systemd/README.md` and `packaging/ndnsf-di-systemd/README_ch.md`
- [ ] T060 [US4] Execute clean-directory canary 1 once and retain install-to-cleanup evidence (depends on T058-T059)
- [ ] T061 [US4] Execute clean-directory canary 2 once without reusing mutable runtime state (depends on T060)
- [ ] T062 [US4] Execute restart, N-to-N+1 upgrade, N+1-to-N rollback, Repo preservation, stop, and cleanup once (depends on T061)
- [ ] T063 [US4] Verify performance, correctness, security, recovery, disk, canary, output-exclusivity, and projected-soak-growth prerequisites; emit `SOAK_NOT_ELIGIBLE` and do not start if any fail (depends on T042, T053, T062)
- [ ] T064 [US4] If and only if T063 passes, execute one unreplaced 24-hour 1 RPS soak with the preregistered provider restart and bounded sampled evidence (depends on T063)
- [ ] T065 [US4] Derive canary, operations, resource-growth, restart, and soak verdicts in `specs/107-ndnsf-di-minindn-gate-recovery/evidence/operations-verdict.md` (depends on T060-T064)

**Checkpoint**: Local operational evidence remains explicitly non-physical.

---

## Phase 7: User Story 5 — Successor Release Decision (P2)

**Goal**: Produce a tamper-evident local PASS/BLOCK without erasing Spec 105 or
claiming Spec 106 evidence.

**Independent test**: Every tampered/missing/mixed case blocks; only complete
local evidence may PASS while physical always remains DEFERRED.

- [X] T066 [US5] Implement digest-bound dimension aggregation, predecessor-BLOCK preservation, and forced physical-DEFERRED output in `NDNSF-DistributedInference/ndnsf_distributed_inference/release_gate.py` (depends on T014)
- [X] T067 [US5] Add Spec 107 manifest/bundle generation with secret/payload/tensor/KV/token-content scanning in `tools/ndnsf-di/build_spec107_release_bundle.py` (depends on T066)
- [ ] T068 [US5] Run release-gate positive/negative/tamper tests plus full relevant C++/Python/MiniNDN security regression and retain results in `specs/107-ndnsf-di-minindn-gate-recovery/evidence/final-tests.md` (depends on T042, T053, T065, T067)
- [ ] T069 [US5] Generate the final successor manifest and mechanical local verdict in `specs/107-ndnsf-di-minindn-gate-recovery/release-gate.json`; never edit or supersede Spec 105 evidence (depends on T068)
- [X] T070 [P] [US5] Update current candidate instructions in `NDNSF-DistributedInference/README.md`, `NDNSF-DistributedInference/README_ch.md`, and Spec 107 `quickstart.md` with identical English/Chinese commands and physical deferral
- [ ] T071 [US5] Update Spec 106 prerequisite to consume only a digest-bound Spec 107 PASS while retaining its physical-only scope; do not mark any Spec 106 task complete (depends on T069 PASS; skip and record deferral on BLOCK)
- [ ] T072 [US5] Run `speckit-analyze`, strict `speckit-audit`, CodeGraph sync/status, agent-context update, GSD health/resume capture, quickstart validation, and final Spec 105 digest verification; remediate all HIGH/BLOCK findings (depends on T069-T071)

**Checkpoint**: Spec 107 is complete. PASS authorizes only the start of Spec 106;
BLOCK is a valid terminal result and preserves all failed evidence.

---

## Dependencies and Stop Rules

```text
T001-T009 identity/disk setup
        -> T010-T020 shared contracts
        -> T021-T027 attribution
             -> if unique >=25% generation-session branch: T028-T042 performance
             -> otherwise: REPLAN_REQUIRED (no optimization implementation)

T010-T020 -> T043-T053 live recovery (independent of performance verdict)

T042 PASS + T053 PASS + T060-T062 PASS
        -> T063 soak eligibility
        -> T064 only if eligible

T042 + T053 + T065 -> T066-T072 release gate
```

- T025, T026, T039-T041, T048-T052, T060-T062, and T064 are once-only. No
  automatic retry, replacement cell, result deletion, or pooled rescue.
- Cleanup failure stops subsequent live-fault cells.
- Correctness/security/preflight failure stops performance before a measured
  role starts.
- Disk preflight must account for model delta, logs, sampled traces, soak growth,
  and the 1 GiB reserve; current free space is not itself an authorization.
- Spec 105 digest failure stops all Spec 107 work.
- T071 is conditional on a real Spec 107 PASS and is never a route to fabricate
  physical readiness.

## Parallel Opportunities

- T003/T005/T007 may proceed after T001 because they touch disjoint tools/tests.
- T011-T014 may proceed after identity setup because their test files are disjoint.
- T022/T023 are separate C++ and Python instrumentation surfaces.
- T029/T030 can be written while T028 extends the C++ RED suite.
- T054/T055 and T059 are independent test/document surfaces.
- No campaign execution task is parallel: ordering is part of evidence identity,
  cleanup safety, disk budgeting, and the no-retry contract.

## Completion Definition

- Every task is checked or has an explicit terminal `SKIPPED_BY_GATE` record.
- Frozen Spec 105 hashes still match `lineage-lock.json`.
- All commands, environment, candidate/campaign IDs, output paths, thresholds,
  failures, cleanup, and evidence digests are retained.
- A BLOCK verdict is complete work; it must not be converted to PASS by tuning,
  rerunning, shortening, pooling, or deleting evidence.
