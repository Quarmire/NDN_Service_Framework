# Tasks: DI Rate-Aware Cost Model

- [x] R001 Add `targetRps` to NativeTracer planner evidence.
- [x] R002 Add provider utilization and provider capacity queue pressure fields.
- [x] R003 Add dependency byte-rate and link utilization fields.
- [x] R004 Propagate `targetRps` through `plan_tracer.py` and MiniNDN harness.
- [x] R005 Propagate new pressure fields into layout and auto campaign summaries.
- [x] R006 Validate default c1/c4/c8 recommendations are unchanged.
- [x] R007 Validate positive target RPS produces non-zero rate-aware fields.
- [x] R008 Run syntax, focused DI tests, and diff checks.
- [x] R009 Add and run rate sweep helper for planner and MiniNDN smoke evidence.

## Validation Evidence

Python syntax:

```bash
python3 -m py_compile \
  Experiments/NDNSF_DI_NativeTracer_Minindn.py \
  examples/python/NDNSF-DistributedInference/native_di_tracer/optimize_native_tracer_plan.py \
  examples/python/NDNSF-DistributedInference/native_di_tracer/plan_tracer.py \
  examples/python/NDNSF-DistributedInference/native_di_tracer/run_layout_campaign.py \
  examples/python/NDNSF-DistributedInference/native_di_tracer/run_auto_assignment_campaign.py
```

Result: pass.

Default recommendation boundary:

```bash
for c in 1 4 8; do
  python3 examples/python/NDNSF-DistributedInference/native_di_tracer/plan_tracer.py \
    --out /tmp/ndnsf-di-025-boundary-c${c} \
    --runtime-candidate shared-backbone-current \
    --role-execution-delay-ms 75 \
    --workload-concurrency ${c} \
    --target-rps 0 \
    --summary-json /tmp/ndnsf-di-025-boundary-c${c}/summary.json
done
```

Result:

| Concurrency | Recommended candidate | targetRps |
| ---: | --- | ---: |
| 1 | `single-provider-serial` | 0.0 |
| 4 | `shared-backbone-current` | 0.0 |
| 8 | `shared-backbone-current` | 0.0 |

Positive target-rate planner evidence:

```bash
python3 examples/python/NDNSF-DistributedInference/native_di_tracer/plan_tracer.py \
  --out /tmp/ndnsf-di-025-rate-c4-rps2 \
  --runtime-candidate shared-backbone-current \
  --role-execution-delay-ms 75 \
  --workload-concurrency 4 \
  --target-rps 2 \
  --summary-json /tmp/ndnsf-di-025-rate-c4-rps2/summary.json
```

Result:

| Candidate | Provider utilization | Provider capacity queue ms | Dependency crossing bytes | Dependency byte-rate Mbps |
| --- | ---: | ---: | ---: | ---: |
| `shared-backbone-current` | 0.158 | 14.824 | 768.0 | 0.012288 |
| `single-provider-serial` | 0.621 | 508.761 | 0.0 | 0.0 |

Full-network MiniNDN auto smoke with rate-aware evidence:

```bash
python3 examples/python/NDNSF-DistributedInference/native_di_tracer/run_auto_assignment_campaign.py \
  --runs 1 \
  --out-root /tmp/ndnsf-di-025-auto-rate-smoke \
  --provider-check-timeout 60 \
  --role-execution-delay-ms 75 \
  --target-rps 2 \
  --workloads c4:4:4
```

Result: success, 4/4 requests completed, auto resolved to `default` and
selected `shared-backbone-current`. The campaign summary included:

```text
selectedProviderMaxUtilization=0.158
selectedProviderCapacityQueuePressureMs=14.824
selectedDependencyByteRateMbps=0.012288
selectedDependencyMaxLinkUtilization=0.000041
selectedDependencyRatePressureMs=0.007
providerQueueWaitMeanMs=0.028
```

Rate sweep helper:

```bash
python3 examples/python/NDNSF-DistributedInference/native_di_tracer/run_rate_sweep_campaign.py \
  --out-root /tmp/ndnsf-di-025-rate-sweep-minindn \
  --target-rps-list 0,1,2,4,8 \
  --minindn-rps-list 0,2,8 \
  --requests 4 \
  --concurrency 4 \
  --role-execution-delay-ms 75 \
  --provider-check-timeout 60
```

Planner sweep result:

| targetRps | Recommended | Shared estimate ms | Shared utilization | Shared capacity queue ms | Single estimate ms | Single utilization | Single capacity queue ms |
| ---: | --- | ---: | ---: | ---: | ---: | ---: | ---: |
| 0 | `shared-backbone-current` | 463.531 | 0.000 | 0.000 | 776.250 | 0.000 | 0.000 |
| 1 | `shared-backbone-current` | 470.310 | 0.079 | 6.776 | 916.076 | 0.3105 | 139.826 |
| 2 | `shared-backbone-current` | 478.362 | 0.158 | 14.824 | 1285.011 | 0.621 | 508.761 |
| 4 | `shared-backbone-current` | 500.042 | 0.316 | 36.497 | 16052.850 | 1.242 | 15276.600 |
| 8 | `shared-backbone-current` | 599.232 | 0.632 | 135.674 | 54616.950 | 2.484 | 53840.700 |

MiniNDN smoke subset:

| targetRps | Selected | Successes | Workload mean ms | Workload p95 ms | Provider queue mean ms | Selected utilization | Selected capacity queue ms |
| ---: | --- | ---: | ---: | ---: | ---: | ---: | ---: |
| 0 | `shared-backbone-current` | 4/4 | 458.899 | 495.286 | 0.030 | 0.000 | 0.000 |
| 2 | `shared-backbone-current` | 4/4 | 440.564 | 482.220 | 0.037 | 0.158 | 14.824 |
| 8 | `shared-backbone-current` | 4/4 | 442.404 | 475.137 | 0.028 | 0.632 | 135.674 |

The MiniNDN subset validates that the new evidence fields propagate through
full-network auto assignment. It is not an open-loop RPS load test: the current
NativeTracer user driver still submits a fixed number of concurrent requests, so
measured latency is not expected to rise with `targetRps` yet.
