# Implementation Plan: Repo Core Discovery Selection

## Summary

Bridge DistributedRepo ACK selection to the core service-discovery helper
introduced by Spec058. The Repo client will parse `ProviderCapabilityHint` from
ACK metadata into `ServiceDiscoveryRecord`, skip draining/unready typed hints in
capacity selection, and keep legacy ACK fallback behavior.

## Technical Context

- Python 3.8+
- Existing `py_repoclient.capability_from_ack`
- Existing `ndnsf.ProviderCapabilityHint` and `ndnsf.ServiceDiscoveryRecord`
- Tests: Python unittest

## Design

1. Add `discovery_record_from_ack(candidate)` in `py_repoclient`.
2. Add `ready_capability_from_ack(candidate)` as a small wrapper that returns
   `None` when the core discovery record is not ready.
3. Keep `capability_from_ack(candidate)` focused on parsing storage capability
   with legacy fallback.
4. Update `_capacity_selector` to use `ready_capability_from_ack`.
5. Add direct tests for ready, draining, unready, legacy fallback, and
   all-unready empty selection.

## Verification

```bash
PYTHONPATH=.:pythonWrapper:NDNSF-DistributedRepo/pythonWrapper python3 tests/python/test_ndnsf_repo_core_discovery_selection.py
PYTHONPATH=.:pythonWrapper:NDNSF-DistributedInference:NDNSF-DistributedRepo/pythonWrapper python3 tests/python/test_ndnsf_app_core_envelope_migration.py
PYTHONPATH=pythonWrapper python3 tests/python/test_ndnsf_core_service_discovery.py
```

