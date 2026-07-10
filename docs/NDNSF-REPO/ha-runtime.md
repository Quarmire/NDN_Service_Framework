# NDNSF-REPO HA Runtime

This document is the implementation entry point for Spec 077.

## Boundaries

- C++ owns manifest, write intent/receipt, operation status, exact Data wire,
  and local store contracts.
- Python owns NDNSF request orchestration, persistent catalog membership,
  anti-entropy, repair scheduling, telemetry-aware placement, and experiments.
- Repo stores named immutable Data or named opaque objects. SegmentFetcher is
  used for finite large objects; stream publication remains an NDNSF core/UAV
  concern.

## Write path

1. Discover candidates and evaluate live capacity/runtime/network telemetry.
2. Reserve bytes on the selected failure-domain-diverse replicas.
3. Submit the same write intent and operation ID to every selected replica.
4. Each replica commits content and its receipt in one SQLite transaction.
5. Return a committed manifest containing only validated receipt owners.
6. Report structured incomplete evidence when the configured W is not met.

## Read path

One total deadline is divided among health-ordered replicas. A failed exact
packet attempt discards the whole packet set before trying another replica.
Opaque objects may explicitly enable delayed hedged reads; the first complete,
size- and digest-valid object wins.

## Catalog and repair

Every catalog entry contains source repo, boot incarnation, monotonic sequence,
generation, state, and digest. Empty deltas still carry heartbeats. Bucket
digests detect divergent buckets before full entry exchange. Under-replication
creates durable repair jobs that are claimed with leases, retried with backoff,
and recreated when a later topology change creates a new repair epoch.
Empty deltas always merge their source status, so membership heartbeat
progress does not depend on object changes. Repair jobs execute under the
target Repo identity and may cross the original publisher namespace only when
the request matches a currently RUNNING durable repair job; ordinary
cross-publisher stores remain rejected.

## Operations

The Python network API exposes `CAPABILITY`, `CACHE_STATUS`,
`RESERVE_CAPACITY`, `RELEASE_CAPACITY`, `CATALOG_BUCKET_DIGEST`,
`CATALOG_BUCKET_ENTRIES`, `REPAIR_SCAN`, `REPAIR_CLAIM`, `REPAIR_COMPLETE`,
`REPAIR_FAIL`, and `SCRUB`, in addition to existing store/fetch/catalog calls.

## Concurrency boundary

One `NetworkDistributedRepoClient` owns one control dispatcher thread. All
NDNSF Request/ACK/Selection operations enter the native `ServiceUser` from
that stable thread; exact-name and segmented Data fetches remain concurrent.
Creating multiple `ServiceUser` sessions with the same identity is not a
supported scaling mechanism.

The 2026-07-10 MiniNDN campaign shows that this design is correct but not yet
production-throughput ready. At 2 offered RPS and concurrency 16, failure rate
was 0.83% but p95 reached 28.1 seconds. At 4 offered RPS, failure rose to
15.83%. A fault-triggered repair run kept 12/12 reads available and repaired
RepoC from RepoB 43.95 seconds after RepoA failed. Canonical evidence and exact
commands are recorded in `specs/077-repo-ha-concurrency/`.
