# Fault Adapter and Orchestration Tests

Date: 2026-07-12

No once-only MiniNDN fault cell was executed in this phase. These are build,
ownership, authority, cleanup, lease, and security-contract checks required
before T048 may claim the positive-control output.

## Build

```bash
./waf build -j4
```

Result: PASS. Both `build/examples/di-native-provider` and the separately
compiled `build/examples/di-native-fault-provider` linked successfully.

Isolation checks:

```text
fault_invalid_rc=2
NDNSF_DI_EXPERIMENT_FAULT_PROVIDER_ERROR NDNSF_DI_FAULT_CONFIG_INVALID
normal_fault_surface=ABSENT
fault_surface=PRESENT
```

The normal provider contains no `NDNSF_DI_EXPERIMENT_FAULT` surface. The fault
provider rejects missing/unknown configuration before serving.

## Focused C++

```bash
./build/unit-tests --run_test='DiNativeFaultInjection/*' --log_level=test_suite
```

Result: PASS, 3/3. Covered fail-closed configuration, exact role/point matching,
one-shot injection, and bounded straggler delay.

## Python ownership, authority, cleanup, lease, and security

```bash
PYTHONPATH=tests/python:NDNSF-DistributedInference:. python3 -m unittest -q \
  test_ndnsf_di_spec107_faults \
  test_ndnsf_di_deployment_readiness \
  test_ndnsf_di_execution_lease_codec \
  test_ndnsf_di_execution_lease_integration \
  test_ndnsf_di_execution_lease_restart
```

Result: PASS, 46/46. The fault subset includes exclusive process-group
adoption, PID/PGID/PPID/start-time/command/executable/boot matching, trigger and
effect observation, one replacement, stale-attempt rejection, unchanged
deadline, unique terminal authority, immutable cell claims, and cleanup.

## Full relevant suites

```bash
./build/unit-tests --log_level=message
PYTHONPATH=tests/python:NDNSF-DistributedInference:. \
  python3 -m unittest discover -s tests/python \
  -p 'test_ndnsf_di_spec107_*.py' -q
```

Result: PASS, C++ 253/253 and Spec 107 Python 75/75. Real ONNX fixture tests
whose optional environment variables were absent remained explicitly skipped;
this does not authorize T048 or any release dimension.
