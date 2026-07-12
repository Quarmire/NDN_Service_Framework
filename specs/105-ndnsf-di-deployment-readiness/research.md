# Research Decisions

## Decision 1: Correct evidence before optimization

**Decision**: Provider-observed evidence is authoritative. Existing Spec 093
throughput is reclassified as synthetic-compute scheduling/dataflow evidence.

**Rationale**: The harness forces deterministic execution for `llm-proportional`
and then writes an optimistic aggregate label. Optimization against that label
would target the wrong bottleneck.

**Alternatives considered**:

- Keep the label and add a footnote: rejected because machines consume summaries.
- Delete old results: rejected because negative and misleading evidence must be
  preserved and corrected, not erased.

## Decision 2: Qwen ONNX and ONNX Runtime CUDA are the sole pilot path

**Decision**: Extend the active three-stage Qwen ONNX contract to CUDA execution.

**Rationale**: Real stage artifacts, deterministic dependency names, MiniNDN
evidence, and a native ONNX runner already exist. vLLM and llama-server are good
replicated-serving systems but do not expose the selected stage hidden-state
contract. TensorRT-LLM would add a conversion/runtime program before correctness
and operations are closed.

**Alternatives considered**:

- Python Transformers: retained as correctness oracle, rejected as production
  provider hot path.
- llama-server: retained as replicated baseline, rejected as model-split path.
- vLLM/TensorRT-LLM: deferred until the one-backend pilot closes.

## Decision 3: A bounded generation profile, not general LLM serving

**Decision**: <=512 input tokens, 1-32 greedy output tokens, batch one.

**Rationale**: This is enough to require real prefill, decode, KV lifecycle,
network transfer, and terminal output while keeping artifact, memory, and
reproducibility costs bounded.

**Alternatives considered**:

- One next-token forward only: rejected as insufficient for actual serving.
- Long-context/multi-tenant/batching: deferred because each adds a separate
  scheduling and cache-policy research problem.

## Decision 4: Provider-local KV state with full-context fallback

**Decision**: Keep KV tensors local to their stage owner; authenticate and version
references. Full context is the correctness fallback.

**Rationale**: Moving all KV data defeats decode efficiency. Treating hidden
runtime state as unversioned authority breaks restart and replan safety.

**Alternatives considered**:

- Repo-backed KV in v1: rejected due latency, confidentiality, and write
  amplification.
- Always recompute full context: retained only as fallback, rejected as primary.

## Decision 5: Measured and configured facts never share a truth label

**Decision**: Separate `ProviderCapability` from expiring
`ProviderTelemetrySnapshot`. Unsupported probes are explicit.

**Rationale**: Static capacity is useful for compatibility; dynamic placement
requires fresh measured state. Silent fallback recreates the present false
resource-awareness problem.

**Alternatives considered**:

- Continue environment-only facts: allowed only in MiniNDN fixtures and labeled
  configured.
- Put GPU policy in Core: rejected; GPU/model semantics belong to DI.

## Decision 6: Fixed dependency wait pool

**Decision**: Replace one thread per pending role with a bounded scheduler owned
by DI.

**Rationale**: The current vector of waiter threads scales with pending roles and
cannot satisfy a deployment resource bound.

**Alternatives considered**:

- Increase OS thread limits: rejected as masking the ownership problem.
- Move scheduling into Core: rejected because role/dependency cancellation is DI
  policy.

## Decision 7: One replacement attempt with an attempt epoch

**Decision**: At most one replacement. Dependency names, leases, cancellation,
and final-result authority include the attempt epoch.

**Rationale**: This is the smallest recovery design that prevents late results
and duplicate authority without introducing a distributed transaction protocol.

**Alternatives considered**:

- Unlimited retries: rejected due load amplification and hidden tail latency.
- No recovery: rejected for deployment.
- Exactly-once execution: rejected as unnecessary; one authoritative result and
  idempotent attempt identity are sufficient.

## Decision 8: Systemd before containers or Kubernetes

**Decision**: Ship systemd-compatible units and release directories, then
validate them in local namespace staging before Spec 106 uses physical hosts.

**Rationale**: It exercises real process identity, filesystem permissions, NFD
socket ownership, restart policy, logs, and rollback with minimal control-plane
work.

**Alternatives considered**:

- Docker Compose: deferred; NFD sockets, GPU devices, identities, and local
  caches still need host integration.
- Kubernetes: rejected for the shortest route.

## Decision 9: INFO metrics, TRACE diagnostics

**Decision**: Release gates consume structured INFO events/snapshots. TRACE is
excluded from performance evidence.

**Rationale**: Recent UAV experiments proved TRACE can materially perturb the
hot path. Operational metrics must be cheap and stable.

## Decision 10: Complete the candidate in MiniNDN; defer hardware to Spec 106

**Decision**: Real-compute, telemetry, bounded scheduler, fault, packaging, local
operations, and soak cells close in MiniNDN. Hardware validation is a separate
Spec 106 feature and cannot block completion of the candidate.

**Rationale**: This preserves the current development policy, keeps every Spec
105 task executable on the available host, and prevents local evidence from
being relabeled as physical production proof.

## Decision 11: Bounded `nvidia-smi` probe before an NVML library dependency

**Decision**: The first measured GPU probe uses the driver's stable query CLI
with an exact field list, background cadence and hard timeout.

**Rationale**: It is present with supported NVIDIA drivers, adds no new ABI/link
dependency, and is outside the request hot path. Device UUID and numeric fields
remain typed after parsing.

**Alternatives considered**:

- Direct NVML C API: a later optimization if probe overhead or portability
  requires it.
- Python GPU libraries: rejected for the native provider lifecycle.
