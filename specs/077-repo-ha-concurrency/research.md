# Research Decisions

## NDN Repo Baseline

The standard Repo model directly answers ordinary Interests for stored Data and uses asynchronous command/status semantics for insertion and deletion. NDNSF-REPO retains its authenticated NDNSF control plane but adopts a long-lived Interest-serving data plane and bounded operation-status retention.

## Distributed Storage Baseline

Ceph demonstrates that a successful replicated write must reflect secondary confirmations and that membership, recovery, and scrubbing are distinct mechanisms. Cassandra demonstrates that temporary retry/hints do not replace anti-entropy. NDNSF-REPO adopts these principles without importing distributed consensus for immutable Data.

## Consistency Decision

- Exact signed Data: immutable, content/wire identity enforced, no update consensus.
- Versioned objects/manifests: generation and expected-generation CAS.
- Default write acknowledgement: all requested replicas.
- Optional ONE/QUORUM: explicit caller policy, with incomplete replicas queued for repair.
- Catalog: eventual convergence with durable per-source sequence and anti-entropy.

## Data Plane Decision

One callback-backed native producer is preferred over static packet producers because the complete repository cannot be copied into process memory and producer count must not scale with objects or fetches. Local Interest filters may be registered per stored prefix on one Face; routing advertises only the stable Repo forwarding route.

## Storage Concurrency Decision

SQLite WAL supports concurrent readers with one writer. The implementation therefore uses one guarded writer connection and thread-local readers, while per-object striped locks protect cache/DB coherence. This is lower risk than a general actor rewrite and removes the global read serialization found in the current implementation.

## Repair Decision

Repair remains orchestration above immutable object transfer, but its intent/state must be durable. A local SQLite repair-job queue plus peer catalog exchange is sufficient for the initial 3-5 node deployment. Full consensus membership and distributed task ownership are out of scope.

## Interoperability Decision

Full repo-ng command-wire compatibility is not required for HA correctness and would introduce a second security entry point. Spec 077 aligns command states and direct Data retrieval; a separately reviewed adapter may later translate standard signed Repo Commands into these operations.
