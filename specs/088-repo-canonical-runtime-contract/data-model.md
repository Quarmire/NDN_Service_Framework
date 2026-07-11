# Data Model

- `RepoObjectManifest`: object identity, exact packet names, digest, size,
  replication and policy metadata.
- `RepoOperationStatus`: operation ID, typed state/reason and progress.
- `ReplicaReceipt`: node, generation, digest and committed state.
- `CatalogEntry`: object generation, live/tombstone/conflict state and epoch.
- `RepairJob`: idempotency key, lease, attempt/backoff and target replica.
- `RepoCacheStatus`: SQLite authority plus bounded-memory counters.

All persisted schemas carry a version. Unknown versions fail closed.
