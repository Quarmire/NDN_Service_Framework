# Confirmed Write Contract

Requests carry `operationId`, `generation`, optional `expectedGeneration`, `digest`, `replicationFactor`, `requiredWriteAcks`, and the object/packet payload metadata.

A successful replica response carries:

```json
{
  "status": "COMMITTED",
  "operationId": "uuid",
  "repoNode": "/example/repo/A",
  "objectName": "/publisher/object",
  "generation": 4,
  "digest": "sha256",
  "persistedBytes": 1234,
  "completedAtMs": 0
}
```

The same operation ID and tuple is idempotent. A mismatched tuple returns `OPERATION_CONFLICT`. A stale expected generation returns `GENERATION_CONFLICT`. A client manifest is committed only after validating the required number of unique Repo receipts.
