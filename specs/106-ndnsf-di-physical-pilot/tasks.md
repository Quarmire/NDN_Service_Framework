# Tasks: NDNSF-DI Physical Production Pilot

**Entry gate**: All tasks are deferred until Spec 105 passes and three physical
GPU nodes plus a second operator are available. The completed scope transfer is
recorded in `evidence/spec105-hardware-migration.md`; it is not an authorization
to execute any physical task.

## Phase 1: Immutable Candidate and Hardware Preflight

- [ ] T001 Record the passing Spec 105 candidate release, source commit and release-gate digest in `specs/106-ndnsf-di-physical-pilot/evidence/candidate-intake.md`.
- [ ] T002 Verify the Spec 105 candidate manifest contract and reject every non-PASS or digest-drift fixture in `tests/python/test_ndnsf_di_physical_pilot.py`.
- [ ] T003 Inventory three physical hosts, GPU UUIDs, memory, drivers, CUDA/ONNX Runtime, NFD and systemd versions in `specs/106-ndnsf-di-physical-pilot/evidence/hardware-inventory.md`.
- [ ] T004 Inventory real controller/user/provider/Repo identities, certificates, trust schema and policy ownership without recording secrets in `specs/106-ndnsf-di-physical-pilot/evidence/identity-inventory.md`.
- [ ] T005 Freeze exactly three hosts, primary/fallback roles, routes, devices and artifact placement in `packaging/ndnsf-di-systemd/config/physical/cluster.json`.
- [ ] T006 Freeze canary, security, restart, upgrade/rollback and 24-hour soak commands in `packaging/ndnsf-di-systemd/config/physical/campaigns/`.
- [ ] T007 Freeze host safety limits, clock-skew bound, telemetry age, deadlines and stop rules in `packaging/ndnsf-di-systemd/config/physical/acceptance.json`.
- [ ] T008 Run the Spec 106 pre-implementation audit and record hardware/operator availability in `specs/106-ndnsf-di-physical-pilot/evidence/preflight-audit.md`.

## Phase 2: Physical Profile and Security Fixtures

- [ ] T009 [P] Add failing physical-profile tests for host count, unique device UUIDs, CUDA-provider requirement/no-fallback, stage/fallback mapping, routes and configured-only resource rejection in `tests/python/test_ndnsf_di_physical_pilot.py`.
- [ ] T010 [P] Add failing doctor tests for identity, trust, NFD route, backend/device, artifact, plan, permissions and filesystem faults in `tests/python/test_ndnsf_runtime_doctor.py`.
- [ ] T011 [P] Add failing real-identity security campaign fixtures for forged trust, replayed tokens, wrong digest and stale attempt epoch in `tests/python/test_ndnsf_di_physical_pilot.py`.
- [ ] T012 Extend physical profile validation and doctor output, enable/verify the CUDA Execution Provider build, and add bounded NVIDIA device telemetry probing in `tools/ndnsf_runtime.py`, `NDNSF-DistributedInference/cpp/ndnsf-di/OnnxRuntimeModelRunner.cpp`, `NDNSF-DistributedInference/cpp/ndnsf-di/ProviderResourceProbe.cpp`, and `NDNSF-DistributedInference/ndnsf_distributed_inference/runtime_v1.py` without changing Qwen algorithms.
- [ ] T013 Generate protected identity/trust path references and verify release bundles contain no key material in `packaging/ndnsf-di-systemd/config/physical/`.
- [ ] T014 Generate the frozen production-security campaign from the approved fixtures in `packaging/ndnsf-di-systemd/config/physical/campaigns/security.json`.
- [ ] T015 Install the immutable release on the first clean three-host profile and record every command in `specs/106-ndnsf-di-physical-pilot/evidence/install-1.md`.
- [ ] T016 Repeat installation from clean hosts using only the runbook and record deviations in `specs/106-ndnsf-di-physical-pilot/evidence/install-2.md`.

## Phase 3: User Story 1 - Production Security (Priority: P1)

