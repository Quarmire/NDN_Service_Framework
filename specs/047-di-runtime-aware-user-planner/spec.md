# Feature Specification: DI Runtime-Aware User-Side Planner

**Feature Branch**: `047-di-runtime-aware-user-planner`

**Created**: 2026-07-05

**Status**: Draft

**Input**: User description: "Keep the planner as a user-side role, but make NDNSF-DI handle multiple users planning concurrently by considering provider fragment residency, provider queue/admission state, and provider-to-provider bandwidth/RTT. Use Spec Kit to produce a complete design and task list."

## User Scenarios & Testing *(mandatory)*

### Scope Split

This feature deliberately separates reusable NDNSF framework mechanisms from
NDNSF-DI-specific planning semantics:

- **NDNSF core owns generic mechanisms**: ACK metadata envelopes, generic
  admission leases, selection lease validation hooks, generic provider runtime
  hints, directed peer network telemetry, reason codes, and diagnostics that
  can be reused by UAV, repository, payment/workflow, and other service
  applications.
- **NDNSF-DI owns inference semantics**: model fragment identity, GPU/CPU/disk
  residency, KV-cache locality, model-stage roles, dependency byte estimates,
  and graph-placement cost models.

The core must not learn about model layers, GPU-loaded fragments, KV cache, or
LLM stages. NDNSF-DI must express those details through DI payloads carried by
the generic core metadata and lease envelopes.

### User Story 1 - Runtime-aware assignment from provider ACKs (Priority: P1)

An NDNSF-DI user sends an inference request while keeping planning local to the user process. The user-side planner starts from a reusable plan template, collects provider ACKs containing current runtime state and lease offers, and chooses a runtime assignment that is valid for the current provider state.

**Why this priority**: This is the minimum change that preserves the current user-side planner direction while preventing stale static plans from becoming execution decisions.

**Independent Test**: Can be tested with a deterministic planner fixture where providers report different fragment residency and queue states; the chosen assignment must prefer valid leases and already-resident fragments.

**Acceptance Scenarios**:

1. **Given** two providers can run the same role and only one reports the required fragment as GPU-loaded, **When** the user-side planner scores ACKs, **Then** it selects the GPU-loaded provider when lease, queue, and network constraints are otherwise acceptable.
2. **Given** a provider reports `QUEUE_OVERLOADED` or no valid lease, **When** the user-side planner builds a runtime assignment, **Then** that provider is excluded from selected roles.
3. **Given** a provider reports a disk-resident fragment and another reports repo-only availability, **When** both can execute the role, **Then** the disk-resident provider receives a lower ready-cost than the repo-only provider.

---

### User Story 2 - Multi-user conflict control through provider leases (Priority: P1)

Multiple users send inference requests at the same time. Each user still creates its own plan, but providers are the authority for real resource availability. Providers grant short-lived leases for roles and validate those leases when a selection arrives.

**Why this priority**: Without leases, concurrent user-side planners can all choose the same apparently-idle provider and overload it.

**Independent Test**: Can be tested with two user requests racing for one provider role slot; only one valid lease should be consumed immediately while the other is delayed, rejected, or assigned to another provider.

**Acceptance Scenarios**:

1. **Given** User A has a valid lease for Provider P role R, **When** User B asks for the same constrained role before the slot is released, **Then** Provider P returns a later estimated start time or a rejection reason instead of another identical immediate lease.
2. **Given** a selection carries an expired lease, **When** the provider receives it, **Then** the provider rejects the selection with `LEASE_EXPIRED` and does not execute the role.
3. **Given** a selection carries a lease for a different role or fragment key, **When** the provider validates it, **Then** the provider rejects it with a mismatch reason and does not consume execution resources.

---

### User Story 3 - Edge-aware placement using provider-to-provider network metrics (Priority: P1)

The user-side planner treats distributed inference as graph placement. It scores provider nodes using capability, queue, lease, and fragment residency, and scores dependency edges using provider-to-provider RTT, bandwidth, packet loss, and confidence.

**Why this priority**: Distributed inference latency depends heavily on inter-provider transfer of hidden states, activations, KV shards, and partial outputs. User-provider RTT alone is not enough.

**Independent Test**: Can be tested with a small stage graph where the compute-optimal assignment uses a poor provider-provider edge and the edge-aware assignment chooses a slightly slower provider pair with much better transfer cost.

**Acceptance Scenarios**:

1. **Given** Stage 0 output must move from P1 to P2 and P1-to-P2 bandwidth is low, **When** an alternative assignment keeps both stages on P1 or uses P1-to-P3 with higher bandwidth, **Then** the planner includes edge cost and may choose the lower end-to-end assignment.
2. **Given** provider-peer metrics are stale or low-confidence, **When** the planner scores an assignment, **Then** it applies a configurable uncertainty penalty.
3. **Given** no provider-to-provider metric exists for an edge, **When** scoring the dependency, **Then** the planner falls back to a conservative transfer estimate rather than treating the edge as free.

