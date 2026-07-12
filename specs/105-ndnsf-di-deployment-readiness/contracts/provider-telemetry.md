# Provider Capability and Telemetry Contract

## Capability

`ndnsf-di-provider-capability-v3` remains inside `GenericAckMetadata` and includes
configured support only. Each value carries a `source` when ambiguity is possible.

## Dynamic Snapshot

```json
{
  "schema": "ndnsf-di-provider-telemetry-v1",
  "providerName": "/provider/A",
  "providerBootId": "uuid",
  "sequence": 42,
  "measuredAtMs": 0,
  "probe": {"source": "nvidia", "status": "measured", "durationMs": 3},
  "gpu": {"deviceId": "GPU-uuid", "totalMb": 8192, "freeMb": 6012, "usedMb": 2180, "utilizationPct": 41},
  "runtime": {"resident": true, "evidenceEpoch": 1, "artifactDigests": {}},
  "scheduler": {"readyQueue": 0, "waitingDependencies": 1, "activeWorkers": 1, "idleWorkers": 0},
  "serviceRate": {"stageRpsEwma": 1.2, "latencyMsEwma": 35, "samples": 120},
  "cache": {"budgetMb": 512, "usedMb": 128, "sessions": 1, "hits": 20, "misses": 2, "evictions": 0}
}
```

Production rules:

- maximum planner age: 2,000 ms;
- failed/unsupported probes remain explicit and exclude GPU placement;
- profile values may fill capability but never measured fields;
- telemetry is advisory; provider admission and leases remain authoritative;
- snapshots are signed/protected through existing NDNSF message security.
