# Tasks: NDNSF-DI OCI Deployment Adapters

**Input**: `spec.md`, `plan.md`, `research.md`, `data-model.md`, `contracts/`, `quickstart.md`
**Tests**: Contract, negative, integration, security, and bounded live acceptance tasks are mandatory.
**Authority**: Spec 108 closes container-substrate and candidate containerization evidence only. Spec 106 retains physical-production authority.

## Format

`- [ ] T### [P?] [US#?] Action with exact file path`

- `[P]`: can run in parallel after its phase prerequisites.
- `[US#]`: belongs to a user story; setup/foundation/final audit tasks omit it.
- Live iTiger tasks MUST use unique run IDs, bounded time, and no automatic rerun after a measured failure.

## Phase 1: Setup and protected baselines

**Purpose**: Establish the new package without disturbing current runtime, Spec 107 candidate work, or systemd rollback.

- [X] T001 Create the container package directories and ownership README at `packaging/ndnsf-di-container/README.md`
- [X] T002 [P] Create test directories and fixture policy at `tests/container/README.md`
- [X] T003 [P] Record the current systemd operator commands and rollback behavior in `tests/container/fixtures/systemd-baseline.json`
- [X] T004 [P] Record current `OnnxRuntimeModelRunner` CPU/GPU/fallback evidence fields in `tests/container/fixtures/backend-evidence-baseline.json`
- [X] T005 [P] Add a fixture from iTiger substrate job 145855, explicitly labeled non-candidate and non-production, at `tests/container/fixtures/itiger/job-145855-substrate.json`
- [X] T006 Add package-wide shell/Python test entrypoint at `tests/container/run.sh`
- [X] T007 Add deterministic temporary-directory and command-mocking helpers at `tests/container/lib/test_helpers.sh`
- [X] T008 Add Python test dependencies and version locks at `tests/container/requirements.txt`
- [X] T009 Add generated-artifact, SIF, local credential, and evidence scratch exclusions to `.gitignore`
- [X] T010 Verify existing `packaging/ndnsf-di-systemd/` install/rollback tests remain green and store the command/result in `specs/108-ndnsf-di-container-deployment/checklists/pre-implementation-audit.md`

**Checkpoint**: New work has isolated paths and protected baselines; no runtime protocol or existing package behavior changed.

---

## Phase 2: Foundational OCI, profile, evidence, and adapter contracts

**Purpose**: Build the shared layer required by both execution adapters.

### Contract tests first

- [X] T011 [P] Add valid CPU Compose profile fixture at `tests/container/fixtures/profiles/cloud-cpu-valid.yaml`
- [X] T012 [P] Add valid iTiger RTX 5000 profile fixture at `tests/container/fixtures/profiles/itiger-rtx5000-valid.yaml`
- [X] T013 [P] Add invalid tag-only release and mismatched-digest fixtures under `tests/container/fixtures/releases/`
- [X] T014 [P] Add invalid mixed-adapter, GPU-without-GRES, `/home` bulk-storage, and multi-node-without-network-evidence fixtures under `tests/container/fixtures/profiles/invalid/`
- [X] T015 Add profile JSON Schema contract tests at `tests/container/contract/test_profile_schema.py`
- [X] T016 Add evidence JSON Schema contract and authority-invariant tests at `tests/container/contract/test_evidence_schema.py`
- [X] T017 Add OCI release manifest schema and digest-pinning tests at `tests/container/contract/test_release_manifest.py`
- [X] T018 Add operator exit-code/stdout/stderr contract tests at `tests/container/contract/test_operator_cli.py`
- [X] T019 Add adapter-interface conformance tests at `tests/container/contract/test_adapter_interface.py`

### Common implementation