---

### User Story 4 - Replan after stale runtime state (Priority: P2)

A user-side planner may choose a provider based on ACKs that become stale before execution. The user should recover by excluding failed leases/providers, refreshing state when needed, and retrying within bounded attempts.

**Why this priority**: Runtime state changes quickly under multi-user load, and bounded replan is the practical safety net.

**Independent Test**: Can be tested by forcing one selected provider to reject a lease after selection; the user must replan with the next valid assignment and record the replan reason.

**Acceptance Scenarios**:

1. **Given** a provider rejects selection because the fragment was evicted, **When** the user receives the rejection, **Then** the user excludes that lease and attempts a bounded replan.
2. **Given** all candidate leases expire, **When** replan attempts are exhausted, **Then** the request fails with a structured planner reason instead of timing out silently.
3. **Given** replan succeeds, **When** final response is received, **Then** metrics include replan count and the reason for the original rejection.

---

### User Story 5 - Evidence through MiniNDN multi-user campaigns (Priority: P2)

Researchers run a MiniNDN campaign that compares static user-side planning with runtime-aware lease planning under multiple simultaneous users and asymmetric provider-to-provider links.

**Why this priority**: The feature is only useful if it improves or explains latency, success rate, and resource utilization under realistic contention.

**Independent Test**: Can be tested by running one documented MiniNDN command that emits per-provider lease, residency, network-edge, and latency metrics.

**Acceptance Scenarios**:

1. **Given** a topology with asymmetric provider-provider bandwidth/RTT, **When** static and edge-aware planners run the same workload, **Then** the output reports selected assignments, edge costs, p50/p95 latency, success rate, and provider utilization.
2. **Given** two or more users submit overlapping requests, **When** lease-aware planning is enabled, **Then** the campaign reports lease granted/rejected/expired/consumed counts.
3. **Given** providers report GPU-loaded, CPU-resident, disk-resident, and repo-available fragments, **When** the campaign finishes, **Then** it reports residency hit rate and load/fetch counts.

### Edge Cases

- Provider ACK omits runtime state or lease fields; the planner must fall back to legacy compatibility or conservative scoring.
- Provider-to-provider metrics are asymmetric; P1-to-P2 and P2-to-P1 must be treated as different edges.
- Provider state is stale by the time selection arrives; lease validation must be authoritative.
- A provider evicts a fragment after ACK but before execution; selection/execution must fail with structured reason and trigger bounded replan.
- Multiple users receive valid future-start leases from the same provider; the provider must respect lease ordering or reject stale selections.
- A dependency edge has large data size but no bandwidth estimate; the planner must not treat the transfer as zero cost.
- A provider has the fragment GPU-loaded but lacks enough free KV-cache or execution memory for the requested context.
- Long-context requests have exact KV prefix locality on a provider that is otherwise less compute-optimal; the planner must include KV locality as a node-cost benefit.
- A provider reports very strong state but low confidence or old timestamp; the planner must penalize it.
- User-side planner reaches max replan attempts; the request must fail with structured evidence.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The system MUST keep planning as a user-side role for this feature; no dedicated planner service or provider-as-planner role is required for the MVP.
- **FR-002**: The system MUST separate reusable plan templates from per-request runtime assignments.
- **FR-003**: NDNSF core MUST provide a generic ACK metadata envelope that can carry structured service-defined runtime hints without requiring the core to understand application-specific fields.
- **FR-004**: NDNSF core MUST provide a generic admission lease envelope that binds request id, service name, provider identity, expiration time, estimated start/finish time, status, reason code, and an opaque service-defined resource binding payload.
- **FR-005**: NDNSF core MUST allow provider selection to carry a selected lease id and resource binding proof so the provider can validate admission before execution.
- **FR-006**: NDNSF core MUST reject expired, missing, mismatched, or already-consumed leases before executing the selected service handler.
- **FR-007**: NDNSF core MUST expose reusable provider runtime hints such as queue length, active work count, estimated queue wait, available capacity hints, timestamp, and confidence without defining DI-specific resources.
- **FR-008**: NDNSF core MUST expose directed peer network telemetry such as peer name, RTT, bandwidth, loss, jitter, timestamp, and confidence in a form reusable by non-DI applications.
- **FR-009**: NDNSF-DI MUST define a canonical model fragment identity that distinguishes model id, model version or digest, runtime backend, precision, split strategy, stage/layer range, shard index, and fragment digest.
- **FR-010**: NDNSF-DI MUST define fragment residency levels including GPU-loaded, CPU-resident, disk-resident, repo-available, and missing as DI-specific runtime hints.
- **FR-011**: NDNSF-DI MUST define a DI lease resource binding payload that binds role id, model fragment identity, residency, reserved GPU/CPU memory hints, and DI-specific readiness estimates inside the generic NDNSF core lease.
- **FR-012**: The NDNSF-DI planner MUST score an assignment as graph placement: role/provider node costs plus dependency/provider-pair edge costs.
- **FR-013**: The DI edge cost MUST include provider-to-provider RTT and transfer time derived from dependency bytes and bandwidth, with penalties for loss, jitter, stale metrics, and unknown metrics.
- **FR-014**: Provider ACKs MUST be able to carry generic core runtime metadata plus DI-specific runtime payloads while preserving compatibility with existing ACK selection behavior.
- **FR-015**: Providers MUST release or expire leases predictably when roles complete, selections time out, or lease expiration is reached.
- **FR-016**: The user-side planner MUST exclude invalid leases and providers during bounded replan attempts.
- **FR-017**: The user-side planner MUST record structured reasons for assignment choice, rejected candidates, replan events, and final failure.
- **FR-018**: The feature MUST expose metrics for lease granted/rejected/expired/consumed, residency hits, provider queue wait, dependency edge cost, replan count, selected assignments, p50/p95 latency, success rate, and provider utilization.
- **FR-019**: The MiniNDN validation MUST include at least one multi-user scenario and one asymmetric provider-to-provider network scenario.
- **FR-020**: Existing NDNSF security behavior MUST remain intact: NAC-ABE attributes, user/provider tokens, replay protection, provider permissions, and V2 naming must not be bypassed.
- **FR-021**: Legacy non-runtime-aware ACKs MUST either remain usable through conservative scoring or fail with a clear unsupported-feature reason when runtime-aware mode is explicitly required.
- **FR-022**: The design MUST support exact KV-cache locality as a future or optional DI node-cost input without confusing it with semantic cache matching.

