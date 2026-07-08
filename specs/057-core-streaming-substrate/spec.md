# Feature Specification: Core Streaming Substrate

**Feature Branch**: `057-core-streaming-substrate`

**Created**: 2026-07-07

**Status**: Draft

**Input**: User description: "Move the reusable streaming mechanism into NDNSF so NDNSF-UAV-APP can call it instead of owning all generic stream behavior."

## User Scenarios & Testing

### User Story 1 - App-neutral stream contract (Priority: P1)

An NDNSF C++ application developer can describe a live or near-live stream with a stream id, session epoch, stream prefix, monotonically increasing sequence numbers, content type, timing metadata, and optional reliability metadata without using UAV-specific video packet types.

**Why this priority**: This creates the reusable boundary. UAV video,
telemetry, logs, and other continuous or near-live publication sequences need
the same stream/session/chunk vocabulary. Named large objects, model artifacts,
files, and DI tensor dependencies normally do not: they should use exact-name
segmented Data retrieval through the existing large-data path.

**Independent Test**: Create stream info and chunks, serialize them, deserialize them, and verify all generic fields and opaque app metadata survive round trip.

**Acceptance Scenarios**:

1. **Given** a stream info object with prefix, id, session epoch, content type, and policy fields, **When** it is converted to and from a dictionary, **Then** the result preserves the same stream identity and policy values.
2. **Given** a chunk with sequence, timing, deadline, payload, and opaque metadata, **When** it is encoded to bytes and decoded, **Then** the payload and all generic metadata are preserved.

---

### User Story 2 - Reusable producer/consumer buffering (Priority: P2)

An application can publish chunks into a bounded producer buffer and consume chunks through a reorder buffer without writing per-application duplicate, stale-session, and missing-sequence tracking.

**Why this priority**: UAV currently owns packet history and decoder reorder logic. A reusable buffer provides the common stream behavior while leaving application decoding outside the core.

**Independent Test**: Add chunks out of order, include duplicates and old-session chunks, and verify the consumer emits only current-session chunks in sequence.

**Acceptance Scenarios**:

1. **Given** chunks for a current stream session, **When** they arrive out of order, **Then** the reorder buffer emits them in sequence once gaps are filled.
2. **Given** a duplicate chunk or a chunk from an old session, **When** it is inserted, **Then** it is ignored and metrics record the duplicate or stale drop.

---

### User Story 3 - Adaptive fetch state and reliability hooks (Priority: P3)

An application can use NDNSF-provided adaptive stream state to choose fetch windows, lookahead, interest lifetime, and missing timeout from observed RTT, timeout, NACK, duplicate, and backlog pressure. Optional FEC metadata is represented generically, but the actual codec remains application-owned.

**Why this priority**: Streaming performance depends on reusable feedback loops. The core should expose generic state and policy helpers without embedding H264, YOLO, or UAV-specific bitrate rules.

**Independent Test**: Feed a policy state with stable and congested observations and verify the resulting decisions stay within configured bounds and react to pressure.

**Acceptance Scenarios**:

1. **Given** low RTT and low pressure, **When** the policy is evaluated, **Then** the window and lookahead stay near the configured steady-state values.
2. **Given** timeout or backlog pressure, **When** the policy is evaluated, **Then** the decision reduces effective concurrency and increases timeout tolerance within configured limits.

### Edge Cases

- Chunks from an old stream id or old session epoch must be dropped before application callbacks see them.
- Duplicate sequence numbers must not be emitted twice.
- Missing chunks must block ordered delivery until they arrive or the application chooses to skip them.
- FEC metadata must be optional and codec-neutral.
- Stream Data payloads may be encrypted by the application; the core must not inspect payload bytes.
- Large files or one-shot named objects are not streams. They should remain on
  NDN segmented object retrieval so consumers can fetch an exact name with
  `SegmentFetcher` semantics.

## Requirements

### Functional Requirements

- **FR-001**: NDNSF C++ core MUST provide app-neutral stream info, chunk, reliability, and metrics entities that do not mention UAV, H264, camera, distributed inference, or file-transfer semantics.
- **FR-002**: NDNSF MUST provide stable TLV encoding for C++ stream info/chunks and JSON-compatible dictionary conversion in the Python wrapper mirror.
- **FR-003**: NDNSF MUST provide Block or bytes encoding and decoding for stream chunks that keeps metadata separate from payload bytes.
- **FR-004**: NDNSF MUST provide a bounded producer buffer that stores chunks by sequence and evicts old chunks deterministically.
- **FR-005**: NDNSF MUST provide a consumer reorder buffer that rejects stale-session chunks, rejects duplicates, and emits current-session chunks in sequence.
- **FR-006**: NDNSF MUST provide generic adaptive fetch state and decision helpers for window, lookahead, interest lifetime, and missing timeout.
- **FR-007**: NDNSF MUST document how NDNSF-UAV-APP maps its existing video stream fields onto the core stream substrate.
- **FR-008**: NDNSF MUST keep application-specific capture, codec, FEC recovery algorithm, decoder queue, and bitrate-control semantics outside the core substrate.
- **FR-009**: NDNSF MUST document that exact-name large object transfer is a
  separate large-data/SegmentFetcher path, not a streaming-substrate use case.

### Key Entities

- **StreamInfo**: A started stream's identity and fetch contract, including stream id, session epoch, prefix, next sequence, content type, freshness, and policy metadata.
- **StreamChunk**: One named stream Data payload plus generic metadata such as sequence, capture time, deadline, content type, FEC info, and opaque application metadata.
- **StreamFecInfo**: Codec-neutral FEC metadata describing data shards, parity shards, symbol index/count, and data lengths.
- **StreamProducerBuffer**: In-memory helper that remembers recent encoded chunks by sequence for serving pending Interests or local tests.
- **StreamConsumerReorderBuffer**: Helper that tracks in-flight/completed/current-session chunks and emits in-order chunks.
- **StreamAdaptiveFetcherState**: Metrics and bounds used to compute fetch window, lookahead, interest lifetime, and missing timeout.

## Success Criteria

### Measurable Outcomes

- **SC-001**: Unit tests cover stream info/chunk round trip, producer buffer eviction, consumer reorder/duplicate/stale handling, and adaptive policy decisions.
- **SC-002**: The C++ library exports the stream substrate through installed headers, and the public Python package exports mirror entities from `ndnsf`.
- **SC-003**: Documentation clearly states that NDNSF core owns generic streaming substrate while UAV owns H264/camera/video-specific behavior.
- **SC-004**: Existing core coordination and Python wrapper tests continue to pass.

## Assumptions

- The primary reusable API lands in the C++ core because NDNSF-UAV-APP is C++ and should eventually call this substrate directly.
- The Python wrapper mirror is retained for orchestration, docs, and cross-application tests, but it is not the authoritative runtime path.
- C++ UAV streaming is not migrated in this first step; it is documented as a compatibility mapping and future migration target.
- NDNSF core will not standardize a video codec, ROI model, or FEC codec in this feature.
