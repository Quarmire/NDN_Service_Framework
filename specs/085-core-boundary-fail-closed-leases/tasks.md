# Tasks: Core Boundary And Fail-Closed Execution Leases

**Parent gates**: `specs/084-ndnsf-occam-simplification/contracts/`

**Tests**: Test-first. No production edit begins until T001-T004 pass and dirty
target ownership is explicit.

## Phase 1: Entry Gate And Frozen Tests

- [x] T001 Record current branch/HEAD, target-file dirty ownership, excluded files, and rollback boundaries in `evidence/entry-baseline.md`; BLOCK if any target dirty change lacks an owner.
- [ ] T002 Re-run parent Core/DI/Repo baseline commands and, before treatment, run at least three matched 60-second current coordinator-on multi-user campaigns with frozen topology/profile/load/seeds; link raw results and exact commands in `evidence/regression-baseline.md`.
- [ ] T003 [US1] Add `tests/python/test_ndnsf_di_execution_lease_fallback.py` that captures the current coordinator-unavailable/missing-`ExecutionLease` failure without changing production behavior.
- [x] T004 Add the external Python API decision for moved Core symbols and any temporary import adapter with owner/expiry in `evidence/python-api-decision.md`.
- [ ] T005 [P] [US1] Add C++ state-machine tests for prepare/commit/atomic activate/abort/renew/release/validate, overlapping conflict keys, duplicate replay, idempotency conflict, ordinary expiry, execution hard-deadline, stale epoch, identity/digest/binding mismatch, and counters in `tests/unit-tests/generic-execution-lease.t.cpp`.
- [ ] T006 [P] [US1] Add C++/Python binding parity tests in `tests/python/test_ndnsf_execution_lease_table.py`; Python must call the bound C++ table rather than a second implementation.
- [ ] T007 [P] [US2] Add DI multi-provider transaction and cross-language codec tests in `tests/python/test_ndnsf_di_execution_lease_transaction.py`, `tests/python/test_ndnsf_di_execution_lease_codec.py`, and `tests/unit-tests/di-execution-lease-service.t.cpp` for identical C++/Python fixtures, malformed/unknown versions, prepare rejection, partial prepare/commit, delayed/duplicate response, cleanup, and no execution before all commits.
- [ ] T008 [P] [US2] Add restart/eviction tests in `tests/python/test_ndnsf_di_execution_lease_restart.py` for new boot epoch, delayed old messages, active binding pin, renewal race, expiry, and release loss.
- [ ] T009 [P] [US3] Add import-boundary tests in `tests/python/test_ndnsf_core_boundary_imports.py` for DI artifacts/deployment/retry and Repo producer target imports plus forbidden generic exports.

**Checkpoint**: Tests describe current failure and target behavior; production is unchanged.

## Phase 2: Canonical Core Execution Lease State

- [ ] T010 [US1] Create `ndn-service-framework/ExecutionLease.hpp` with application-neutral `GenericExecutionLease`, opaque binding proof/conflict keys, `ExecutionLeaseState`, `ExecutionLeaseResult`, reason constants, and thread-safe `ProviderExecutionLeaseTable` declarations.
- [ ] T011 Create `ndn-service-framework/ExecutionLease.cpp` implementing boot epoch, prepare, commit, atomic validate-and-activate, abort, renew, release, validate, ordinary/hard-deadline cleanup, idempotent replay, typed rejection, counters, and active committed/executing binding queries.
- [ ] T012 Add `ExecutionLease.cpp` and `tests/unit-tests/generic-execution-lease.t.cpp` to exact `wscript` source/test targets.
- [ ] T013 Bind the Core types/table in `pythonWrapper/src/ndnsf/_ndnsf.cpp`, including immutable snapshots/results and no callback while holding the table mutex.
- [ ] T014 Replace any Python execution-lease algorithm in `pythonWrapper/ndnsf/runtime_telemetry.py` with thin conversion/helpers over `_ndnsf`; export only generic names from `pythonWrapper/ndnsf/__init__.py`.
- [ ] T015 Run T005-T006, `./waf build --targets=unit-tests -j4`, and the full C++ unit suite; record exact results in `evidence/core-lease-implementation.md`.

**Checkpoint**: One C++ provider-local lease state machine exists with Python parity.

## Phase 3: DI Lease Service And Transaction

