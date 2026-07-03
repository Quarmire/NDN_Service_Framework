# Implementation Plan: Runtime Doctor and Usability Harness

## Design

The first usability layer is deliberately outside the C++ hot path. A Python
stdlib-only CLI reads a profile, checks the built binaries and control files,
generates missing bootstrap tokens when requested, writes a resolved config,
and emits JSONL events that later GUI, MiniNDN, and CI tools can consume.

The profile is JSON for now because JSON is available without dependencies in
fresh Ubuntu and MiniNDN environments. YAML can be added later as a thin parser
layer.

## Files

- `tools/ndnsf_runtime.py`: profile doctor and structured event writer.
- `examples/hello.runtime.json`: canonical HELLO runtime profile.
- `examples/common_regression.sh`: common shell helpers for regressions.
- `tests/python/test_ndnsf_runtime_doctor.py`: unit coverage for token
  generation, resolved config, and event output.

## Validation

Run:

```bash
python3 tests/python/test_ndnsf_runtime_doctor.py
python3 tools/ndnsf_runtime.py doctor --profile examples/hello.runtime.json --fix
examples/run_token_certificate_bootstrap_regression.sh
examples/run_security_regressions.sh
```