- [X] T020 Define OCI release manifest schema at `packaging/ndnsf-di-container/schemas/oci-release.schema.json`
- [X] T021 Copy and version the deployment profile schema at `packaging/ndnsf-di-container/schemas/deployment-profile.schema.json`
- [X] T022 Copy and version the deployment evidence schema at `packaging/ndnsf-di-container/schemas/deployment-evidence.schema.json`
- [X] T023 Implement YAML loading, environment expansion allowlist, and schema validation at `packaging/ndnsf-di-container/lib/profile.py`
- [X] T024 Implement cross-field validation for adapter exclusivity, GPU/GRES, storage, fallback, and network gates at `packaging/ndnsf-di-container/lib/profile.py`
- [X] T025 Implement OCI manifest loading, digest normalization, and tag-only rejection at `packaging/ndnsf-di-container/lib/release.py`
- [X] T026 Implement local artifact checksum and OCI/SIF materialization records at `packaging/ndnsf-di-container/lib/release.py`
- [X] T027 Define the adapter lifecycle interface at `packaging/ndnsf-di-container/lib/adapters/base.py`
- [X] T028 Implement common evidence initialization, canonical JSON, checksums, and outcome rules at `packaging/ndnsf-di-container/lib/evidence.py`
- [X] T029 Implement substrate/candidate/physical-production authority evaluation with forced `physicalProduction=DEFERRED` at `packaging/ndnsf-di-container/lib/evidence.py`
- [X] T030 Implement secret-value and credential-pattern redaction at `packaging/ndnsf-di-container/lib/redaction.py`
- [X] T031 Implement manifest-based evidence staging and atomic durable promotion at `packaging/ndnsf-di-container/lib/evidence.py`
- [X] T032 Implement the non-interactive command dispatcher and documented exit codes at `packaging/ndnsf-di-container/bin/ndnsf-di-deploy`
- [X] T033 Add `validate-profile`, `verify-release`, and `verify-evidence` commands at `packaging/ndnsf-di-container/bin/ndnsf-di-deploy`
- [X] T034 Run T015-T019 and record the exact command/result in `specs/108-ndnsf-di-container-deployment/checklists/pre-implementation-audit.md`

**Checkpoint**: Profiles, releases, adapters, and evidence have one tested contract before either runtime integration exists.

---

## Phase 3: User Story 1 - OCI build and single-host Docker Compose (P1)

**Goal**: Build/pull one immutable release and operate a CPU cloud node without target-host compilation.

**Independent test**: Build a CPU image, install a digest-pinned release on a clean Compose host, pass readiness/smoke, recreate it, and verify durable identity/state.

### Tests first

- [X] T035 [P] [US1] Add OCI build-context secret exclusion test at `tests/container/unit/test_oci_build_context.py`
- [X] T036 [P] [US1] Add reproducible manifest/SBOM/provenance test at `tests/container/unit/test_oci_release.py`
- [X] T037 [P] [US1] Add Compose render and host-scoped NFD contract test at `tests/container/unit/test_compose_render.py`
- [X] T038 [P] [US1] Add identity/state mount persistence test at `tests/container/unit/test_compose_storage.py`
- [X] T039 [US1] Add local CPU image smoke test at `tests/container/integration/test_oci_cpu_smoke.sh`
- [X] T040 [US1] Add single-host Compose readiness/recreate test at `tests/container/integration/test_compose_cpu_node.sh`

### OCI source

- [X] T041 [P] [US1] Add pinned CPU build inputs at `packaging/ndnsf-di-container/oci/locks/cpu.lock`
- [X] T042 [P] [US1] Add pinned common OS/NDN dependency inputs at `packaging/ndnsf-di-container/oci/locks/common.lock`
- [X] T043 [US1] Implement multi-stage CPU OCI build at `packaging/ndnsf-di-container/oci/Dockerfile.cpu`
- [X] T044 [US1] Add container entrypoint that execs one declared role and does not launch systemd at `packaging/ndnsf-di-container/oci/scripts/entrypoint.sh`
- [X] T045 [US1] Add OCI health probe using existing runtime/NFD readiness semantics at `packaging/ndnsf-di-container/oci/scripts/healthcheck.sh`
- [X] T046 [US1] Implement build manifest, SBOM/provenance references, and digest capture at `packaging/ndnsf-di-container/oci/scripts/build-release.sh`
- [X] T047 [US1] Implement OCI layer/build-context secret scan at `packaging/ndnsf-di-container/oci/scripts/scan-release.sh`

