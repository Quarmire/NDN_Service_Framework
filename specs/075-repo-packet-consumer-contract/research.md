# Research: Repo Packet Consumer Contract

## Decision 1: Do not migrate all large objects to packet storage

**Decision**: Keep DI model/runtime files and UAV encrypted recording chunks on
the opaque-object API.

**Rationale**: These callers own bytes, not pre-signed NDN Data. Repo may use
segmented Data internally for transport, but those packet names are not part of
the application contract.

**Alternatives considered**: Expose internal transport packets as application
objects. Rejected because it leaks Repo placement names and signatures.

## Decision 2: Manifest packet names are the read authority

**Decision**: Retrieve a packet-backed object by iterating its ordered,
validated `packetNames` list.

**Rationale**: Segment number and prefix are insufficient to distinguish
versions or non-standard application naming. Exact names preserve producer
intent and wire identity.

**Alternatives considered**: Derive `prefix/seg=N`. Rejected because it can
rename packets and select the wrong version.

## Decision 3: Fail atomically

**Decision**: Return no packet set when any indexed packet is missing,
duplicated, or name-mismatched.

**Rationale**: A partial ordered set cannot satisfy the manifest contract and
would force every application to reproduce integrity checks.

**Alternatives considered**: Return partial results plus status. Deferred; a
streaming/repair API has different semantics and is outside this feature.
