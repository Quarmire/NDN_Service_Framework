# Feature Specification: NDNSF-DI Advisory Coordinator

**Feature Branch**: `048-di-advisory-coordinator`
**Created**: 2026-07-05
**Status**: Implemented MVP
**Input**: User-side planner remains the default, but multiple users may plan
at the same time and should avoid making conflicting choices when a lightweight
coordination hint is available.

## User Scenarios

### Scenario 1: User plans without coordinator

A user receives provider ACK metadata and builds a runtime-aware assignment
locally. If no coordinator is configured, the current Spec047 behavior is
unchanged.

### Scenario 2: Multiple users receive advisory suggestions

Several users submit `PlanIntent` objects to an optional coordinator. The
coordinator groups intents in a short window, scores each user's current
provider candidates, and returns `AdvisorySuggestion` objects that try to avoid
overusing the same provider.

### Scenario 3: User validates a suggestion before use

The user treats a suggestion as a hint. Before accepting it, the user verifies
the suggestion freshness/proof and rechecks that every suggested provider is
still present in the user's own ACK candidates and still has valid runtime
metadata or leases.

## Requirements

- **REQ-048-001**: User-side planning remains the default execution path.
- **REQ-048-002**: The coordinator is optional and disabled by default.
- **REQ-048-003**: A `PlanIntent` must identify request, user, template, nonce,
  lifetime, and optional utility weight.
- **REQ-048-004**: An `AdvisorySuggestion` must carry role assignments,
  coordinator name, window id, expiry, score breakdown, and proof field.
- **REQ-048-005**: Coordinator suggestions must be non-binding. Providers must
  still enforce admission leases and runtime availability.
- **REQ-048-006**: Users must ignore stale suggestions, context-mismatched
  suggestions, tampered proofs, and suggestions whose providers are not valid in
  the current local ACK/lease candidate set.
- **REQ-048-007**: The MVP must include deterministic tests for disabled mode,
  multi-user provider balancing, valid suggestion acceptance, stale suggestion
  rejection, tampered-proof rejection, and lease-validation rejection.

## Non-Goals

- No central executor.
- No direct user-to-user distributed consensus.
- No provider-side trust in coordinator suggestions.
- No new C++ wire protocol in the MVP.
- No replacement of UserToken, ProviderToken, NAC-ABE, provider permissions, or
  admission leases.

## Success Criteria

- Python runtime tests pass.
- Existing runtime-aware planner tests continue to pass.
- Documentation explains when to use coordinator and when to keep pure
  user-side planning.
