# Tasks: DI Closed-Loop Workload Campaign

- [x] W001 Add `--requests` closed-loop support to `user_driver.py`.
- [x] W002 Preserve backward-compatible aggregate execution output.
- [x] W003 Thread request count through MiniNDN harness and summaries.
- [x] W004 Thread request count through layout campaign runner.
- [x] W005 Run syntax/build checks.
- [x] W006 Run closed-loop full-network smoke.
- [x] W007 Run a small closed-loop campaign and record results.
- [x] W008 Update docs with interpretation and next step.

## Validation Commands

- `python3 -m py_compile examples/python/NDNSF-DistributedInference/native_di_tracer/user_driver.py Experiments/NDNSF_DI_NativeTracer_Minindn.py examples/python/NDNSF-DistributedInference/native_di_tracer/run_layout_campaign.py`
- `PYTHONPATH=pythonWrapper:NDNSF-DistributedInference python3 examples/python/NDNSF-DistributedInference/native_di_tracer/user_driver.py --dry-run --requests 3`
- `./waf build --targets=di-native-provider`
- `./waf build --targets=di-native-plan-manifest-smoke`
- `sudo -n python3 Experiments/NDNSF_DI_NativeTracer_Minindn.py --full-network --assignment default --role-execution-delay-ms 75 --requests 3 --out /tmp/ndnsf-di-closed-loop-default-smoke --provider-check-timeout 60`
- `python3 examples/python/NDNSF-DistributedInference/native_di_tracer/run_layout_campaign.py --runs 2 --out-root /tmp/ndnsf-di-closed-loop-campaign-3req-75 --provider-check-timeout 60 --role-execution-delay-ms-list 75 --requests 3`

## Accepted Results

| Assignment | Runtime candidate | Runs | Requests/run | Makespan mean ms | Makespan p95 ms | Workload p95 mean ms | Throughput mean rps |
| --- | --- | ---: | ---: | ---: | ---: | ---: | ---: |
| `default` | `shared-backbone-current` | 2 | 3 | 1128.025 | 1137.698 | 480.911 | 2.660 |
| `single-provider` | `single-provider-serial` | 2 | 3 | 1231.884 | 1260.203 | 489.938 | 2.437 |

Shared-backbone reduced closed-loop makespan by `103.859 ms` mean and improved
throughput by about `9.2%` under 75 ms per-role capacity pressure.
