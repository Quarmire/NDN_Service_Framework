# Feature Specification: Typed Envelopes Without Semantic Loss

**Parent**: `specs/084-ndnsf-occam-simplification/`
**Status**: Complete

## Intent

Make `ProviderCapabilityHint` the authoritative ACK capability envelope and
stop current DI and Repo producers from also emitting duplicate flat aliases.
Preserve application-domain state inside declared service payloads. Support a
bounded, observable legacy-reader epoch without allowing malformed or unknown
typed data to fall back silently.

## User Stories

### User Story 1 - One authoritative ACK view (Priority: P1)

A current consumer receives one typed provider capability, runtime, rejection,
network, lease, operation-status, and service-specific payload view. If typed
and legacy values conflict, typed wins and the conflict is counted.

### User Story 2 - Bounded rolling upgrade (Priority: P1)

A mixed-version deployment can explicitly enable legacy-only reads while all
current producers emit typed-only ACK metadata. Unknown typed versions and
malformed typed envelopes fail closed instead of using legacy aliases.

### User Story 3 - Domain semantics survive simplification (Priority: P1)

Repo storage/residency, DI fragment/model state, and UAV mission/video/safety
state remain in their owning application schema and are not removed merely
because they contain fields named status, state, load, or availability.

## Functional Requirements

- **FR-001** `ProviderCapabilityHint` MUST use explicit schema version v2 and
  be the typed authority for current ACK capability metadata.
- **FR-002** A shared decoder MUST expose typed-only and mixed modes, with typed
  authority and process-local legacy/conflict/malformed/unknown counters.
- **FR-003** A present malformed or unknown typed envelope MUST fail closed;
  legacy fallback is allowed only when no typed envelope is present and mixed
  mode is explicit.
- **FR-004** Current DI Python, DI native C++, and Repo Python producers MUST
  emit no duplicate flat capability/runtime/rejection/lease aliases.
- **FR-005** Current consumers MUST obtain common fields from the typed envelope
  and domain fields from `servicePayload`, not from duplicate top-level aliases.
- **FR-006** Generic Core `GenericAckMetadata`, network metrics, lease objects,
  and operation status MUST retain their independent semantics and schemas.
- **FR-007** Repo storage/cache/catalog state, DI residency/model/role state,
  and UAV mission/video/safety state MUST remain application-owned.
- **FR-008** Stored Repo SQLite data, exact Data wire, DI plan/cache artifacts,
  and UAV mission/config files MUST require no rewrite; any stored alias not
  proven safe to remove MUST be retained and classified.
- **FR-009** Compatibility mode, deadline, counters, and removal exit criteria
  MUST be documented and machine-tested.
- **FR-010** Typed-only, legacy-only, matching dual, conflicting dual,
  malformed typed, unknown typed version, restart, and rollback fixtures MUST
  pass across Core, DI, Repo, and UAV ownership boundaries.

## Success Criteria

- **SC-001** Current producer ACK payloads contain `providerCapabilityHint` and
  no inventoried flat legacy alias.
- **SC-002** Conflict fixtures select typed values and increment counters;
  malformed/unknown typed fixtures fail closed.
- **SC-003** Mixed-reader and typed-only MiniNDN smokes complete with zero
  unexplained conflicts and typed-only producer evidence.
- **SC-004** Full Core, DI, Repo, UAV, security, persistence, and exact-wire
  regressions pass without semantic field loss.

## Non-Goals

- Replacing domain payload schemas with one generic status object.
- Removing `GenericAckMetadata` or app-specific persisted state.
- Changing Request/ACK/Selection/Response names, NAC-ABE, tokens, or security.
