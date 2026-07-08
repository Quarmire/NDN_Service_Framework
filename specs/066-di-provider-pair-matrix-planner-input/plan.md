# Implementation Plan: DI Provider-Pair Matrix Planner Input

## Summary

Add a narrow planner input path for core `ProviderNetworkMatrix` data. Spec065
made NativeTracer summaries emit provider-pair telemetry; this feature lets
later NativeTracer planning runs consume that matrix for dependency edge costs.

## Design

1. Add `--provider-network-matrix-json` to `plan_tracer.py`.
2. Add a loader that accepts:
   - raw matrix JSON with `metrics`;
   - wrapper JSON with `matrix`;
   - previous NativeTracer summary JSON with `providerPairTelemetry.matrix`.
3. Pass the loaded matrix to `choose_edge_aware_runtime_assignment`.
4. Add `providerNetworkMatrix.source` and `metricCount` to the policy summary.
5. Add the same CLI option to the MiniNDN harness and forward it into every
   NativeTracer plan-tracer invocation.
6. Keep missing option behavior unchanged by continuing to use the fixture
   matrix.

## Validation

```bash
python3 -m py_compile \
  Experiments/NDNSF_DI_NativeTracer_Minindn.py \
  examples/python/NDNSF-DistributedInference/native_di_tracer/plan_tracer.py

PYTHONPATH=pythonWrapper:NDNSF-DistributedInference:Experiments \
  python3 tests/python/test_ndnsf_di_runtime_aware_campaign.py

PYTHONPATH=pythonWrapper:NDNSF-DistributedInference \
  python3 tests/python/test_ndnsf_di_runtime_aware_planner.py

git diff --check
```

