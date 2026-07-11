# Final Core And Security Evidence

**Date**: 2026-07-11

| Check | Result |
|---|---|
| `./waf configure --with-tests` | PASS, Boost 1.71 and required libraries found |
| `./waf build -j$(nproc)` | PASS, 72 full targets from a clean build directory |
| `./waf build --targets=unit-tests -j$(nproc)` | PASS |
| `./build/unit-tests --log_level=message` | PASS, 215 cases |
| Core Python discovery | PASS, 29 cases |
| `pythonWrapper/setup.py build_ext --inplace` | PASS |
| `examples/run_security_regressions.sh` | PASS, all six security suites |

The aggregate covered HELLO authorization, ACK payload, selective custom
selection, NAC-ABE routing, token/replay negatives, and certificate bootstrap.
The complete C++ suite additionally covered normal, Targeted bootstrap/fast
path, collaboration, policy epoch, malformed request, and exact authorization
behavior. No security assertion was weakened.

The first aggregate attempt exposed stale fixture strings that expected the old
provider-only permission log. Runtime execution succeeded. Fixtures were
updated to match canonical `providerServiceName=/provider/service`; the original
aggregate then passed.
