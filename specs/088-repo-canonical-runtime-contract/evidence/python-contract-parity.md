# Python Contract Parity

The `py_repoclient` binding now exposes every public object-state field needed
by the canonical C++ contract, including exact packet names, generation/write
lifecycle, Data references, operation status, catalog state and cache status.
JSON parsers use the C++ `RepoProtocol` implementation rather than duplicate
Python parsing rules.

```bash
CFLAGS='-O0 -g0' CXXFLAGS='-O0 -g0' \
  python3 setup.py build_ext --inplace --force
PYTHONPATH=pythonWrapper:NDNSF-DistributedRepo/pythonWrapper:NDNSF-DistributedInference:. \
  python3 tests/python/test_ndnsf_repo_core_discovery_selection.py
```

Result: extension build PASS; 7/7 parity and discovery-selection tests PASS.
