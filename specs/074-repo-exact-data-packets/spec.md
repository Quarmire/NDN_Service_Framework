# Feature Specification: Exact NDN Data Packet Repository

**Feature Branch**: `Experimental`
**Created**: 2026-07-10
**Status**: In Progress

## User Scenarios & Testing

### User Story 1 - Preserve published Data packets exactly (Priority: P1)

An application publishes already-signed segmented NDN Data under names such as
`/data/model/v=42/seg=0`. A Repo stores each packet under that exact Data name
and later returns the original wire packet without changing its name, content,
metadata, final block, or signature.

**Independent Test**: Store several signed Data packets, restart the Repo, fetch
each exact segment name, and compare both the returned name and SHA-256 of the
complete wire encoding with the original.

**Acceptance Scenarios**:

1. **Given** a valid signed packet named `/data/x/v=7/seg=2`, **when** it is
   inserted, **then** the authoritative storage key is that complete name.
2. **Given** a stored packet, **when** it is fetched by its complete name,
   **then** the returned wire encoding is byte-identical to the inserted packet.
3. **Given** a Repo restart, **when** the same name is fetched, **then** the
   packet remains available without being re-signed or renamed.

---

### User Story 2 - Use manifests as indexes, not alternate packet names (Priority: P1)

An application may use a logical object manifest to discover a packet set. The
manifest records the exact packet names in order. It does not cause the Repo to
create names such as `<object>/seg/N` or `<object>/ndn-data/N`.

**Independent Test**: Store one packet set through a manifest, inspect the
inventory, and verify that every stored packet key is an original Data name and
that no Repo-generated segment alias exists.

**Acceptance Scenarios**:

1. **Given** a packet-set manifest, **when** it is stored, **then** it indexes
   the exact Data names and their order.
2. **Given** a known exact Data name, **when** a client fetches it, **then** the
   client does not need the logical object name to identify the packet.
3. **Given** a logical object lookup, **when** the manifest is returned, **then**
   a client can fetch and reassemble the object from its exact packet names.

---

### User Story 3 - Share immutable packets safely (Priority: P2)

Two manifests may refer to the same immutable Data packet. The Repo stores one
authoritative packet copy, tracks both references, and does not remove the
packet until the final reference is deleted.

**Independent Test**: Reference one exact Data name from two manifests, delete
one manifest, verify the packet remains, then delete the second and verify that
the unreferenced packet is reclaimed according to policy.

### Edge Cases

- A wire packet cannot be decoded as NDN Data.
- The caller-provided Data name differs from the name encoded in the wire.
- The same exact name is inserted with different wire bytes.
- A packet set contains duplicate names, missing segment numbers, mixed base
  names, mixed versions, or inconsistent FinalBlockId values.
- A manifest update replaces some packet references while retaining others.
- A packet is larger than the memory cache budget but fits authoritative storage.
- A Repo restarts with packets in SQLite and an empty memory cache.
- An Interest asks for an unstored exact name or a prefix that is ambiguous.

## Requirements

### Functional Requirements

- **FR-001**: The Repo MUST parse every submitted wire packet as NDN Data before
  committing it.
- **FR-002**: The authoritative key of a stored packet MUST be the complete Data
  name encoded in that packet, including version and segment components.
- **FR-003**: The Repo MUST preserve and return the complete wire encoding
  byte-for-byte; it MUST NOT re-sign, rename, re-segment, or reconstruct it.
- **FR-004**: A caller-supplied packet name, when present, MUST exactly match the
  name encoded in the wire or the insertion MUST fail.
- **FR-005**: Re-inserting the same name and same wire MUST be idempotent.
- **FR-006**: Re-inserting the same name with different wire bytes MUST fail as
  an immutable-name conflict.
- **FR-007**: A logical object manifest MUST index an ordered list of exact Data
  names and MUST NOT create Repo-owned segment aliases.
- **FR-008**: The Repo MUST support lookup of one packet by exact Data name and
  lookup of a packet set through its manifest.
- **FR-009**: A network fetch MUST register/advertise the original packet prefix
  derived from stored names, never a Repo-derived replacement prefix.
- **FR-010**: Packet retrieval after restart MUST read through the SQLite
  authority and may admit the packet to the bounded memory cache.
- **FR-011**: Packet cache entries MUST be keyed by exact Data name.
- **FR-012**: Shared packet references MUST not duplicate authoritative wire
  storage, and deleting one manifest MUST not remove packets still referenced by
  another manifest.
- **FR-013**: Existing opaque-object STORE/FETCH APIs MAY remain for compatibility,
  but new packet insertion/fetch paths MUST NOT use `<object>/seg/N` or
  `<object>/ndn-data/N` as storage keys.
- **FR-014**: Operation status and errors MUST distinguish invalid wire,
  name/wire mismatch, immutable-name conflict, missing packet, and invalid packet
  set.
- **FR-015**: Documentation MUST explain that SegmentFetcher is a client-side
  reassembly helper over exact packet names; it does not authorize Repo renaming.

### Key Entities

- **ExactDataPacket**: Complete Data name, immutable wire bytes, wire digest,
  wire size, timestamps, and access counters.
- **RepoObjectManifest**: Logical object metadata plus an ordered list of exact
  Data names; it is an index, not packet storage.
- **ObjectPacketReference**: Relationship between one manifest, ordinal/segment,
  and one ExactDataPacket.
- **PacketServingSession**: Temporary producer registration for an original Data
  prefix; it serves stored packet wires without modification.

## Assumptions

- NDN Data names are immutable content identifiers within this Repo contract:
  the same full name cannot legitimately identify different signed wire bytes.
- Versioned packet names are available in manifests, so clients need not rely on
  ambiguous version discovery after restart.
- SQLite remains authoritative and memory remains a bounded disposable cache.
- Opaque object compatibility does not define the canonical segmented Data path.

## Success Criteria

- **SC-001**: Every packet in the acceptance test is fetched using its original
  complete name with a byte-identical wire digest before and after Repo restart.
- **SC-002**: Inventory inspection finds zero Repo-generated packet aliases for
  objects inserted through the exact-packet API.
- **SC-003**: Same-name/same-wire insertion succeeds idempotently and
  same-name/different-wire insertion is rejected deterministically.
- **SC-004**: Shared-reference deletion tests preserve a packet until its last
  manifest reference is removed.
- **SC-005**: The MiniNDN test completes manifest discovery and exact segment
  retrieval through the real NDNSF Repo service path with all checks passing.
