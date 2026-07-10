# Data Model: SQLite-Authoritative Repo Hot Cache

## RepoCacheStatus

| Field | Type | Meaning |
|---|---:|---|
| `storageBackend` | string | Active composition: `memory`, `sqlite`, or `tiered` |
| `authoritativeBackend` | string | Backend defining existence and durable state |
| `cachePolicy` | string | `lru` or `disabled` |
| `budgetBytes` | uint64 | Configured logical cache budget |
| `usedBytes` | uint64 | Sum of current entry charges |
| `entryCount` | uint64 | Number of typed entries currently admitted |
| `hits` | uint64 | Successful memory lookups |
| `misses` | uint64 | Memory lookups requiring backing access |
| `admissions` | uint64 | Entries admitted after committed writes or backing reads |
| `evictions` | uint64 | Entries removed by capacity enforcement |
| `invalidations` | uint64 | Entries removed by overwrite or delete |
| `oversizedBypasses` | uint64 | Admission attempts skipped because charge exceeds budget |
| `backingReads` | uint64 | Reads sent to the authoritative backend after a miss |
| `backingWrites` | uint64 | Successful authoritative put, manifest, or erase operations |

### Invariants

- `usedBytes <= budgetBytes` when caching is enabled.
- `entryCount == 0` and `usedBytes == 0` when `budgetBytes == 0`.
- Counters never decrease during one process lifetime.
- Cache status does not claim persistence; it resets after restart.

## HotCacheKey

| Field | Type | Meaning |
|---|---|---|
| `kind` | enum | `object` or `packets` |
| `objectName` | string | Canonical Repo object name |

Typed keys prevent an opaque object payload and a packet-set representation with the same object name from overwriting one another accidentally.

## HotCacheEntry

| Field | Type | Meaning |
|---|---|---|
| `key` | HotCacheKey | Entry identity |
| `manifest` | RepoObjectManifest | Complete metadata snapshot |
| `value` | bytes or packet list | Complete cached representation |
| `chargeBytes` | uint64 | Deterministic logical memory charge |
| `recency` | LRU position | Updated on successful get or put |

### State Transitions

```text
absent --committed write/read miss--> admitted
admitted --read/write--> most-recently-used
admitted --capacity pressure--> evicted
admitted --successful overwrite/delete--> invalidated
absent/admitted --oversized admission--> unchanged
```

## Authoritative Record

SQLite retains the existing object and packet tables. This feature changes ordering and ownership, not external object identity or manifest schema.

```text
objects(object_name, manifest_json, payload, payload_size, sha256, object_type, updated_at, ...)
data_segments(object_name, segment_no, data_name, wire, wire_size, updated_at, ...)
```

For Python packet sets, the `objects` manifest row and all replacement `data_segments` rows commit in one SQLite transaction before cache admission.
