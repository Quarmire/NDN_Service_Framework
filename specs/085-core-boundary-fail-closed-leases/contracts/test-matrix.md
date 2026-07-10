# Test Matrix

| Concern | Exact target | Required cases |
|---|---|---|
| Core lease table | `tests/python/test_ndnsf_execution_lease_table.py` | state transitions, duplicate, conflict, expiry, renew, epoch, identity/binding |
| DI transaction | `tests/python/test_ndnsf_di_execution_lease_transaction.py` | all commit, prepare reject, partial commit, abort/release loss, no early execute |
| C++/Python DI payload parity | `tests/unit-tests/di-execution-lease-service.t.cpp`; `tests/python/test_ndnsf_di_execution_lease_codec.py` | identical fixtures, malformed/unknown version, every operation/reason |
| Provider restart/eviction | `tests/python/test_ndnsf_di_execution_lease_restart.py` | stale epoch, restart, active pin, expiry cleanup |
| Current fallback regression | `tests/python/test_ndnsf_di_execution_lease_fallback.py` | reproduce missing import baseline, treatment typed failure |
| Boundary imports | `tests/python/test_ndnsf_core_boundary_imports.py` | generic exports absent, DI/Repo imports work |
| Existing Core | `./build/unit-tests`; Core Python patterns | parent baseline remains green |
| Security | `examples/run_security_regressions.sh` | all six pass |
| DI/Repo | exact parent baseline commands | no behavior regression |
| MiniNDN | child campaign script/fixture | 3 x 60 s coordinator-off, C++ native provider activation/release, zero conflict |
