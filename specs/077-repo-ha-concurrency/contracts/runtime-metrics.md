# Runtime Metrics Contract

ACK capability metadata includes:

```text
ready, queueDepth, inflightReads, inflightWrites, inflightRepair,
usedBytes, reservedBytes, freeBytes,
cacheHits, cacheMisses, storageReadLatencyMs, storageWriteLatencyMs,
failureDomain, availability, RTT, bandwidth, metricsTimestampMs
```

Metrics are snapshots with a timestamp and TTL. Placement ignores stale snapshots. Capacity uses `freeBytes = capacity - usedBytes - reservedBytes`.
