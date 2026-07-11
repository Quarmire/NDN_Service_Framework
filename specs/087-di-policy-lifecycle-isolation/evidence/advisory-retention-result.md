# Advisory Retention Result

## Frozen Campaign

- Ten matched pure user-side/advisory pairs.
- Seeds: `PYTHONHASHSEED=8701` through `8710`.
- Same capacity-pool topology, four requests, concurrency four, 2 RPS,
  one-second role delay, and five-second overload fast-fail timeout.
- Raw summaries: `results/spec087-advisory-gate/run-01` through `run-10`.
- Analysis: `results/spec087-advisory-gate/gate-analysis.json`.

## Result

| Metric | Pure user-side | Advisory |
|---|---:|---:|
| Mean lease-conflict rate | 0.06615 | 0.10191 |
| Mean completion | 70.0% | 52.5% |
| Mean p50 | 4247.26 ms | 4923.75 ms |
| Mean p95 | 5514.96 ms | 5640.08 ms |
| Max stable RPS at this overload point | 0 | 0 |

The primary metric's relative improvement was -54.06%. The paired mean
pure-minus-advisory difference was -0.03576 with bootstrap 95% CI
[-0.07136, 0.00387]. The CI crosses zero and completion/latency regress.

## Decision

The frozen gate fails. Delete DI advisory coordination and retain the simpler
distributed design: each user plans locally, while provider-owned execution
leases fail closed under contention. Keep the generic Core coordination
envelopes because they are application-neutral and independently tested.