### Compose adapter

- [X] T048 [US1] Add host-scoped NFD and application service definitions at `packaging/ndnsf-di-container/adapters/docker-compose/compose.yaml`
- [X] T049 [US1] Add CPU cloud profile template at `packaging/ndnsf-di-container/adapters/docker-compose/profiles/cloud-cpu.yaml`
- [X] T050 [US1] Implement Compose install/render/preflight/start/stop/status operations at `packaging/ndnsf-di-container/lib/adapters/docker_compose.py`
- [X] T051 [US1] Implement explicit Unix-socket ownership/mount validation at `packaging/ndnsf-di-container/lib/adapters/docker_compose.py`
- [ ] T052 [US1] Run T035-T040 and retain canonical CPU release/Compose evidence under `results/spec108-container/cloud-cpu/`

> T052 is currently `BLOCKED`: this managed execution environment cannot access `/var/run/docker.sock`; measured diagnostics are retained in `results/spec108-container/cloud-cpu/blocked-summary.json`. Offline tests and `docker compose config` are PASS, but no image/readiness/recreate PASS is claimed.

**Checkpoint**: US1 is independently deployable and reversible on one CPU cloud host.

---

## Phase 4: User Story 2 - Multi-host cloud NFD routing (P1)

**Goal**: Connect cloud deployments through explicit, evidenced NFD faces/routes.

**Independent test**: Two hosts pass TCP/UDP and NFD route preflight, then complete one remote service invocation.

- [ ] T053 [P] [US2] Add two-node topology fixtures at `tests/container/fixtures/profiles/cloud-two-node/`
- [ ] T054 [P] [US2] Add missing-port, wrong-route, and duplicate-node-identity negative fixtures at `tests/container/fixtures/profiles/invalid/cloud-network/`
- [ ] T055 [US2] Add route rendering and prefix ownership tests at `tests/container/unit/test_compose_routes.py`
- [ ] T056 [US2] Add TCP/UDP 6363 preflight tests at `tests/container/unit/test_network_preflight.py`
- [ ] T057 [US2] Implement remote endpoint and route validation at `packaging/ndnsf-di-container/lib/adapters/docker_compose.py`
- [ ] T058 [US2] Implement idempotent NFD face/route apply and observation at `packaging/ndnsf-di-container/adapters/docker-compose/scripts/configure-routes.sh`
- [ ] T059 [US2] Implement bounded TCP/UDP reachability checks at `packaging/ndnsf-di-container/adapters/docker-compose/scripts/network-preflight.py`
- [ ] T060 [US2] Add node A and node B profile examples at `packaging/ndnsf-di-container/adapters/docker-compose/profiles/cloud-two-node/`
- [ ] T061 [US2] Add a two-host remote invocation integration test at `tests/container/integration/test_compose_two_host.sh`
- [ ] T062 [US2] Verify non-selected/remote service behavior still uses existing NDNSF-DI security and selection regressions via `tests/container/integration/test_packaged_minindn_regressions.sh`
- [ ] T063 [US2] Retain canonical two-host network and invocation evidence under `results/spec108-container/cloud-two-host/`

**Checkpoint**: Cloud multi-host readiness is based on measured routes and invocation, not container health alone.

---

## Phase 5: User Story 3 - iTiger Slurm + Apptainer execution (P1)

**Goal**: Submit bounded, digest-bound iTiger jobs with correct storage, GRES, GPU mapping, and durable evidence.

**Independent test**: One five-minute RTX 5000 job passes Slurm allocation, SIF, `--nv`, compute `/tmp`, and durable evidence acceptance.

### Offline tests first

