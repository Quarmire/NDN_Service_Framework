# Implementation Plan: DI Execution Operation Status

## Summary

Add a source-level summary bridge in `Experiments/NDNSF_DI_NativeTracer_Minindn.py`
that attaches core `ServiceOperationStatus` to `userExecution` and
`dependencyExecution` dictionaries before `summary.json` is written.

## Design

1. Add `_execution_operation_state()` with explicit status/reason mapping.
2. Add `_execution_operation_progress()` that derives progress from
   success/request counts when available.
3. Add `_execution_operation_metadata()` that copies important legacy evidence
   into the core status metadata while leaving the legacy top-level fields
   unchanged.
4. Add `with_execution_operation_status()` for one execution dictionary.
5. Add `attach_execution_operation_statuses()` and call it from the final
   summary write path so gated, local-baseline, full-network success, and
   failure summaries all get the envelope.
6. Add focused unit coverage in the existing NativeTracer campaign test module.

## Verification

```bash
PYTHONPATH=pythonWrapper:NDNSF-DistributedInference:Experiments python3 tests/python/test_ndnsf_di_runtime_aware_campaign.py
git diff --check
```
