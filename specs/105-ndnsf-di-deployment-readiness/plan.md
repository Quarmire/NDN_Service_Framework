# Implementation Plan: NDNSF-DI MiniNDN Deployment Candidate

**Branch**: `Experimental` | **Date**: 2026-07-12 | **Spec**: [spec.md](spec.md)

## Summary

Turn the current Qwen research paths into one honest, bounded, deployable pilot
without creating a second runtime. The implementation first repairs evidence
identity, then makes the existing three-stage Qwen ONNX path the canonical real
compute path, connects fresh measured provider facts to reusable plan validity,
replaces unbounded dependency waiter threads with a bounded scheduler, adds one
attempt-epoch-based recovery path, and packages the same binaries and profiles as
systemd-compatible services. MiniNDN closes algorithm, failure, packaging, and
local operations gates; physical GPU production acceptance moves to Spec 106.

## Revision R1 Control Decision

The original three-run 1 RPS campaign is retained as a failed candidate, not
rerun. Its zero-complete outcome is confounded by a verified four-worker FIFO
execution pattern: every token completion enqueues the same generation's next
token after already queued work from later generation sessions. Revision R1
therefore separates three decisions:

1. SC-002 and H3 remain failed for the original candidate and all thresholds stay
   frozen.
2. Independent telemetry, recovery, packaging, and local-operations implementation
   may continue; the failure does not justify skipping their tests or moving them
   to the physical pilot.
3. A corrected generation-level load driver must pass deterministic scheduling
   validity tests before one newly registered three-run campaign may execute.
   Failure of that campaign blocks the candidate and causes the 24-hour soak to
   close as `NOT RUN / BLOCK` under the preregistered stop rule.

This is an experiment-validity repair, not a performance waiver. It neither
changes the per-token collaboration protocol nor claims that corrected scheduling
will make the CPU profile serviceable.

## Technical Context

**Language/Version**: C++17; Python 3.8+

**Primary Dependencies**: ndn-cxx, NFD, ndn-svs, NAC-ABE/OpenABE, Boost,
ONNX Runtime CPU Execution Provider, Linux host resource interfaces, systemd

**Storage**: Digest-bound model/runtime artifacts and authoritative Repo SQLite;
provider-local model/activation/KV caches are disposable

**Testing**: Boost.Test C++ unit tests, Python unittest, focused regression
scripts, MiniNDN 60-second campaigns, local namespace/systemd staging and soak

**Target Platform**: Ubuntu x86_64 development host running MiniNDN with three
logical provider nodes and local NFD instances

**Project Type**: C++ runtime plus Python planning/operator CLI and deployment
assets

**Performance Goals**: Fixed Qwen pilot at 1 offered RPS reaches >=99% completion,
>=95% offered throughput, distributed p95 <=2x matched single-node p95; discover
rather than assume higher stable rates

**Constraints**: No synthetic evidence in real-compute gates; no security bypass;
one bounded replacement attempt; telemetry age <=2 seconds; dependency waiting
threads bounded; no Kubernetes or multi-backend expansion

**Scale/Scope**: One Qwen2.5-0.5B model, three stages, batch one, <=512 input
tokens, <=32 output tokens, one controller, one user workload, exactly three
logical primary providers with same-host fallback roles, and optional Repo nodes

## Constitution Check

### Pre-design Gate

- **Canonical dynamic runtime**: PASS. Existing V2/Targeted/Collaboration paths
  remain authoritative; no generated or split-name API returns.
- **Security in the data path**: PASS. Permission, NAC-ABE, token, replay, digest,
  and fail-closed lease checks remain mandatory.
- **CodeGraph first**: PASS. Existing runner, readiness, worker, planner, CLI,
  test, and experiment paths were traced before design.
- **Spec-driven durable work**: PASS. This feature owns requirements, contracts,
  tasks, evidence gates, migration, and rollback.
- **Validation scope**: PASS. Measured windows are 60 seconds; MiniNDN security
  exercises application paths but stays explicitly non-production. Spec 106 owns
  real identities and physical acceptance.
- **GSD/ARS**: PASS. Long-running phases have resumable state and the experiment
  plan freezes controls, repetitions, stop rules, and fallacy checks.

### Post-design Gate

PASS. All new semantics live in DI unless they are generic execution facts already
owned by Core. No new wire protocol is required: typed evidence and telemetry use
the existing generic capability/service-payload envelope, dependency objects use
existing large-data APIs, and systemd packaging remains outside runtime logic.

## Evidence Baseline and Controlling Finding

The implementation MUST begin by preserving and correcting the following fact:
Spec 093 provider logs record `tracerDeterministicRunner=1` even though aggregate
summaries report `runnerMode=qwen-onnx-native`. Existing 1-8 RPS values remain
valid only for scheduler/control/dependency transport. They are not real Qwen
compute throughput. No later task may use them as a production capacity baseline.

The valid real-compute anchors are:

- real three-stage Qwen ONNX MiniNDN one-token forward: 119 measured requests,
  p50 287.14 ms, p95 358.52 ms;