- [ ] T017 [US1] Execute positive permission bootstrap and bounded Qwen inference with real identities and record signed/encrypted path evidence in `specs/106-ndnsf-di-physical-pilot/evidence/security-positive.md`.
- [ ] T018 [US1] Execute forged-trust and unauthorized-provider negative cells and record exact fail-closed reasons in `specs/106-ndnsf-di-physical-pilot/evidence/security-negative.md`.
- [ ] T019 [US1] Execute UserToken, ProviderToken, execution-lease and attempt-epoch replay cells and record rejection evidence in `specs/106-ndnsf-di-physical-pilot/evidence/security-negative.md`.
- [ ] T020 [US1] Verify no keys, prompts, tokens, tensors or KV payload bytes occur in logs, metrics or release bundles in `specs/106-ndnsf-di-physical-pilot/evidence/security-log-audit.md`.
- [ ] T021 [US1] Generate the production security dimension from immutable positive/negative artifacts in `specs/106-ndnsf-di-physical-pilot/evidence/security-gate.json`.

## Phase 4: User Story 2 - Physical Operation and Recovery (Priority: P1)

- [ ] T022 [US2] Execute matched physical single-node and three-stage canaries with real CUDA/device evidence in unique `results/spec106-physical-canary-*` directories.
- [ ] T023 [US2] Verify physical telemetry source, device/boot binding, freshness and plan-commit predicates in `specs/106-ndnsf-di-physical-pilot/evidence/physical-telemetry.md`.
- [ ] T024 [US2] Compare token correctness, completion, throughput, p50/p95/p99, TTFT, inter-token and stage/resource decomposition in `specs/106-ndnsf-di-physical-pilot/evidence/physical-canary.md`.
- [ ] T025 [US2] Execute scheduled provider restart and same-three-node fallback, retaining old-boot/KV/attempt rejection in `specs/106-ndnsf-di-physical-pilot/evidence/restart-fallback.md`.
- [ ] T026 [US2] Execute N->N+1 upgrade and N+1->N rollback twice with cache incompatibility and Repo preservation checks in `specs/106-ndnsf-di-physical-pilot/evidence/upgrade-rollback.md`.
- [ ] T027 [US2] Re-run doctor/security/correctness canaries after rollback and record failed-release preservation in `specs/106-ndnsf-di-physical-pilot/evidence/post-rollback.md`.

## Phase 5: User Story 3 - Physical Soak (Priority: P2)

- [ ] T028 [US3] Preflight the frozen 24-hour 1 RPS campaign, output directory uniqueness, disk budget, monitoring and scheduled restart in `specs/106-ndnsf-di-physical-pilot/evidence/soak-preflight.md`.
- [ ] T029 [US3] Execute exactly one 24-hour physical soak with INFO metrics and no replacement run under `results/spec106-physical-soak-*`.
- [ ] T030 [US3] Validate every request outcome, correctness, completion interval, latency, stage/resource metrics, restart interruption and resource growth in `specs/106-ndnsf-di-physical-pilot/evidence/physical-soak.md`.
- [ ] T031 [US3] Run the full 11-item fallacy and evidence-integrity scan over canary and soak results in `specs/106-ndnsf-di-physical-pilot/evidence/physical-validation.md`.

## Phase 6: Production Release Decision

- [ ] T032 Run full maintained C++/Python/security/Repo/DI regressions against the exact candidate and record results in `specs/106-ndnsf-di-physical-pilot/evidence/integrated-regression.md`.
- [ ] T033 Generate `physicalProductionOverall=PASS|BLOCK` with mechanical missing/failed-artifact precedence in `specs/106-ndnsf-di-physical-pilot/evidence/release-gate.json`.
- [ ] T034 Run post-implementation `speckit-analyze`, `speckit-audit` and convergence and record all findings in `specs/106-ndnsf-di-physical-pilot/evidence/post-implementation-audit.md`.
- [ ] T035 Synchronize physical deployment, security, operations and rollback documentation in `packaging/ndnsf-di-systemd/README.md`, `README_ch.md` and `docs/`.
- [ ] T036 Synchronize CodeGraph, agent context and GSD state, verify worktree ownership, commit the physical pilot and play the completion bell.

## Dependencies

```text
Spec 105 PASS + hardware/operator available
  -> candidate/hardware preflight
  -> physical profile/security fixtures
  -> real security
  -> operation/recovery
  -> 24-hour soak
  -> production release decision
```
