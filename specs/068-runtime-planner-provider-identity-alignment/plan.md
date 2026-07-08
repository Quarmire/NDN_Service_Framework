# Implementation Plan: Runtime Planner Provider Identity Alignment

## Summary

Previous runtime-aware evidence used fixture provider names such as
`/provider/gpu` and `/provider/disk`, while MiniNDN execution uses concrete
provider names such as `/NDNSF-DI/Tracer/provider/backbone`. This breaks
matching against provider-pair telemetry collected from MiniNDN. The fix is to
derive runtime-aware candidate metadata from `provider-profiles.json` when the
harness provides it.

## Design

1. Add a `load_provider_ack_metadata_input()` helper in `plan_tracer.py`.
2. If `--provider-profiles-json` exists, build `GenericAckMetadata` records
   using provider names from that file and fragment keys from the generated
   `PlanTemplate`.
3. If no usable profiles file exists, preserve the existing fixture fallback.
4. Record `providerRuntimeMetadata.source` and provider count in policy summary.
5. Add tests for provider-profile metadata construction.

## Validation

```bash
python3 -m py_compile \
  examples/python/NDNSF-DistributedInference/native_di_tracer/plan_tracer.py

PYTHONPATH=pythonWrapper:NDNSF-DistributedInference:Experiments \
  python3 tests/python/test_ndnsf_di_runtime_aware_campaign.py

git diff --check
```

