# Implementation Plan: NDNSF-DI OCI Deployment Adapters

**Branch**: `108-ndnsf-di-container-deployment` | **Date**: 2026-07-12 | **Spec**: [spec.md](spec.md)
**Input**: Feature specification from `specs/108-ndnsf-di-container-deployment/spec.md`

## Summary

Create one immutable OCI build/release source for NDNSF-DI and place environment-specific execution behind two thin adapters: Docker Compose for long-lived cloud hosts and Slurm + Apptainer for bounded iTiger jobs. Preserve the existing runtime, NDN security model, evidence semantics, candidate lineage, and systemd rollback surface. Add explicit iTiger storage classes, GPU GRES selection, OCI-to-SIF identity, job lifecycle handling, and substrate/candidate authority separation.

## Technical Context

**Languages/Tools**: Bash, Python 3, YAML/JSON, Dockerfile/Containerfile, Docker Compose, Slurm CLI, Apptainer 1.3.x
**Primary Dependencies**: current NDNSF-DI binaries and NDN stack, Docker Engine/Compose for cloud, Slurm + Apptainer for iTiger, NVIDIA host driver, CUDA/ONNX Runtime user-space libraries in GPU OCI images
**Storage**: cloud bind mounts/volumes; iTiger `/home` for small account config, `/project/$USER/ndnsf-di` for durable artifacts, compute-node `/tmp` for per-job scratch
**Testing**: shell/Python contract tests, JSON Schema validation, OCI/SIF secret scans, Compose integration, Slurm dry-run/parser fixtures, bounded live iTiger acceptance jobs
**Target Platform**: supported Linux cloud VM and iTiger `bigTiger` compute nodes
**Project Type**: deployment packaging and execution adapters
**Performance Goals**: packaging overhead must be measurable and bounded; candidate performance PASS remains Spec 106 authority
**Constraints**: no secrets in images; no login-node daemon; digest pinning; fail-closed GPU; no unverified multi-node iTiger claim; preserve physical-production `DEFERRED`
**Scale/Scope**: one OCI release, CPU/GPU variants as needed, two adapters, single-node iTiger first, multi-host cloud Compose

## Constitution Check

| Principle | Design response | Gate |
|---|---|---|
| Security and evidence | External identities, read-only secret mounts, redaction, digest-bound evidence | Required before integration acceptance |
| Dynamic runtime API | Packages existing generic runtime; no generated static API or wire changes | Code review must confirm no runtime protocol fork |
| MiniNDN default | Functional network/security regression remains MiniNDN; iTiger jobs validate execution substrate and candidate-bound container behavior only | Required in test plan |
| Honest physical authority | Spec 108 does not promote iTiger substrate results to physical-production PASS | Schema and negative test |
| Incremental migration | Existing systemd package remains intact and usable | Rollback acceptance |
| Reproducibility | OCI digest, SIF digest, commands, resources, versions, and results are retained | Evidence schema gate |

**Pre-design verdict**: PASS. No constitution exception is required.

## Architecture

### 1. Shared OCI release plane

`oci/` owns reproducible build inputs, CPU/GPU image variants, dependency locks, SBOM/provenance, release manifest generation, digest verification, and secret scanning. An image tag is only a discovery hint; the release manifest and all accepted evidence use immutable digests.

The image contains the runtime and user-space dependencies. It does not contain host GPU drivers, node identities, Slurm/Docker credentials, deployment routes, or durable evidence.

### 2. Common adapter contract

`ndnsf-di-deploy` parses and validates a `DeploymentProfile`, resolves the OCI release, delegates environment operations, and emits a common `DeploymentEvidenceBundle`. The common layer owns:

- profile/schema validation;
- candidate and release lineage;
- digest and provenance verification;
- identity reference validation and redaction;
- backend/fallback assertions;
- evidence finalization and gate evaluation.

Adapters own only preflight, materialization, lifecycle, resource/network/storage binding, and adapter-specific observation.

### 3. Docker Compose adapter

The cloud adapter uses one Compose project per host. A host-scoped NFD is the default; application containers connect through an explicit Unix socket or declared local endpoint. Multi-host operation requires explicit faces/routes and TCP/UDP 6363 preflight.

CPU execution requires Docker Engine and Compose. Docker GPU execution additionally requires a compatible NVIDIA host driver and NVIDIA Container Toolkit. These are host prerequisites; neither is embedded in the image.

Compose lifecycle maps to install/pull, preflight, start, status, logs/evidence, stop, upgrade, and rollback. Release swaps preserve declared durable identity/state bindings and never reuse mutable tags as rollback identity.

