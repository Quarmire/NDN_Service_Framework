# Research Decisions: NDNSF Occam Simplification

## Decision 1: Correctness Before Minimality

**Decision**: Retain security, provider admission, execution leases,
prepare/commit/abort, rollback, exact-name Data, Repo quorum/repair, and UAV
safety mechanisms.

**Rationale**: These mechanisms prevent unauthorized execution, double
allocation, partial plans, data corruption, or unsafe vehicle commands. Their
complexity corresponds to real failure modes.

**Alternative rejected**: Remove all leases/coordinator-related concepts. This
would confuse optional advisory planning with mandatory resource admission.

## Decision 2: Advisory Coordination Is Optional Policy

**Decision**: Pure user-side planning plus provider-local leases is the default.
Advisory coordination may remain only as a DI experiment.

**Rationale**: Provider admission resolves asynchronous multi-user conflicts
without leader election. Advice may improve allocation but is not authority.

**Alternative rejected**: Make one coordinator mandatory. This adds a failure
domain, discovery/election problem, and a second source of scheduling truth.

## Decision 3: Provider-Local Authority And Fail-Closed Loss

**Decision**: Remove the broken/untracked local execution lease fallback and
replace coordinator-owned global refCount with provider-authoritative leases
and the prepare/commit/abort state machine in
`contracts/di-lease-authority.md`.

**Rationale**: A lease that other participants cannot observe is not a lease and
can permit eviction or conflicting execution.

**Alternative rejected**: Keep it as availability fallback. Availability gained
by bypassing safety is invalid.

## Decision 4: V2 Is The Only Core Invocation Protocol

**Decision**: Remove V1 split names, Bloom-filter provider encoding, deprecated
permission token indexing, and unused compatibility handlers after an ABI gate.

**Rationale**: Repository callers already use unified V2 APIs. Maintaining both
paths multiplies naming, parsing, authorization, and test logic.

**Alternative rejected**: Permanent aliases. They retain the implementation and
security burden rather than providing a narrow adapter.

## Decision 5: DI Concepts Belong In DI

**Decision**: Move deployment manager, execution artifact schema/materializer,
application retry, semantic cache, and optional coordinator out of Core.

**Rationale**: Repo and UAV do not need model fragments, backends, semantic
patterns, or DI refCount policy. Generic Core status/lease/large-data primitives
are sufficient building blocks.

## Decision 6: One Repo Runtime, Selected By An ADR

**Decision**: Use current implementations as behavior candidates and freeze
black-box parity fixtures. Child feature 088 selects the authoritative runtime
through `contracts/repo-decision-gate.md`; Spec 084 does not preselect C++ or
Python.

**Rationale**: Current HA evidence and C++ integration goals pull in different
directions. Exact packet, persistence, security, operational complexity,
maintainability, and performance must be evaluated together before deletion.

**Alternative rejected**: Keep both as equal implementations. Their service
contracts already differ and parity cost grows with every HA feature.

**Alternative rejected**: Select the language from one control-plane benchmark.
That measurement cannot establish persistence, correctness, or security fit.

## Decision 7: Small Public Repo API, Rich Private Protocol

**Decision**: Expose stable object operations; keep packet batching, pull,
capacity reservation, finalization, repair, and anti-entropy internal.

**Rationale**: Internal operations are required for HA but do not need to become
application-facing concepts.

## Decision 8: Core C++ Owns Stream State

**Decision**: Use one C++ reorder/health/adaptive implementation with Python
bindings after the observable parity contract passes. UAV owns codec and
vehicle-domain policy.

**Rationale**: Sequence, gaps, duplicates, stale sessions, and window decisions
are generic. H264 frame assembly and FEC recovery are application-specific.

## Decision 9: Bounded Compatibility, Not Permanent Fallback

**Decision**: Typed/legacy dual encoding and Normal/pull fallbacks must have
schema versions, counters, end conditions, and deletion tasks.

**Rationale**: Compatibility without an expiry becomes a second architecture.

## Decision 10: Experimental Features Must Look Experimental

**Decision**: Remove handler-less planner placeholders. Keep semantic cache and
advisory coordinator under explicit experimental namespaces and flags.

**Rationale**: A public backend that throws `NotImplementedError`, or an
approximate cache that silently affects selection, overstates supported behavior.

## Decision 11: Umbrella Governance, Not One Rewrite

**Decision**: Spec 084 owns program invariants and gates. Implementation occurs
in child specs 085-090 as defined by `contracts/child-feature-map.md`.

**Rationale**: Core protocol deletion, DI distributed state, Repo persistence,
stream bindings, and cross-project schema migration have different rollback and
evidence boundaries. Combining them makes review and recovery unsafe.

## Decision 12: Domain State Is Not A Legacy Alias

**Decision**: Typed migration first inventories every field. A flat field is
removed only when it is an alias or compatibility encoding, not when it carries
application-domain state such as deployment residency.

**Rationale**: Generic operation lifecycle and domain state may both use the
word `status` while representing different facts.

## Adversarial Review

- The plan would fail if “simplification” became a broad rewrite. Mitigation:
  migration-before-deletion, one concern per commit, matched regressions.
- Repo convergence may be too large for one child. Mitigation: first approve the
  runtime ADR, then use independently revertible parity slices for storage,
  catalog, write quorum, read failover, repair, and duplicate deletion; split
  child 088 again if any slice still contains an architecture decision.
- External V1 callers are not visible in repository scans. Mitigation: explicit
  release/ABI decision and optional separately packaged adapter.
- Replacing UAV reorder logic could alter latency-sensitive skip behavior.
  Mitigation: encode current skip policy as tests before migration.
