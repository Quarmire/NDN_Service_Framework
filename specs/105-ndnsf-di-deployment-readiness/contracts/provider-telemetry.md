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
  "probe": {"source": "linux-proc", "status": "measured", "durationMs": 1},
  "host": {"deviceId": "cpu0", "totalMemoryMb": 32768, "freeMemoryMb": 12000, "processRssMb": 2048},
  "runtime": {"resident": true, "evidenceEpoch": 1, "artifactDigests": {}},
  "scheduler": {"readyQueue": 0, "waitingDependencies": 1, "activeWorkers": 1, "idleWorkers": 0},
  "serviceRate": {"stageRpsEwma": 1.2, "latencyMsEwma": 35, "samples": 120},
  "cache": {"budgetMb": 512, "usedMb": 128, "sessions": 1, "hits": 20, "misses": 2, "evictions": 0}
}
```

Production rules:

- maximum planner age: 2,000 ms;
- failed/unsupported probes remain explicit and exclude local placement;
- profile values may fill capability but never measured fields;
- telemetry is advisory; provider admission and leases remain authoritative;
- snapshots are signed/protected through existing NDNSF message security.

Physical GPU fields and NVIDIA probe sources are additive Spec 106 profile data;
they are not synthesized by Spec 105.
