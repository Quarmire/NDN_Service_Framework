# Feature Specification: Repo Packet Consumer Contract

**Feature Branch**: existing worktree
**Created**: 2026-07-10
**Status**: In progress

## User Scenarios & Testing

### User Story 1 - Retrieve original signed packets (Priority: P1)

An application stores an ordered set of signed NDN Data packets and later asks
the Repo for that packet-backed object. The application receives the same
packets, in manifest order, with complete names and wire bytes unchanged.

**Independent Test**: Store `/data/.../v=.../seg=N`, retrieve the packet set
through one high-level operation, and compare every name, wire digest, and
position with the original set.

**Acceptance Scenarios**:

1. **Given** a packet-backed manifest with ordered packet names, **When** the
   application retrieves the packet set, **Then** each exact name is fetched in
   manifest order and the returned wire name equals the requested name.
2. **Given** a missing or altered packet, **When** the packet set is retrieved,
   **Then** retrieval fails explicitly instead of returning a partial object.

### User Story 2 - Keep opaque objects separate (Priority: P1)

An application stores model files, encrypted recording chunks, JSON, or other
arbitrary bytes without treating those bytes as pre-signed NDN packets.

**Independent Test**: Store and fetch an opaque object through the existing
object API and verify that the payload is unchanged and no packet-only API is
required.

**Acceptance Scenarios**:

1. **Given** arbitrary bytes, **When** the object API is used, **Then** existing
   object storage and reassembly behavior remains unchanged.
2. **Given** a packet-backed manifest, **When** its ordinary payload view is
   requested, **Then** content may be reassembled from the original packet
   names without changing the stored names or wires.

### User Story 3 - Use one contract across applications (Priority: P2)

Repo examples and future UAV or DI packet-producing paths use a shared packet
consumer operation rather than open-coding loops over individual packet names.

**Independent Test**: The exact-packet example and MiniNDN acceptance scenario
use the shared operation and still pass restart and wire-identity checks.

## Edge Cases

- A packet-backed manifest has no packet names.
- A manifest repeats the same packet name.
- One exact Data name is unavailable while earlier packets are available.
- A Repo returns a packet whose decoded complete name differs from the request.
- An opaque object has multiple internally generated transport segments but is
  not an application-produced packet set.
- A packet-backed object is stored under an application namespace such as
  `/data`, outside the publisher identity prefix.

## Requirements

### Functional Requirements

- **FR-001**: The system MUST expose a high-level operation that retrieves all
  packets named by a packet-backed manifest.
- **FR-002**: The operation MUST preserve manifest order and MUST request every
  packet by its exact complete Data name.
- **FR-003**: The operation MUST reject empty, duplicate, missing, or mismatched
  packet indexes and MUST NOT return a partial packet set.
- **FR-004**: The operation MUST preserve each packet's original wire bytes.
- **FR-005**: Opaque-object APIs MUST continue to support arbitrary bytes,
  including model files, encrypted UAV recording chunks, and JSON documents.
- **FR-006**: A packet-backed manifest MAY expose a reassembled payload view,
  but that view MUST consume the original packet set and MUST NOT create renamed
  or re-signed stored packets.
- **FR-007**: Documentation MUST define the decision rule for choosing packet
  storage versus object storage and classify current Repo, UAV, and DI callers.
- **FR-008**: Existing exact-packet restart, persistence, cache, and MiniNDN
  behavior MUST remain valid.

### Key Entities

- **Packet-backed manifest**: Logical object metadata whose ordered
  `packetNames` identify original signed NDN Data packets.
- **Opaque object manifest**: Logical object metadata without `packetNames`;
  transport segmentation is an implementation detail.
- **Exact packet set**: Complete, ordered packet result returned only after all
  packet names and wires have been validated.

## Success Criteria

- **SC-001**: A multi-packet object can be retrieved with one public API call,
  returning 100% of packet names in manifest order.
- **SC-002**: Tests detect every missing, duplicate, and wrong-name packet case.
- **SC-003**: Exact packet wires remain byte-identical before and after Repo
  restart in MiniNDN.
- **SC-004**: Existing DI model/runtime object and UAV encrypted recording paths
  continue to pass without conversion to packet storage.

## Assumptions

- The producer remains responsible for Data segmentation, signing, naming, and
  payload encryption before calling the packet storage API.
- `packetNames` is authoritative for packet-backed objects; `segmentCount`
  alone does not define packet identity.
- Current UAV recording chunks and DI artifacts are arbitrary bytes, not
  pre-encoded NDN Data packets.

## Out of Scope

- Replacing ordinary object storage with packet storage.
- Moving UAV H264/FEC/ROI policy or DI artifact semantics into Repo core.
- Changing Data names, signatures, encryption, or packet wire encodings.
