# Quickstart: Validate Core Runtime Contract Completion

Run from the repository root:

```bash
PYTHONPATH=pythonWrapper python3 tests/python/test_ndnsf_core_boundary_envelopes.py
PYTHONPATH=pythonWrapper python3 tests/python/test_ndnsf_core_service_discovery.py
PYTHONPATH=.:pythonWrapper:NDNSF-DistributedInference:NDNSF-DistributedRepo/pythonWrapper python3 tests/python/test_ndnsf_app_core_envelope_migration.py
./build/unit-tests --run_test=GenericAdmissionLease
```

Expected outcomes:

- C++ generic ACK metadata tests include provider capability and operation
  status round trips.
- Python core discovery tests classify ready, draining, stale, and unavailable
  providers.
- Repo/UAV/DI migration tests continue to pass with core-first parsing.
- No app-specific catalog, video, mission, or model planning semantics are
  moved into NDNSF core.
