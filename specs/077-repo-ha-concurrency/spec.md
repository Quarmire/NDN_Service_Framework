# Feature Specification: High-Availability Concurrent Distributed Repo

**Feature Branch**: `Experimental`

**Created**: 2026-07-10

**Status**: Complete

**Input**: User description: "Turn NDNSF-REPO into a high-concurrency distributed repository that remains correct and available when repository nodes disappear; plan every required change and execute all tasks."

## User Scenarios & Testing

### User Story 1 - Confirmed Replicated Writes (Priority: P1)

As an application storing immutable NDN Data or a versioned object, I need a successful store result to identify the replicas that durably accepted the exact generation so that a configured replication factor is an observed fact rather than a placement intention.

**Why this priority**: A repository cannot claim fault tolerance if it acknowledges an object before the required replicas have persisted it.

**Independent Test**: Store an object with three selected replicas, make one replica reject or time out, and verify that the client either receives the configured number of valid write receipts or a structured incomplete-write result whose persisted partial replicas are repairable.

**Acceptance Scenarios**:

1. **Given** three healthy selected replicas and required write acknowledgements of three, **When** a store completes, **Then** the result contains three matching durable receipts and the committed manifest lists only those confirmed replicas.
2. **Given** one unavailable selected replica, **When** an ALL write cannot meet its acknowledgement requirement, **Then** the operation fails without publishing a falsely committed manifest and records the successful partial replicas for repair.
3. **Given** a repeated operation identifier and identical object generation, **When** the store is retried, **Then** each replica returns the existing receipt without duplicating data or changing the result.
4. **Given** a mutable object at generation N, **When** a writer supplies expected generation N, **Then** generation N+1 can commit; a stale expected generation is rejected with a conflict result.

---

### User Story 2 - Concurrent Always-On Data Serving (Priority: P1)

As a group of concurrent consumers, I need repository nodes to answer exact Data-name Interests through a long-lived data plane without creating one Face, producer thread, timer, or routing advertisement per fetch.

**Why this priority**: Per-fetch producers are the largest architectural obstacle to high read concurrency.

**Independent Test**: Store many packet-backed objects, issue concurrent exact-name and segmented-object fetches, and verify that producer/thread counts remain bounded while every returned wire packet is byte-identical.

**Acceptance Scenarios**:

1. **Given** exact signed Data packets in SQLite, **When** their prefixes are activated, **Then** one long-lived Repo data-plane producer answers exact Interests by loading the requested wire from cache or SQLite.
2. **Given** repeated FETCH_PREPARE requests, **When** the same object is fetched many times, **Then** no additional producer thread or retention timer is created.
3. **Given** an opaque object, **When** it is prepared for SegmentFetcher, **Then** its deterministic served packets are persisted once and subsequently served by the same long-lived data plane.
4. **Given** a restarted Repo, **When** persisted serving prefixes are restored, **Then** stored Data becomes available without re-uploading the object.

---

### User Story 3 - Durable Membership, Catalog, and Repair (Priority: P1)

As a repository operator, I need node incarnation, catalog changes, tombstones, peer progress, and repair work to survive process restart so that node loss and stale replicas converge automatically.

**Why this priority**: In-memory epochs and repair suppression can lose deletes, miss a failed node, or permanently leave an object under-replicated.

**Independent Test**: Replicate an object, stop a node, restart catalog processes, and verify that heartbeat-driven liveness detection creates a durable repair job, restores the required replica count, and preserves deletion tombstones.

**Acceptance Scenarios**:

1. **Given** a Repo restart, **When** it opens its database, **Then** it resumes a new boot incarnation while preserving catalog journal, tombstones, peer watermarks, and unfinished repair jobs.
2. **Given** no object delta but a replica heartbeat becomes stale, **When** the repair scan runs, **Then** an under-replication job is created without waiting for another object update.
3. **Given** a completed repair whose target later disappears, **When** under-replication recurs, **Then** a new repair attempt is allowed.
4. **Given** a delete tombstone, **When** processes restart and old AVAILABLE catalog entries arrive, **Then** the deleted generation is not resurrected.
5. **Given** a bounded journal limit, **When** peers acknowledge old sequences, **Then** obsolete deltas are compacted without losing snapshot correctness.

---

### User Story 4 - Health-Aware Placement and Fast Failover (Priority: P2)

As a client under load or partial failure, I need placement and reads to use fresh capacity, queue, network, and health information so that overloaded or dead replicas do not dominate latency.

**Why this priority**: Replication improves availability only if clients avoid stale placement decisions and fail over within a bounded budget.

**Independent Test**: Inject queue pressure and terminate the primary replica during a fetch, then verify that placement excludes saturated nodes and the client retries a healthy replica before the operation deadline.

**Acceptance Scenarios**:

1. **Given** a cached placement older than its TTL or containing a failed node, **When** another write begins, **Then** candidates are rediscovered.
2. **Given** live Repo metrics, **When** replicas are scored, **Then** free/reserved bytes, queue depth, inflight operations, storage latency, RTT, bandwidth, availability, and failure domain can influence selection.
3. **Given** a failed read replica, **When** Nack, timeout, or integrity failure occurs, **Then** its health is penalized and the remaining operation budget is used for another replica.
4. **Given** a capacity reservation, **When** concurrent clients attempt to place objects, **Then** reserved bytes prevent them from all selecting capacity that is no longer available.

