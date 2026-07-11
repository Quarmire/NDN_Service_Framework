# Feature Specification: Repo Canonical Runtime And Contract

**Parent**: `specs/084-ndnsf-occam-simplification/`
**Status**: In progress

## User Stories

### User Story 1 - Exact persistent objects (Priority: P1)

A client inserts app-signed Data packets under their exact names and retrieves
the identical wire bytes after process restart.

### User Story 2 - Available replicated storage (Priority: P1)

A client writes with ONE, QUORUM, or ALL consistency and can read/repair after
a replica fails without accepting conflicting finalized content.

### User Story 3 - One maintainable Repo contract (Priority: P2)

Repo maintainers evolve one C++ object/protocol contract. Python adapts that
contract for NDNSF networking and operations; DI/UAV do not own Repo policy.

## Functional Requirements

- **FR-001** C++ `RepoCore`, `RepoNode`, `RepoClient`, and `RepoProtocol` MUST be
  the canonical object and protocol contract.
- **FR-002** Deployed nodes MUST use SQLite as authority; a bounded memory LRU
  MAY accelerate reads but MUST never become authority.
- **FR-003** Exact app-signed Data names and wire bytes MUST be immutable.
- **FR-004** The public object API MUST remain small: insert/store, manifest,
  fetch, inventory/status, and delete.
- **FR-005** Replication, reservation, receipt, repair, catalog, tombstone, and
  anti-entropy operations MUST use a versioned private protocol and reject
  ordinary unauthorized clients.
- **FR-006** Raw payload convenience helpers MUST segment/sign client-side and
  call exact-Data insertion rather than introduce another storage model.
- **FR-007** Python Repo orchestration MUST live in
  `NDNSF-DistributedRepo/pythonWrapper/py_repoclient`, not the DI package.
- **FR-008** DI and UAV MAY depend on the public Repo client but MUST NOT own
  storage/catalog/repair policy.
- **FR-009** Restart, migration, rollback-open behavior, concurrency,
  backpressure, metrics, and malformed input MUST be tested.

## Success Criteria

- **SC-001** Exact packet and restart tests preserve names and wire bytes.
- **SC-002** Cache tests prove cold SQLite read, hit, eviction, fallback, and
  SQLite availability for oversized entries.
- **SC-003** At least three matched 60-second RF=2/W=ALL campaigns retain 30/30,
  zero invalid finalized replicas, and required repair coverage.
- **SC-004** Public DI default imports no Repo node/server implementation.
- **SC-005** One authoritative source implements each object/protocol rule.
