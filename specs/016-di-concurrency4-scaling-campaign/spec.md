# Feature 016: DI Concurrency-4 Scaling Campaign

Status: Accepted

## Goal

Measure whether the concurrency-2 result from Feature 015 scales to four
outstanding NativeTracer requests.

The model remains the existing smallest Qwen-derived NativeTracer ONNX artifact
set.

## Scope

- Run the full-network MiniNDN NativeTracer harness.
- Use `requests=4`, `concurrency=4`.
- Keep `roleExecutionDelayMs=75` to match the previous concurrency campaign.
- Compare both executable assignments:
  - `default` -> `shared-backbone-current`
  - `single-provider` -> `single-provider-serial`
- Record whole-run makespan, per-request workload mean/p50/p95, success/failure
  counts, and throughput.

## Non-Goals

- New planner policy.
- New protocol changes.
- Larger model artifacts.
- Lossy-network or RPS stress testing.

## Acceptance

- [x] Campaign completes repeated full-network MiniNDN runs with
  `requests=4`, `concurrency=4`.
- [x] Both assignments report success/failure counts.
- [x] Results show whether shared-backbone still improves per-request latency
  or begins to lose to fixed dependency/queueing overhead.
- [x] Spec records accepted artifact paths and interpretation.

## Evidence Command

```bash
python3 examples/python/NDNSF-DistributedInference/native_di_tracer/run_layout_campaign.py \
  --runs 10 \
  --requests 4 \
  --concurrency 4 \
  --role-execution-delay-ms-list 75 \
  --out-root /tmp/ndnsf-di-concurrency4-campaign-10 \
  --provider-check-timeout 60
```

## Accepted Campaign

Artifacts:

- Summary: `/tmp/ndnsf-di-concurrency4-campaign-10/campaign-summary.json`
- Per-run CSV: `/tmp/ndnsf-di-concurrency4-campaign-10/campaign-runs.csv`
- Per-run evidence:
  `/tmp/ndnsf-di-concurrency4-campaign-10/{default,single-provider}/run-*`

Results:

| Assignment | Runtime candidate | Runs | Requests | Success | Failure | Workload mean ms | Workload p50 ms | Workload p95 ms | Throughput RPS |
| --- | --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| `default` | `shared-backbone-current` | 10 | 40 | 40 | 0 | 458.414 | 441.790 | 510.167 | 0.387 |
| `single-provider` | `single-provider-serial` | 10 | 40 | 40 | 0 | 619.563 | 588.057 | 709.669 | 0.378 |

Comparison based on per-request workload latency:

- Workload mean delta: `+161.149 ms` for `single-provider`.
- Workload p50 delta: `+146.267 ms` for `single-provider`.
- Workload p95 delta: `+199.502 ms` for `single-provider`.

Interpretation: both layouts remained stable at concurrency 4, with 40/40
requests successful in each assignment. `shared-backbone-current` continued to
outperform `single-provider-serial` on per-request mean, p50, and p95 latency.
This strengthens the Feature 015 conclusion: for the smallest Qwen NativeTracer
artifact, a single request favors avoiding dependency exchange, but concurrent
requests favor shared-backbone because parallel role execution reduces
per-request latency enough to offset NDNSF dependency exchange.

The next planner step should be concurrency-aware scoring. The planner should
not treat `single-provider-serial` or `shared-backbone-current` as universally
best; it should estimate request concurrency, provider queueing, and dependency
exchange cost together.
