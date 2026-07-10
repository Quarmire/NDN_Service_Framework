# Core Execution Lease Implementation Evidence

**Date**: 2026-07-10

## Implementation

- `ndn-service-framework/ExecutionLease.hpp/.cpp` now owns the only execution
  lease state machine.
- `ProviderExecutionLeaseTable` implements prepare, commit, atomic
  validate-and-activate, validate, abort, renew, release, expiry cleanup,
  provider boot epochs, opaque conflict keys, binding checks, idempotent replay,
  typed rejection, snapshots, active-key queries, and counters.
- EXECUTING uses a separate hard deadline, so ordinary prepare/commit TTL does
  not release a resource while business logic is running.
- `pythonWrapper/src/ndnsf/_ndnsf.cpp` binds the C++ types/table directly.
  `runtime_telemetry.py` and `ndnsf.__init__` only alias the native types; there
  is no second Python algorithm and no callback is invoked under the table lock.

## Red-Green Findings

The first tracer test failed at link time before `ExecutionLease.cpp` existed.
After the minimal implementation, the expanded expiry test found that repeated
cleanup calls reported the same expired lease more than once. The implementation
was corrected so transition and accounting happen once. The activate replay
fingerprint was also expanded to cover the complete authenticated binding, not
only proof bytes.

## Verification

```bash
./waf build --targets=unit-tests -j4
./build/unit-tests --run_test='GenericExecutionLeaseTable/*' --log_level=test_suite
PYTHONPATH=pythonWrapper:NDNSF-DistributedInference \
  python3 -m unittest discover -s tests/python \
    -p 'test_ndnsf_execution_lease_table.py' -v
./build/unit-tests --log_level=message
```

Results:

- seven focused C++ lease tests passed;
- three Python/C++ identity and behavior parity tests passed;
- full C++ suite: 206 tests, no errors;
- real ONNX tests remained environment-gated exactly as in the baseline;
- editable Python extension rebuilt from the workspace and loaded from
  `pythonWrapper/ndnsf/_ndnsf.cpython-38-x86_64-linux-gnu.so`.