### 4. Slurm + Apptainer adapter

The iTiger adapter submits bounded jobs through `sbatch` or runs job steps through `srun`. It never starts persistent application/NFD processes on the login node. The profile declares partition, account/QOS when applicable, wall time, nodes, tasks, CPUs, memory, GRES type/count, and job name.

The adapter resolves a pinned OCI digest and materializes it as a SIF under durable project storage or per-job scratch. Formal evidence records both OCI digest and SIF SHA-256 because conversion creates an adapter-specific executable artifact. Login-node and compute-node Apptainer versions are observed independently; execution facts are collected inside the allocation.

GPU jobs use `apptainer exec --nv`. Slurm chooses the physical device; container-local `CUDA_VISIBLE_DEVICES` is not treated as a physical index. Evidence correlates requested GRES, Slurm allocation fields, physical GPU UUID/model from `nvidia-smi`, and container-visible device.

Supported planned GRES values are:

| GRES value | Current iTiger nodes | Planned acceptance use |
|---|---|---|
| `h100_80gb` | itiger01 | high-end compatibility probe |
| `rtx_6000` | itiger02-06 | alternative GPU compatibility probe |
| `rtx_5000` | itiger07-11 | initial five-minute acceptance slice |

Values are revalidated at submission time because cluster configuration is external and mutable.

### 5. iTiger storage layout

| Class | Default | Contents | Lifecycle |
|---|---|---|---|
| account | `/home/$USER` | SSH and small user config only | durable, quota constrained |
| project | `/project/$USER/ndnsf-di` | source, release manifests, SIF, models, identity bindings, evidence | durable, backed by user quota |
| scratch | compute-node `/tmp/ndnsf-di-$SLURM_JOB_ID-$RUN_ID` | extraction, cache, transient models/work files | job-local, disposable |
| evidence staging | scratch then `/project/$USER/ndnsf-di/evidence/$RUN_ID` | logs, manifests, status, checksums | atomically/manifest promoted |

Preflight checks the actual target paths and quota/space; shared `df` capacity is never interpreted as the user's quota. Exit and termination traps finalize a manifest and copy admissible artifacts to project storage while retaining the original workload exit status.

### 6. Networking and NFD topology

Cloud Compose multi-host networking is supported after explicit reachability and route validation. iTiger begins with a single allocation/node. Multi-node iTiger is a gated extension: a compute-node probe must demonstrate stable address selection and required NFD TCP/UDP reachability before the adapter enables multi-node readiness. Failure or absence of evidence remains `DEFERRED`/`BLOCKED`.

### 7. GPU and ONNX Runtime truthfulness

Profiles distinguish `cpu`, `cuda`, and `onnxruntime-cuda`; they include `allowCpuFallback`. The existing runtime chooses the provider and emits observed backend evidence. The adapter validates allocation and library/device visibility but does not reimplement model-provider selection.

GPU requested + unavailable + fallback false is a startup failure. GPU requested + explicit fallback true is `DEGRADED`, never GPU PASS. Host drivers remain external; OCI images supply compatible CUDA/ONNX Runtime user-space closure.

### 8. Security and authority boundaries

Node/job identities are external references with per-deployment uniqueness. Secret mounts are read-only where possible and excluded from layers, SIF, logs, and evidence. OCI/SIF scans and evidence redaction are acceptance gates.

Evidence separates:

- `substrate`: Slurm allocation, GPU pass-through, Apptainer, storage;
- `candidate`: packaged NDNSF-DI behavior and functional tests; and
- `physicalProduction`: owned by Spec 106 and retained as `DEFERRED` here.

The measured iTiger probe job `145855` is useful substrate evidence only; it is not a candidate GPU inference, performance, security, or physical-production PASS.

## Project Structure

### Documentation

```text
specs/108-ndnsf-di-container-deployment/
├── spec.md
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── traceability.md
├── contracts/
│   ├── node-profile.schema.json
│   ├── container-evidence.schema.json
│   └── operator-cli.md
├── checklists/
│   ├── requirements.md
│   └── pre-implementation-audit.md
└── tasks.md
```

### Source

