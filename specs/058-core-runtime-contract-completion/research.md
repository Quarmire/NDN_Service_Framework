# Research: Core Runtime Contract Completion

## Decision: Keep Core Envelopes Generic And Opaque

**Rationale**: Repo storage policy, UAV video/MAVLink policy, and DI model
planning policy are not reusable framework semantics. Core should expose common
facts: provider readiness, reason code, runtime queue, operation lifecycle,
data-product name, stream health, admission lease, and network telemetry.

**Alternatives considered**:

- Move Repo/UAV/DI status types into core. Rejected because it would make core
depend on app semantics.
- Leave all fields app-local. Rejected because it keeps duplicated parsing and
diagnostics.

## Decision: C++ Helper First, Not Wire-Format Rewrite

**Rationale**: The current C++ path already carries semicolon ACK fields and
json64 app payloads. Adding typed helpers around the existing format gives
immediate reviewable value without disrupting MiniNDN or existing examples.

**Alternatives considered**:

- Introduce a new binary TLV envelope. Rejected for this feature because it
would require broad compatibility work.
- Keep JSON builders in app code. Rejected because DI native provider already
had to hand-build `providerCapabilityHint` JSON.

## Decision: Discovery Facade Over Existing NDNSD Health

**Rationale**: NDNSD already provides service announcement state. A core facade
can combine NDNSD entries with provider capability hints and classify providers
as ready, draining, stale, or unavailable without knowing app policy.

**Alternatives considered**:

- A DI-specific provider registry. Rejected because Repo and UAV also need the
same discovery facts.
- A new controller-owned scheduler. Rejected because users should still be able
to plan locally and use advisory coordination only when useful.

## Decision: Preserve Stream vs Large-Data Boundary

**Rationale**: Continuous video/telemetry needs stream identity, sequence, gap,
and freshness. Files, recordings, model artifacts, and DI tensor bundles need
exact names and segmented retrieval. The core should document and enforce this
distinction through helpers and examples.

**Alternatives considered**:

- Treat all large data as stream chunks. Rejected because exact-name retrieval
and reproducible data names are central to NDN and DI planning.

