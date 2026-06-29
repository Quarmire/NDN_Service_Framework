# Plan: DI Concurrency-4 Scaling Campaign

## Approach

Reuse `run_layout_campaign.py` with:

```text
requests=4
concurrency=4
roleExecutionDelayMs=75
runs=10 per assignment
```

The two assignments remain:

```text
default -> shared-backbone-current
single-provider -> single-provider-serial
```

## Validation

1. Run Python syntax validation for the NativeTracer user and campaign scripts.
   Complete.
2. Run the 10x2 full-network MiniNDN campaign. Complete.
3. Inspect `campaign-summary.json` and `campaign-runs.csv`. Complete.
4. Record success counts and per-request workload latency in the spec.
   Complete.

## Interpretation Rule

For concurrent workloads, prefer the workload fields (`meanMs`, `p50Ms`,
`p95Ms`, and `throughputRps`) over whole-run `elapsedMs`, because `elapsedMs`
includes full-network setup, parent process coordination, and makespan.

If shared-backbone is still faster at concurrency 4, the next design step is to
turn planner scoring into a concurrency-aware policy. If single-provider wins
again, the next step is to identify the dependency exchange or provider queueing
limit.

## Observed Result

Both assignments completed 10/10 runs:

```text
default / shared-backbone-current:        40 success, 0 failure
single-provider / single-provider-serial: 40 success, 0 failure
```

Accepted workload latency comparison:

```text
default workload mean/p50/p95:         458.414 / 441.790 / 510.167 ms
single-provider workload mean/p50/p95: 619.563 / 588.057 / 709.669 ms
```

At concurrency 4, `shared-backbone-current` remains faster. This points toward
single-provider serial queueing as the dominant cost at this concurrency level,
while NDNSF dependency exchange remains acceptable for this small model and
topology.
