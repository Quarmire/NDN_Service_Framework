# Requirements Checklist

- [x] User scenarios cover writes, reads, restart, node loss, repair, and load.
- [x] Replication success is defined by durable receipts.
- [x] Immutable Data and mutable metadata use separate consistency rules.
- [x] Data-plane producer/thread bounds are testable.
- [x] Catalog ordering and tombstone persistence do not rely only on wall time.
- [x] Repair is durable, retryable, and foreground-limited.
- [x] Concurrency and capacity oversubscription have explicit requirements.
- [x] Existing security and exact-wire behavior remain mandatory.
- [x] MiniNDN campaign duration, variables, controls, and metrics are specified.
- [x] Full repo-ng wire compatibility is explicitly separated from this feature.