- [X] T064 [P] [US3] Add `sinfo`, `scontrol`, `squeue`, and `sacct` fixtures for observed iTiger output at `tests/container/fixtures/itiger/slurm/`
- [X] T065 [P] [US3] Add login/compute Apptainer version-difference fixtures at `tests/container/fixtures/itiger/apptainer/`
- [X] T066 [P] [US3] Add `/home`, `/project`, compute `/tmp`, quota-full, and partial-copy fixtures at `tests/container/fixtures/itiger/storage/`
- [X] T067 [P] [US3] Add GRES/node mapping fixtures for all three GPU types at `tests/container/fixtures/itiger/gres/`
- [X] T068 [US3] Add Slurm resource rendering and injection-safety tests at `tests/container/unit/test_slurm_render.py`
- [X] T069 [US3] Add scheduler state/exit parsing tests at `tests/container/unit/test_slurm_state.py`
- [X] T070 [US3] Add iTiger storage policy and quota-signal tests at `tests/container/unit/test_itiger_storage.py`
- [X] T071 [US3] Add OCI-to-SIF digest/materialization tests at `tests/container/unit/test_apptainer_materialization.py`
- [X] T072 [US3] Add job trap, original-exit preservation, and partial-promotion tests at `tests/container/unit/test_slurm_evidence_trap.py`
- [X] T073 [US3] Add GRES-to-physical-GPU/container-mapping tests at `tests/container/unit/test_itiger_gpu_mapping.py`
- [X] T074 [US3] Add submit-exactly-once/no-auto-rerun test at `tests/container/unit/test_slurm_submit.py`

### Adapter implementation

- [X] T075 [P] [US3] Add `bigTiger` RTX 5000 five-minute profile at `packaging/ndnsf-di-container/adapters/slurm-apptainer/profiles/itiger-rtx5000.yaml`
- [X] T076 [P] [US3] Add parameterized RTX 6000 profile at `packaging/ndnsf-di-container/adapters/slurm-apptainer/profiles/itiger-rtx6000.yaml`
- [X] T077 [P] [US3] Add parameterized H100 80GB profile at `packaging/ndnsf-di-container/adapters/slurm-apptainer/profiles/itiger-h100.yaml`
- [X] T078 [US3] Implement login-node discovery for account/QOS/partition/GRES and path policy at `packaging/ndnsf-di-container/adapters/slurm-apptainer/scripts/preflight-login.sh`
- [X] T079 [US3] Implement actual-path capacity/quota capture without treating shared `df` as quota at `packaging/ndnsf-di-container/adapters/slurm-apptainer/scripts/check-storage.py`
- [X] T080 [US3] Implement pinned OCI-to-SIF materialization and checksum record at `packaging/ndnsf-di-container/adapters/slurm-apptainer/scripts/materialize-sif.sh`
- [X] T081 [US3] Add deterministic `sbatch` template with explicit resources and no embedded secrets at `packaging/ndnsf-di-container/adapters/slurm-apptainer/templates/ndnsf-di.sbatch.in`
- [X] T082 [US3] Implement compute-node allocation, Apptainer version, scratch, and GPU preflight at `packaging/ndnsf-di-container/adapters/slurm-apptainer/scripts/preflight-compute.sh`
- [X] T083 [US3] Implement `apptainer exec --nv` invocation with explicit binds and clean environment policy at `packaging/ndnsf-di-container/adapters/slurm-apptainer/scripts/run-container.sh`
- [X] T084 [US3] Implement host/container GPU UUID/model observation and Slurm mapping at `packaging/ndnsf-di-container/adapters/slurm-apptainer/scripts/collect-gpu.py`
- [X] T085 [US3] Implement bounded compute-node `/tmp` write/fsync validation at `packaging/ndnsf-di-container/adapters/slurm-apptainer/scripts/check-scratch.py`
- [X] T086 [US3] Implement exit/TERM/INT trap, canonical manifest, checksums, and project promotion at `packaging/ndnsf-di-container/adapters/slurm-apptainer/scripts/finalize-evidence.sh`
- [X] T087 [US3] Implement Slurm preflight/materialize/render/submit/status/wait/cancel/log/evidence operations at `packaging/ndnsf-di-container/lib/adapters/slurm_apptainer.py`
- [X] T088 [US3] Add bounded polling, `sacct` terminal reconciliation, and cancellation reason capture at `packaging/ndnsf-di-container/lib/adapters/slurm_apptainer.py`
- [X] T089 [US3] Run T068-T074 entirely offline and preserve results at `results/spec108-container/offline-slurm-contract/`
- [ ] T090 [US3] Submit exactly one final five-minute RTX 5000 substrate acceptance job and preserve the unique `/project/$USER/ndnsf-di/evidence/<run-id>` bundle at `results/spec108-container/itiger-rtx5000-substrate/manifest.json`