---

### User Story 5 - Reproducible Availability and Throughput Evidence (Priority: P2)

As a researcher, I need deterministic unit, integration, and MiniNDN campaigns that quantify concurrency and failure behavior so claims are based on measured evidence.

**Why this priority**: Functional tests alone cannot establish high concurrency or useful node-loss tolerance.

**Independent Test**: Run the documented campaign and inspect machine-readable summaries for correctness, stable RPS, latency percentiles, failure rates, failover time, repair time, and resource bounds.

**Acceptance Scenarios**:

1. **Given** 1, 4, 16, and 32 concurrent clients, **When** 60-second read-heavy, write-heavy, and mixed workloads run, **Then** summaries report p50/p95/p99, throughput, failure rate, queue rejection, cache hit rate, and SQLite latency.
2. **Given** one Repo termination during read, write, and repair phases, **When** the campaign finishes, **Then** acknowledged data remains available according to its write policy and missing replicas are repaired.
3. **Given** a process restart, **When** catalog and serving state reload, **Then** committed objects, exact packet names, tombstones, and pending repair work remain correct.

### Edge Cases

- A write reaches some replicas but not the configured acknowledgement count.
- A client retries after losing the final response even though all replicas committed.
- Two writers race on the same mutable object generation.
- An exact Data name is submitted with different wire bytes.
- A Repo restarts with the same identity but a new boot incarnation.
- A node is slow rather than dead and later rejoins with stale catalog entries.
- A delete races with repair or a delayed write receipt.
- The hot cache evicts an item while the data plane is serving it.
- SQLite reports busy, disk-full, corruption, or an integrity-check failure.
- Repair traffic competes with foreground reads and writes.
- All replicas are unavailable or the remaining request deadline is too short.

## Requirements

### Functional Requirements

- **FR-001**: Every replicated store MUST use a globally unique operation identifier and a deterministic object generation/digest tuple.
- **FR-002**: Each replica MUST durably persist an idempotent write receipt before reporting success.
- **FR-003**: The client MUST distinguish selected, attempted, confirmed, failed, and pending-repair replicas.
- **FR-004**: A committed manifest MUST list only confirmed replicas and MUST NOT be returned until the configured write acknowledgement count is met.
- **FR-005**: The default required write acknowledgement count MUST equal the requested replication factor; callers MAY explicitly choose ONE or QUORUM.
- **FR-006**: Partial writes MUST remain discoverable as uncommitted replica state and MUST be eligible for repair or garbage collection.
- **FR-007**: Retrying the same operation identifier with the same generation and digest MUST be idempotent; reuse with different content MUST be rejected.
- **FR-008**: Mutable manifest/object updates MUST support generation and expected-generation compare-and-set semantics.
- **FR-009**: Exact signed NDN Data names MUST remain immutable and reject same-name/different-wire writes.
- **FR-010**: Catalog summaries MUST identify divergent live hashes for the same object generation as CONFLICT rather than selecting an arbitrary entry.
- **FR-011**: A Repo node MUST use a bounded number of long-lived Faces/threads for stored-data serving independent of concurrent fetch count.
- **FR-012**: Exact Data Interests MUST be answered directly from the bounded hot cache or SQLite using the original full Data name and wire.
- **FR-013**: Opaque segmented-object preparation MUST persist deterministic served packets once and reuse them across fetches and restarts.
- **FR-014**: Stored serving prefixes MUST survive restart and MUST use a stable Repo forwarding route rather than one NLSR advertisement per fetch.
- **FR-015**: SQLite MUST remain authoritative; memory remains a disposable bounded cache.
- **FR-016**: Writes MUST use a bounded writer admission queue or semaphore and reads MUST not be serialized behind unrelated object writes.
- **FR-017**: Independent object reads MUST be able to use separate SQLite read connections under WAL mode.
- **FR-018**: Cache/read consistency MUST prevent a stale backing read from replacing a newer committed cache value.
- **FR-019**: Capacity accounting MUST be maintained incrementally and MUST NOT scan all object and packet rows for every ACK.
- **FR-020**: SQLite connections MUST configure a bounded busy timeout and explicit transaction behavior.
- **FR-021**: Catalog journal entries, tombstones, node incarnation, peer progress, and repair jobs MUST be persisted in SQLite.
- **FR-022**: Catalog ordering MUST use source identity, source incarnation, and monotonic source sequence; wall-clock time MUST NOT be the sole conflict authority.
- **FR-023**: Membership heartbeat processing MUST occur even when a peer returns no object delta.
- **FR-024**: Under-replication scans MUST run periodically and after relevant writes, deletes, heartbeats, or repair results.
- **FR-025**: Repair jobs MUST have durable states, attempts, retry deadlines, leases, and completion/failure results.
- **FR-026**: Journal growth MUST be bounded through snapshots, peer watermarks, and compaction.
- **FR-027**: A lightweight bucket-digest anti-entropy operation MUST detect missing or divergent catalog ranges.
- **FR-028**: Foreground work MUST have priority over repair and repair concurrency/bandwidth MUST be bounded.
- **FR-029**: Provider ACK metadata MUST expose live queue, inflight read/write, reserved bytes, cache state, and recent storage-latency metrics.
- **FR-030**: Placement cache entries MUST have a TTL and MUST be invalidated after failure or capacity rejection.
- **FR-031**: Placement MUST support failure-domain diversity and fresh storage/network telemetry.
- **FR-032**: Capacity reservations MUST expire automatically and be consumed or released by the associated write operation.
- **FR-033**: Read failover MUST honor one total deadline, use per-replica budgets, record replica health, and preserve whole-packet-set atomicity.
- **FR-034**: Repository command status MUST expose received, running, committed, incomplete, failed, cancelled, and expired states with bounded retention.
- **FR-035**: Server-side authorization MUST validate publisher ownership for mutable object names; exact signed Data may be stored as opaque wire according to configured validation policy.
- **FR-036**: C++ and Python contracts MUST use the same field names, defaults, state values, and compatibility behavior.
- **FR-037**: Existing Spec 073-076 persistence, exact-wire, packet-consumer, and replica-failover behavior MUST remain compatible.
- **FR-038**: Automated tests MUST cover idempotency, write acknowledgement thresholds, partial writes, CAS conflict, durable restart, anti-entropy, repair recurrence, bounded producers, concurrent reads/writes, capacity reservations, and fast failover.
- **FR-039**: A 60-second MiniNDN campaign MUST compare concurrency levels and exercise node loss during read, write, and repair phases.
- **FR-040**: Campaign output MUST be machine-readable and report correctness, stable RPS, p50/p95/p99, failure rate, queue rejection, replica receipts, failover latency, repair latency, cache hit ratio, and resource bounds.

