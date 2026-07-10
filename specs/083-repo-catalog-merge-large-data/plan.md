# Implementation Plan: Repo Catalog Merge Large-Data Path

## Design

Small deltas remain inline. For a large delta, the sidecar serializes:

```json
{"schemaVersion":1,"entries":[],"sourceStatus":{}}
```

It publishes those bytes with `SegmentedObjectProducer` under its Repo identity
and sends one protected `CATALOG_MERGE_PULL` containing the exact versioned
name, digest, byte count, and entry count. The target uses SegmentFetcher,
validates every declaration, then calls the same merge implementation.

If pull fails, the sidecar logs the reason and uses the existing bounded inline
batches. This supports mixed deployments without changing catalog meaning.

## Safety

- Maximum transfer size is 16 MiB.
- Hash and byte count are checked before JSON parsing.
- Schema and entry count are checked before merge.
- Producer cleanup runs in `finally`.
- The authenticated request binds the expected hash to the exact Data name.

## Experiment

- Baseline: accepted Spec 082 workers=3, seed 78004.
- Treatment: identical 60-second RF=3/W=QUORUM campaign.
- Primary: initial merge mode/batches/duration and 4/4 outage repair.
- Secondary: first repair, request p50/p95, merge total, receipt floor, invalid
  repairs.
- Execute once and retain negative results.