### Key Entities

- **PlanIntent**: The user request for model, input, context size, output constraints, latency target, and required service name.
- **PlanTemplate**: Reusable model split description that defines roles, fragments, dependency edges, estimated memory/compute cost, and valid provider constraints.
- **RuntimeAssignment**: Per-request mapping from template roles to provider leases.
- **GenericAckMetadata**: NDNSF core envelope for structured service-defined provider metadata.
- **GenericAdmissionLease**: NDNSF core lease envelope for reusable provider admission control.
- **GenericProviderRuntimeHint**: NDNSF core provider state summary that avoids application-specific semantics.
- **PeerNetworkMetric**: NDNSF core directed peer telemetry usable by DI and non-DI services.
- **ModelFragmentKey**: Canonical identity for a model fragment, stage, or shard.
- **FragmentRuntimeState**: Provider-reported residency and readiness estimate for a specific fragment.
- **DiProviderRuntimeState**: DI-specific provider state built from generic hints plus fragment and KV-cache state.
- **DiLeaseResourceBinding**: DI-specific payload carried inside a generic admission lease.
- **DependencyEdgeCost**: Planner estimate for moving a dependency object between two selected providers.
- **LeaseValidationResult**: Provider decision when a selection attempts to consume a lease.
- **ReplanRecord**: User-side evidence describing why a plan had to be retried or failed.
- **PlannerMetrics**: Aggregated evidence for latency, assignment, lease, residency, edge-cost, and utilization behavior.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: In a deterministic fixture, the planner chooses a GPU-loaded fragment provider over CPU/disk/repo alternatives when other costs are equal.
- **SC-002**: In a deterministic fixture, the planner chooses a lower end-to-end graph assignment when provider-to-provider edge cost outweighs a compute-only ranking.
- **SC-003**: In a two-user contention test, a provider does not execute two immediate role selections for the same single-slot resource unless it issued two valid non-conflicting leases.
- **SC-004**: A selection with an expired or mismatched lease is rejected before role execution and produces a structured reason.
- **SC-005**: A forced stale-state failure triggers bounded replan and records the failed lease/provider reason.
- **SC-006**: MiniNDN campaign output contains p50/p95 latency, success rate, selected assignments, lease counters, residency hit counters, inter-provider edge costs, and provider utilization.
- **SC-007**: In an asymmetric network MiniNDN topology, the edge-aware planner avoids a poor provider-provider dependency edge when an alternative assignment has lower estimated end-to-end cost.
- **SC-008**: Existing focused security and token regressions remain passing after lease/ACK changes.

## Assumptions

- The first implementation keeps the planner in the user process and does not introduce a centralized planner service.
- Lease/admission is the concurrency-control boundary; users do not coordinate directly with each other.
- Provider-to-provider network metrics can initially come from configured MiniNDN topology, passive dependency transfer timing, or synthetic provider ACK fixtures.
- Runtime-aware mode may be enabled per DI workload/profile so existing non-DI examples do not need to change immediately.
- Provider state reports are hints for planning; lease validation is authoritative.
- Generic leases and peer telemetry should be useful for future non-DI applications; DI-specific fields must stay in opaque service-defined payloads from the core perspective.
- Model fragment identity is digest-based where possible; human-readable stage names are not sufficient for equality.
- Exact KV-cache locality is considered only when exact model, split, stage, prefix digest, and token prefix metadata match.
- This feature plans and tasks the design; implementation will follow in later work unless explicitly requested.