**Checkpoint**: iTiger substrate PASS is admissible but explicitly not candidate inference or physical-production PASS.

---

## Phase 6: User Story 4 - Truthful CPU/GPU/ONNX Runtime backend (P1)

**Goal**: Correlate resource visibility with the backend actually selected by NDNSF-DI and fail closed on undeclared fallback.

**Independent test**: CPU, GPU PASS, GPU/no-fallback failure, and explicit fallback/degraded cases satisfy the same evidence rules on both adapters.

- [ ] T091 [P] [US4] Add pinned GPU user-space dependency lock at `packaging/ndnsf-di-container/oci/locks/gpu.lock`
- [ ] T092 [P] [US4] Add driver/CUDA/ONNX Runtime compatibility matrix at `packaging/ndnsf-di-container/oci/compatibility/gpu-matrix.yaml`
- [ ] T093 [US4] Add GPU OCI build using user-space CUDA/ORT only at `packaging/ndnsf-di-container/oci/Dockerfile.gpu`
- [ ] T094 [US4] Add Docker GPU profile and NVIDIA Container Toolkit preflight test at `tests/container/unit/test_docker_gpu_preflight.py`
- [ ] T095 [US4] Add Apptainer `--nv` and no-NVIDIA-Container-Toolkit requirement test at `tests/container/unit/test_apptainer_gpu_preflight.py`
- [ ] T096 [US4] Add backend evidence correlation and GPU UUID matching tests at `tests/container/unit/test_backend_evidence.py`
- [ ] T097 [US4] Add GPU requested/no allocation/no fallback negative test at `tests/container/integration/test_gpu_fail_closed.sh`
- [ ] T098 [US4] Add explicit fallback/degraded/not-GPU-PASS test at `tests/container/integration/test_gpu_fallback_degraded.sh`
- [ ] T099 [US4] Implement Docker driver/toolkit and runtime compatibility checks at `packaging/ndnsf-di-container/lib/adapters/docker_compose.py`
- [ ] T100 [US4] Implement Apptainer host-driver/user-space compatibility checks at `packaging/ndnsf-di-container/lib/adapters/slurm_apptainer.py`
- [ ] T101 [US4] Map existing runtime provider/device evidence into `BackendCompatibilityRecord` at `packaging/ndnsf-di-container/lib/evidence.py`
- [ ] T102 [US4] Reject GPU PASS when observed provider is CPU or fallback occurred at `packaging/ndnsf-di-container/lib/evidence.py`
- [ ] T103 [US4] Run one candidate-bound iTiger RTX 5000 ONNX Runtime inference job and preserve release/backend/model/result evidence under `results/spec108-container/itiger-rtx5000-candidate/`

**Checkpoint**: GPU acceptance proves actual candidate inference provider, not only `nvidia-smi` visibility.

---

## Phase 7: User Story 5 - Identity, secrets, storage, and evidence safety (P1)

**Goal**: Keep unique identity and secret material outside immutable artifacts and retain only redacted, durable evidence.

**Independent test**: Two deployments use distinct identities; OCI/SIF/evidence scans find zero private-key/password/token disclosure; scratch loss does not destroy accepted evidence.

