# Data Model and State Machines

## ExecutionEvidence

Immutable for one provider boot and runner installation.

| Field | Rule |
|---|---|
| `schemaVersion` | Exact supported version; unknown major fails closed |
| `providerName` | Authenticated provider identity |
| `providerBootId` | Changes on process restart |
| `evidenceEpoch` | Increases on runner/artifact reinstall within a boot |
| `runnerKind` | `synthetic-delay`, `wiring-only`, `onnxruntime-cpu`, `onnxruntime-cuda`, `transformers`, `llama-server`, or `unknown` |
| `realCompute` | Derived from runner kind and runtime initialization, never caller supplied |
| `deviceKind` / `deviceId` | CPU or stable GPU UUID/index |
| `runtimeVersion` | Exact ONNX Runtime/CUDA/backend version |
| `artifactDigests` | Role-to-artifact SHA-256 mapping |
| `planDigest` | Installed native-plan digest |
| `roles` | Exact installed stage roles |
| `createdAtMs` | Evidence creation time |

Aggregate classification requires all selected providers to agree on real/synthetic
class, model/plan identity, and compatible backend. Missing or mixed evidence is
`invalid-evidence`, not `unknown-real`.

## ProviderCapability

Configured, slow-changing support contract:

- provider and device identity;
- supported runner kinds and tensor dtypes;
- total GPU/RAM ceilings;
- supported model families and artifact formats;
- maximum workers, context tokens, batch, stage bytes;
- capability version and source (`profile`, `build`, `operator`).

Capability never claims current free memory, queue, residency, or performance.

## ProviderTelemetrySnapshot

Measured, expiring state:

- provider boot and evidence epoch;
- measurement timestamp, monotonic sequence, source and probe duration;
- supported/unsupported/error status;
- GPU total/free/used bytes, utilization and device UUID;
- model/runner residency and installed artifact digests;
- ready queue, dependency waits, active/idle workers;
- stage service-rate and latency EWMA with sample count;
- cache budget/used bytes, session count, hits/misses/evictions;
- network profile version/reference.

Validation:

- timestamp cannot be in the future beyond bounded clock skew;
- sequence is monotonic within one boot;
- device/evidence identity must match capability and installed runner;
- production planner rejects age >2,000 ms;
- configured-only fields cannot set `source=measured`.

## PlanLease

| Section | Contents |
|---|---|
| Identity | model, tokenizer, artifact, planner, native-plan digests |
| Workload | context/output/batch class and target RPS |
| Assignment | stage, provider, device, runner and fallback candidates |
| Validity | provider boots, evidence epochs, telemetry sequences/age, memory/queue limits, network profile, cache assumptions |
| Authority | issue/expiry time, user identity, execution-lease references |
| Explanation | selected/rejected candidates and normalized costs |

State: `PROPOSED -> VALID -> ACTIVE -> EXPIRED|INVALIDATED|SUPERSEDED`.
`ACTIVE` does not override provider admission or execution leases.

## ExecutionAttempt

Key: `(requestId, attemptEpoch)`.

Fields:

- user, service, model/plan and deadline;
- assigned provider boots and role mapping;
- execution-lease IDs;
- dependency namespace prefix including attempt epoch;
- context/KV bindings;
- cancellation and terminal reason;
- authoritative-result flag.

State:

```text
CREATED -> ADMITTED -> RUNNING -> COMPLETED
                    \-> CANCELLING -> CANCELLED -> REPLACED
                    \-> FAILED
                    \-> EXPIRED
```

Only one epoch per request may enter `COMPLETED` as authoritative. Epoch 1 may
start only after epoch 0 enters a terminal/superseded transition. Late epoch-0
data is counted and ignored.

## KvStateBinding

- session and context epoch;
- stage role and provider boot;
- model/artifact/plan/security epoch digests;
- cache kind and opaque provider-local reference;
- token range and byte estimate;
- created/access/expiry time.

State: `RESIDENT -> STALE|EVICTED|LOST`; full-context rebuild produces a new
binding. A provider restart makes every old binding `LOST`.

## DeploymentRelease

- release ID, source commit and build manifest;
- binary/package digests;
- supported profile/plan/evidence/telemetry schema ranges;
- systemd unit and environment versions;
- artifact and trust prerequisites;
- active/previous release paths;
- activation and rollback records.

Authoritative Repo data is external to this entity. Model/KV/activation caches
are disposable and may be deleted during activation or rollback.

## AcceptanceRecord

- immutable experiment ID and purpose;
- source commit and release ID;
- full command/environment/profile/topology;
- backend/evidence classification;
- warmup, measurement window, request cap and seeds;
- all run paths including failures;
- correctness, performance, security, recovery, operations and evidence gates;
- Material Passport, fallacy scan, final PASS/BLOCK.

No replacement run may reuse an acceptance-record ID or output directory.
