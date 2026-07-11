# Feature Specification: Core Stream Parity And UAV Migration

**Parent**: `specs/084-ndnsf-occam-simplification/`
**Status**: Acceptance complete

## User Stories

### User Story 1 - One generic stream state engine (Priority: P1)

C++ and Python consumers observe identical reorder, duplicate, stale-session,
gap, overflow and adaptive-fetch behavior from the Core C++ engine.

### User Story 2 - UAV keeps domain policy (Priority: P1)

UAV video uses the Core stream state engine while retaining H264 framing, FEC
repair, ROI, decoder backlog, MAVLink, mission and safety policy in UAV code.

### User Story 3 - Static objects remain exact-name retrieval (Priority: P2)

Models, files, catalog snapshots and planned tensor bundles use segmented
exact-name retrieval, never continuous StreamChunk semantics.

## Functional Requirements

- **FR-001** C++ `StreamProducerBuffer`, `StreamConsumerReorderBuffer` and
  `StreamAdaptiveFetcherState` MUST be the sole generic state algorithms.
- **FR-002** Python MUST bind or thinly adapt the C++ engine with deterministic
  field/default/error conversion.
- **FR-003** Session mismatch, duplicate, reorder, gap, skip, pending overflow,
  metrics and adaptive decisions MUST have C++/Python parity.
- **FR-004** Thread-safety and callback ownership MUST be explicit.
- **FR-005** UAV MUST reuse generic sequence/reorder/gap/health/adaptive state.
- **FR-006** H264, FEC codec, ROI, MAVLink, mission, authority, decoder backlog
  and user-visible labels MUST remain UAV-owned.
- **FR-007** Static/finite objects MUST use exact-name segmented retrieval.
- **FR-008** Unknown versions and malformed chunks MUST fail closed.

## Success Criteria

- **SC-001** Shared parity vectors produce identical state and metrics in C++
  and Python.
- **SC-002** No second Python reorder/adaptive algorithm remains.
- **SC-003** Three matched UAV MiniNDN loss campaigns preserve stale rejection,
  FEC recovery, bounded buffering, gap/drop counts, completion and latency.
- **SC-004** Forbidden static-object StreamChunk tests pass.