### Key Entities

- **WriteIntent**: Operation ID, object name, generation, expected generation, digest, replication factor, required acknowledgements, selected replicas, state, and timestamps.
- **WriteReceipt**: Replica identity, operation ID, generation, digest, durable state, persisted bytes, and completion time.
- **Versioned Manifest**: Object metadata, generation, parent generation, digest, confirmed replicas, consistency policy, and lifecycle state.
- **Catalog Journal Entry**: Source Repo, source incarnation, monotonic sequence, object generation, state, digest, and manifest snapshot.
- **Repo Membership Record**: Repo identity, boot incarnation, latest heartbeat, capability snapshot, and liveness state.
- **Repair Job**: Durable repair ID, object generation, source, target, reason, state, lease owner/deadline, attempts, and retry time.
- **Capacity Reservation**: Reservation ID, client/operation, bytes, expiry, and consumed/released state.
- **Repo Runtime Metrics**: Queue depth, inflight reads/writes, reserved/used/free bytes, cache counters, storage latency, and network telemetry.
- **Serving Prefix Record**: Original Data prefix, associated object, generation, activation state, and stable forwarding route.

## Success Criteria

### Measurable Outcomes

- **SC-001**: No successful ALL write reports fewer durable receipts than its replication factor in fault-injection tests.
- **SC-002**: Replaying 100 identical operation IDs changes neither stored bytes nor receipt count and returns the same committed generation.
- **SC-003**: Concurrent exact-name reads create no per-fetch producer thread or timer and return byte-identical wires in every test.
- **SC-004**: With at least one confirmed live replica, read failover completes within the configured total operation deadline and never mixes one packet set across replicas.
- **SC-005**: After restart, catalog journal, tombstones, peer watermarks, serving prefixes, and unfinished repair jobs retain their expected state.
- **SC-006**: A stale replica is detected and a replacement replica is repaired without requiring a new object delta.
- **SC-007**: Reported used plus reserved bytes never exceeds configured capacity under concurrent placement and writes.
- **SC-008**: The 60-second campaign completes at every configured concurrency and records stable RPS and latency percentiles; the implementation introduces no correctness failures at its reported stable operating point.
- **SC-009**: Existing Spec 073-076 focused tests and MiniNDN exact-packet paths remain passing.
- **SC-010**: GSD, Spec Kit, CodeGraph, and ARS verification artifacts identify every implemented requirement and any residual production risk.

## Assumptions

- Immutable, signed NDN Data is the preferred storage unit; consensus is not required for immutable packet bytes.
- Mutable aliases and manifests require generation/CAS but this feature does not implement a general distributed transaction system.
- SQLite WAL remains the authoritative per-node store; a distributed SQL database is out of scope.
- The initial anti-entropy design uses deterministic hash buckets rather than a full Merkle-tree library.
- NDNSF NAC-ABE, permission, token, and replay-protection mechanisms remain mandatory and unchanged.
- Standard repo-ng command-wire interoperability is a separate compatibility adapter; this feature aligns asynchronous status semantics but prioritizes NDNSF security and HA behavior.
- C++ owns the reusable Repo contracts and storage primitives; the current Python network runtime remains the production experiment adapter until a dedicated C++ network data plane fully replaces it.
