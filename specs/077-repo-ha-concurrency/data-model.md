# Data Model

## WriteIntent

Required fields: `operationId`, `objectName`, `generation`, `digest`, `replicationFactor`, `requiredAcks`, `selectedReplicas`, `state`, `createdAtMs`, `updatedAtMs`.

Optional fields: `expectedGeneration`, `parentGeneration`, `reservationIds`, `error`.

State transitions:

```text
RECEIVED -> RUNNING -> COMMITTED
                   -> INCOMPLETE -> RUNNING/COMMITTED/FAILED/EXPIRED
                   -> FAILED/CANCELLED/EXPIRED
```

## WriteReceipt

Identity is `(operationId, repoNode)`. A receipt binds `objectName`, `generation`, `digest`, `persistedBytes`, `state`, and `completedAtMs`. The same identity cannot bind different content.

## VersionedManifest

Adds `generation`, `parentGeneration`, `writeConsistency`, `requiredWriteAcks`, `confirmedReplicaNodes`, `operationId`, and `lifecycleState` to the existing manifest. Missing fields decode to legacy defaults.

## CatalogJournalEntry

Identity is `(sourceRepo, sourceBootId, sourceSequence)`. Ordering is source-local and monotonic. Entry payload includes object generation, digest, AVAILABLE/DELETED/PARTIAL/CONFLICT state, and a manifest snapshot.

## RepairJob

Identity is a deterministic digest of object generation, target Repo, and repair epoch. State is `PENDING`, `LEASED`, `RUNNING`, `SUCCEEDED`, `FAILED`, or `EXPIRED`. Lease expiry permits another worker to resume abandoned work.

## CapacityReservation

Identity is random and bound to one operation. `ACTIVE` reservations reduce advertised free bytes; `CONSUMED`, `RELEASED`, and `EXPIRED` reservations do not. Expiration is mandatory.

## RuntimeMetrics

Snapshot fields: `queueDepth`, `inflightReads`, `inflightWrites`, `inflightRepair`, `usedBytes`, `reservedBytes`, `freeBytes`, cache counters, `storageReadLatencyMs`, `storageWriteLatencyMs`, and network telemetry.

## ServingPrefix

Persistent association between original Data prefix and object generation. The data plane registers active prefixes locally and loads packet wire by exact Interest name.
