# Feature Specification: NDNSF-DI MiniNDN Deployment Candidate

**Feature Branch**: `Experimental`

**Created**: 2026-07-12

**Status**: Revision R1 — Local MiniNDN Implementation After Retained Gate Failure

**Input**: Convert the shortest audited NDNSF-DI deployment route into a complete design, detailed executable task list, and adversarial pre-implementation audit.

## Scope and Product Boundary

This feature produces the shortest credible route from the current research
harness to a reproducible local MiniNDN deployment candidate. It closes
algorithm, evidence, bounded-runtime, packaging, and local-operations readiness
without claiming physical production deployment. The supported profile is
deliberately narrow:

- Qwen2.5-0.5B split into three contiguous ONNX stages;
- one primary provider per stage plus predeclared fallback roles represented by
  three MiniNDN provider nodes on the same development host, one controller, one
  user, and optional Repo nodes;
- short-context, batch-one requests with at most 512 input tokens and 32 generated
  tokens;
- MiniNDN development, acceptance, restart, upgrade/rollback, and soak on the
  local development host;
- systemd-compatible packaging validated in local namespace staging; no
  Kubernetes control plane;
- one optimized execution backend, not simultaneous vLLM/TensorRT/llama.cpp work.

The replicated llama-server path remains a supported comparison baseline, not
the distributed-model implementation delivered by this feature. Physical-node
security and operations acceptance is owned by Spec 106 and cannot be inferred
from Spec 105.

### Revision R1: Retained Capacity Failure

The first candidate campaign executed exactly three prespecified 60-second runs
and failed SC-002 with zero complete 32-token generations at the measurement
cutoff. Those runs are immutable negative evidence; Revision R1 does not replace,
average away, or relabel them. Code inspection established that the campaign also
used a four-worker FIFO client executor while each token callback submitted the
next token behind work from other generation sessions. The observed zero-complete
result therefore applies to the tested combination of CPU runtime, per-token
collaboration protocol, and breadth-first client scheduling. It is not sufficient
evidence to attribute the failure solely to CPU compute capacity.

Revision R1 permits implementation work that is independent of the failed gate
to continue. Before any new acceptance campaign, the generation driver must be
tested as a bounded generation-level scheduler, expose its queue discipline and
per-session progress, and receive a new immutable candidate/campaign identity.
The 1 RPS, completion, correctness, and latency thresholds remain unchanged. A
new failure remains BLOCK; it is not compensated by a higher timeout, retry, a
lower offered rate, or an unregistered replacement run.

## User Scenarios & Testing

### User Story 1 - Trust Every Performance Label (Priority: P1)

An evaluator can tell from every summary whether providers ran synthetic delay,
wiring-only validation, CPU ONNX, CUDA ONNX, Transformers, or llama-server. A
synthetic run can never satisfy a real-compute release gate.

**Why this priority**: Incorrect evidence labels invalidate every optimization,
capacity, and deployment decision that follows.

**Independent Test**: Run one synthetic and one real ONNX fixture and verify the
summary derives its runner identity from observed provider evidence, rejects
mixed/unknown evidence, and classifies the two runs differently.

**Acceptance Scenarios**:

1. **Given** a provider launched with a deterministic runner, **When** a summary
   is produced, **Then** it reports synthetic execution and cannot pass the
   real-compute gate.
2. **Given** every provider reports verified CPU ONNX execution with matching
   artifact hashes, **When** a summary is produced, **Then** it reports real
   CPU execution and retains the per-provider evidence.
3. **Given** providers disagree about backend or artifact identity, **When** the
   run closes, **Then** the run is rejected rather than assigned an optimistic
   aggregate label.

---

### User Story 2 - Run a Real Bounded Qwen Service (Priority: P1)

An application user can submit a short-context Qwen request and receive up to 32
generated tokens from a three-stage ONNX pipeline whose stage computation and
intermediate tensors are real, with results checked against a single-node
baseline.

**Why this priority**: This is the smallest workload that demonstrates the
actual distributed-inference product rather than only its control and dataflow
scaffolding.

**Independent Test**: On MiniNDN, run the fixed Qwen profile for three matched
60-second repetitions at 1 offered RPS and compare token output, completion,
throughput, latency, stage timing, and resource use with the frozen single-node
baseline.

**Acceptance Scenarios**:

1. **Given** valid Qwen stage artifacts and a short prompt, **When** the user
   invokes the service, **Then** all generated tokens match the single-node
   greedy-decoding baseline.
2. **Given** a 60-second measured window at 1 RPS, **When** three repetitions
   finish, **Then** at least 99% of submitted requests complete, achieved
   throughput is at least 95% of offered load, and p95 is no greater than twice
   the matched single-node p95.
