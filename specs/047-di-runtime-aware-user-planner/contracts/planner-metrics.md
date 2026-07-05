# Contract: Planner Metrics Output

## Purpose

Provide evidence for debugging and MiniNDN campaigns.

## Required Fields

```json
{
  "schema": "ndnsf-di-planner-metrics-v1",
  "requestId": "<request-id>",
  "plannerMode": "runtime-aware-user-side",
  "selectedAssignments": [],
  "scoreBreakdown": {},
  "leaseCounters": {
    "granted": 0,
    "rejected": 0,
    "expired": 0,
    "consumed": 0
  },
  "residencyCounters": {
    "gpuLoadedHit": 0,
    "cpuResidentHit": 0,
    "diskResidentHit": 0,
    "repoAvailable": 0,
    "missing": 0
  },
  "edgeCostSummary": {
    "dependencyEdges": 0,
    "estimatedTransferMs": 0,
    "unknownEdges": 0,
    "staleMetricEdges": 0
  },
  "queueWaitMs": 0,
  "replanCount": 0,
  "latencyMs": 0,
  "success": true,
  "failureReason": ""
}
```

## Aggregation Requirements

Campaign tooling must be able to aggregate:

- p50 and p95 latency;
- success rate;
- provider utilization;
- lease rejection reasons;
- residency hit rates;
- edge-cost contribution to selected assignments;
- replan counts.
