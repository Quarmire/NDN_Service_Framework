# Implementation Plan: Deployment Operation Status

## Summary

Add deployment lifecycle helpers in `pythonWrapper/ndnsf/service.py` that create
and consume core `ServiceOperationStatus` while keeping legacy deployment fields
intact.

## Design

1. Add `_deployment_operation_state(status)` and `_deployment_progress(status)`.
2. Add `_deployment_operation_status(deployment, operation="DEPLOYMENT")`.
3. Add `_deployment_sort_key(deployment)` that prefers parsed
   `operationStatus.state` and then the legacy deployment status.
4. Add `operationStatus` to deployment, eviction, rejection, and not-found
   dictionaries.
5. Keep `wait_deployment(target_status=...)` compatible with legacy status.

## Verification

```bash
PYTHONPATH=pythonWrapper python3 tests/python/test_ndnsf_deployment_operation_status.py
PYTHONPATH=pythonWrapper python3 tests/python/test_ndnsf_core_boundary_envelopes.py
```

