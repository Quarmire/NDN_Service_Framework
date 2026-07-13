# Feature Specification: NDNSF-DI OCI Deployment Adapters

**Feature Branch**: `108-ndnsf-di-container-deployment`
**Created**: 2026-07-12
**Status**: Draft
**Input**: Abstract the Docker cloud deployment as an OCI build source and add an iTiger Slurm + Apptainer execution adapter, storage layout, GPU GRES selection, and acceptance tasks.

## Scope and authority

This feature defines one immutable OCI release and two execution adapters:

1. `docker-compose` for long-lived Linux cloud hosts; and
2. `slurm-apptainer` for scheduled iTiger compute allocations.

The adapters own lifecycle, resource, network, and storage integration only. They MUST NOT duplicate NDNSF-DI planning, provider selection, NDN security, evidence semantics, or release-gate logic. Existing systemd packaging remains the non-container rollback surface.

Spec 108 may establish container-substrate readiness and candidate-bound software evidence. It MUST NOT turn iTiger substrate evidence into a physical-production claim. Physical deployment authority, production security closure, hardware performance, and soak acceptance remain owned by Spec 106.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Build and launch one OCI release on a cloud host (Priority: P1)

An operator builds or pulls a digest-pinned NDNSF-DI OCI release, selects the Docker Compose adapter, installs host-scoped configuration and identity references, and starts a CPU node without rebuilding source on the target host.

**Why this priority**: It is the shortest repeatable cloud deployment path and establishes the shared OCI source used by all adapters.

**Independent Test**: On a clean supported Linux VM with Docker Engine and Compose, install a pinned release, start the CPU profile, pass readiness and a local smoke request, restart it, and verify that identity/state survive while runtime processes are recreated.

**Acceptance Scenarios**:

1. **Given** only Docker prerequisites and a deployment bundle, **When** the operator installs a pinned OCI digest and starts the CPU profile, **Then** NFD and NDNSF-DI services become ready without host compilation.
2. **Given** a running node, **When** the Compose project is recreated, **Then** durable identity/state remain and mutable runtime state is rebuilt.
3. **Given** an image tag whose resolved digest differs from the release manifest, **When** installation is attempted, **Then** the adapter fails closed before starting services.

---

### User Story 2 - Connect multiple cloud nodes through explicit NFD routes (Priority: P1)

An operator deploys the same OCI release on multiple Docker hosts, gives each host its own NFD, configures explicit inter-host NFD faces/routes, and verifies remote service discovery and invocation.

**Why this priority**: NDNSF-DI is distributed; single-host container success alone is not deployment readiness.

**Independent Test**: Start two cloud nodes, verify NFD reachability and routes, invoke a provider on the remote node, and preserve route/readiness evidence.

**Acceptance Scenarios**:

1. **Given** two hosts with distinct node profiles, **When** explicit TCP/UDP 6363 policy and routes are applied, **Then** each node can reach the remote service prefix.
2. **Given** a missing route or blocked port, **When** preflight runs, **Then** deployment reports a precise network failure instead of claiming readiness.
3. **Given** application containers on one host, **When** they access NFD, **Then** they use the host deployment's Unix socket or declared local endpoint and do not run their own hidden NFD.

---

### User Story 3 - Submit an iTiger Slurm + Apptainer deployment (Priority: P1)

An iTiger user stores the project and immutable artifacts under `/project/$USER`, submits a bounded Slurm job requesting an explicit GPU GRES, materializes the pinned OCI release as a SIF, runs it through Apptainer inside the allocation, uses compute-node `/tmp` as scratch, and copies evidence back to durable project storage before exit.

**Why this priority**: iTiger is the available GPU execution environment and does not expose the Docker-daemon deployment model.

**Independent Test**: Submit a five-minute `rtx_5000` acceptance job that records allocation metadata, validates `nvidia-smi` on the host and in `apptainer exec --nv`, verifies OCI/SIF identity, writes and fsyncs a scratch artifact under compute-node `/tmp`, copies the evidence bundle to `/project/$USER/ndnsf-di/evidence`, and exits successfully.

**Acceptance Scenarios**:

1. **Given** a pinned OCI digest and iTiger account, **When** the user requests `--partition=bigTiger --gres=gpu:rtx_5000:1`, **Then** the workload runs only inside its allocation and records the requested GRES, assigned node, physical GPU UUID, and container-visible device.
2. **Given** the OCI artifact, **When** Apptainer materializes it, **Then** the evidence binds the OCI digest, SIF SHA-256, Apptainer version, and materialization command.
3. **Given** job-local scratch output, **When** the job exits normally or through the controlled failure trap, **Then** admissible logs and results are copied to `/project/$USER/ndnsf-di/evidence/<run-id>`.
4. **Given** no working compute-node route between allocations/nodes, **When** a multi-node deployment is requested, **Then** the adapter fails its NFD network preflight and does not claim multi-node readiness.

