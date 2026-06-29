# Tasks: DI Threshold Campaign

- [x] T001 Add encoded output-bundle padding support to the C++ ONNX runner.
- [x] T002 Add activation padding to NativeTracer policy bundle generation.
- [x] T003 Thread activation padding through the MiniNDN harness summary.
- [x] T004 Extend the layout campaign runner for multiple padding values.
- [x] T005 Run syntax/build validation.
- [x] T006 Run smoke threshold campaign.
- [x] T007 Record accepted results and next recommendation.

## Validation

- `python3 -m py_compile Experiments/NDNSF_DI_NativeTracer_Minindn.py examples/python/NDNSF-DistributedInference/native_di_tracer/plan_tracer.py examples/python/NDNSF-DistributedInference/native_di_tracer/run_layout_campaign.py`
- `PYTHONPATH=NDNSF-DistributedInference:pythonWrapper python3 examples/python/NDNSF-DistributedInference/native_di_tracer/plan_tracer.py --out /tmp/ndnsf-di-pad-policy-smoke --summary-json /tmp/ndnsf-di-pad-policy-smoke-summary.json --activation-pad-bytes 65536`
- `./waf build --targets=di-native-provider`
- `./waf build --targets=di-native-plan-manifest-smoke`
- `python3 Experiments/NDNSF_DI_NativeTracer_Minindn.py --local-execution-only --assignment default --activation-pad-bytes 65536 --out /tmp/ndnsf-di-pad-local-smoke`
- `sudo -n python3 Experiments/NDNSF_DI_NativeTracer_Minindn.py --full-network --assignment default --activation-pad-bytes 65536 --out /tmp/ndnsf-di-pad-full-default-smoke --provider-check-timeout 60`
- `python3 examples/python/NDNSF-DistributedInference/native_di_tracer/run_layout_campaign.py --runs 1 --out-root /tmp/ndnsf-di-threshold-smoke --provider-check-timeout 60 --activation-pad-bytes-list 0,65536`
- `python3 examples/python/NDNSF-DistributedInference/native_di_tracer/run_layout_campaign.py --runs 3 --out-root /tmp/ndnsf-di-threshold-campaign-3 --provider-check-timeout 60 --activation-pad-bytes-list 0,65536,262144,1048576`

Smoke result:

- Pad `0`: default `253.303 ms`, single-provider `201.471 ms`.
- Pad `65536`: default `256.534 ms`, single-provider `191.265 ms`.
- All four full-network smoke runs executed successfully.

Threshold campaign result:

| Activation padding | Default mean ms | Default p95 ms | Single-provider mean ms | Single-provider p95 ms | Mean delta ms |
| ---: | ---: | ---: | ---: | ---: | ---: |
| 0 | 218.907 | 291.893 | 211.313 | 223.850 | -7.594 |
| 65536 | 292.584 | 300.988 | 178.912 | 199.410 | -113.672 |
| 262144 | 355.196 | 380.050 | 197.816 | 205.440 | -157.380 |
| 1048576 | 3709.943 | 5302.937 | 199.368 | 216.767 | -3510.575 |

All 24 full-network MiniNDN runs completed with the expected executable
candidates. The broader campaign did not find a threshold where
`shared-backbone-current` beats `single-provider-serial`; instead it confirmed
that artificial Backbone activation growth increases cross-provider dependency
exchange cost for this smallest Qwen NativeTracer workload.