- real three-stage Qwen Transformers correctness: matching top token, Python
  execution and `torch.save` hidden state;
- real Qwen GGUF/llama-server: replicated-provider comparison, not model split;
- focused native DI C++ tests and Runtime v1 contract tests.

## Architecture

### 1. One Release-Gate Truth Chain

```text
provider runner factory
  -> observed ExecutionEvidence per boot/role/artifact
  -> readiness ACK service payload
  -> user candidate evidence
  -> run summary consistency check
  -> release gate
```

`runnerMode` is removed as a caller-selected truth source. The aggregate label is
derived from provider evidence. Evidence is immutable for one provider boot and
changes only after runner/artifact reinstallation creates a new evidence epoch.
Synthetic, wiring-only, mixed, missing, or unknown evidence blocks real-compute
classification.

### 2. Canonical Pilot Execution Path

```text
tokenize at user
  -> Stage 0: embedding + layers 0-7
  -> Stage 1: layers 8-15
  -> Stage 2: layers 16-23 + norm/head
  -> greedy token
  -> repeat decode with stage-local KV bindings, max 32 tokens
```

The existing Qwen ONNX exporter and three-stage dependency names are retained.
The provider executable uses `OnnxRuntimeModelRunner`; the runner gains the dtype,
dynamic-shape, explicit execution-provider, and cache-tensor support required by
these artifacts. The local profile requires CPU; CUDA requests fail closed and
are accepted only by Spec 106 on physical GPU hosts.
The application-facing request remains an NDNSF-DI payload or standard
LargeDataReference. Core Request/ACK/Selection/Response and large-data wire
semantics do not change.

Stage-local KV tensors stay provider-local. Requests carry authenticated cache
bindings, never opaque authority. A valid cache binding is an optimization; full
context is the correctness fallback. Delta-only requests with missing/stale
bindings fail explicitly.

### 3. Capability and Telemetry Separation

```text
ProviderCapability (configured, slow-changing)
  device/backend support, total memory ceiling, stage formats

ProviderTelemetrySnapshot (measured, expiring)
  free/used host memory, process RSS, model residency, queue/waiting/active,
  service-rate EWMA, boot id, timestamp, source and freshness
```

A `ProviderResourceProbe` runs off the request hot path. The Spec 105 backend
reads bounded Linux host/process resource interfaces and records their exact
source. Missing, malformed, stale, or unsupported samples remain explicit rather
than copying configured values. Physical NVIDIA telemetry is owned by Spec 106.
Readiness and admission combine measured facts with the existing worker snapshot.

Plan keys and leases include provider boot/membership, runtime/artifact versions,
telemetry version, network-profile version, and cache-binding assumptions. Reuse
is allowed only after every predicate is revalidated.

### 4. Bounded Dependency Scheduling

Replace `m_inputWaiters` (one thread per pending role) with one
`DependencyWaitScheduler` owned by `ProviderRoleWorker`:

- fixed worker count and bounded queue;
- cancellation token per execution attempt;
- deadline and priority metadata;
- readiness completion enqueues the role on the existing compute queue;
- shutdown cancels queued waits, resolves promises once, and joins fixed workers;
- counters expose queued, active, expired, cancelled, rejected, and completed
  waits.

No generic Core scheduler is added. This is DI role/dependency policy and stays in
`NDNSF-DistributedInference`.

### 5. Attempt Epoch and Recovery

```text
ExecutionAttemptKey = requestId + attemptEpoch
```

The user creates epoch 0. Provider loss, straggler deadline, stale telemetry, or
recoverable cache failure may create epoch 1 only. Each epoch has distinct
dependency object names and execution-lease bindings. The user owns final-result
authority; late data from a superseded epoch is observable and ignored.

Recovery order:

1. cancel old attempt and revoke/expire its execution lease;
2. exclude failed boot/provider and invalidate incompatible plan/cache facts;
3. select one compatible standby or fallback plan;
4. start epoch 1 with remaining request deadline;
5. otherwise return one exact terminal reason.

There is no recursive retry, no same-epoch replay, and no bypass of permission,
token, lease, digest, or deadline checks.

### 6. Operator Surface and Packaging

Systemd is the first and only supervisor. Deployment assets live under
`packaging/ndnsf-di-systemd/` and include controller, provider, user/bench, Repo
templates, environment files, tmpfiles, log rotation, and an install/uninstall
script. Units use dependency ordering but do not assume simultaneous readiness;
NDNSF readiness remains the authority.

The existing `ndnsf-di` CLI becomes the supported operator entry point:

- `doctor`: host, NFD, identity, artifact, backend, device, profile checks;
- `provider`: run the real provider service, not emit sample metadata;
- `plan`: build/validate/explain a real plan lease;
- `run`: invoke the real bounded Qwen service;
- `status`: aggregate role health and recent terminal reasons;
- `metrics`: export structured JSON and Prometheus textfile format;
- `bench`: execute a frozen campaign profile;
- `inspect`: read result/release-gate artifacts.

The former simulated Runtime v1 commands move under an explicit `contract-smoke`
namespace or are removed after callers migrate. They cannot retain production
names.

