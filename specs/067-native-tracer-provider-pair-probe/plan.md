# Implementation Plan: NativeTracer Provider-Pair Probe

## Summary

Spec065 added provider-pair telemetry collection from an existing
`dependency-edge-ndnping-rtt-stats.json`; Spec066 lets later planner runs
consume a previous matrix. This feature closes the loop by making NativeTracer
full-network runs generate that dependency-edge ndnping evidence before the
user workload starts.

## Design

1. Add lightweight helpers in the NativeTracer harness:
   - parse ndnping RTT output;
   - map launch rows to provider metadata by role;
   - load dependency edges from the generated native plan;
   - run dependency-edge ndnping and write the expected JSON file.
2. Run the probe after provider provisioning readiness is observed and before
   user-driver execution.
3. Keep the probe best-effort. If it fails, write failure status and continue.
4. Add `--skip-provider-pair-telemetry-probe` and matching runtime profile key.
5. Keep the existing `collect_provider_pair_telemetry()` finalizer as the
   single summary collection point.

## Validation

```bash
python3 -m py_compile Experiments/NDNSF_DI_NativeTracer_Minindn.py

PYTHONPATH=pythonWrapper:NDNSF-DistributedInference:Experiments \
  python3 tests/python/test_ndnsf_di_runtime_aware_campaign.py

git diff --check
```