- [ ] T104 [P] [US5] Add synthetic identity/secret fixtures containing detectable markers under `tests/container/fixtures/secrets/`
- [ ] T105 [P] [US5] Add duplicate identity, writable secret mount, and identity-in-build-context negative tests at `tests/container/unit/test_identity_policy.py`
- [ ] T106 [P] [US5] Add OCI/SIF filesystem and string scan tests at `tests/container/integration/test_artifact_secret_scan.sh`
- [ ] T107 [P] [US5] Add logs/evidence redaction and false-positive allowlist tests at `tests/container/unit/test_redaction.py`
- [ ] T108 [US5] Implement identity reference ownership, mode, uniqueness, and expected-name validation at `packaging/ndnsf-di-container/lib/profile.py`
- [ ] T109 [US5] Implement adapter-specific read-only identity/secret bind rendering at `packaging/ndnsf-di-container/lib/adapters/docker_compose.py`
- [ ] T110 [US5] Implement minimal read-only Apptainer identity/secret binds at `packaging/ndnsf-di-container/lib/adapters/slurm_apptainer.py`
- [ ] T111 [US5] Add release build-context denylist and positive allowlist at `packaging/ndnsf-di-container/oci/.dockerignore`
- [ ] T112 [US5] Add artifact/evidence secret scanner wrapper at `packaging/ndnsf-di-container/oci/scripts/scan-secrets.py`
- [ ] T113 [US5] Add storage cleanup dry-run protection for identities, accepted evidence, active/prior releases, and referenced models at `packaging/ndnsf-di-container/lib/cleanup.py`
- [ ] T114 [US5] Run two-identity restart/materialization and secret-scan acceptance, retaining only redacted results under `results/spec108-container/security/`

**Checkpoint**: Same OCI release is reusable without cloned credentials, and no accepted artifact exposes secret material.

---

## Phase 8: User Story 6 - Operations, upgrade, cancellation, and rollback (P2)

**Goal**: Provide predictable adapter-specific operations and preserve lineage across failure/recovery.

**Independent test**: Compose upgrade failure rolls back; a Slurm job is submitted, inspected, cancelled, and archived without a login-node daemon.

- [ ] T115 [P] [US6] Add Compose failed-upgrade/prior-digest rollback fixtures at `tests/container/fixtures/compose/rollback/`
- [ ] T116 [P] [US6] Add Slurm PENDING/RUNNING/CANCELLED/TIMEOUT/PREEMPTED fixtures at `tests/container/fixtures/itiger/lifecycle/`
- [ ] T117 [US6] Add Compose upgrade and rollback state-machine tests at `tests/container/unit/test_compose_upgrade.py`
- [ ] T118 [US6] Add Slurm wait/cancel/terminal-evidence state-machine tests at `tests/container/unit/test_slurm_lifecycle.py`
- [ ] T119 [US6] Implement last-accepted/prior release registry at `packaging/ndnsf-di-container/lib/release.py`
- [ ] T120 [US6] Implement staged Compose upgrade, readiness gate, and digest rollback at `packaging/ndnsf-di-container/lib/adapters/docker_compose.py`
- [ ] T121 [US6] Implement bounded status/log collection for both adapters at `packaging/ndnsf-di-container/bin/ndnsf-di-deploy`
- [ ] T122 [US6] Implement exact-job cancellation and already-terminal handling at `packaging/ndnsf-di-container/lib/adapters/slurm_apptainer.py`
- [ ] T123 [US6] Add login-node process audit proving no post-command NDNSF-DI/NFD daemon at `tests/container/live/test_itiger_no_login_daemon.sh`
- [ ] T124 [US6] Run Compose failed-upgrade rollback acceptance and preserve evidence under `results/spec108-container/cloud-rollback/`
- [ ] T125 [US6] Run one bounded iTiger cancellation-path test without consuming a GPU when CPU allocation suffices and preserve evidence under `results/spec108-container/itiger-cancel/`
- [ ] T126 [US6] Verify systemd fallback installation remains documented and functional after container package installation in `tests/container/integration/test_systemd_fallback.sh`

**Checkpoint**: Operational failure paths preserve exact release/job truth and a usable rollback surface.

---

## Phase 9: Cross-adapter integration and acceptance gates

**Purpose**: Prove shared semantics, close supported substrate slices, and keep unsupported claims fail-closed.

