# Tasks: NDNSF-DI MiniNDN Deployment Candidate

**Input**: Design documents from `specs/105-ndnsf-di-deployment-readiness/`

**Tests**: Test-first is mandatory for evidence truth, tensor/runtime behavior,
telemetry freshness, bounded scheduling, recovery, security, and release gates.

## Phase 1: Setup and Frozen Baseline

**Purpose**: Preserve the controlling evidence and prevent scope drift.

- [X] T001 Record the exact Spec 093 deterministic-provider log evidence, optimistic summary field, source commit, and non-compute interpretation in `specs/105-ndnsf-di-deployment-readiness/evidence/baseline.md`.
- [X] T002 [P] Inventory all maintained producers and readers of `runnerMode`, `tracerDeterministicRunner`, runner backend metadata, and provider readiness evidence in `specs/105-ndnsf-di-deployment-readiness/evidence/runner-reader-inventory.md` using CodeGraph then exact text verification.
- [X] T003 [P] Inventory Qwen ONNX stage artifacts, tokenizer/model revisions, current dtype/shape contracts, single-node proof, MiniNDN proof, and dependency object formats in `specs/105-ndnsf-di-deployment-readiness/evidence/qwen-runtime-inventory.md`.
- [X] T004 [P] Inventory production-relevant install, CLI, profile, identity, NFD, metrics, service supervision, release, and rollback surfaces in `specs/105-ndnsf-di-deployment-readiness/evidence/operator-surface-inventory.md`.
- [X] T005 Freeze the pilot model revision, prompt corpus, tokenizer settings, three-stage ranges, generation limits, numerical tolerance, and artifact digest procedure in `examples/ndnsf-di-qwen-pilot.model.json`.
- [X] T006 Freeze matched single-node, MiniNDN, fault, local-operations and 24-hour soak controls, including the same-host three-provider fallback role/artifact layout and explicit Spec 106 physical deferral, in `examples/ndnsf-di-qwen-pilot.campaign.json` and `examples/ndnsf-di-qwen-pilot-faults.campaign.json`.
- [X] T007 Run strict pre-implementation Spec Kit structure/audit and record the verdict without repairs in `specs/105-ndnsf-di-deployment-readiness/evidence/pre-implementation-audit.md`.
- [X] T008 Verify GSD health, synchronize CodeGraph and agent context, and record source/worktree ownership before implementation in `specs/105-ndnsf-di-deployment-readiness/evidence/preflight.md`.

**Checkpoint**: No runtime edit begins until the deterministic evidence mismatch,
pilot inputs, comparison rules and audit verdict are immutable.

---

## Phase 2: Foundational Schemas and Test Fixtures

**Purpose**: Establish shared types and negative fixtures required by every story.

- [X] T009 Add failing C++ round-trip, required-field, unknown-major-version, and secret-exclusion tests for `ExecutionEvidence` in `tests/unit-tests/distributed-inference-async-runtime.t.cpp`.
- [X] T010 [P] Add failing Python fixtures for synthetic, wiring-only, CPU, CUDA, mixed, missing, artifact-mismatch, and plan-mismatch evidence in `tests/python/test_ndnsf_di_deployment_readiness.py`.
- [X] T011 [P] Add failing Python serialization/freshness tests for configured capability versus measured telemetry in `tests/python/test_ndnsf_di_runtime_v1.py`.
- [X] T012 [P] Add failing C++ attempt-epoch and terminal-reason codec tests in `tests/unit-tests/distributed-inference-async-runtime.t.cpp`.
- [X] T013 Define `ExecutionEvidence` and stable runner-kind enums without wire or policy logic in `NDNSF-DistributedInference/cpp/ndnsf-di/ExecutionEvidence.hpp` and `ExecutionEvidence.cpp`.
- [X] T014 Add execution-evidence JSON encode/decode, validation, digest normalization, and redaction implementation in `NDNSF-DistributedInference/cpp/ndnsf-di/ExecutionEvidence.cpp`.
- [X] T015 Extend the DI Python typed models with provider capability, measured telemetry, execution evidence, plan predicates, attempt epoch, and terminal reason in `NDNSF-DistributedInference/ndnsf_distributed_inference/runtime_v1.py`.
- [X] T016 Add schema samples and version compatibility fixtures under `examples/python/NDNSF-DistributedInference/runtime_v1_schemas/` matching every contract in `specs/105-ndnsf-di-deployment-readiness/contracts/`.
- [X] T017 Update `wscript` to compile the new DI C++ translation units and prove a no-ONNX/no-CUDA build fails clearly only when a requested backend is unavailable.

