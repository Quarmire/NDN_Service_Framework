# Feature 019: DI Auto Full-Network Campaign

Status: Accepted

## Goal

Turn the `--assignment auto` full-network smoke from Feature 018 into a small,
repeatable MiniNDN campaign.

The campaign should show, with repeated measured runs, that the concurrency-aware
planner can choose the executable NativeTracer runtime layout automatically:

- concurrency 1 -> `single-provider-serial`
- concurrency 2 -> `shared-backbone-current`
- concurrency 4 -> `shared-backbone-current`

## Scope

- Add a campaign helper for full-network `--assignment auto`.
- Run c1/c2/c4 workloads with repeated MiniNDN runs.
- Produce machine-readable JSON and CSV summaries.
- Record p50/p95/stddev style evidence for slides and paper.

## Non-Goals

- New planner cost model.
- New runtime layout.
- Larger model artifacts.
- Changing NDNSF service invocation or NDN-SVS behavior.

## Acceptance

- [x] Campaign helper runs `--full-network --assignment auto`.
- [x] Summary records per-workload selected candidates.
- [x] Summary records success/failure counts and latency statistics.
- [x] c1 resolves to `single-provider-serial`.
- [x] c2 and c4 resolve to `shared-backbone-current`.
- [x] Campaign evidence is recorded in this feature.

## Accepted Evidence

Campaign path:

```text
results/ndnsf_di_auto_assignment_campaign_20260629
```

Command:

```bash
python3 examples/python/NDNSF-DistributedInference/native_di_tracer/run_auto_assignment_campaign.py \
  --runs 5 \
  --out-root results/ndnsf_di_auto_assignment_campaign_20260629 \
  --role-execution-delay-ms 75 \
  --workloads c1:1:1,c2:2:2,c4:4:4
```

Results:

| Workload | Runs | Requests | Failures | Resolved assignment | Selected candidate | Workload mean ms | Workload p95 ms |
| --- | ---: | ---: | ---: | --- | --- | ---: | ---: |
| c1 | 5 | 5 | 0 | `single-provider` | `single-provider-serial` | 511.529 | 511.529 |
| c2 | 5 | 10 | 0 | `default` | `shared-backbone-current` | 469.389 | 502.756 |
| c4 | 5 | 20 | 0 | `default` | `shared-backbone-current` | 449.850 | 487.826 |

The campaign summary also records full user-driver elapsed time. For slides and
paper latency discussion, the primary per-request evidence is the workload
mean/p50/p95 fields, because the full elapsed time includes fixed driver/window
overhead around the closed-loop request set.

Follow-up queue-pressure campaign:

```text
/tmp/ndnsf-di-024-auto-queue-campaign-2
```

Command:

```bash
python3 examples/python/NDNSF-DistributedInference/native_di_tracer/run_auto_assignment_campaign.py \
  --runs 2 \
  --out-root /tmp/ndnsf-di-024-auto-queue-campaign-2 \
  --provider-check-timeout 60 \
  --role-execution-delay-ms 75 \
  --workloads c1:1:1,c4:8:4,c8:16:8
```

Results:

| Workload | Runs | Requests | Failures | Resolved assignment | Selected candidate | Provider queue mean ms | Selected queue pressure ms | Single-provider queue pressure ms |
| --- | ---: | ---: | ---: | --- | --- | ---: | ---: | ---: |
| c1 | 2 | 2 | 0 | `single-provider` | `single-provider-serial` | 18.980 | 0.000 | 0.000 |
| c4 | 2 | 16 | 0 | `default` | `shared-backbone-current` | 0.558 | 0.000 | 43.664 |
| c8 | 2 | 32 | 0 | `default` | `shared-backbone-current` | 0.086 | 0.000 | 50.941 |

This follow-up keeps the same smallest Qwen-derived NativeTracer artifacts and
75 ms controlled per-role work. It extends the original c1/c2/c4 campaign with
explicit queue-pressure evidence: low concurrency stays single-provider, while
c4 and c8 select shared-backbone because single-provider would build a serial
ready queue.
