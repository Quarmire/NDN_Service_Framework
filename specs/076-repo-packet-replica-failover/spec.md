# Feature Specification: Repo Packet Replica Failover

**Created**: 2026-07-10
**Status**: Complete

## User Scenarios & Testing

### User Story 1 - Survive a Repo failure during packet-set retrieval (P1)

An application retrieves a signed packet set from two replicas. After the first
packet arrives from the primary Repo, that Repo stops. The application discards
the incomplete attempt and retrieves the complete ordered set from the secondary
Repo.

**Independent Test**: Store the same exact packet set at Repo A and Repo B in
MiniNDN, terminate Repo A after its first successful packet, and verify that Repo
B serves every manifest packet in order with identical wires.

**Acceptance Scenarios**:

1. **Given** two complete replicas, **When** Repo A fails after one packet,
   **Then** the final result contains the full set obtained by a fresh Repo B
   attempt and no partial mixed result.
2. **Given** a target Repo preparation response, **When** the exact packet
   Interest is sent, **Then** the target Repo identity is carried as a forwarding
   hint so identical application Data prefixes can be disambiguated.

### User Story 2 - Produce reproducible failover evidence (P2)

An experiment operator receives a machine-readable record of failure timing,
per-replica packet attempts, total latency, exact names, wire identity, and the
fact that the secondary restarted from packet zero.

## Edge Cases

- The trigger is never reached because the primary fails before one packet.
- The secondary is missing one packet.
- A stale route remains after the primary process exits.
- Both replicas advertise the same application-owned Data prefix.
- The control resume file is delayed or absent.

## Functional Requirements

- **FR-001**: Exact packet retrieval MUST use the forwarding hints returned by
  the selected Repo preparation response.
- **FR-002**: Packet-set failover MUST restart retrieval at the first manifest
  name on the next replica and MUST NOT merge partial attempts.
- **FR-003**: The MiniNDN harness MUST terminate the real Repo A process after
  exactly one successful packet fetch and then release the client to continue.
- **FR-004**: The seed path MUST store the same packet wires at Repo A and Repo B.
- **FR-005**: The result MUST record per-replica call order, successful primary
  packet count, secondary complete-set count, failover latency, names, and hashes.
- **FR-006**: Existing one-replica exact packet and opaque-object behavior MUST
  remain unchanged.

## Success Criteria

- **SC-001**: The failover result reports all checks true after Repo A is killed.
- **SC-002**: Repo A successfully supplies exactly one packet before failure.
- **SC-003**: Repo B is asked for every manifest packet from the first name.
- **SC-004**: Every returned packet name and wire hash matches the seed record.
- **SC-005**: Focused Python and existing Repo regressions pass.

## Assumptions

- MiniNDN namespaces share the host filesystem, allowing deterministic
  trigger/resume files without adding a production protocol hook.
- Failure injection belongs only to the example/test harness, never production
  Repo or NDNSF wire APIs.

## Out of Scope

- Parallel hedged reads or packet-by-packet multi-source assembly.
- Catalog repair after permanent replica loss.
- Changing NLSR convergence or NDN-SVS timing.