**Checkpoint**: Shared contracts round-trip, invalid evidence fails closed, and no
request behavior has changed.

---

## Phase 3: User Story 1 - Trust Every Performance Label (Priority: P1) MVP

**Goal**: Make provider-observed execution identity the only real-compute truth.

**Independent Test**: Synthetic and real providers produce different immutable
evidence; mixed/missing evidence blocks; summaries never infer reality from flags.

- [X] T018 [P] [US1] Add failing provider-factory tests proving deterministic, wiring-only, CPU ONNX, and CUDA ONNX factories emit distinct evidence in `tests/unit-tests/distributed-inference-async-runtime.t.cpp`.
- [X] T019 [P] [US1] Add failing campaign parser tests proving `runnerMode=qwen-onnx-native` cannot override `tracerDeterministicRunner=1` in `tests/python/test_ndnsf_native_tracer_runtime_profile.py`.
- [X] T020 [P] [US1] Add failing release-gate tests for missing, mixed, contradictory, synthetic, digest-mismatched, and all-real evidence in `tests/python/test_ndnsf_di_deployment_readiness.py`.
- [X] T021 [US1] Make runner factories create `ExecutionEvidence` only after backend/session initialization succeeds in `NDNSF-DistributedInference/cpp/ndnsf-di/NativeModelRunner.cpp` and `OnnxRuntimeModelRunner.cpp`.
- [X] T022 [US1] Bind runner evidence to provider boot ID, evidence epoch, installed roles, model/artifact digests, plan digest, runtime version, and device ID in `examples/DI_NativeProviderExecutable.cpp`.
- [X] T023 [US1] Publish execution evidence through the existing typed readiness service payload without token/key/payload bytes in `NDNSF-DistributedInference/cpp/ndnsf-di/NativeProviderReadiness.cpp`.
- [X] T024 [US1] Parse and preserve per-provider execution evidence in user ACK candidates and run artifacts in `Experiments/NDNSF_DI_NativeTracer_Minindn.py`.
- [X] T025 [US1] Replace caller-assigned aggregate `runnerMode` with derived `runnerClassification` plus a deprecated derived compatibility field in `Experiments/NDNSF_DI_NativeTracer_Minindn.py`.
- [X] T026 [US1] Implement the six-dimension machine-readable release gate and mechanical BLOCK precedence in `NDNSF-DistributedInference/ndnsf_distributed_inference/release_gate.py`.
- [X] T027 [US1] Reclassify Spec 091-093 metadata in a new immutable correction artifact at `specs/105-ndnsf-di-deployment-readiness/evidence/historical-evidence-correction.md` without editing raw results.
- [X] T028 [US1] Update maintained profile, GUI, report, and documentation readers to consume execution evidence in `tools/ndnsf_runtime.py`, `NDNSF-DistributedInference/ndnsf_distributed_inference/gui.py`, `docs/NDNSF-DI-runtime-workflow.md`, `NDNSF-DistributedInference/README.md`, and `README_ch.md`.
- [X] T029 [US1] Run CodeGraph and exact zero-reader scans, then remove caller-controlled `runnerMode` only when the migration conditions in `migration-and-rollback.md` pass.
- [X] T030 [US1] Execute unique synthetic, real CPU, unavailable-CUDA rejection, mixed, missing, and digest-mismatch evidence cells and record the release-gate matrix in `specs/105-ndnsf-di-deployment-readiness/evidence/evidence-gate-results.md`.

**Checkpoint**: US1 is independently deployable. No synthetic or unknown provider
can pass a real-compute gate, and historical throughput claims are correctly scoped.

---

## Phase 4: User Story 2 - Run a Real Bounded Qwen Service (Priority: P1)

**Goal**: Deliver real three-stage CPU ONNX prefill/decode for the fixed local pilot.

**Independent Test**: Three MiniNDN stages produce the same 32 greedy tokens as
the frozen single-node baseline and close the 1 RPS acceptance gate.