```text
packaging/ndnsf-di-container/
├── bin/ndnsf-di-deploy
├── lib/
│   ├── profile.py
│   ├── release.py
│   ├── evidence.py
│   ├── redaction.py
│   └── adapters/
│       ├── base.py
│       ├── docker_compose.py
│       └── slurm_apptainer.py
├── oci/
│   ├── Dockerfile.cpu
│   ├── Dockerfile.gpu
│   ├── locks/
│   └── scripts/
├── adapters/
│   ├── docker-compose/
│   │   ├── compose.yaml
│   │   ├── profiles/
│   │   ├── scripts/
│   │   └── templates/
│   └── slurm-apptainer/
│       ├── scripts/
│       ├── templates/
│       └── profiles/
└── schemas/

tests/container/
├── contract/
├── unit/
├── integration/
├── fixtures/
└── live/
```

**Structure Decision**: Use a new generic container package rather than extending a Docker-named root. This keeps OCI release production independent of the execution environment and makes the adapters explicit. Existing systemd files remain unchanged except for documented interoperability checks.

## Implementation Phases

1. **Contracts and baselines**: freeze schemas, authority boundaries, release lineage, existing systemd behavior, and current runtime GPU evidence.
2. **OCI build source**: reproducible CPU/GPU build inputs, manifest, SBOM/provenance, digest verification, secret scan.
3. **Common adapter core**: profile validation, operator CLI, materialization records, evidence and redaction.
4. **Docker Compose adapter**: host-scoped NFD, CPU/GPU profiles, networking, lifecycle, rollback.
5. **Slurm + Apptainer adapter**: resource/GRES validation, storage layout, job template, SIF materialization, GPU mapping, traps, cancellation.
6. **Integration and acceptance**: MiniNDN/container regressions, cloud two-host probe, bounded iTiger live jobs, negative and authority tests.
7. **Audit and handoff**: traceability, security scan, reproducibility bundle, strict Spec Kit audit, Spec 106 handoff.

## Validation Strategy

### Offline and CI-safe

- JSON Schema and profile fixture validation;
- release/digest/SIF checksum logic;
- Slurm command rendering and `scontrol`/`sacct` parser fixtures;
- GRES allowlist/count/resource-negative tests;
- storage-path and quota-policy tests;
- job trap/finalization tests with mocked scheduler commands;
- backend/fallback and authority-boundary tests;
- OCI context and evidence secret scans.

### Local/container integration

- CPU OCI build and smoke;
- Docker Compose readiness/restart/upgrade/rollback;
- host-scoped NFD socket and multi-host route checks;
- existing MiniNDN functional/security regressions against the packaged candidate where practical;
- GPU negative tests on non-GPU hosts without claiming GPU acceptance.

### Live iTiger acceptance

1. preflight VPN/SSH, account, partition, GRES, quota/path, and Apptainer;
2. submit a bounded `rtx_5000:1`, 2 CPU, 8 GiB, five-minute job;
3. record host and in-container GPU facts;
4. bind OCI digest and SIF SHA-256;
5. validate compute-node `/tmp` with a bounded fsync write;
6. copy a checksummed evidence bundle to project storage;
7. require successful Slurm terminal state and schema validation;
8. separately run candidate-bound inference only after its OCI release exists.

The already observed job `145855` provides preliminary substrate evidence and implementation fixtures. It does not close the future candidate-bound acceptance tasks.

## Rollback Strategy

- **OCI**: retain the last accepted digest and manifest; never roll back by mutable tag.
- **Compose**: stop failed release, restore prior digest and declared durable bindings, rerun readiness.
- **Slurm**: cancel failed/obsolete jobs, preserve terminal evidence, submit the prior accepted OCI/SIF materialization as a new job; jobs themselves are immutable historical executions.
- **Host package**: existing `packaging/ndnsf-di-systemd` remains available if container deployment is withdrawn.
- **Data**: no destructive migration is allowed without a versioned, tested restore path.

## Complexity Tracking

| Added complexity | Why required | Simpler alternative rejected |
|---|---|---|
| Two runtime adapters | Cloud hosts and iTiger expose different lifecycle/resource authority | Pretending Docker Compose can run on iTiger would require unavailable daemon/privileges |
| OCI plus SIF identity | Apptainer conversion creates a distinct executable artifact | Recording only a tag or OCI digest cannot prove the executed SIF |
| Separate substrate/candidate/physical gates | Prevents overclaiming from a GPU visibility probe | One overall PASS would erase authority and evidence boundaries |
| Multi-node iTiger network gate | Compute-node NFD reachability is not yet verified | Assuming port/addressability would create a false deployment claim |

## Post-Design Constitution Check

**Verdict**: PASS, subject to the pre-implementation audit in `checklists/pre-implementation-audit.md`. The plan introduces packaging and lifecycle integrations only; no runtime protocol fork or production-authority expansion is planned.
