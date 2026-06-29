# Plan: DI Concurrency Stability Campaign

## Approach

Reuse `run_layout_campaign.py`, which already wraps
`Experiments/NDNSF_DI_NativeTracer_Minindn.py` and emits per-run plus aggregate
campaign artifacts.

This campaign fixes the workload at:

```text
requests=2
concurrency=2
roleExecutionDelayMs=75
```

The campaign compares the two executable NativeTracer layouts:

```text
default -> shared-backbone-current
single-provider -> single-provider-serial
```

## Validation

1. Run Python syntax validation for the changed user driver and campaign
   runner. Complete.
2. Run the 10x2 MiniNDN campaign. Complete.
3. Read `campaign-summary.json` and `campaign-runs.csv`. Complete.
4. Record success counts, latency, throughput, and interpretation in the spec.
   Complete.

## Interpretation Rule

If both assignments complete 10/10 successful runs, treat Feature 014 as stable
for concurrency 2. If any run fails, record it as remaining concurrency evidence
instead of hiding the failure.

## Observed Result

Both assignments completed 10/10 runs and 20/20 requests:

```text
default / shared-backbone-current:       20 success, 0 failure
single-provider / single-provider-serial: 20 success, 0 failure
```

For concurrent workloads, use the workload latency fields (`meanMs`, `p50Ms`,
`p95Ms`) rather than only whole-run `elapsedMs`, because `elapsedMs` includes
fixed full-network setup and workload makespan. The accepted comparison is:

```text
default workload mean/p50/p95:         478.957 / 446.470 / 511.443 ms
single-provider workload mean/p50/p95: 567.271 / 537.988 / 596.554 ms
```

This makes `shared-backbone-current` faster in the tested concurrency-2
campaign.