- [X] T031 [P] [US2] Add failing tensor codec tests for int64 token IDs, bool/int64 attention masks, float16/float32 hidden state, named multi-output cache tensors, dynamic dimensions, and malformed shapes in `tests/unit-tests/distributed-inference-async-runtime.t.cpp`.
- [X] T032 [P] [US2] Add failing ONNX runner tests for required CPU selection, unavailable-CUDA rejection, explicit CPU fallback policy, dynamic shapes, multiple dtypes, and backend/device evidence in `tests/unit-tests/distributed-inference-async-runtime.t.cpp`.
- [X] T033 [P] [US2] Add failing Python correctness fixtures for 1, 2, and 32 greedy tokens, max-input/output admission, cache hit, full-context rebuild, and delta-only cache failure in `tests/python/test_ndnsf_di_deployment_readiness.py`.
- [X] T034 [US2] Extend `NamedTensor` and `TensorBundleCodec` with bounded dtype/shape metadata and exact byte-size validation in `NDNSF-DistributedInference/cpp/ndnsf-di/TensorBundleCodec.hpp` and `TensorBundleCodec.cpp`.
- [X] T035 [US2] Extend `OnnxRuntimeModelRunner` to construct typed tensors and decode typed outputs for the Qwen pilot in `NDNSF-DistributedInference/cpp/ndnsf-di/OnnxRuntimeModelRunner.cpp`.
- [X] T036 [US2] Add explicit ONNX Runtime Execution Provider selection, required-provider failure, CPU device binding, runtime-version capture, unavailable-CUDA rejection, and no-silent-CPU-fallback behavior in `NDNSF-DistributedInference/cpp/ndnsf-di/OnnxRuntimeModelRunner.cpp`.
- [X] T037 [US2] Extend the Qwen exporter to expose three stage artifacts with declared token, mask, hidden, logits, and per-stage KV inputs/outputs in `examples/python/NDNSF-DistributedInference/native_di_tracer/generate_qwen_native_tracer_artifacts.py`.
- [X] T038 [US2] Validate exported stage artifacts against the frozen full-model baseline and write digests/shape contracts into the service manifest in `examples/python/NDNSF-DistributedInference/llm_pipeline/plan_pipeline.py`.
- [X] T039 [US2] Implement bounded request validation, tokenization, greedy decode orchestration, and token-by-token result comparison in new `NDNSF-DistributedInference/ndnsf_distributed_inference/qwen_pilot.py`.
- [X] T040 [US2] Implement provider-local `KvStateBinding` storage, lookup, replacement, eviction and boot invalidation with bounded memory in `NDNSF-DistributedInference/cpp/ndnsf-di/NativeProviderSession.hpp` and `NativeProviderSession.cpp`.
- [X] T041 [US2] Bind KV references to session, stage, context/model/plan/security epochs and reject cross-binding reuse in `NDNSF-DistributedInference/cpp/ndnsf-di/NativeProviderHandler.cpp`.
- [X] T042 [US2] Add full-context rebuild and exact delta-only cache-miss terminal behavior in `NDNSF-DistributedInference/ndnsf_distributed_inference/qwen_pilot.py`.
- [X] T043 [US2] Replace Python `torch.save`/NPZ hidden-state transport in the pilot path with the typed tensor bundle while retaining the old path only as a labeled comparison fixture in `examples/python/NDNSF-DistributedInference/llm_pipeline/provider.py` and `user.py`.
- [X] T044 [US2] Wire the real C++ CPU ONNX Qwen provider and bounded-generation user into `Experiments/NDNSF_DI_LlmPipeline_Minindn.py` without reusing the deterministic NativeTracer switch.
- [X] T045 [US2] Generate a matched single-node baseline artifact with identical model/tokenizer/prompts/generation/backend/logging in `Experiments/NDNSF_DI_QwenFull_OnnxVsTransformers_LocalBenchmark.py`.
- [X] T046 [US2] Execute all frozen correctness cells, including cache hit/rebuild/fail-closed cases, and record tokens/digests in `specs/105-ndnsf-di-deployment-readiness/evidence/qwen-correctness.md`.
- [X] T047 [US2] Execute exactly three unique 60-second 1 RPS MiniNDN repetitions plus matched single-node cells and preserve every outcome under distinct `results/spec105-qwen-pilot-*` directories.
- [X] T048 [US2] Validate completion, throughput, p50/p95/p99, p95 ratio, TTFT, inter-token, stage decomposition, resource metrics, and all 11 fallacies in `specs/105-ndnsf-di-deployment-readiness/evidence/qwen-minindn-performance.md`.