---

### User Story 4 - Select CPU or GPU truthfully across adapters (Priority: P1)

An operator selects CPU, CUDA GPU, or ONNX Runtime GPU execution and receives evidence of the backend actually used. GPU requests fail closed unless an explicitly configured CPU fallback policy allows degradation.

**Why this priority**: Silent fallback invalidates both performance claims and deployment evidence.

**Independent Test**: Run CPU, valid GPU, invalid GPU/no-fallback, and invalid GPU/explicit-fallback cases through both adapter contract tests; verify the declared and observed backend fields.

**Acceptance Scenarios**:

1. **Given** a GPU profile and valid allocation/runtime, **When** inference starts, **Then** evidence reports the selected CUDA/ONNX Runtime provider and physical GPU identity.
2. **Given** a GPU profile without an allocated or visible GPU, **When** fallback is disabled, **Then** startup fails before readiness.
3. **Given** fallback explicitly enabled, **When** the GPU provider is unavailable, **Then** the run is marked degraded and never satisfies a GPU acceptance gate.
4. **Given** iTiger execution, **When** the container starts, **Then** host driver compatibility is checked but NVIDIA Container Toolkit is neither required nor installed by the adapter.

---

### User Story 5 - Preserve identity, secrets, state, and evidence safely (Priority: P1)

An operator provisions node-specific identities and secrets outside the OCI image, mounts them read-only where possible, separates durable state from scratch/cache data, and obtains a redacted evidence bundle.

**Why this priority**: Cloned credentials and leaked private material are deployment blockers.

**Independent Test**: Scan OCI layers and evidence for private-key markers, verify per-node identity bindings, exercise restart/materialization, and confirm that scratch deletion does not remove durable evidence.

**Acceptance Scenarios**:

1. **Given** two deployments from the same OCI digest, **When** identities are inspected, **Then** each uses its own external identity binding.
2. **Given** generated evidence, **When** redaction validation runs, **Then** private keys, passwords, tokens, and raw secret values are absent.
3. **Given** iTiger storage policy, **When** a job runs, **Then** `/home` contains only small account configuration, `/project/$USER/ndnsf-di` contains durable artifacts, and compute-node `/tmp` contains disposable per-job scratch.

---

### User Story 6 - Operate, upgrade, cancel, and roll back predictably (Priority: P2)

An operator can inspect status, collect logs, stop or cancel work, upgrade to a new digest, and roll back to the last accepted release using adapter-appropriate lifecycle commands.

**Why this priority**: Repeatable recovery is required before broad deployment, but depends on the P1 release and adapter contracts.

**Independent Test**: Upgrade and rollback a Compose node; submit, inspect, cancel, and archive a Slurm job; verify that release and evidence lineage remain intact.

**Acceptance Scenarios**:

1. **Given** a healthy Compose release, **When** the next digest fails readiness, **Then** the operator restores the prior accepted digest and state binding.
2. **Given** a queued or running Slurm job, **When** the operator requests cancellation, **Then** `scancel` is issued, termination evidence is archived, and no login-node daemon remains.
3. **Given** a failed job or host, **When** evidence is collected, **Then** it contains adapter, release, configuration, backend, network, storage, timing, and failure-cause records.

### Edge Cases

