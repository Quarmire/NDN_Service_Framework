# Feature Specification: DI Policy And Lifecycle Isolation

**Feature Branch**: `Experimental`
**Parent**: `specs/084-ndnsf-occam-simplification/`
**Status**: Acceptance complete

## Intent

Keep the default NDNSF-DI runtime distributed and executable without an
advisory coordinator or semantic cache. Provider admission leases remain the
only execution authority. Deployment records are descriptive. Optional
research policies must be imported explicitly from `experimental/` and cannot
silently change default planning or authorization.

## User Stories

### User Story 1 - Coordinator-free execution (Priority: P1)

A user builds and executes a plan using provider ACK/runtime hints and
provider-owned admission leases. Multiple users may contend, but correctness
does not depend on a coordinator process.

**Acceptance**: NativeTracer and Qwen workflows pass with no advisory module
imported; lease rejection is explicit and bounded.

### User Story 2 - Explicit experimental policy (Priority: P2)

A researcher may explicitly import semantic caching, enable it, and collect
evidence. It is not reachable through the default
`ndnsf_distributed_inference` import surface. Advisory coordination was removed
after failing its frozen retention gate.

**Acceptance**: default-import isolation tests pass; disabled experimental
configuration cannot modify assignments or responses.

### User Story 3 - Safe application retry (Priority: P2)

A DI user driver retries only when the caller declares an operation idempotent
and supplies a typed retry reason. Human-readable error text never authorizes a
retry.

## Functional Requirements

- **FR-001** Default DI planning MUST be pure user-side planning plus
  provider-owned admission.
- **FR-002** Deployment publication MUST be descriptive and MUST NOT carry
  ref-count or eviction authority.
- **FR-003** The default planner registry MUST contain executable handlers
  only; unsupported planner kinds MUST be absent and fail by lookup.
- **FR-004** Advisory coordination MUST be deleted when its frozen retention
  gate fails; the generic Core coordination envelopes remain available to
  other applications without making advisory planning part of DI.
- **FR-005** Semantic cache MUST live under `experimental/semantic_cache/`, be
  default-off, and remain application/provider policy.
- **FR-006** Provider-local Exact Forward Cache MUST remain available as a
  strict exact-match optimization and MUST NOT be conflated with semantic
  cache.
- **FR-007** Retry decisions MUST accept explicit idempotency metadata and a
  typed reason; string matching MUST NOT determine safety.
- **FR-008** Coordinator-off multi-user operation MUST fail closed through
  provider lease rejection without global deadlock.
- **FR-009** Advisory retention MUST use the frozen experiment gate in the
  parent task T032; failure deletes the feature rather than weakening the gate.

## Success Criteria

- **SC-001** Default import exposes no advisory or semantic-cache symbol.
- **SC-002** Every default registry entry can execute a valid request.
- **SC-003** Unit tests prove non-idempotent operations never retry, regardless
  of error text, and typed retryable reasons retry within the configured bound.
- **SC-004** Coordinator-off NativeTracer/Qwen and multi-user MiniNDN workflows
  complete within their existing thresholds.
- **SC-005** Ten matched runs determine retention. The measured gate failed, so
  DI contains no advisory coordinator implementation, wire service, CLI mode,
  or advisory campaign branch.

## Non-Goals

- Centralizing plan authority.
- Moving provider admission leases into DI.
- Making semantic similarity a framework claim.
- Publishing provider-local exact cache contents into NDN in-network cache.
