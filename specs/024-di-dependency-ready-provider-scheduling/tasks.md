# Tasks: DI Dependency-Ready Provider Scheduling

- [x] D001 Add provider-worker head-of-line blocking regression test.
- [x] D002 Move dependency input waiting out of the compute worker.
- [x] D003 Preserve provider timing logs and make queue wait mean ready-queue wait.
- [x] D004 Run focused C++ tests.
- [x] D005 Run Python syntax validation.
- [x] D006 Run concurrent full-network MiniNDN smoke.
- [x] D007 Update experiment docs with the new provider scheduling semantics.
- [x] D008 Record validation evidence and remaining risk.
- [x] D009 Add provider timing aggregation to the layout campaign runner.
- [x] D010 Run c4/c8 small campaigns and record the queueing result.
- [x] D011 Add calibrated provider queue-pressure fields to planner evidence.
- [x] D012 Validate planner recommendations and queue-pressure fields for c1/c4/c8.
- [x] D013 Propagate planner queue-pressure fields into auto campaign summaries.
- [x] D014 Run c1/c4/c8 auto-assignment campaign and record measured queueing evidence.

## Validation Evidence

Focused worker regression:

```bash
./build/unit-tests -t ProviderRoleWorkerDoesNotOccupyComputeWorkerWhileWaitingForInputs
```

Result: pass. With `workerCount=1`, a dependency-waiting consumer no longer
blocks a later producer role from running and publishing the input.

Focused DI regression set:

```bash
./build/unit-tests -t AsyncDataflowRuntimeRunsStageFrontierHeadsInParallelBeforeMerge,ProviderRoleWorkerDoesNotOccupyComputeWorkerWhileWaitingForInputs,ProviderRoleWorkerPrefetchesAllInputsBeforeRunningRole,NativeProviderRuntimeDispatchesRegisteredRoleRunner,NativeExecutionPlanGeneratedJsonDrivesProviderRoleWorkers,NativeExecutionPlanGeneratedJsonDrivesProviderSessionSkeleton
```

Result: pass.

Python syntax:

```bash
python3 -m py_compile Experiments/NDNSF_DI_NativeTracer_Minindn.py examples/python/NDNSF-DistributedInference/native_di_tracer/*.py
```

Result: pass.

Full-network MiniNDN smoke:

```bash
sudo -n python3 Experiments/NDNSF_DI_NativeTracer_Minindn.py \
  --full-network \
  --assignment default \
  --role-execution-delay-ms 75 \
  --requests 4 \
  --concurrency 4 \
  --out /tmp/ndnsf-di-024-ready-scheduler-smoke \
  --provider-check-timeout 60
```

Result: `status=SUCCESS`, `userExecution=executed`,
`dependencyExecution=executed`, 4/4 requests succeeded.

Observed workload metrics:

```text
makespanMs=10291.654
meanMs=426.024
p95Ms=464.222
throughputRps=0.389
```

Provider timing logs emitted 16 role executions. Ready-queue wait was
`mean=0.048 ms`, `max=0.253 ms`.

Small c4 campaign:

```bash
python3 examples/python/NDNSF-DistributedInference/native_di_tracer/run_layout_campaign.py \
  --runs 2 \
  --out-root /tmp/ndnsf-di-024-c4-campaign-2 \
  --provider-check-timeout 60 \
  --role-execution-delay-ms-list 75 \
  --requests 8 \
  --concurrency 4
```

Result: all 32 role executions succeeded across 16 requests per assignment
group. Shared-backbone had workload mean `474.611 ms`, p95 `643.655 ms`, and
provider queue mean `0.281 ms`; single-provider had workload mean `561.850 ms`,
p95 `695.538 ms`, and provider queue mean `48.960 ms`.

Small c8 campaign:

```bash
python3 examples/python/NDNSF-DistributedInference/native_di_tracer/run_layout_campaign.py \
  --runs 2 \
  --out-root /tmp/ndnsf-di-024-c8-campaign-2 \
  --provider-check-timeout 60 \
  --role-execution-delay-ms-list 75 \
  --requests 16 \
  --concurrency 8
```

Result: all 64 role executions succeeded across 32 requests per assignment
group. Shared-backbone had workload mean `454.132 ms`, p95 `616.933 ms`, and
provider queue mean `0.033 ms`; single-provider had workload mean `617.723 ms`,
p95 `946.931 ms`, and provider queue mean `56.535 ms`.

Planner queue-pressure calibration:

```bash
for c in 1 4 8; do
  python3 examples/python/NDNSF-DistributedInference/native_di_tracer/plan_tracer.py \
    --out /tmp/ndnsf-di-024-planner-c${c} \
    --runtime-candidate shared-backbone-current \
    --role-execution-delay-ms 75 \
    --workload-concurrency ${c}
done
```

Result:

| Concurrency | Recommended | Shared providerReadyQueuePressureMs | Single providerReadyQueuePressureMs |
| ---: | --- | ---: | ---: |
| 1 | `single-provider-serial` | 0.000 | 0.000 |
| 4 | `shared-backbone-current` | 0.000 | 43.664 |
| 8 | `shared-backbone-current` | 0.000 | 50.941 |

These fields match the direction of the c4/c8 measured provider timing:
shared-backbone keeps one role per provider and near-zero ready queue pressure;
single-provider places all four roles on one provider and shows visible serial
queue pressure.

Auto-assignment queue-pressure campaign:

```bash
python3 examples/python/NDNSF-DistributedInference/native_di_tracer/run_auto_assignment_campaign.py \
  --runs 2 \
  --out-root /tmp/ndnsf-di-024-auto-queue-campaign-2 \
  --provider-check-timeout 60 \
  --role-execution-delay-ms 75 \
  --workloads c1:1:1,c4:8:4,c8:16:8
```

Result: all 6 full-network MiniNDN runs succeeded with
`userExecution=executed` and `dependencyExecution=executed`.

| Workload | Selected candidate | Requests | Failures | Workload mean ms | Workload p95 ms | Provider queue mean ms | Provider queue max ms | Selected queue pressure ms | Single-provider queue pressure ms |
| --- | --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| c1 | `single-provider-serial` | 2 | 0 | 509.996 | 509.996 | 18.980 | 75.892 | 0.000 | 0.000 |
| c4 | `shared-backbone-current` | 16 | 0 | 466.559 | 766.057 | 0.558 | 23.359 | 0.000 | 43.664 |
| c8 | `shared-backbone-current` | 32 | 0 | 447.802 | 619.082 | 0.086 | 3.596 | 0.000 | 50.941 |

The c4/c8 auto runs connect the planner estimate to the measured runtime
effect: auto selects the shared-backbone layout when the single-provider layout
would introduce serial ready-queue pressure, and the selected layout keeps
provider queue wait near zero in the full-network run.

## Remaining Risk

Full `./build/unit-tests` is not clean in this checkout because
`NdnSvsSmoke/ServiceUserRequestServiceReachesProviderAndReturnsResponse` fails
in the existing NDN-SVS smoke path. The focused DI worker tests and the
full-network NativeTracer smoke passed. Pending dependency waiters are currently
unbounded; production work should add admission limits if campaign sizes grow.