- [ ] T016 [US2] Create `NDNSF-DistributedInference/ndnsf_distributed_inference/deployment.py`, `NDNSF-DistributedInference/cpp/ndnsf-di/ExecutionLeaseService.hpp`, and `NDNSF-DistributedInference/cpp/ndnsf-di/ExecutionLeaseService.cpp` with one versioned deterministic `LeaseOperationRequest/Response` contract, descriptive `DeploymentRecord`, Python client/adapter, and C++ native-provider service over the bound Core table.
- [ ] T017 Register `/Inference/Control/Lease` in `examples/DI_NativeProviderExecutable.cpp` and Python providers through `ServiceProvider.add_context_handler`; add exact sources/tests to `examples/wscript`; update generated controller policies in `Experiments/NDNSF_DI_NativeTracer_Minindn.py`; require ordinary NDNSF permission, NAC-ABE, token/replay, authenticated context, and provider authorization.
- [ ] T018 Bind `requesterIdentity`, `providerName`, `serviceName`, and `requestId` from authenticated C++/Python request context, not payload fields; canonicalize the DI binding to proof bytes, have the provider derive non-empty conflict keys from trusted local worker/GPU slot inventory rather than requester input, and validate request/service/plan/binding/idempotency against the Core table.
- [ ] T019 Implement prepare-all/commit-all in `deployment.py` and wire `examples/python/NDNSF-DistributedInference/native_di_tracer/user_driver.py` to run it before collaboration; use bounded provider deadlines, cleanup, immutable plan digest, and carry provider lease IDs/bindings in assignments only after every commit.
- [ ] T020 Implement provider restart epoch, renewal/expiry, typed rejection diagnostics, and counters in both DI adapters; prove C++/Python fixture parity.
- [ ] T021 Integrate `ProviderExecutionLeaseTable` with `NDNSF-DistributedInference/cpp/ndnsf-di/NativeProviderHandler.hpp/.cpp`: atomically validate-and-activate before model work, release in every completion/error path, enforce hard deadline, and block fragment eviction for committed/executing bindings.
- [ ] T022 Export DI deployment/lease APIs from `NDNSF-DistributedInference/ndnsf_distributed_inference/__init__.py`, remove global deployment `refCount` from authority decisions while retaining descriptive migration parsing where required, and update exact provider/user callers in `evidence/di-caller-migration.md`.
- [ ] T023 Run T007-T008 and a concurrent two-user fake-runtime stress loop of at least 1,000 transactions; require zero conflicting commits and bounded live lease count in `evidence/di-lease-local-validation.md`.

**Checkpoint**: DI correctness is coordinator-independent and partial commit is non-executable.

## Phase 4: Core/Application Boundary Migration

- [ ] T024 [US3] Move `ExecutionArtifact`, `ExecutionArtifactSpec`, `ExecutionContext`, and materialization helpers from `pythonWrapper/ndnsf/service.py` into `NDNSF-DistributedInference/ndnsf_distributed_inference/artifact_deployment.py`; preserve serialized bytes and hash/path safety.
- [ ] T025 Replace deployment publish/get/evict/wait and acquire/release methods in `pythonWrapper/ndnsf/service.py` with DI-owned APIs and migrate every repository caller before deleting the old methods.
- [ ] T026 Remove the `GRANTED_LOCAL` fallback, missing import, coordinator/global-refCount authority calls, and swallowed lease-publication exceptions from `pythonWrapper/ndnsf/service.py` after T003/T007/T008 pass.
- [ ] T027 [US4] Move `RepoDataPlaneProducer` implementation/export to `NDNSF-DistributedRepo/pythonWrapper/py_repoclient/` and migrate exact Repo/DI/UAV callers without changing packet names, persistence, catalog, or repair.
- [ ] T028 Move application retry-by-error-string behavior out of generic `ndnsf`; DI retry accepts explicit idempotency metadata and tested retryable operations only.
- [ ] T029 Remove migrated DI/Repo/retry exports from `pythonWrapper/ndnsf/__init__.py`; coordination exports remain untouched for Spec 087.
- [ ] T030 Run `tools/maintenance/ndnsf_occam_audit.py` plus CodeGraph/exact `rg` queries and write zero/unexplained-caller decisions to `evidence/final-caller-inventory.md`.
- [ ] T031 Complete parent removal-gate records for each moved/deleted symbol with migration/deletion commits, exact tests, and rollback commands.

**Checkpoint**: Generic Core exports only generic mechanisms; app behavior lives with its owner.

## Phase 5: Regression And MiniNDN Acceptance

- [ ] T032 [US5] Run the full Core C++ suite, Core Python baseline, and `examples/run_security_regressions.sh`; record pass/skip deltas in `evidence/final-core-security.md`.
- [ ] T033 Run the parent DI and Repo focused tests plus NativeTracer/Qwen local execution and GUI preflight; record in `evidence/final-app-regressions.md`.
- [ ] T034 Create a capacity-overlap multi-user fixture and exact coordinator-off MiniNDN command under `specs/085-core-boundary-fail-closed-leases/fixtures/`; include authority failure, restart, and partial-commit scenarios.
- [ ] T035 Run at least three matched 60-second coordinator-off MiniNDN campaigns with frozen topology/profile/seed/load; record raw completion, conflict, lease reason/counter, cleanup, p50/p95, CPU/memory, and exact environment under `results/` and summarize in `evidence/minindn-acceptance.md`.
- [ ] T036 Verify zero synthetic lease, zero conflicting commit, zero execution before all commits, completion decrease <=0.5 percentage points, and median p50/p95 <=110% of baseline.
- [ ] T037 Run CodeGraph boundary audit, Spec Kit analyze/converge, GSD verify, and `speckit-audit` post-implementation; append real gaps before completion.
- [ ] T038 Mark 085 complete only after every removal gate is READY, dirty-file ownership is resolved, all evidence is reproducible, and rollback is independently tested.

## Dependency Order

```text
T001-T004 -> T005-T009
T005-T006 -> T010-T015
T010-T015 -> T016-T023
T003 + T007-T009 + T023 -> T024-T031
T024-T031 -> T032-T038
```

No task may modify V1 permission semantics, advisory coordinator ownership,
Repo storage/runtime architecture, Stream behavior, NDN-SVS, proposal slides,
or credentials.
