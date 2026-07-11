# Final Core And Security Verification

**Implementation commit**: `3918c98`

| Check | Result |
|---|---|
| `./waf build --targets=unit-tests,di-native-provider -j$(nproc)` | PASS |
| `./build/unit-tests --log_level=message` | 210 cases, no errors; optional ONNX fixtures skipped when environment variables were absent |
| `python3 -m unittest discover -s tests/python -p 'test_ndnsf_core*.py' -q` | 29 passed |
| Core lease Python binding tests | 3 passed |
| `examples/run_security_regressions.sh` | PASS: HELLO auth, ACK payload, custom selection, NAC-ABE routing, token negative, certificate bootstrap |

The security run used a temporary host NFD only for these existing regression
scripts. It was stopped after the run. No permission, NAC-ABE, token, replay,
or certificate-bootstrap semantics were weakened by 085.

