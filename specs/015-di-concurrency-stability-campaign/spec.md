# Feature 015: DI Concurrency Stability Campaign

Status: Accepted

## Goal

Validate that the ACK/RESPONSE request-scoped key fix from Feature 014 is
stable across repeated full-network NativeTracer runs, not just one successful
diagnostic run.

The model remains the existing smallest Qwen-derived NativeTracer ONNX artifact
set.

## Scope

- Run the existing full-network MiniNDN NativeTracer harness.
- Use `requests=2`, `concurrency=2`.
- Use the default 75 ms per-role execution delay that exposed the earlier
  second-request starvation failure.
- Compare both executable assignments:
  - `default` -> `shared-backbone-current`
  - `single-provider` -> `single-provider-serial`
- Produce JSON/CSV campaign artifacts with success count, latency, p50, p95,
  stddev, and throughput.

## Non-Goals

- Larger model artifacts.
- New planner policy.
- New service protocol changes.
- Concurrency above 2. Higher concurrency should be a later campaign after this
  one is stable.

## Acceptance

- [x] Campaign runner completes repeated full-network MiniNDN runs with
  `requests=2`, `concurrency=2`.
- [x] Both assignments report all runs successful.
- [x] Results include per-run and aggregate latency/throughput metrics.
- [x] Spec records accepted artifact paths and interpretation.

## Evidence Command

```bash
python3 examples/python/NDNSF-DistributedInference/native_di_tracer/run_layout_campaign.py \
  --runs 10 \
  --requests 2 \
  --concurrency 2 \
  --role-execution-delay-ms-list 75 \
  --out-root /tmp/ndnsf-di-concurrency-campaign-10 \
  --provider-check-timeout 60
```

## Accepted Campaign

Artifacts:

- Summary: `/tmp/ndnsf-di-concurrency-campaign-10/campaign-summary.json`
- Per-run CSV: `/tmp/ndnsf-di-concurrency-campaign-10/campaign-runs.csv`
- Per-run evidence:
  `/tmp/ndnsf-di-concurrency-campaign-10/{default,single-provider}/run-*`

Results:

| Assignment | Runtime candidate | Runs | Requests | Success | Failure | Workload mean ms | Workload p50 ms | Workload p95 ms | Throughput RPS |
| --- | --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| `default` | `shared-backbone-current` | 10 | 20 | 20 | 0 | 478.957 | 446.470 | 511.443 | 0.204 |
| `single-provider` | `single-provider-serial` | 10 | 20 | 20 | 0 | 567.271 | 537.988 | 596.554 | 0.201 |

Comparison based on per-request workload latency:

- Workload mean delta: `+88.314 ms` for `single-provider`.
- Workload p50 delta: `+91.518 ms` for `single-provider`.
- Workload p95 delta: `+85.111 ms` for `single-provider`.

Interpretation: Feature 014 is stable for the tested concurrency-2 workload:
both executable layouts completed 20/20 requests with no observed starvation or
missing selection. Under this two-request concurrent workload,
`shared-backbone-current` is faster than `single-provider-serial` on
per-request mean, p50, and p95 latency. This differs from the earlier
single-request campaign, where the single-provider layout avoided dependency
exchange overhead. The current result suggests that once two requests are
outstanding, the shared-backbone layout can use parallel role execution enough
to offset its NDNSF dependency exchange cost.