**Checkpoint (initial candidate)**: Stop the unchanged campaign if tokens differ,
real CPU ONNX evidence is incomplete, or the fixed 1 RPS gate fails. Preserve the
three failed runs and do not compensate with higher timeout, retry, lower load, or
an unregistered replacement. Revision R1 permits independent implementation to
continue, but a new acceptance campaign is forbidden until T049-T051 close the
verified generation-scheduling validity defect with a new preregistered identity.

---

## Phase 5: User Story 3 - Plan From Fresh Measured Capacity (Priority: P2)

**Goal**: Make measured provider state and explicit validity predicates control
reuse, placement, defer and rejection.

**Independent Test**: Fresh measured facts admit/reuse; stale, unsupported,
identity-mismatched or infeasible facts reject/defer; configured values remain labeled.

- [X] T049 [P] [US3] Add deterministic Python tests proving the generation load driver owns one bounded generation per admitted job, preserves per-session token order, bounds active/queued generations, prevents FIFO breadth-first token-step starvation, and reports per-session progress plus worker/queue occupancy in `tests/python/test_ndnsf_di_deployment_readiness.py`.
- [X] T050 [US3] Replace callback-resubmitted token-step scheduling with a bounded generation-level scheduler in `examples/python/NDNSF-DistributedInference/llm_pipeline/user.py`; retain fixed offered times, four-worker default, zero retry, original timeout, cancellation, and all offered/failed/unfinished accounting.
- [X] T051 [US3] Execute the deterministic driver-validity fixtures, freeze a new candidate/campaign ID and exact command before measurement, and record why the original three runs remain immutable and non-combinable in `specs/105-ndnsf-di-deployment-readiness/evidence/qwen-scheduler-revision.md`.
- [X] T052 [US3] Add failing C++ resource-probe tests for measured host/process memory facts, read failure, unsupported source, malformed input, identity mismatch, stale sample, and redaction, then define the background `ProviderResourceProbe` interface, snapshot status, timeout and freshness semantics in `tests/unit-tests/distributed-inference-async-runtime.t.cpp` and `NDNSF-DistributedInference/cpp/ndnsf-di/ProviderResourceProbe.hpp`.
- [X] T053 [US3] Implement bounded Linux `/proc/meminfo` and provider-process RSS readers, exact unit/parser validation, and explicit unsupported/error snapshots in `NDNSF-DistributedInference/cpp/ndnsf-di/ProviderResourceProbe.cpp`; physical NVIDIA probing remains in Spec 106.
- [X] T054 [US3] Merge resource snapshots with worker queue/wait/active state and stage service-rate EWMA outside the request hot path in `NDNSF-DistributedInference/cpp/ndnsf-di/NativeProviderReadiness.cpp`.
- [X] T055 [US3] Publish configured capability and measured telemetry as distinct typed sections with source, boot, sequence and timestamp in `NDNSF-DistributedInference/cpp/ndnsf-di/NativeProviderReadiness.cpp`.
- [ ] T056 [US3] Add failing Python telemetry tests for provider boot, evidence epoch, artifact/runtime identity, source, age, memory, queue, membership, network version and cache assumptions, then parse, validate and retain those fields in `tests/python/test_ndnsf_di_runtime_aware_planner.py`, `NDNSF-DistributedInference/ndnsf_distributed_inference/runtime_v1.py`, and `deployment.py`.
- [ ] T057 [US3] Add failing admission tests proving configured-only or stale free memory cannot satisfy a candidate memory gate, then implement mandatory plan-feasibility predicates before scoring and observable `reuse|replan|defer|reject` decisions in `tests/python/test_ndnsf_di_runtime_aware_campaign.py` and `NDNSF-DistributedInference/ndnsf_distributed_inference/runtime_v1.py`.
- [ ] T058 [US3] Bind cached plan leases to provider membership/boot, evidence, artifact/runtime, telemetry, network-profile and cache versions in `NDNSF-DistributedInference/ndnsf_distributed_inference/runtime_v1.py`.
- [ ] T059 [US3] Revalidate telemetry and plan predicates immediately before execution-lease commit in `NDNSF-DistributedInference/ndnsf_distributed_inference/deployment.py`.
- [ ] T060 [US3] Update MiniNDN fixtures to label static 2/4/8 GB facts `configured` and add controlled measured host-telemetry injection without presenting it as physical GPU evidence in `Experiments/NDNSF_DI_NativeTracer_Minindn.py`.
- [ ] T061 [US3] Execute fresh/stale/memory-pressure/queue-pressure/membership/device-mismatch plan cells and record every predicate decision in `specs/105-ndnsf-di-deployment-readiness/evidence/telemetry-plan-validation.md`.
- [ ] T062 [US3] Execute exactly one newly preregistered three-repetition real-Qwen MiniNDN acceptance campaign with the validated generation scheduler and INFO telemetry; retain the original failed runs separately and verify correctness, fixed 1 RPS thresholds, queue/progress accounting, and telemetry perturbation in `specs/105-ndnsf-di-deployment-readiness/evidence/telemetry-performance-check.md`.

