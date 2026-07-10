# Feature Specification: SQLite-Authoritative Repo Hot Cache

**Feature Branch**: `Experimental`

**Created**: 2026-07-09

**Status**: Complete

**Input**: User description: "Use SQLite as the authoritative store and a capacity-limited memory tier for newly written and frequently read repository content; design, implement, and validate the complete feature with MiniNDN."

## User Scenarios & Testing

### User Story 1 - Durable Repository Authority (Priority: P1)

As an application storing a model, media object, or segmented NDN Data in a persistent NDNSF repository, I need a successful store response to mean that SQLite contains the authoritative object so that a process restart never loses an acknowledged object.

**Why this priority**: Durability and a single source of truth are required before a memory optimization can be trusted.

**Independent Test**: Store objects through the repository API, restart the repository with an empty memory tier, and fetch the exact bytes and manifests from the same database.

**Acceptance Scenarios**:

1. **Given** a persistent repository with an empty database, **When** an object store completes successfully, **Then** the object and manifest are committed to SQLite before they are visible through the memory tier.
2. **Given** acknowledged objects in SQLite, **When** the repository process restarts, **Then** all objects remain discoverable and fetchable without preloading them into memory.
3. **Given** an authoritative write failure, **When** a store is attempted, **Then** the operation fails and no stale or uncommitted cache entry becomes readable.
4. **Given** an existing object, **When** it is deleted or overwritten, **Then** SQLite and the memory tier expose the same new state.

---

### User Story 2 - Bounded Hot-Object Acceleration (Priority: P1)

As a repository operator serving repeatedly requested objects, I need recent and frequently reused objects to be read from a bounded memory tier while cold objects remain only in SQLite, so reads improve without unbounded RAM growth.

**Why this priority**: This is the requested performance behavior and must remain safe under mixed object and packet workloads.

**Independent Test**: Configure a small memory budget, store and fetch objects in a controlled order, and verify admission, cache hits, least-recently-used eviction, cold SQLite fallback, and the byte limit.

**Acceptance Scenarios**:

1. **Given** a cacheable newly committed object, **When** it is stored, **Then** it is admitted as the most recently used entry.
2. **Given** a cold object in SQLite, **When** it is fetched, **Then** the first read comes from SQLite and admits it, while the next read is a memory hit.
3. **Given** a full memory tier, **When** another object is admitted, **Then** least-recently-used entries are evicted until the configured logical byte budget is respected.
4. **Given** an object larger than the entire memory budget, **When** it is stored or fetched, **Then** it remains available from SQLite but bypasses memory admission.
5. **Given** ordinary objects and segmented Data packets, **When** both are cached, **Then** they share one total memory budget rather than independent limits.

---

### User Story 3 - Configurable and Observable Operation (Priority: P2)

As a repository operator or experiment author, I need to configure the persistent path and memory budget and retrieve cache counters through the same repository service path, so I can verify behavior and tune deployments without inspecting process internals.

**Why this priority**: A cache that cannot be measured cannot be safely evaluated or operated.

**Independent Test**: Start repositories with different backend and budget settings, request cache status locally and over NDNSF, and compare returned counters with a known access sequence.

**Acceptance Scenarios**:

1. **Given** a tiered repository configuration, **When** the node starts, **Then** its summary reports SQLite authority, the database path, the memory budget, and the LRU policy.
2. **Given** known misses, hits, admissions, evictions, invalidations, and oversized objects, **When** cache status is requested, **Then** the corresponding monotonic counters and current usage are returned.
3. **Given** a zero-byte memory budget, **When** the node starts, **Then** SQLite remains authoritative and cache status reports a disabled memory tier.

---

### User Story 4 - Network-Level Validation (Priority: P2)

As an NDNSF experiment author, I need a MiniNDN scenario that uses the real repository service path to prove persistence and hot-cache behavior across nodes, so the result is stronger than an in-process unit test.

**Why this priority**: The final acceptance requirement explicitly calls for MiniNDN verification.

**Independent Test**: Run the dedicated MiniNDN tiered-cache scenario and inspect its machine-readable summary and node logs.

**Acceptance Scenarios**:

1. **Given** a client and persistent repo on different MiniNDN nodes, **When** the client performs a cold fetch followed by a repeated fetch, **Then** cache status shows at least one miss followed by at least one hit and the returned bytes are identical.
2. **Given** a small configured budget and multiple objects, **When** the client drives an eviction sequence, **Then** cache usage remains within budget and the evicted object is still retrievable from SQLite.
3. **Given** a stored object and a restarted repo process using the same database, **When** the client fetches it again, **Then** the fetch succeeds after a cold backing-store read and subsequent repeat fetch hits memory.

### Edge Cases

