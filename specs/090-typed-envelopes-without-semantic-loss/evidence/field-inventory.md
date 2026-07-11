# Field Disposition Inventory

| Module / field family | Kind | Typed replacement / owner | Decision |
|---|---|---|---|
| ACK `provider`, `repoNode` | legacy-alias | `ProviderCapabilityHint.providerName` | remove from current producers |
| ACK `status`, `runtimeStatus`, `negativeAckReason` | legacy-alias | typed ready/drain/reason and service payload | remove top-level duplicate |
| ACK `queue`, `queueDepth`, `activeWorkers`, wait estimates | legacy-alias | `runtimeHint` | remove top-level duplicate |
| ACK GPU/RAM/FLOPS/backend/model capacity | legacy-alias | `runtimeHint.capacityHints` | remove top-level duplicate |
| ACK RTT/bandwidth/peer metrics | legacy-alias | `runtimeHint.peerMetrics` | remove top-level duplicate |
| ACK lease id/status/expiry/binding | legacy-alias | `leaseOffers` | remove top-level duplicate |
| ACK operation lifecycle aliases | legacy-alias | `operationStatus` | remove top-level duplicate |
| Repo storage/cache/catalog/replica fields | domain-state | `ndnsf-repo-capability-v1` service payload | retain |
| DI role/model/fragment/residency fields | domain-state | `ndnsf-di-capability-v1` service payload | retain |
| UAV mission/video/FEC/authority/safety fields | domain-state | UAV schemas | retain |
| `GenericAckMetadata` | transport-metadata | Core | retain; not an alias |
| `ServiceOperationStatus` | transport-metadata | Core | retain |
| Repo SQLite rows and exact Data wire | domain-state | Repo | retain; no migration |
| DI plan/cache/artifact JSON | domain-state | DI | retain; no migration |
| UAV mission/config files | domain-state | UAV | retain; no migration |

No in-scope field remains classified as unknown. GUI profile compatibility and
Repo historical database migration fields are stored-format concerns, not ACK
aliases, and remain under their existing owners.
