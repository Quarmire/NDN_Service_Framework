# Data Model

## ExactDataPacket

| Field | Meaning | Invariant |
|---|---|---|
| `dataName` | Complete URI decoded from wire | Primary key; includes version/segment when present |
| `wire` | Complete encoded Data packet | Returned byte-for-byte |
| `wireSha256` | Integrity/diagnostic digest | Must match `wire` |
| `wireSize` | Stored byte count | Must equal wire length |
| `updatedAt` | First/last idempotent insertion time | Never licenses mutation |
| `hitCount` | Persistent access counter | Diagnostic only |

## ObjectPacketReference

| Field | Meaning | Invariant |
|---|---|---|
| `objectName` | Logical manifest key | References an existing manifest |
| `ordinal` | Reassembly order | Unique within object |
| `segmentNo` | Parsed segment number | Matches Data name when segmented |
| `dataName` | Referenced exact packet | References `ExactDataPacket` |

## State Transitions

```text
wire received
 -> decode Data
 -> validate declared name and set shape
 -> begin SQLite transaction
 -> insert packet or verify identical existing wire
 -> replace manifest references
 -> reclaim packets with zero references
 -> commit
 -> update/invalidate hot cache
 -> publish catalog availability
```

Failure before commit leaves authority, cache, and catalog unchanged.
