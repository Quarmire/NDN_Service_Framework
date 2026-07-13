# Data Model: NDNSF-DI OCI Deployment Adapters

## 1. OCIReleaseManifest

Immutable description of a releasable candidate.

| Field | Type | Rules |
|---|---|---|
| `schemaVersion` | string | exact supported schema version |
| `releaseId` | string | stable, unique |
| `candidateId` | string | links Spec 107 candidate/evidence |
| `sourceRevision` | string | full VCS revision |
| `createdAt` | timestamp | UTC |
| `images` | map | architecture/backend to digest-pinned OCI reference |
| `buildInputs` | array | lock/checksum references |
| `sbom` | object | digest and location |
| `provenance` | object | builder and attestation reference |
| `compatibility` | object | CPU arch, CUDA/ORT requirements |

**Invariant**: accepted release images use `@sha256:` references; tags alone are invalid.

## 2. DeploymentProfile

Declarative desired execution.

| Field | Type | Rules |
|---|---|---|
| `profileId` | string | unique within deployment |
| `runId` | string | unique per execution |
| `releaseManifest` | path/URI | immutable manifest |
| `runtime.kind` | enum | `docker-compose` or `slurm-apptainer` |
| `backend.requested` | enum | `cpu`, `cuda`, `onnxruntime-cuda` |
| `backend.allowCpuFallback` | boolean | default false for GPU profiles |
| `identity.reference` | path/reference | external, never image content |
| `storage` | StorageBinding | adapter-specific validated paths |
| `network` | NetworkProfile | explicit NFD topology |
| `compose` | object/null | required only for Compose |
| `slurm` | object/null | required only for Slurm |

**Invariant**: exactly one adapter configuration is present and matches `runtime.kind`.

## 3. RuntimeMaterialization

What the adapter actually executes.

| Field | Type | Rules |
|---|---|---|
| `adapter` | enum | two supported adapters |
| `ociReference` | string | digest-pinned |
| `ociDigest` | string | SHA-256 |
| `materializationType` | enum | `docker-image` or `sif` |
| `materializationId` | string | Docker image ID or SIF SHA-256 |
| `path` | string/null | required for SIF |
| `runtimeVersion` | string | Docker/Compose or Apptainer |
| `createdAt` | timestamp | UTC |
| `verified` | boolean | true only after digest/checksum validation |

## 4. SlurmResourceRequest

| Field | Type | Rules |
|---|---|---|
| `partition` | string | default `bigTiger`, preflight validated |
| `account` | string/null | optional, discovered/declared |
| `qos` | string/null | optional, discovered/declared |
| `wallTime` | duration | positive, within partition limit |
| `nodes` | integer | >= 1; initial acceptance 1 |
| `tasksPerNode` | integer | >= 1 |
| `cpusPerTask` | integer | >= 1 |
| `memory` | string | explicit Slurm memory unit |
| `gpu.type` | enum | `h100_80gb`, `rtx_6000`, `rtx_5000` |
| `gpu.count` | integer | >= 1 for GPU backend |
| `networkProbeRequired` | boolean | true when nodes > 1 |

**Invariant**: GPU backend requires a GPU request; CPU backend must not accidentally reserve a GPU unless explicitly justified.

## 5. SlurmAllocationRecord

Observed scheduler truth.

| Field | Type |
|---|---|
| `jobId` | string |
| `jobName` | string |
| `account` / `qos` / `partition` | string/null |
| `nodeList` | array[string] |
| `requestedTres` / `allocatedTres` | object/string |
| `slurmJobGpus` | string/null |
| `cudaVisibleDevices` | string/null |
| `submitTime` / `startTime` / `endTime` | timestamp/null |
| `state` | enum/string |
| `exitCode` | string/null |
| `elapsedSeconds` | number/null |

## 6. StorageBinding

