# Spec 107 Foundation Verification

**Date:** 2026-07-12
**Scope:** T001–T019
**Verdict:** `PASS`

## Commands and results

### Build

```bash
./waf build -j4
```

Result: PASS, 267 build steps, including `unit-tests`,
`di-native-provider`, and the native DI smoke targets. The first attempt exposed
an existing build-glob defect: `tests/sanitizer/dependency-wait-scheduler-sanitizer.cpp`
was linked into `unit-tests` and supplied a second `main`. `tests/wscript` now
excludes `sanitizer/**` from the unit-test glob; the sanitizer source remains
available for its documented standalone ASan/UBSan command.

### Full C++ unit suite

```bash
./build/unit-tests --log_level=message
```

Result: PASS, 250/250. Environment-dependent real-ONNX and generated-plan
smokes reported their existing explicit skips because their optional environment
variables were unset; the new codec/state/queue tests executed.

### Spec 107 Python contracts

```bash
PYTHONPATH=tests/python:NDNSF-DistributedInference:. \
  python3 -m unittest discover -s tests/python \
  -p 'test_ndnsf_di_spec107_*.py' -v
```

Result: PASS, 47/47. Coverage includes frozen lineage, mutation denial,
candidate/campaign identity, exclusive preflight, artifact materialization,
stable timing reconciliation, live-fault schemas, evidence binding, and
release-input tamper rejection.

### Existing deployment-readiness contracts

```bash
PYTHONPATH=tests/python:NDNSF-DistributedInference:. \
  python3 tests/python/test_ndnsf_di_deployment_readiness.py -v
```

Result: PASS, 21/21. Existing Qwen bounds, generation scheduler, recovery,
release gate, runtime CLI, and packaging contracts remain intact.

## Boundary confirmation

- No MiniNDN, model export, performance, fault, canary, or soak campaign ran.
- No new Core TLV, V2 name, permission path, token path, or authorization bypass
  was added. `QwenGenerationSession` is an application-layer JSON codec/state
  contract compiled only with the DI runtime and native example targets.
- Spec 105 locked files retain all four recorded SHA-256 values.
- Diagnostic and release-input models reject payload, token, tensor, KV, secret,
  mixed-candidate, physical-PASS, and diagnostic-as-acceptance evidence.
