# CACHE_STATUS Contract

## C++ Service Name

```text
<repo-service-prefix>/CACHE_STATUS
```

The request payload is empty. The normal NDNSF Request/ACK/Selection/Response security flow is unchanged.

## Python Shared-Service Operation

```json
{"operation":"CACHE_STATUS"}
```

## Successful Response

```json
{
  "storageBackend": "tiered",
  "authoritativeBackend": "sqlite",
  "cachePolicy": "lru",
  "budgetBytes": 67108864,
  "usedBytes": 8192,
  "entryCount": 2,
  "hits": 7,
  "misses": 3,
  "admissions": 5,
  "evictions": 2,
  "invalidations": 1,
  "oversizedBypasses": 0,
  "backingReads": 3,
  "backingWrites": 4
}
```

All numeric fields are non-negative integers. Unknown additive fields must be ignored by clients. Existing Repo operations and payloads are unchanged.

## Mode Semantics

| Mode | `storageBackend` | `authoritativeBackend` | `cachePolicy` |
|---|---|---|---|
| SQLite with zero-byte hot cache | `sqlite` | `sqlite` | `disabled` |
| SQLite plus bounded RAM | `tiered` | `sqlite` | `lru` |

`memory` is not a supported deployed Repo-node mode. Internal tests may use an in-memory backing test double without exposing it through node configuration.

## Failure Semantics

- An unavailable or malformed backend returns the existing unsuccessful NDNSF `ResponseMessage` form.
- Counter snapshots are observational and need not be mutually atomic with a client operation completing on another thread.
- A missing `CACHE_STATUS` service on an older node is handled as a normal unsupported/timeout condition by clients.