**Checkpoint**: The planner is capacity-aware only when fresh measured facts are
present; otherwise it rejects/defer rather than pretending configuration is telemetry.

---

## Phase 6: User Story 4 - Recover Without Unbounded Runtime Growth (Priority: P2)

**Goal**: Bound dependency waiting and implement one safe replacement attempt.

**Independent Test**: 1,000 waits use fixed threads and cancel cleanly; provider
faults produce one authoritative result or one exact terminal failure.

- [ ] T063 [P] [US4] Add failing C++ stress tests for 1,000 waits, bounded queue overflow, deadline expiry, cancellation, promise completion, shutdown, and thread-count ceiling in `tests/unit-tests/distributed-inference-async-runtime.t.cpp`.
- [ ] T064 [P] [US4] Add failing C++ tests for attempt epoch monotonicity, distinct dependency names, old-epoch rejection, cancellation and terminal exactly-once behavior in `tests/unit-tests/distributed-inference-async-runtime.t.cpp`.
- [ ] T065 [P] [US4] Add failing Python user recovery tests for provider loss, straggler, stale telemetry, cache miss, no replacement and deadline exhaustion in `tests/python/test_ndnsf_di_deployment_readiness.py`.
- [ ] T066 [US4] Define fixed worker/queue/deadline/cancellation contracts in new `NDNSF-DistributedInference/cpp/ndnsf-di/DependencyWaitScheduler.hpp`.
- [ ] T067 [US4] Implement the bounded dependency wait scheduler, counters, cancellation and shutdown in `NDNSF-DistributedInference/cpp/ndnsf-di/DependencyWaitScheduler.cpp`.
- [ ] T068 [US4] Replace `m_inputWaiters` with the owned scheduler and preserve immediate-ready fast path in `NDNSF-DistributedInference/cpp/ndnsf-di/ProviderRoleWorker.hpp` and `ProviderRoleWorker.cpp`.
- [ ] T069 [US4] Add bounded wait/admission snapshots and explicit scheduler-overload reason to `NDNSF-DistributedInference/cpp/ndnsf-di/NativeProviderReadiness.cpp`.
- [ ] T070 [US4] Add `ExecutionAttemptKey` and attempt epoch to role assignments, dependency object-name derivation and authenticated DI payload metadata in `NDNSF-DistributedInference/cpp/ndnsf-di/NativeExecutionPlan.hpp` and `NativeExecutionPlan.cpp`.
- [ ] T071 [US4] Validate attempt epoch, execution lease, provider boot and plan binding before provider execution in `NDNSF-DistributedInference/cpp/ndnsf-di/NativeProviderHandler.cpp`.
- [ ] T072 [US4] Add user-side attempt state machine, one bounded replacement, remaining-deadline calculation and final-result authority in `NDNSF-DistributedInference/ndnsf_distributed_inference/deployment.py`.
- [ ] T073 [US4] Implement cancel/supersede messages through existing DI service payloads without a new Core wire name in `NDNSF-DistributedInference/ndnsf_distributed_inference/deployment.py` and `NativeProviderHandler.cpp`.
- [ ] T074 [US4] Make provider boot ID invalidate old execution and KV state and expose the new boot before readiness in `examples/DI_NativeProviderExecutable.cpp`.
- [ ] T075 [US4] Instrument old-epoch data, duplicate terminal attempts, cancelled waits, replacement decisions and exact terminal reasons at INFO in `ProviderRoleWorker.cpp`, `NativeProviderHandler.cpp`, and `deployment.py`.
- [ ] T076 [US4] Add same-three-node fallback role activation plus provider kill/restart, straggler, missing segment, hash mismatch, stale telemetry, cache eviction and late-old-output injection to `Experiments/NDNSF_DI_LlmPipeline_Minindn.py`.
- [ ] T077 [US4] Execute the frozen 1,000-wait stress and record threads/memory/state cleanup in `specs/105-ndnsf-di-deployment-readiness/evidence/bounded-scheduler.md`.
- [ ] T078 [US4] Execute the frozen fault matrix with all failures retained and record recovery/terminal outcomes and 11/11 fallacy scan in `specs/105-ndnsf-di-deployment-readiness/evidence/fault-recovery.md`.

