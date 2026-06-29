# Tasks: DI Provider Capacity Campaign

- [x] C001 Add `executionDelayMs` support to the C++ ONNX runner.
- [x] C002 Add matching fake-runner support in `di-native-plan-manifest-smoke`.
- [x] C003 Thread `--role-execution-delay-ms` through policy generation and
  summary metadata.
- [x] C004 Thread `--role-execution-delay-ms` through MiniNDN harness.
- [x] C005 Extend layout campaign runner for delay sweeps.
- [x] C006 Run syntax/build validation.
- [x] C007 Run local and full-network smoke.
- [x] C008 Run a small capacity campaign and record results.
- [x] C009 Update docs with interpretation and next step.

## Validation Commands

- `python3 -m py_compile Experiments/NDNSF_DI_NativeTracer_Minindn.py examples/python/NDNSF-DistributedInference/native_di_tracer/plan_tracer.py examples/python/NDNSF-DistributedInference/native_di_tracer/run_layout_campaign.py examples/python/NDNSF-DistributedInference/native_di_tracer/optimize_native_tracer_plan.py`
- `./waf build --targets=di-native-provider`
- `./waf build --targets=di-native-plan-manifest-smoke`
- `PYTHONPATH=NDNSF-DistributedInference:pythonWrapper python3 examples/python/NDNSF-DistributedInference/native_di_tracer/plan_tracer.py --out /tmp/ndnsf-di-capacity-policy-smoke --summary-json /tmp/ndnsf-di-capacity-policy-smoke-summary.json --role-execution-delay-ms 25`
- `python3 Experiments/NDNSF_DI_NativeTracer_Minindn.py --local-execution-only --assignment default --role-execution-delay-ms 25 --out /tmp/ndnsf-di-capacity-local-smoke`
- `sudo -n python3 Experiments/NDNSF_DI_NativeTracer_Minindn.py --full-network --assignment default --role-execution-delay-ms 25 --out /tmp/ndnsf-di-capacity-full-default-smoke --provider-check-timeout 60`
- `python3 examples/python/NDNSF-DistributedInference/native_di_tracer/run_layout_campaign.py --runs 1 --out-root /tmp/ndnsf-di-capacity-campaign-smoke --provider-check-timeout 60 --role-execution-delay-ms-list 0,25`
- `python3 examples/python/NDNSF-DistributedInference/native_di_tracer/run_layout_campaign.py --runs 1 --out-root /tmp/ndnsf-di-capacity-campaign-smoke-50-75 --provider-check-timeout 60 --role-execution-delay-ms-list 50,75`
- `python3 examples/python/NDNSF-DistributedInference/native_di_tracer/run_layout_campaign.py --runs 3 --out-root /tmp/ndnsf-di-capacity-campaign-75x3 --provider-check-timeout 60 --role-execution-delay-ms-list 75`

## Accepted Results

Single-run sweep:

| Role delay ms | Shared-backbone ms | Single-provider ms | Single minus shared ms |
| ---: | ---: | ---: | ---: |
| 0 | 285.664 | 179.470 | -106.194 |
| 25 | 345.130 | 318.482 | -26.648 |
| 50 | 419.097 | 388.562 | -30.535 |
| 75 | 469.112 | 473.932 | 4.820 |

Confirmed 75 ms campaign:

| Layout | Runs | Mean ms | Stddev ms | p50 ms | p95 ms |
| --- | ---: | ---: | ---: | ---: | ---: |
| `shared-backbone-current` | 3 | 494.673 | 33.628 | 488.016 | 531.132 |
| `single-provider-serial` | 3 | 512.909 | 31.059 | 506.711 | 546.599 |

At 75 ms per role, shared-backbone was faster by 18.236 ms mean.