- [ ] T127 [P] Add a common adapter conformance matrix runner at `tests/container/contract/run_adapter_matrix.py`
- [ ] T128 [P] Add evidence mutation tests for tag-only, wrong SIF digest, wrong job ID, partial promotion, secret finding, and false physical PASS at `tests/container/contract/test_evidence_mutations.py`
- [ ] T129 [P] Add exact GRES profile/render tests for H100, RTX 6000, and RTX 5000 at `tests/container/contract/test_itiger_gres_matrix.py`
- [ ] T130 Add packaged MiniNDN network/security regression orchestration at `tests/container/integration/test_packaged_minindn_regressions.sh`
- [ ] T131 Add candidate/release/evidence cross-link validation at `tests/container/contract/test_candidate_lineage.py`
- [ ] T132 Add iTiger multi-node TCP/UDP 6363 and NFD face/route probe at `packaging/ndnsf-di-container/adapters/slurm-apptainer/scripts/probe-multinode-network.sh`
- [ ] T133 Add multi-node probe parser and admissibility tests at `tests/container/unit/test_itiger_network_probe.py`
- [ ] T134 Submit exactly one five-minute two-node CPU network probe and retain PASS/FAIL as measured evidence under `results/spec108-container/itiger-multinode-network/`
- [ ] T135 Enable multi-node profile validation only if T134 supplies an admissible PASS reference at `packaging/ndnsf-di-container/lib/profile.py`
- [ ] T136 Run the full offline contract/unit suite with zero network/GPU dependency and retain JUnit/summary at `results/spec108-container/offline-suite/`
- [ ] T137 Run the local OCI/Compose integration suite and retain canonical evidence at `results/spec108-container/local-integration/`
- [ ] T138 Run existing NDNSF-DI/MiniNDN security regressions against the packaged candidate and retain command/result pointers at `results/spec108-container/minindn-regressions/`
- [ ] T139 Run a canonical OCI/SIF/evidence secret scan with zero findings and retain scanner/version/digest data at `results/spec108-container/secret-scan/`
- [ ] T140 Validate the final RTX 5000 substrate and candidate bundles against the checked-in schema at `results/spec108-container/itiger-validation/`
- [ ] T141 Submit one RTX 6000 compatibility probe only if the release claims RTX 6000 support and preserve the measured outcome under `results/spec108-container/itiger-rtx6000/`
- [ ] T142 Submit one H100 compatibility probe only if the release claims H100 support and preserve the measured outcome under `results/spec108-container/itiger-h100/`
- [ ] T143 Compare OCI source digest, SIF digest, candidate revision, model digest, and backend evidence across all live bundles in `results/spec108-container/cross-runtime-lineage.json`
- [ ] T144 Assert all Spec 108 evidence has `physicalProduction=DEFERRED` and owner `Spec 106` in `tests/container/contract/test_physical_authority.py`
- [ ] T145 Generate the acceptance summary without converting failed/deferred cells to PASS at `results/spec108-container/acceptance-summary.json`
- [ ] T146 Re-run `tests/container/run.sh` from a clean environment and record exact commands, durations, pass/fail/skip counts, and artifact paths in `specs/108-ndnsf-di-container-deployment/checklists/pre-implementation-audit.md`

**Checkpoint**: Every supported claim has candidate-bound evidence; failed or unverified matrix cells remain explicit.

---

## Phase 10: Documentation, audit, and Spec 106 handoff

**Purpose**: Make deployment reproducible and prevent authority or implementation drift.