**Checkpoint**: Stop if any stale attempt becomes authoritative, any security
check is bypassed, or threads/state grow with pending roles.

---

## Phase 7: User Story 5 - Package and Operate a Local Deployment Candidate (Priority: P3)

**Goal**: Package the accepted runtime as a reproducible systemd-compatible
deployment and prove local install, operation, application-security paths,
upgrade, rollback and soak.

**Independent Test**: Two clean local staging directories execute the runbook,
close the MiniNDN candidate gate, perform restart/upgrade/rollback and a 24h soak.

- [ ] T079 [P] [US5] Add failing CLI tests proving production `provider|run|bench|status|metrics` execute real adapters and simulated behavior is only under `contract-smoke` in `tests/python/test_ndnsf_di_deployment_readiness.py`.
- [ ] T080 [P] [US5] Add failing deployment-profile tests for role identity, NFD endpoint, trust paths, release paths, device/backend, writable directories, startup/shutdown bounds and secret redaction in `tests/python/test_ndnsf_runtime_doctor.py`.
- [ ] T081 [P] [US5] Add failing packaging tests for unit hardening, dedicated users, dependency order, restart bounds, tmpfiles, log rotation, release symlinks and uninstall safety in `tests/python/test_ndnsf_di_deployment_readiness.py`.
- [ ] T082 [US5] Move simulated Runtime v1 `run|bench|context-sweep` implementations under explicit `contract-smoke` commands and preserve callers through a bounded deprecation error in `NDNSF-DistributedInference/ndnsf_distributed_inference/runtime_v1.py`.
- [ ] T083 [US5] Wire production `provider`, `plan`, `run`, `bench`, `status`, `metrics`, `inspect` and `doctor` commands to real deployment adapters in `NDNSF-DistributedInference/ndnsf_distributed_inference/runtime_v1.py`.
- [ ] T084 [US5] Extend the runtime doctor with NFD, identity/certificate, trust schema, backend/device, model/artifact, plan/evidence, disk/permissions and telemetry probe checks in `tools/ndnsf_runtime.py`.
- [ ] T085 [US5] Implement low-overhead structured metrics snapshots and atomic JSON/Prometheus textfile export in `NDNSF-DistributedInference/ndnsf_distributed_inference/operations.py`.
- [ ] T086 [US5] Create hardened controller, provider, Repo, user/bench and target units in `packaging/ndnsf-di-systemd/units/` with bounded restart and shutdown behavior.
- [ ] T087 [US5] Create versioned environment/profile examples, tmpfiles and logrotate definitions in `packaging/ndnsf-di-systemd/config/` without embedding secrets.
- [ ] T088 [US5] Implement idempotent install, activate, rollback and uninstall scripts with digest checks and authoritative-Repo protection in `packaging/ndnsf-di-systemd/install.sh` and `rollback.sh`.
- [ ] T089 [US5] Add release manifest generation with source commit, binary/profile/schema compatibility and artifact digests in `packaging/ndnsf-di-systemd/create-release.sh`.
- [ ] T090 [US5] Write the complete local-operator runbook for MiniNDN identity setup, install, doctor, start, status, canary, restart, upgrade, rollback, evidence collection, emergency stop and Spec 106 deferral in `packaging/ndnsf-di-systemd/README.md` and synchronized `README_ch.md`.
- [ ] T091 [US5] Validate the systemd release in an isolated local/namespace staging profile and record unit/security/rollback evidence in `specs/105-ndnsf-di-deployment-readiness/evidence/systemd-staging.md`.
- [ ] T092 [US5] Run the local MiniNDN canary twice from clean staging directories with matched single/distributed Qwen cells, recording host/backend/profile facts in unique `results/spec105-local-canary-*` directories.
- [ ] T093 [US5] Execute scheduled MiniNDN provider-process restart, staged N->N+1 upgrade, N+1->N rollback, cache incompatibility and Repo preservation drills and record them in `specs/105-ndnsf-di-deployment-readiness/evidence/local-operations.md`.
- [ ] T094 [US5] Preflight the frozen 24-hour local MiniNDN 1 RPS soak against the T062 gate; if PASS, execute it once without replacement and record correctness, completion, latency, resource growth, restart interruption, application-security-path evidence and all failures, otherwise record `NOT RUN / BLOCK` with the controlling immutable evidence in `specs/105-ndnsf-di-deployment-readiness/evidence/local-soak.md`.