### 7. Observability

INFO-level structured events and periodic snapshots cover:

- provider boot, backend, device, artifact and plan evidence;
- readiness/admission and telemetry freshness;
- request/attempt/role lifecycle;
- compute queue and dependency wait scheduler;
- stage compute/fetch/publish bytes and latency;
- cache hit/miss/rebuild/evict;
- cancellation/replan/recovery;
- security rejection and terminal outcome.

TRACE remains diagnostic and is excluded from performance gates. Metrics failure
never changes request correctness.

### 8. Rollout and Rollback

1. Correct evidence labels with no runtime behavior change.
2. Run real Qwen MiniNDN correctness and the initial 1 RPS acceptance; retain a
   failed campaign without replacement.
3. Repair and validate generation-level load scheduling, add measured telemetry
   and plan invalidation, then run one newly registered acceptance campaign.
4. Add bounded dependency scheduler; stress and rerun acceptance.
5. Add one replacement attempt; execute deterministic fault cells.
6. Package systemd profile; validate in MiniNDN/namespace staging.
7. Run the local MiniNDN canary, staged upgrade/rollback, and 24-hour soak.

Each slice is independently revertible. New summary fields are additive until
all maintained readers migrate. Caches are disposable. Authoritative Repo schema
is unchanged. Rollback activates the prior release directory/profile and rejects
cache entries whose version/digest/epoch bindings do not match.

## Experiment and Acceptance Design

See [experiment-plan.md](experiment-plan.md). The controlling sequence is:

1. **Evidence cells**: synthetic, real CPU, unavailable-CUDA rejection, mixed,
   missing.
2. **Correctness cells**: single-node vs three-stage, fixed prompt corpus,
   token-by-token comparison, full-context and cache-hit decode.
3. **MiniNDN performance**: 1 RPS x 3 repetitions x 60 seconds per immutable
   candidate campaign; application
   permission/token paths execute, but the MiniNDN dummy-keychain environment is
   not cryptographic-strength evidence. Higher rates are
   discovery only until frozen in a later spec.
4. **Stress**: 1,000 pending waits and cancellation; no network performance claim.
5. **Faults**: provider loss, restart, straggler, missing segment, hash mismatch,
   stale telemetry, cache loss; five fixed seeds/cells where stochastic.
6. **Application-security performance**: normal MiniNDN permission, NAC-ABE,
   token, replay, and provider-permission paths, explicitly non-cryptographic-
   strength because of the dummy keychain.
7. **Local operations**: matched single-node/distributed canary, provider-process
   restart, staged upgrade/rollback, then a 24-hour MiniNDN soak only after its
   performance preflight passes; otherwise preserve `NOT RUN / BLOCK` evidence.

## Project Structure

### Documentation

```text
specs/105-ndnsf-di-deployment-readiness/
├── spec.md
├── plan.md
├── research.md
├── data-model.md
├── experiment-plan.md
├── migration-and-rollback.md
├── security.md
├── quickstart.md
├── traceability.md
├── tasks.md
├── contracts/
└── evidence/
```

### Source and Test Impact

```text
NDNSF-DistributedInference/cpp/ndnsf-di/
├── ExecutionEvidence.{hpp,cpp}              # new
├── ProviderResourceProbe.{hpp,cpp}           # new
├── DependencyWaitScheduler.{hpp,cpp}         # new
├── OnnxRuntimeModelRunner.{hpp,cpp}           # extend
├── TensorBundleCodec.{hpp,cpp}                # extend
├── NativeProviderReadiness.{hpp,cpp}          # extend
├── NativeProviderHandler.{hpp,cpp}            # attempt epoch
├── ProviderRoleWorker.{hpp,cpp}               # bounded waiting
└── NativeExecutionPlan*.{hpp,cpp}              # validity/evidence fields

NDNSF-DistributedInference/ndnsf_distributed_inference/
├── runtime_v1.py                              # operator CLI/migration
├── deployment.py                              # lease/attempt planning
├── runtime_telemetry.py                       # typed DI facts
└── qwen_pilot.py                              # new bounded product adapter

examples/DI_NativeProviderExecutable.cpp
examples/python/NDNSF-DistributedInference/llm_pipeline/
Experiments/NDNSF_DI_NativeTracer_Minindn.py
Experiments/NDNSF_DI_LlmPipeline_Minindn.py
packaging/ndnsf-di-systemd/
tests/unit-tests/distributed-inference-async-runtime.t.cpp
tests/python/test_ndnsf_di_runtime_v1.py
tests/python/test_ndnsf_native_tracer_runtime_profile.py
tests/python/test_ndnsf_di_deployment_readiness.py             # new
```

**Structure Decision**: Extend the existing DI runtime and operator packages.
Do not create a new service protocol, planner package, container orchestrator, or
parallel LLM runtime.

## Complexity Tracking

No constitution violation is required. Three new C++ components exist because
they have separate ownership and invariants: immutable execution identity,
resource probing, and bounded dependency scheduling. Combining them into
`ProviderRoleWorker` would mix evidence, OS integration, and scheduling state.