3. **Given** a request exceeds 512 input or 32 output tokens, **When** it reaches
   admission, **Then** it fails explicitly before partial pipeline execution.

---

### User Story 3 - Plan From Fresh Measured Capacity (Priority: P2)

An operator can see measured host memory, model residency, queue pressure, worker
occupancy, and stage service rate. The user planner accepts a plan only while
the facts that justified it remain fresh and compatible.

**Why this priority**: Static 2/4/8 GB labels cannot safely drive placement on
real heterogeneous machines.

**Independent Test**: Change measured free host memory and queue pressure in a
controlled fixture, verify the typed snapshot changes, and verify reuse,
invalidation, defer, or rejection follows the declared freshness and capacity
rules.

**Acceptance Scenarios**:

1. **Given** a healthy local CPU provider, **When** telemetry is sampled, **Then** the
   snapshot is measured, timestamped, bounded in age, and bound to provider,
   device, model, artifact, and runtime identities.
2. **Given** stale or unavailable dynamic telemetry, **When** planning occurs,
   **Then** the provider is excluded or conservatively rejected; configured
   capacity is never silently presented as measured capacity.
3. **Given** a material change in free memory, queue pressure, model residency,
   or provider membership, **When** a cached plan is considered, **Then** its
   validity is re-evaluated and the decision is observable.

---

### User Story 4 - Recover Without Unbounded Runtime Growth (Priority: P2)

An operator can lose or restart one stage provider without corrupting outputs,
leaking unbounded waiter threads, or leaving requests permanently stuck. The
user either completes through a bounded replacement attempt or receives a
terminal failure with an exact reason.

**Why this priority**: A service that only works while every provider and every
dependency succeeds is not deployable.

**Independent Test**: Under MiniNDN, inject missing dependency, slow stage,
provider kill, provider restart, stale cache, and hash mismatch while observing
bounded threads, cancellation, replan decisions, attempt epochs, and terminal
outcomes.

**Acceptance Scenarios**:

1. **Given** 1,000 pending dependency waits, **When** none is ready, **Then**
   waiter thread growth remains bounded by the configured scheduler pool plus a
   fixed allowance.
2. **Given** a selected provider dies, **When** one bounded replacement is
   permitted, **Then** the old attempt cannot publish an authoritative final
   result and the new attempt uses a distinct epoch.
3. **Given** no compatible replacement exists, **When** the deadline expires,
   **Then** all work and dependency state is cancelled and the user receives one
   terminal reason.

---

### User Story 5 - Package and Operate a Local Deployment Candidate (Priority: P3)

An operator can install, configure, start, inspect, stop, upgrade, and roll back
the controller, three MiniNDN providers, user, and optional Repo services using
one documented local staging profile without reading experiment scripts.

**Why this priority**: The runtime is not a credible deployment candidate until
the complete operator workflow is reproducible and diagnosable through stable
interfaces on the environment currently available.

**Independent Test**: From a clean local staging directory, execute the MiniNDN
runbook, complete a 24-hour 1 RPS soak with one scheduled provider-process
restart, then roll forward and back between two compatible staged releases.

**Acceptance Scenarios**:

1. **Given** the supported local host and a clean staging directory, **When** the
   runbook is followed, **Then** every MiniNDN role reaches ready without editing
   source files or requiring a physical node.
2. **Given** a running deployment, **When** the operator inspects it, **Then**
   health, backend identity, artifact identity, plan identity, queues, host-resource facts,
   request outcomes, and recent terminal errors are available in structured
   output.
3. **Given** an incompatible or failed release, **When** rollback is requested,
   **Then** the previous release and configuration resume without migrating or
   trusting disposable model/KV caches.

### Edge Cases

- A local profile requests CUDA even though the provider only supports CPU ONNX,
  or a provider reports an artifact hash
  different from the installed plan.
- One provider reports measured telemetry while another reports configured-only
  capacity.
- Telemetry becomes stale between ACK collection and execution-lease commit.
- A provider dies after publishing an output but before the next stage fetches it.
- A late output from attempt epoch N arrives after epoch N+1 becomes authoritative.
- A dependency is partially segmented, corrupted, duplicated, or never reaches
  its declared FinalBlockId.
- KV state is evicted, mismatched to model/plan/security epoch, or lost on restart.
- Real certificate validation succeeds but materially changes the performance
  profile compared with dummy-keychain MiniNDN.
- The local host lacks the required CPU execution provider or cannot fit the
  assigned stage in its measured memory budget.
- The metrics sink is unavailable; request correctness must not depend on it.
- Rollback sees a newer disposable cache format; the cache must be discarded.
- Physical nodes or production identities are unavailable; candidate gates must
  remain executable while production status stays explicitly deferred.

