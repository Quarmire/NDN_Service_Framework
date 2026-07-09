# Quickstart: UAV QGC-Parity Boundary Slice

Build and test the foundational protocol contracts:

```bash
./waf build --targets=unit-tests
./build/unit-tests --run_test=UavProtocolState
PYTHONPATH=.:pythonWrapper:NDNSF-DistributedInference:NDNSF-DistributedRepo/pythonWrapper \
  python3 tests/python/test_ndnsf_app_core_envelope_migration.py
git diff --check
```

The next runtime slice should expose the new contracts through NDNSF-UAV-APP
services:

- `/UAV/MAVLink/ParameterEdit`
- `/UAV/Preflight/Checklist`
- `/UAV/MAVLink/AnalyzeSnapshot`