| Field | Type | Rules |
|---|---|---|
| `accountRoot` | path | iTiger default `/home/$USER`; small config only |
| `projectRoot` | path | durable; iTiger default `/project/$USER/ndnsf-di` |
| `scratchRoot` | path | unique per run; compute `/tmp` on iTiger |
| `identityRoot` | path | durable external reference |
| `imageRoot` | path | durable SIF/cache location |
| `modelRoot` | path | durable model source/cache policy |
| `evidenceRoot` | path | durable unique run directory |
| `capacityChecks` | array | path, free bytes, quota result, timestamp |
| `promotion` | object | staging, destination, manifest/checksum, result |

## 7. NetworkProfile

| Field | Type | Rules |
|---|---|---|
| `topology` | enum | `single-node`, `multi-host`, `multi-node-allocation` |
| `nfdMode` | enum | `host-scoped`, `job-scoped` |
| `localEndpoint` | string | Unix socket or declared local URI |
| `routes` | array | explicit prefix/remote endpoint |
| `requiredPorts` | array | normally TCP/UDP 6363 for remote NFD |
| `preflightEvidence` | reference/null | mandatory for multi-host/multi-node |
| `status` | enum | `PASS`, `FAIL`, `BLOCKED`, `DEFERRED` |

## 8. BackendCompatibilityRecord

| Field | Type |
|---|---|
| `requestedBackend` | enum |
| `observedBackend` | string/null |
| `allowCpuFallback` | boolean |
| `fallbackOccurred` | boolean |
| `status` | `PASS`, `DEGRADED`, `FAIL` |
| `hostDriverVersion` | string/null |
| `hostReportedCudaVersion` | string/null |
| `runtimeCudaVersion` | string/null |
| `onnxRuntimeVersion` | string/null |
| `executionProvider` | string/null |
| `physicalGpus` | array[GPUObservation] |
| `containerVisibleDevices` | string/null |

## 9. GPUObservation

| Field | Type |
|---|---|
| `uuid` | string |
| `model` | string |
| `physicalIndex` | integer/null |
| `memoryMiB` | integer/null |
| `driverVersion` | string/null |
| `source` | enum (`host`, `container`) |

## 10. DeploymentEvidenceBundle

Top-level admissible evidence record.

| Field | Type | Rules |
|---|---|---|
| `schemaVersion` | string | supported version |
| `runId` | string | matches profile and directory |
| `candidate` | object | ID, source revision, Spec 107 lineage |
| `release` | OCIReleaseManifest reference/summary | digest-bound |
| `profileDigest` | SHA-256 | exact rendered profile |
| `materialization` | RuntimeMaterialization | verified |
| `adapterEvidence` | ComposeEvidence or SlurmEvidence | matches adapter |
| `storage` | StorageBinding summary | no secret contents |
| `network` | NetworkProfile result | explicit status |
| `backend` | BackendCompatibilityRecord | observed truth |
| `tests` | array | commands, results, artifact checksums |
| `authority` | object | substrate/candidate/physicalProduction |
| `redaction` | object | scan tool/result |
| `outcome` | enum | `PASS`, `DEGRADED`, `FAIL`, `BLOCKED`, `DEFERRED` |
| `startedAt` / `finishedAt` | timestamp | UTC |

**Authority invariant**: Spec 108 MUST emit `physicalProduction=DEFERRED`.

## State Transitions

### Common run

```text
DECLARED -> PREFLIGHTED -> MATERIALIZED -> STARTED -> OBSERVED
         -> EVIDENCE_FINALIZED -> {PASS|DEGRADED|FAIL|BLOCKED|DEFERRED}
```

Any failed digest, identity, resource, storage, network, or backend precondition goes to `BLOCKED` or `FAIL`; it never skips directly to `PASS`.

### Slurm job

```text
RENDERED -> SUBMITTED -> PENDING -> RUNNING
         -> {COMPLETED|FAILED|TIMEOUT|CANCELLED|PREEMPTED}
         -> EVIDENCE_ARCHIVED
```

`EVIDENCE_ARCHIVED` preserves the scheduler terminal state; it does not convert it to success.

### Compose release

```text
INSTALLED -> STARTING -> READY -> UPGRADING -> READY
                      \-> FAILED -> ROLLING_BACK -> READY|FAILED
```