## Requirements

### Functional Requirements

- **FR-001**: Every provider MUST emit an observed execution-evidence record
  containing backend kind, synthetic/real status, runtime version, device kind,
  artifact digest, plan digest, and stage roles.
- **FR-002**: Aggregate summaries MUST derive runner classification from provider
  evidence and MUST reject missing, mixed, contradictory, or synthetic evidence
  for real-compute gates.
- **FR-003**: Historical Spec 091-093 evidence MUST be reclassified without
  deleting original artifacts or rewriting measured values.
- **FR-004**: The supported pilot MUST execute Qwen2.5-0.5B as three contiguous
  ONNX stages and support bounded greedy generation of 1-32 tokens.
- **FR-005**: Distributed outputs MUST be compared token-by-token with a frozen
  single-node baseline using identical model, tokenizer, prompt, generation
  settings, and numerical tolerance.
- **FR-006**: The runtime MUST support the tensor dtypes and dynamic shapes needed
  by token IDs, attention masks, hidden states, logits, and declared cache tensors.
- **FR-007**: Per-stage KV state MUST be bound to session, stage, context epoch,
  model digest, plan digest, provider, and security epoch; invalid bindings MUST
  fail closed or rebuild from full context.
- **FR-008**: Admission MUST enforce the pilot limits before any stage executes.
- **FR-009**: Providers MUST expose static configured capability separately from
  dynamic measured telemetry.
- **FR-010**: Dynamic telemetry MUST include timestamp, source, freshness,
  device identity, free/total host memory, model residency, queue depth, waiting
  dependencies, active workers, and measured stage service rate.
- **FR-011**: Stale, missing, or unsupported measured telemetry MUST never be
  silently replaced by configured values in a production plan.
- **FR-012**: Plan reuse MUST validate provider membership, runtime/artifact
  identity, telemetry freshness, memory feasibility, queue threshold, network
  profile version, and cache binding before execution.
- **FR-013**: Dependency waiting MUST use a bounded scheduler; total wait threads
  MUST not scale linearly with pending roles.
- **FR-014**: Every request MUST have an execution attempt epoch used in
  dependency names, cancellation, final-result authority, and late-message
  rejection.
- **FR-015**: Retry/replan MUST be bounded to one replacement attempt in the
  pilot and MUST never bypass permission, token, lease, digest, or deadline checks.
- **FR-016**: The runtime MUST provide explicit terminal reasons for provider
  loss, straggler deadline, dependency absence, hash mismatch, stale plan,
  telemetry rejection, cache miss, cancellation, and no replacement.
- **FR-017**: Provider restart MUST advertise a new boot identity and invalidate
  in-memory execution/KV state from the previous boot.
- **FR-018**: MiniNDN acceptance MUST execute the normal permission, NAC-ABE,
  token, replay, and provider-permission paths, while labeling dummy-keychain
  evidence `application-auth-path-executed` rather than cryptographic-strength
  production evidence.
- **FR-019**: The pilot MUST provide systemd units, environment/profile files,
  least-privilege filesystem ownership, startup ordering, health checks,
  structured status, and bounded shutdown.
- **FR-020**: Deployment artifacts MUST be versioned and digest-bound; executable
  artifacts MUST retain allowlist and sandbox requirements.
- **FR-021**: Upgrade and rollback MUST preserve authoritative Repo/catalog state
  while treating model, activation, and KV caches as disposable unless their
  schema and identity bindings match.
- **FR-022**: Operators MUST receive structured metrics for request outcomes,
  latency, stage compute/fetch/publish, queueing, host-resource facts, plan decisions,
  retries, cache events, and terminal failures without enabling TRACE.
- **FR-023**: All performance campaigns MUST use frozen commands, 60-second
  measured windows, prespecified repetitions and stop rules, unique output
  directories, no silent retries, and complete failure retention.
- **FR-024**: Spec 105 validation MUST remain on the local MiniNDN host. Physical
  nodes, real-identity production security, cross-host GPU telemetry, and
  production release authority MUST remain deferred to Spec 106.
- **FR-025**: A generation-load driver MUST schedule and account for one bounded
  generation as the application request unit, expose worker/queue discipline and
  per-session token progress, and MUST NOT convert a generation-level campaign
  into unreported breadth-first token-subrequest queueing.
- **FR-026**: The failed initial campaign MUST remain immutable negative evidence.
  Any corrected campaign MUST use a new preregistered candidate/campaign identity,
  retain the original threshold, and report `BLOCK` rather than silently rerun,
  extend, retry, or reduce load after failure.

### Key Entities