- A zero-byte cache budget disables admission while retaining SQLite persistence.
- Empty payloads and manifest-only logical objects remain valid and do not corrupt cache accounting.
- An entry whose logical charge equals the budget is admitted; an entry larger than the budget bypasses admission.
- Updating an existing entry adjusts usage without double-counting the old version.
- A failed SQLite commit, full database, malformed database, or unwritable path never leaves a readable cache-only object.
- Concurrent fetches and writes do not corrupt LRU order, counters, or SQLite state.
- Deleting a non-existent object remains idempotent and does not underflow cache accounting.
- Cache counters may reset on process restart, but authoritative objects and manifests must not.

## Requirements

### Functional Requirements

- **FR-001**: A persistent tiered repository MUST use SQLite as the sole authoritative source for object existence, manifests, payloads, and segmented packet data.
- **FR-002**: A successful persistent write MUST commit to SQLite before the corresponding memory entry is admitted or replaced.
- **FR-003**: Failed authoritative writes MUST NOT create or update a readable memory entry or publish a successful catalog change.
- **FR-004**: Persistent reads MUST use read-through behavior: consult memory first, load from SQLite on a miss, validate data as currently required, and admit eligible content after a successful load.
- **FR-005**: Newly committed eligible content MUST be admitted to memory so immediate reuse does not require another SQLite read.
- **FR-006**: The memory tier MUST enforce one configurable logical-byte budget across every cached representation owned by a repository node.
- **FR-007**: The memory tier MUST use least-recently-used replacement; successful reads and writes MUST refresh recency.
- **FR-008**: Content whose logical charge exceeds the full budget MUST bypass memory without affecting SQLite availability.
- **FR-009**: Overwrite and delete operations MUST invalidate or replace affected memory entries only after the authoritative operation succeeds.
- **FR-010**: The repository MUST expose current budget, current logical usage, entry count, backend/authority, policy, and monotonic hit, miss, admission, eviction, invalidation, oversized-bypass, backing-read, and backing-write counters.
- **FR-011**: Cache status MUST be available through in-process C++ APIs, embedded service registration, remote NDNSF service registration, and the Python repository operation path.
- **FR-012**: Repository CLI/configuration MUST require SQLite-backed operation and support a SQLite path and memory-cache byte budget; it MUST NOT silently fall back to memory-only authority.
- **FR-013**: Persistent repository sample configuration MUST select tiered SQLite-plus-memory operation by default and document how to disable or resize the memory tier.
- **FR-014**: C++ and Python persistent repository paths MUST implement the same authority ordering, shared-budget, LRU, oversized-bypass, deletion, and status semantics.
- **FR-015**: Existing store, fetch, manifest, inventory, capability, catalog, segmented Data, and delete wire contracts MUST remain compatible.
- **FR-016**: Automated tests MUST cover persistence across restart, hit/miss transitions, LRU eviction, shared budget enforcement, oversized bypass, overwrite/delete consistency, failed backing write isolation, and zero-budget SQLite operation.
- **FR-017**: A MiniNDN test MUST exercise the real NDNSF repo request/response path and verify miss-to-hit, eviction with SQLite fallback, and restart persistence from observable status and returned data.
- **FR-018**: Validation output MUST include a machine-readable summary containing cache budget/usage, counters, object integrity checks, restart result, and test pass/fail state.

### Key Entities

- **Authoritative Repository Record**: Durable object name, manifest, payload or segmented Data packets, digest, type, size, and update metadata stored in SQLite.
- **Hot Cache Entry**: One object or segmented-packet representation, its deterministic logical byte charge, and its least-recently-used position.
- **Cache Status**: Current backend and authority, policy, budget, usage, entry count, and cumulative operation counters.
- **Tiered Repository Configuration**: Storage mode, SQLite path, memory budget, service identity, and existing repository capability settings.

## Success Criteria

### Measurable Outcomes

- **SC-001**: Every acknowledged object in the persistence test remains byte-identical and discoverable after repository restart.
- **SC-002**: In all eviction and mixed-representation tests, reported logical memory usage never exceeds the configured budget.
- **SC-003**: A controlled cold-read/repeat-read sequence reports a miss on the first read and a hit on the second without changing returned bytes.
- **SC-004**: An authoritative write failure produces zero new cache admissions and the attempted object remains unavailable.
- **SC-005**: Cache-enabled and zero-budget nodes both retain SQLite authority and pass the same object API tests.
- **SC-006**: The dedicated MiniNDN scenario completes successfully and proves cold read, hot hit, eviction fallback, and restart persistence through the network service path.
- **SC-007**: The full C++ DistributedRepo smoke test and focused Python repository tests pass with no regression in existing APIs.

## Assumptions

- SQLite WAL mode and the repository's current durability setting remain appropriate for this feature; changing fsync policy is outside scope.
- The configured memory limit is a deterministic logical cache charge rather than an exact measurement of allocator/container overhead.
- LRU is sufficient for the first production policy; frequency-aware admission and multi-segment partial caching are future optimizations.
- Cache contents and counters are process-local and disposable; only SQLite records survive restart.
- The in-memory backend may remain as an internal unit-test double, but it is not a supported Repo node deployment mode.
- Existing NDNSF authorization, permission, request/response, and large-data mechanisms are reused unchanged.
