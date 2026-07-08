# Feature Specification: Core Runtime Contract Completion

**Feature Branch**: `058-core-runtime-contract-completion`

**Created**: 2026-07-08

**Status**: Draft

**Input**: User description: "Turn the core/app boundary audit findings for NDNSF-DistributedRepo, NDNSF-UAV-APP, and NDNSF-DistributedInference into detailed Spec Kit design, task list, and complete implementation."

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Core Runtime Envelopes Are Reusable Across C++ and Python (Priority: P1)

As an NDNSF application developer, I need provider readiness, operation status, data-product references, leases, and rejection reasons to use one reusable NDNSF contract instead of app-specific ad-hoc fields.

**Why this priority**: Repo, UAV, and DI all report provider readiness and long-running work. Without a common contract, each app invents parsing and diagnostic logic.

**Independent Test**: A C++ unit test and Python regression can encode/decode the same envelope concepts while legacy ACK fields continue to parse.

**Acceptance Scenarios**:

1. **Given** a C++ provider wants to report readiness, **When** it builds a provider capability hint, **Then** the payload contains provider name, service name, readiness, reason code, runtime hint, and opaque app payload.
2. **Given** an app reports a long-running operation, **When** it builds an operation status, **Then** the status uses the reusable lifecycle vocabulary and can be embedded in provider capability metadata.

---

### User Story 2 - Apps Keep Domain Semantics But Emit Core-First Bridges (Priority: P2)

As a Repo, UAV, or DI maintainer, I need to keep domain-specific policy in the app while exposing common readiness/status/stream evidence through NDNSF core envelopes.

**Why this priority**: The framework should become more reusable without moving Repo catalog policy, UAV video/MAVLink logic, or DI model planning into core.

**Independent Test**: Existing migration tests prove Repo ACK parsing prefers core hints, UAV maps adaptive video state to StreamHealth, and DI provider ACKs expose typed provider capability hints.

**Acceptance Scenarios**:

1. **Given** Repo emits storage capacity, **When** a client parses the ACK, **Then** it prefers ProviderCapabilityHint and falls back to legacy storage fields only if needed.
2. **Given** UAV reports adaptive video state, **When** stream health is requested, **Then** the core StreamHealth shape reports degraded/congested/stale state without taking over codec policy.
3. **Given** DI reports model/runtime readiness, **When** MiniNDN summarizes ACKs, **Then** coreEnvelopeSummary includes provider readiness and reason-code evidence.

---

### User Story 3 - Service Discovery, Drain, and Provider-Pair Telemetry Are Core Surfaces (Priority: P3)

As an experiment operator, I need a reusable way to discover healthy providers, recognize draining providers, and rank provider pairs by network telemetry, so each app does not implement these mechanics differently.

**Why this priority**: DI, Repo, and UAV all need discovery and telemetry, but their scoring policies are different. Core should provide common facts; apps should decide how to use them.

**Independent Test**: Python tests can build discovery snapshots from NDNSD health entries, filter draining/unavailable providers, and rank provider pairs by reusable telemetry.

**Acceptance Scenarios**:

1. **Given** provider health and capability hints are available, **When** a service discovery snapshot is built, **Then** ready providers are listed separately from draining, stale, or unavailable providers.
2. **Given** provider-pair metrics exist, **When** an app asks for transfer ranking, **Then** the core ranks candidate pairs without knowing Repo/UAV/DI semantics.

### Edge Cases

- A provider publishes both legacy fields and typed core hints with conflicting values; core-first parsing must prefer the typed hint while keeping fallback compatibility.
- A provider is ready but draining; selection helpers must mark it non-preferred for new requests without implying existing requests failed.
- A provider has stale NDNSD health but fresh ACK capability; freshness rules must preserve both timestamps so the app can decide.
- A large exact-name object must not be forced through the continuous stream abstraction.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: NDNSF MUST define reusable runtime envelope contracts for provider capability, runtime hints, operation status, data-product references, rejection reasons, and admission leases.
- **FR-002**: C++ and Python NDNSF core MUST expose compatible provider capability and operation status helpers sufficient for Repo, UAV, and DI bridge payloads.
- **FR-003**: Applications MUST keep domain-specific semantics in app payloads rather than moving catalog, video, mission, model, tensor, or cache policy into core.
- **FR-004**: Repo/UAV/DI bridge code MUST prefer typed core envelope fields when present and preserve legacy fallback behavior.
- **FR-005**: NDNSF core MUST provide a service-discovery snapshot helper that combines service name, provider identity, readiness, drain state, reason code, and timestamps.
- **FR-006**: NDNSF core MUST provide a generic drain/unavailable state vocabulary usable by provider capability hints and discovery filtering.
- **FR-007**: NDNSF core MUST expose provider-pair telemetry ranking helpers without embedding app-specific scoring policy.
- **FR-008**: Documentation MUST explicitly distinguish continuous streams from exact-name large-data retrieval.
- **FR-009**: Tests MUST prove core helpers independently and at least one Repo/UAV/DI migration path each.

### Key Entities *(include if feature involves data)*

- **ProviderCapabilityHint**: Provider readiness, service name, reason code, runtime hint, optional operation status, leases, and opaque app payload.
- **ServiceOperationStatus**: Reusable lifecycle status for long-running service work.
- **DataProductReference**: Named Data reference for artifacts, recordings, catalog objects, and other exact-name products.
- **ServiceDiscoverySnapshot**: Core view of providers for a service, including ready, draining, stale, and unavailable categories.
- **ProviderNetworkMatrix**: Core network telemetry for ranking provider-pair transfer cost.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Core C++ and Python envelope tests pass without importing Repo, UAV, or DI domain modules.
- **SC-002**: Repo/UAV/DI migration tests pass and show typed core envelopes are preferred when present.
- **SC-003**: Documentation lists which mechanisms belong in core and which remain in Repo/UAV/DI, with no unresolved boundary ambiguity for stream vs large-data transfer.
- **SC-004**: Existing focused Python regressions for core boundary envelopes and app migrations continue to pass.

## Assumptions

- This feature extends the existing Spec 049 boundary work rather than replacing it.
- The first implementation can use semicolon field payloads and JSON strings already used by the project; a future hard wire-format migration can be separate.
- MiniNDN full performance campaigns are not required for this contract completion; focused unit/regression tests are sufficient unless runtime behavior changes.
