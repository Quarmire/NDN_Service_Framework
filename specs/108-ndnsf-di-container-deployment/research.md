# Research: NDNSF-DI OCI Deployment Adapters

## Decision 1: Treat OCI as the release source, not the runtime contract

**Decision**: Build and publish immutable OCI artifacts once, then execute them through adapter-specific materializations.

**Rationale**: Docker Compose and iTiger expose different control planes. OCI provides a portable content and provenance boundary without pretending that a Docker daemon exists on Slurm compute nodes.

**Alternatives considered**:

- **Docker everywhere**: rejected because iTiger provides Slurm + Apptainer, not a user-controlled Docker daemon.
- **Separate Docker and Apptainer builds**: rejected because duplicated build graphs would weaken candidate identity and dependency consistency.
- **SIF as the only release artifact**: rejected because cloud Docker execution and registries consume OCI directly.

## Decision 2: Use two thin execution adapters

**Decision**: Implement `docker-compose` and `slurm-apptainer` behind one profile, release, evidence, and operator contract.

**Rationale**: Compose owns persistent host services; Slurm owns scheduled allocations. The common core must retain NDNSF-DI behavior, security, candidate lineage, and gate semantics.

**Rejected**: embedding scheduler behavior in NDNSF-DI, running systemd inside containers, or maintaining a second release gate per adapter.

## Decision 3: Preserve host-scoped NFD for cloud Compose

**Decision**: One deployment-scoped NFD per cloud host, with explicit local access and explicit inter-host routes.

**Rationale**: It matches the existing host operational model, avoids hidden per-container forwarding islands, and makes route evidence inspectable.

**Rejected**: one NFD per application container as the default; host-network mode as an undocumented shortcut.

## Decision 4: Slurm allocation is the iTiger lifecycle and authority envelope

**Decision**: All compute work runs through `sbatch`/`srun`; no persistent NDNSF-DI or NFD process runs on the login node.

**Rationale**: Slurm assigns CPU, memory, GPU, node, time, and termination state. Bypassing it is operationally incorrect and produces unauditable resource claims.

**Operational consequences**:

- queued, running, completed, failed, timed-out, and cancelled are first-class states;
- jobs are historical executions, not mutable services;
- restart means submitting a new job linked to the same or a new release;
- `sacct` terminal state and exit code are part of acceptance.

## Decision 5: Bind OCI identity to SIF identity

**Decision**: Record source OCI digest and resulting SIF SHA-256 for every admissible Apptainer run.

**Rationale**: OCI-to-SIF conversion creates a distinct byte artifact. OCI digest proves source selection; SIF checksum proves what Apptainer executed.

**Rejected**: recording only a mutable tag; assuming two independent conversions are byte-identical without measurement.

## Decision 6: Use cluster storage by data lifetime

**Decision**:

- `/home/$USER`: small SSH/account configuration only;
- `/project/$USER/ndnsf-di`: durable source, release manifests, SIF, models, identity references, and evidence;
- compute-node `/tmp/ndnsf-di-$SLURM_JOB_ID-$RUN_ID`: disposable job scratch/cache;
- durable evidence promotion before exit.

**Rationale**: The account guidance gives `/home` a small quota and recommends `/project`; compute-node `/tmp` is large but temporary. The observed filesystem pool size is not the user's quota.

**Rejected**: storing SIF/model caches in `/home`; treating login-node `/tmp` as representative of compute-node scratch; retaining authoritative evidence only in `/tmp`.

## Decision 7: Express GPUs through named GRES and observe actual devices

**Decision**: Profiles request one of `h100_80gb`, `rtx_6000`, or `rtx_5000` with a positive count. Evidence records the request, allocated TRES, physical UUID/model, and container-visible mapping.

**Rationale**: Slurm selects devices and may remap them. `CUDA_VISIBLE_DEVICES=0` inside a job means the first allocated device, not necessarily physical GPU index 0.

**Rejected**: hard-coding node names or physical indices; accepting `nvidia-smi` output without proving Slurm allocation.

## Decision 8: Distinguish Docker GPU prerequisites from Apptainer GPU prerequisites

**Decision**:

- Docker adapter: compatible host driver plus NVIDIA Container Toolkit.
- Apptainer adapter: compatible host driver plus cluster-managed Apptainer and `--nv`; no NVIDIA Container Toolkit installation.
- OCI image: CUDA/ONNX Runtime user-space closure, never kernel driver installation.

**Rationale**: The two runtimes inject GPU devices/libraries differently. Mixing their prerequisite lists would cause unnecessary or impossible cluster changes.

## Decision 9: Keep existing ONNX Runtime backend truth as the source of record

**Decision**: Adapter preflight verifies allocation and visibility, while the existing `OnnxRuntimeModelRunner` reports the actually selected provider/device. Fallback-disabled GPU failure remains fail-closed; explicit fallback is degraded.

**Rationale**: Reimplementing provider selection in packaging would produce two competing truths.

## Decision 10: Gate multi-node iTiger NFD networking

**Decision**: Initial iTiger support is single-node/single-allocation. Multi-node support is disabled until an in-allocation probe proves usable node addresses and required NFD TCP/UDP reachability.

**Rationale**: Scheduler/GPU success does not prove compute-node ingress, egress, firewall policy, or stable addressing.

**Rejected**: inferring network reachability from SSH login access or cloud Compose behavior.

## Decision 11: Separate three evidence authorities

**Decision**:

1. `substrate`: scheduler, GPU pass-through, Apptainer, storage;
2. `candidate`: packaged NDNSF-DI functional/backend evidence; and
3. `physicalProduction`: Spec 106 only.

**Rationale**: A container seeing a GPU is necessary but insufficient evidence for correct model execution, performance, security, or field deployment.

## Current measured iTiger facts

The following facts were measured on 2026-07-12/13 and are planning inputs, not permanent configuration guarantees:

- login endpoint: `itiger.memphis.edu` after university VPN;
- partition: `bigTiger`;
- observed GRES mapping: H100 80GB on itiger01, RTX 6000 on itiger02-06, RTX 5000 on itiger07-11;
- login Apptainer observed 1.3.4; compute node observed 1.3.3;
- `/project/tma1` exists;
- compute job `/tmp` was a job-isolated filesystem of approximately 14 TB, unlike login `/tmp`;
- job `145855`, requested as `gpu:rtx_5000:1`, completed successfully on itiger07;
- Slurm exposed one allocated GPU with a container-local mapping;
- host and `apptainer exec --nv` both observed an RTX 5000 Ada GPU;
- `docker://nvidia/cuda:12.4.1-base-ubuntu22.04` materialized to a SIF with SHA-256 `35603df78f9be6e167e0d4cb5221b40952c9726129cef00d146ccce4f0afaffb`;
- a bounded 64 MiB fsync write under compute-node `/tmp` passed;
- preliminary evidence lives under `/project/tma1/ndnsf-di/probes/rtx5000-20260713T000608Z`.

These results establish a usable iTiger container substrate. They do not prove NDNSF-DI candidate inference, performance, production security, multi-node NFD networking, or physical deployment readiness.

## Open validations

- Registry/authentication path for the final NDNSF-DI OCI release.
- Exact host-driver/user-space CUDA matrix for each final GPU image.
- Compute-allocation multi-node TCP/UDP 6363 policy and addressability.
- Candidate-bound ONNX Runtime GPU inference on all claimed GRES classes.
- Quota behavior and cache eviction under realistic SIF/model sizes.
- Termination-trap evidence preservation on timeout/preemption.
- Long-duration and physical-production acceptance under Spec 106.
