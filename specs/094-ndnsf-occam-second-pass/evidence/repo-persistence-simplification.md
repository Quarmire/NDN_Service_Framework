# Repo Persistence Simplification

Removed:

- public `InMemoryRepoStore` and `makeMemoryRepoStore`;
- default memory-backed `RepoCore` and `RepoNode` constructors;
- ignored `producer_retention_s` constructor/CLI/harness option;
- ignored `isolated_runtime` request-helper option and all call arguments;
- nested `legacyStatus` copies and misleading `legacy_fields` local name.

Canonical behavior:

- SQLite is authoritative;
- one byte-bounded LRU memory tier remains optional acceleration;
- examples and tests explicitly create temporary tiered SQLite stores;
- exact Data packet, catalog, HA, replication, and repair contracts are
  unchanged.

Verification:

```text
waf targets: ndnsf-distributed-repo, Smoke, TieredCache, ExactPacket, HA built
DistributedRepoSmoke: passed
DistributedRepoTieredCacheTest: passed, authority=sqlite, cache=tiered
DistributedRepoExactPacketTest: passed
DistributedRepoHaTest: passed
Repo Python tests: 89 passed
Core-envelope migration tests: 7 passed
ACK compatibility tests: 10 passed
active removed-symbol scan: zero production references
```
