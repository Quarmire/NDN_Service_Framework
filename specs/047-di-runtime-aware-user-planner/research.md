# Research: DI Runtime-Aware User-Side Planner

## Decision: Keep planner as a user-side role for the MVP

**Rationale**: The user explicitly chose User-side Planner. This preserves the current NDNSF-DI direction and avoids introducing a planner service as a new availability/security boundary.

**Alternatives considered**:
- Dedicated planner service: better global optimization, but changes deployment and adds service availability concerns.
- Provider-as-planner: useful later for coordinator-style deployments, but risks mixing resource owner and global assignment authority.

## Decision: Use provider leases as the concurrency-control boundary

**Rationale**: Multiple users can plan independently only if providers are authoritative about real resources. A lease makes an ACK actionable but time-bounded, and a selection must prove it is consuming the intended lease.

**Alternatives considered**:
- Trust user-side static state: fails under concurrent users.
- Let users coordinate with each other: not scalable and contrary to NDNSF service abstraction.
- Central lock service: stronger global control but outside the selected User-side Planner scope.

## Decision: Put generic lease/admission in NDNSF core

**Rationale**: Admission leases are not DI-specific. UAV commands, repository operations, workflow steps, and future service applications may all need short-lived provider admission, structured rejection reasons, and selection-time validation.

**Alternatives considered**:
- DI-only lease model: faster to prototype, but duplicates a reusable service-invocation concern and prevents other NDNSF applications from sharing the mechanism.
- Fully application-opaque ACK payload only: preserves flexibility but gives the core no reusable way to validate leases before service execution.

## Decision: Keep fragment/GPU/KV semantics in NDNSF-DI

**Rationale**: Model layers, fragment residency, GPU memory, CPU-to-GPU load cost, and KV-cache locality are distributed-inference semantics. Putting them into NDNSF core would make the base framework less general and harder for non-DI applications to use.

**Alternatives considered**:
- Core `ModelFragmentKey`: rejected because model split identity is not meaningful for UAV, repository, or generic service applications.
- Core GPU-memory fields: rejected because providers may expose many resource types; DI should encode GPU-specific details as service-defined metadata.

## Decision: Treat planning as graph placement

**Rationale**: Distributed inference has role nodes and dependency edges. Provider capability, queue, memory, and fragment residency are node costs; activation/hidden-state/KV/partial-output transfer between selected providers is edge cost.

**Alternatives considered**:
- Rank providers independently per role: misses poor provider-to-provider links and can choose locally optimal but globally slow assignments.
- Use only topology estimates: misses runtime congestion, queue, cache, and lease state.

## Decision: Make ModelFragmentKey digest-based and split-aware

**Rationale**: `stage0` is not globally meaningful. Fragment identity must include model, version/digest, runtime backend, precision, split strategy, stage/layer range, shard position, and fragment digest to distinguish compatible fragments.

**Alternatives considered**:
- Human-readable stage labels: too ambiguous across split plans.
- Provider-local file paths: not portable or verifiable across providers.

## Decision: Keep ACK payload bounded and layered

**Rationale**: Full provider inventory and all peer metrics can be large. ACKs should carry generic core hints and lease envelopes plus DI-specific relevant fragment states in a service-defined payload. They may also carry top-k relevant peer metrics or a metric digest/name for optional fetch.

**Alternatives considered**:
- Full inventory in every ACK: simple but wasteful and can increase ACK latency.
- Separate metrics fetch only: cleaner but too slow for the MVP critical path unless cached.

## Decision: Provider-to-provider metrics are directed and confidence-scored

**Rationale**: Bandwidth/RTT/loss can be asymmetric and stale. The planner must model `P1 -> P2` independently from `P2 -> P1` and penalize stale or low-confidence values.

**Alternatives considered**:
- Undirected average metrics: hides asymmetric links.
- No confidence/staleness: makes stale measurements look authoritative.

## Decision: Bounded replan instead of unbounded retry

**Rationale**: Runtime state can change after ACK. Bounded replan handles lease expiration, fragment eviction, or selection rejection without letting a single request loop forever.

**Alternatives considered**:
- No replan: brittle under contention.
- Unbounded retry: can amplify load during congestion.