- [ ] T147 [P] Update cloud OCI/Compose operator guide in `packaging/ndnsf-di-container/docs/cloud-compose.md`
- [ ] T148 [P] Update iTiger VPN/SSH/storage/Slurm/Apptainer operator guide in `packaging/ndnsf-di-container/docs/itiger-slurm-apptainer.md`
- [ ] T149 [P] Document GPU driver/toolkit/Apptainer differences and compatibility failure handling in `packaging/ndnsf-di-container/docs/gpu-compatibility.md`
- [ ] T150 [P] Document identity provisioning, secret rotation, and redaction in `packaging/ndnsf-di-container/docs/security.md`
- [ ] T151 [P] Document Compose upgrade/rollback and Slurm cancel/resubmit/recovery in `packaging/ndnsf-di-container/docs/operations.md`
- [ ] T152 [P] Document `/home`, `/project`, compute `/tmp`, quota checks, retention, and cleanup in `packaging/ndnsf-di-container/docs/storage.md`
- [ ] T153 Update repository `README.md` with the OCI source and two adapter entry points
- [ ] T154 Update `README_zh-CN.md` with the same deployment scope and commands as T153
- [ ] T155 Reconcile every FR/SC/task/evidence link in `specs/108-ndnsf-di-container-deployment/traceability.md`
- [ ] T156 Run Spec Kit analyze and resolve all unambiguous cross-artifact inconsistencies in `specs/108-ndnsf-di-container-deployment/`
- [ ] T157 Run strict code-aware Spec Kit audit against current CodeGraph/source/evidence and record the verdict in `specs/108-ndnsf-di-container-deployment/checklists/pre-implementation-audit.md`
- [ ] T158 Demonstrate clean rollback to `packaging/ndnsf-di-systemd/` and record non-destructive migration evidence in `specs/108-ndnsf-di-container-deployment/checklists/pre-implementation-audit.md`
- [ ] T159 Create a Spec 106 handoff that lists physical GPU performance, production security, real-network/UAV, and soak work without claiming completion at `specs/106-ndnsf-di-physical-deployment/handoffs/spec108-container.md`
- [ ] T160 Run the completion bell and record final status, tests, evidence paths, remaining deferred work, and recommended next step in `specs/108-ndnsf-di-container-deployment/completion-summary.md`

---

## Dependencies and execution order

### Phase dependencies

- Phase 1 has no dependencies.
- Phase 2 depends on Phase 1 and blocks all adapter work.
- Phase 3 depends on Phase 2.
- Phase 4 depends on Phase 3.
- Phase 5 depends on Phase 2 and can proceed in parallel with Phases 3-4 after the shared contract is stable.
- Phase 6 depends on the OCI source (Phase 3) and iTiger adapter (Phase 5) for live GPU acceptance.
- Phase 7 depends on Phase 2 and must finish before any artifact is accepted.
- Phase 8 depends on the relevant adapter implementation.
- Phase 9 depends on Phases 3-8; T135 depends specifically on a PASS from T134.
- Phase 10 depends on the supported Phase 9 evidence matrix.

### User-story dependencies

- US1 is the cloud MVP and OCI release source.
- US2 extends US1 but does not block iTiger single-node work.
- US3 depends only on the common foundation and an OCI release fixture for offline work; its final live candidate test depends on US1's final release.
- US4 consumes both adapter and runtime evidence; it does not change runtime provider selection.
- US5 is cross-cutting and gates acceptance of US1-US4.
- US6 uses completed adapters and can be tested independently per adapter.

### Safe parallel examples

- T011-T014 profile/release fixtures.
- T035-T038 OCI/Compose unit tests.
- T041-T042 dependency locks.
- T064-T067 iTiger fixtures.
- T075-T077 iTiger GRES profiles.
- T104-T107 security tests.
- T115-T116 lifecycle fixtures.
- T147-T152 operator documents after interfaces stabilize.

## Required acceptance sequence

1. Offline schema, renderer, parser, redaction, and evidence tests.
2. Local CPU OCI build and single-host Compose.
3. Cloud route and packaged MiniNDN regressions.
4. iTiger five-minute substrate job.
5. Candidate-bound RTX 5000 inference job.
6. Optional RTX 6000/H100 probes only for claimed support.
7. Multi-node iTiger remains gated by its own measured probe.
8. Strict audit, systemd rollback evidence, and Spec 106 handoff.

## Completion rule

Spec 108 is complete only when all applicable tasks are checked with admissible evidence, unsupported cells are explicitly `DEFERRED`/`BLOCKED`, both adapters conform to the common contracts, and no artifact claims physical-production PASS. A queue delay, scheduler failure, GPU incompatibility, or network failure is preserved as the measured outcome; it is not retried or rewritten into success without a separately authorized new run.
