# Core Baseline

Executed on the Phase 1 dirty worktree:

```bash
./waf build --targets=unit-tests -j4
./build/unit-tests --log_level=test_suite
examples/run_security_regressions.sh
PYTHONPATH=pythonWrapper:NDNSF-DistributedInference:NDNSF-DistributedRepo/pythonWrapper \
  python3 -m unittest discover -s tests/python -p 'test_ndnsf_core*.py' -v
```

Additional focused Python commands covered certificate bootstrap, negative ACK,
and Targeted API tests.

Results:

- build passed in 57.891 seconds;
- 199 C++ test cases passed with no errors;
- real ONNX model tests explicitly skipped because their fixture environment
  variables were not set; they are not counted as executed model evidence;
- six security scripts passed: HELLO auth, ACK payload, custom selection,
  NAC-ABE routing, negative token handshake, and token certificate bootstrap;
- Core Python: 24 tests passed; certificate bootstrap 3, negative ACK 8, and
  Targeted API 2 passed.

The security suite temporarily used the host NFD as designed by its script and
cleaned it on exit. Final network acceptance remains MiniNDN-only.
