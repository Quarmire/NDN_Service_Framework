# Feature 017: DI Concurrency-Aware Planner Evidence

Status: Accepted

## Goal

Extend NativeTracer planner evidence so it can explain the observed layout
boundary:

- Single request / small model: `single-provider-serial` can be faster because
  it avoids NDNSF dependency exchange.
- Concurrent requests: `shared-backbone-current` can be faster because
  provider work is distributed and single-provider serial execution queues.

The model remains the existing smallest Qwen-derived NativeTracer ONNX artifact
set.

## Scope

- Add workload concurrency to `planner-optimization.json`.
- Add per-candidate estimates for:
  - critical-path latency,
  - provider bottleneck work,
  - provider ready-queue pressure,
  - concurrency queue cost,
  - transfer/dependency exchange cost.
- Preserve fixed-assignment reproducibility: `--assignment default` and
  `--assignment single-provider` still force the runtime candidate used by the
  MiniNDN harness.
- Add a separate planner recommendation field so evidence can say which
  executable candidate should be preferred for the workload.

## Non-Goals

- New runtime layout.
- New NDNSF wire protocol.
- Larger model artifacts.
- Replacing the campaign measurements.

## Acceptance

- [x] `planner-optimization.json` records `workloadConcurrency`.
- [x] Candidate cost records include `criticalPathMs`,
  `providerBottleneckMs`, `providerReadyQueuePressureMs`, and
  `concurrencyQueueMs`.
- [x] Evidence records a `plannerRecommendedCandidate` independently from the
  forced `selectedCandidate`.
- [x] MiniNDN policy generation passes the requested concurrency into planner
  evidence.
- [x] Validation shows the planner recommends `single-provider-serial` for
  concurrency 1 and `shared-backbone-current` for concurrency 2/4 under the
  current calibrated model.

## Evidence Commands

```bash
python3 examples/python/NDNSF-DistributedInference/native_di_tracer/plan_tracer.py \
  --out /tmp/ndnsf-di-planner-c1 \
  --runtime-candidate shared-backbone-current \
  --role-execution-delay-ms 75 \
  --workload-concurrency 1

python3 examples/python/NDNSF-DistributedInference/native_di_tracer/plan_tracer.py \
  --out /tmp/ndnsf-di-planner-c4 \
  --runtime-candidate shared-backbone-current \
  --role-execution-delay-ms 75 \
  --workload-concurrency 4
```

## Accepted Evidence

Planner evidence paths:

- `/tmp/ndnsf-di-planner-c1/planner-optimization.json`
- `/tmp/ndnsf-di-planner-c2/planner-optimization.json`
- `/tmp/ndnsf-di-planner-c4/planner-optimization.json`
- `/tmp/ndnsf-di-024-planner-c1/planner-optimization.json`
- `/tmp/ndnsf-di-024-planner-c4/planner-optimization.json`
- `/tmp/ndnsf-di-024-planner-c8/planner-optimization.json`
- Harness smoke: `/tmp/ndnsf-di-planner-harness-c4/summary.json`

Validation results with `roleExecutionDelayMs=75`:

| Workload concurrency | selectedCandidate | plannerRecommendedCandidate | Shared total ms | Single total ms |
| ---: | --- | --- | ---: | ---: |
| 1 | `shared-backbone-current` | `single-provider-serial` | 345.031 | 310.500 |
| 2 | `shared-backbone-current` | `shared-backbone-current` | 384.531 | 465.750 |
| 4 | `shared-backbone-current` | `shared-backbone-current` | 463.531 | 776.250 |

Important cost fields:

| Candidate | Concurrency | providerCount | maxRoles/provider | providerReadyQueuePressureMs | concurrencyQueueMs |
| --- | ---: | ---: | ---: | ---: | ---: |
| `shared-backbone-current` | 4 | 4 | 1 | 0.000 | 118.500 |
| `single-provider-serial` | 4 | 1 | 4 | 43.664 | 465.750 |
| `shared-backbone-current` | 8 | 4 | 1 | 0.000 | 276.500 |
| `single-provider-serial` | 8 | 1 | 4 | 50.941 | 1086.750 |

The harness smoke confirmed that `--concurrency 4` reaches the planner evidence:

```text
/tmp/ndnsf-di-planner-harness-c4/summary.json
workloadConcurrency=4
plannerRecommendedCandidate=shared-backbone-current
```

Interpretation: the planner evidence now captures both the recommendation and
the reason behind the high-concurrency boundary. A forced runtime assignment
remains reproducible through `selectedCandidate`, while
`plannerRecommendedCandidate` records the layout the planner would prefer for
the declared workload. `providerReadyQueuePressureMs` is calibrated against the
c4/c8 provider timing campaigns: shared-backbone has one role per provider and
near-zero ready queue pressure, while single-provider has four roles on one
provider and visible serial queue pressure.