**Checkpoint**: A MiniNDN candidate PASS requires every earlier dimension plus
local staging and a completed passing soak. A preregistered stop-rule skip closes
the task honestly but keeps the candidate BLOCKED. Physical production remains
`DEFERRED` and is governed only by Spec 106.

---

## Phase 8: Cross-Cutting Closure and Release Decision

- [ ] T095 [P] Run full C++ tests, all maintained Python tests, security regressions, Repo checks, DI quick checks, and the unchanged UAV launcher checks; record exact commands/results in `specs/105-ndnsf-di-deployment-readiness/evidence/integrated-regression.md`.
- [ ] T096 [P] Run sanitizers or a documented supported memory/thread analysis over execution evidence, tensor codec, bounded scheduler, cancellation and provider restart paths; record limitations in `specs/105-ndnsf-di-deployment-readiness/evidence/runtime-safety.md`.
- [ ] T097 Verify zero secret/payload leakage in evidence, INFO metrics, systemd logs and release bundles and record the negative scan in `specs/105-ndnsf-di-deployment-readiness/evidence/security-log-audit.md`.
- [ ] T098 Generate `release-gate.json` from immutable evidence with a mechanical six-dimension `minindnCandidateOverall` PASS/BLOCK and a separate `physicalProductionOverall` fixed to DEFERRED unless Spec 106 evidence exists.
- [ ] T099 Run `speckit-analyze` and strict post-implementation `speckit-audit`; record findings and remediation status in `specs/105-ndnsf-di-deployment-readiness/evidence/post-implementation-audit.md`.
- [ ] T100 Run `speckit-converge` to append any unbuilt or unsupported requirement as new unchecked tasks rather than weakening the spec.
- [ ] T101 Synchronize English/Chinese README, runtime workflow, architecture, build/test, experiment and release documentation in `README.md`, `NDNSF-DistributedInference/README.md`, `README_ch.md`, and `docs/`.
- [ ] T102 Synchronize CodeGraph, agent context and GSD state, verify clean ownership, commit the completed feature in logical slices, and play the completion bell.

---

## Dependencies and Execution Order

```text
Phase 1 baseline
  -> Phase 2 shared contracts
  -> US1 evidence truth (hard gate)
  -> US2 real Qwen (hard gate)
  -> US3 measured planning
  -> US4 bounded recovery
  -> US5 operator deployment
  -> closure/release decision
```

- US1 and US2 are both P1, but US2 performance evidence depends on US1 truth.
- US3 must use US2's real runtime and US1's evidence identity.
- US4 depends on US3 so replacement never uses stale/configured-only capacity.
- US5 begins packaging tests after Phase 2; local operations remain blocked by
  US1-US4 MiniNDN gates.
- Physical hardware work is outside Spec 105 and cannot interrupt its completion.

## Parallel Opportunities

- T002-T004 inventory different surfaces.
- T009-T012 and T018-T020 create independent negative fixtures.
- US2 tensor/runtime tests and Python product fixtures can be authored in
  parallel before shared implementation.
- US3 probe tests and planner tests are independent until integration.
- US4 scheduler C++ tests and user recovery Python tests are independent.
- US5 CLI/profile/packaging tests can be authored in parallel, but local canary,
  operations and soak execution remain sequential and gated.

## Implementation Strategy

### MVP 1: Evidence Truth

Complete T001-T030. This is immediately valuable even if every later phase is
deferred because it prevents false real-compute claims.

### MVP 2: Credible Research Deployment

Complete through T048. This yields a bounded real-Qwen MiniNDN service with an
honest matched baseline.

### Controlled Pilot

Complete through T078 before local deployment-candidate operations. This closes
measured capacity and bounded failure semantics.

### Local MiniNDN Deployment Candidate

Complete T079-T102. `minindnCandidateOverall` is PASS only if local canary,
operations drills, 24-hour soak and all earlier gates exist. Physical production
readiness remains deferred to Spec 106 regardless of this result.