- **Execution Evidence**: Provider-observed proof of the runner, runtime, device,
  artifacts, plan, and roles actually used.
- **Provider Capability**: Slowly changing configured limits and supported
  backends, explicitly distinct from measured facts.
- **Telemetry Snapshot**: Time-bounded measured provider/runtime/resource state.
- **Plan Lease**: Reusable assignment plus all validity predicates and versions.
- **Execution Attempt**: One authoritative request attempt identified by request
  ID and monotonically increasing epoch.
- **Context/KV Binding**: Provider-local state identity required for safe reuse.
- **Deployment Release**: Versioned binaries, profiles, manifests, units, and
  compatibility metadata that can be activated or rolled back atomically.
- **Acceptance Record**: Immutable command, environment, topology, results,
  failures, and gate verdict for one validation cell.

## Success Criteria

### Measurable Outcomes

- **SC-001**: 100% of measured runs and providers have internally consistent
  observed execution evidence; synthetic or unknown runners pass 0 real-compute
  gates.
- **SC-002**: Three matched 60-second MiniNDN repetitions at 1 RPS complete at
  least 99% of submitted bounded-generation requests, achieve at least 95% of
  offered load, and produce tokens identical to the single-node baseline.
- **SC-003**: Distributed p95 for the fixed pilot workload is no more than 2.0x
  matched single-node p95, with compute/fetch/publish/queue decomposition for
  at least 99% of completed requests.
- **SC-004**: Measured telemetry used for selection is at most 2 seconds old;
  stale-telemetry tests reject or defer 100% of affected plans.
- **SC-005**: A 1,000-wait stress test keeps dependency-wait thread count at or
  below configured scheduler workers plus five and releases all state after
  cancellation.
- **SC-006**: Across five prespecified provider-loss injections, every request
  has exactly one terminal outcome, no stale attempt becomes authoritative, and
  at least four runs recover within two request deadlines when a compatible
  replacement exists.
- **SC-007**: The local deployment candidate can be installed and started twice
  from clean staging directories by the runbook without source edits or
  undocumented commands.
- **SC-008**: A 24-hour local MiniNDN soak at the frozen 1 request/second
  offered load has zero incorrect outputs, zero
  security bypasses, zero unbounded resource growth, at least 99% request
  completion, and no more than one unrecovered service interruption during the
  scheduled provider restart.
- **SC-009**: Local staged upgrade and rollback drills complete twice with
  authoritative Repo state preserved and disposable incompatible caches rejected.
- **SC-010**: The release gate produces a machine-readable MiniNDN-candidate
  PASS/BLOCK plus a separate physical-production `DEFERRED|PASS|BLOCK` status;
  missing physical evidence can never be reported as production PASS.
- **SC-011**: Every generation acceptance record reports offered generations,
  completed/failed/unfinished generations, token progress per session, client
  worker/queue occupancy, and the exact immutable campaign identity; a driver
  fixture proves that a runnable generation cannot be starved behind later-token
  work from unrelated sessions.

## Assumptions

- Only the local MiniNDN development host is currently available. Three logical
  providers and their fallback roles run on that host.
- The local host provides the backend capabilities used by the frozen profile;
  configured fixture values remain labeled and never become physical GPU proof.
- Qwen2.5-0.5B is intentionally retained as the smallest real target; larger,
  MoE, tensor-parallel, batching, and multi-tenant workloads are later features.
- Greedy decoding is sufficient for deterministic correctness comparison.
- ONNX Runtime CPU is the sole Spec 105 pilot backend because it is the real
  backend available on the local MiniNDN host. CUDA execution and GPU capacity
  acceptance are deferred to Spec 106; requesting CUDA locally must fail rather
  than silently fall back.
- Systemd is the sole first-release supervisor. Containers and Kubernetes are
  out of scope except for the existing executable-artifact sandbox contract.
- Provider-local KV caches are disposable; authoritative application/model
  artifacts remain digest-bound and Repo-backed.
- Spec 106 may begin only after Spec 105's MiniNDN candidate gate passes.

## Explicit Non-Goals

- Supporting arbitrary LLM families, arbitrary ONNX graphs, or arbitrary device
  counts in the first pilot.
- Production tensor parallelism, expert parallelism, dynamic batching, speculative
  decoding, or multi-tenant fairness.
- Kubernetes, cloud autoscaling, global controller HA, or cross-region deployment.
- Claiming 8 RPS real-model throughput from existing deterministic evidence.
- Weakening NAC-ABE, token, lease, digest, or executable-artifact checks to meet
  latency targets.
- Claiming physical-node deployment, production cryptographic strength,
  cross-host resource awareness, or production release readiness from MiniNDN.