- OCI registry is unavailable after a digest has already been materialized locally.
- Image tag moves while the release manifest remains pinned to an older digest.
- Login-node and compute-node Apptainer versions differ.
- Slurm job remains queued, is cancelled, times out, or is preempted.
- Requested GRES is unavailable or the scheduler exposes a container-local device index different from the physical index.
- Host NVIDIA driver is incompatible with the image's CUDA user-space runtime.
- `/project` quota is exhausted even though the shared filesystem reports free capacity.
- Compute-node `/tmp` is cleared, differs from login-node `/tmp`, or cannot fit the materialized model/cache.
- Cross-node TCP/UDP 6363 is blocked or node addresses are not stable/reachable.
- A partial evidence copy occurs during abnormal job termination.
- CPU fallback is enabled accidentally in a GPU acceptance profile.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The project MUST produce a versioned, immutable OCI release manifest with image references pinned by digest.
- **FR-002**: The OCI release MUST contain the NDNSF-DI runtime and required NDN user-space dependencies, but MUST NOT contain node private identities, operator credentials, or environment-specific routes.
- **FR-003**: The deployment design MUST expose a common operator contract and explicit `docker-compose` and `slurm-apptainer` runtime adapters.
- **FR-004**: Runtime adapters MUST be thin lifecycle/resource integrations and MUST reuse the same candidate, evidence, security, and release-gate semantics.
- **FR-005**: Existing `packaging/ndnsf-di-systemd` installation and rollback MUST remain usable and MUST NOT be silently replaced.
- **FR-006**: The Docker adapter MUST support Docker Engine plus Compose on supported Linux cloud hosts without target-host source compilation.
- **FR-007**: The Docker adapter MUST run exactly one host-scoped NFD by default and MUST make NFD access by application containers explicit.
- **FR-008**: The Docker adapter MUST support explicit multi-host NFD face/route configuration and preflight required TCP/UDP 6363 reachability.
- **FR-009**: Docker GPU execution MUST declare NVIDIA driver and NVIDIA Container Toolkit as host prerequisites rather than embedding a driver in the OCI image.
- **FR-010**: The iTiger adapter MUST use Slurm allocations (`sbatch`/`srun`) and MUST NOT run persistent NDNSF-DI or NFD processes on login nodes.
- **FR-011**: The iTiger adapter MUST use Apptainer to execute the OCI-derived artifact and MUST NOT require a Docker daemon or NVIDIA Container Toolkit on iTiger.
- **FR-012**: The iTiger adapter MUST materialize or select a SIF from a pinned OCI digest and record both OCI digest and SIF SHA-256.
- **FR-013**: The iTiger adapter MUST support `bigTiger` GRES types `h100_80gb`, `rtx_6000`, and `rtx_5000`, plus a positive GPU count.
- **FR-014**: The iTiger adapter MUST submit CPU, memory, wall-time, partition, node-count, task-count, and GPU GRES as explicit Slurm resources.
- **FR-015**: GPU evidence MUST distinguish requested GRES, Slurm allocation fields, physical GPU UUID/model, and container-visible device mapping.
- **FR-016**: Apptainer GPU execution MUST use `--nv` and verify host-driver/container-user-space compatibility before declaring readiness.
- **FR-017**: The runtime MUST preserve existing fail-closed ONNX Runtime provider selection when CPU fallback is disabled.
- **FR-018**: Explicit CPU fallback MUST mark a run degraded and MUST prevent it from satisfying a GPU acceptance gate.
- **FR-019**: `/home` on iTiger MUST be reserved for small account configuration and SSH material; the adapter MUST NOT place images, models, project trees, or evidence campaigns there.
- **FR-020**: Durable iTiger project source, SIF artifacts, models, release manifests, identity bindings, and evidence MUST reside under `/project/$USER/ndnsf-di` by default.
- **FR-021**: Compute-node `/tmp` MUST be used only for per-job scratch, cache, extraction, and transient work; required output MUST be copied to durable storage before job exit.
- **FR-022**: The adapter MUST validate actual quota/space for each storage class and MUST NOT infer a user's quota from shared filesystem capacity.
- **FR-023**: Each run MUST use a unique run identifier and unique scratch/evidence directory, with atomic or manifest-verified promotion to the durable evidence location.
- **FR-024**: Multi-node iTiger execution MUST remain disabled until a compute-allocation network probe validates the required NFD transport and addressability.
- **FR-025**: Each node/job MUST receive a distinct external identity binding; identities and secrets MUST NOT be baked into OCI layers or SIF artifacts.
- **FR-026**: Secret-bearing mounts MUST be read-only where supported, minimally scoped, and excluded from evidence and diagnostic archives.
- **FR-027**: The common evidence schema MUST bind candidate, OCI digest, runtime materialization, adapter, configuration, identity reference, backend, network, storage, and lifecycle outcome.
- **FR-028**: Compose evidence MUST record project name, container/image digests, health, NFD routes, restart/upgrade state, and host GPU prerequisite checks when applicable.
- **FR-029**: Slurm evidence MUST record job ID, account/QOS, partition, node list, requested/allocated TRES, wall time, exit state/code, Apptainer version, and durable-copy result.
- **FR-030**: The operator interface MUST provide build/verify, install/materialize, preflight, start/submit, status/wait, logs/evidence, stop/cancel, upgrade, and rollback operations where applicable.
- **FR-031**: Slurm jobs MUST install exit/termination traps that attempt evidence finalization without hiding the original exit status.
- **FR-032**: Adapter preflight failures MUST be actionable and MUST prevent readiness or acceptance claims.
- **FR-033**: iTiger substrate checks MAY establish scheduler, GPU pass-through, Apptainer, and scratch capability, but MUST be labeled separately from NDNSF-DI candidate acceptance.
- **FR-034**: Physical-production, real-UAV, real-network, production security, performance, and soak PASS authority MUST remain deferred to Spec 106.
- **FR-035**: The feature MUST include contract, negative, integration, and evidence-schema tests for both adapters without requiring GPU allocation for every unit test.
- **FR-036**: Documentation MUST provide exact cloud and iTiger prerequisites, commands, storage paths, resource requests, failure recovery, and cleanup procedures.

