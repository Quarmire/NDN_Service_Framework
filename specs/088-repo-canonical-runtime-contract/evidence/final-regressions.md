# Final Regression Evidence

Date: 2026-07-11

| Gate | Result |
|---|---|
| `./waf configure --with-tests --with-examples` | PASS |
| `./waf build --targets=unit-tests -j$(nproc)` | PASS |
| `./build/unit-tests --log_level=message` | 214/214 PASS; environment-gated ONNX fixtures skipped |
| Full DI example target build | PASS, including NativeTracer provider/schema/manifest/session binaries |
| `python3 -m unittest discover -s tests/python -p 'test_*.py'` | 343/343 PASS, 1 environment-dependent skip |
| `examples/run_security_regressions.sh` | PASS, all six security suites |

The Python run also verifies that deployment readiness/status helpers now live
under the DI package and that Repo clients/examples import the canonical
`py_repoclient` adapter instead of restoring Core or DI default Repo exports.