### Key Entities

- **OCI Release Manifest**: Immutable mapping from candidate and architecture to OCI image digests, build inputs, SBOM/provenance, and compatibility metadata.
- **Runtime Adapter**: Thin implementation of deployment lifecycle and environment integration for Docker Compose or Slurm + Apptainer.
- **Deployment Profile**: Declarative runtime, node/job resources, networking, storage, backend, identity reference, and fallback policy.
- **Runtime Materialization**: Adapter-specific executable form of an OCI release, such as a Docker image ID or SIF SHA-256.
- **Slurm Allocation Record**: Requested and observed scheduler resources, GRES, node, job state, timing, and exit information.
- **Storage Binding**: Durable, identity, cache, scratch, and evidence paths plus capacity/quota validation results.
- **Backend Compatibility Record**: Requested and observed inference backend, runtime/provider versions, driver/device identity, and fallback outcome.
- **Deployment Evidence Bundle**: Redacted, candidate-bound acceptance artifact conforming to the shared schema.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: A clean supported cloud VM can start a CPU Compose node from a pinned OCI digest and pass readiness plus smoke invocation within 15 minutes after prerequisites are installed.
- **SC-002**: Two cloud hosts can pass NFD route preflight and complete a remote NDNSF-DI invocation with evidence that identifies both nodes and the OCI digest.
- **SC-003**: A five-minute `bigTiger` `rtx_5000:1` Slurm acceptance job can validate allocation, host GPU, OCI-to-SIF materialization, `apptainer exec --nv`, compute-node `/tmp` fsync, and durable evidence copy.
- **SC-004**: Equivalent parameterized acceptance definitions exist for `rtx_6000` and `h100_80gb` without hard-coding physical device indices.
- **SC-005**: Every accepted container run records a pinned OCI digest and adapter-specific materialization digest/ID; tag-only evidence is rejected.
- **SC-006**: GPU/no-fallback tests fail closed when GPU execution is unavailable, while explicit fallback is reported as degraded and cannot pass a GPU gate.
- **SC-007**: An OCI/SIF secret scan and evidence redaction test find zero private-key, password, or raw token disclosures.
- **SC-008**: iTiger storage validation proves durable artifacts under `/project/$USER/ndnsf-di`, disposable job scratch under compute-node `/tmp`, and no bulk project artifacts under `/home`.
- **SC-009**: Compose upgrade rollback restores the last accepted release; Slurm cancellation preserves final job state and best-effort evidence without leaving login-node processes.
- **SC-010**: Multi-node iTiger readiness remains false until the in-allocation NFD network probe passes; an unverified network is represented as `DEFERRED` or `BLOCKED`, never `PASS`.
- **SC-011**: Contract and negative test suites cover all common evidence fields, both adapters, all supported GRES values, invalid resource/storage profiles, and authority-boundary assertions.
- **SC-012**: No Spec 108 artifact converts the observed iTiger substrate probe into a physical-production or candidate-performance PASS; those fields remain deferred to Spec 106.

## Assumptions

- iTiger account, VPN, SSH, Slurm allocation, `/project/$USER`, and Apptainer access are provided by the cluster and are not provisioned by this feature.
- The currently observed iTiger configuration (`bigTiger`, named GPU GRES types, and project/storage conventions) is evidence for planning but remains subject to administrator change and therefore is revalidated by preflight.
- Container images supply user-space CUDA/ONNX Runtime dependencies; GPU kernel drivers remain host/cluster responsibilities.
- The first iTiger acceptance slice is one node and one GPU. Multi-node execution is gated by a dedicated network probe.
- Production registry authentication and image signing policy may vary by deployment, but digest verification and provenance recording are mandatory.

## Dependencies

- Spec 107 provides the active NDNSF-DI candidate/runtime behavior to package.
- Spec 106 owns physical/production validation and final deployment authority.
- Existing systemd packaging supplies rollback and operational behavior that container adapters should align with.
- Docker Engine/Compose are external prerequisites for cloud execution.
- Slurm and Apptainer are cluster-managed prerequisites for iTiger execution.

## Out of Scope

- Installing or administering iTiger Slurm, Apptainer, GPU drivers, VPN, accounts, partitions, or quotas.
- Kubernetes, Docker-in-Docker, systemd inside containers, or a long-running scheduler bypass.
- Treating a Slurm batch job as a permanent public cloud service endpoint.
- Real-UAV, real-NDN-network, production-security, candidate-performance, or long-soak closure owned by Spec 106.
- Replacing the existing systemd package or changing NDNSF-DI wire/security protocols.
