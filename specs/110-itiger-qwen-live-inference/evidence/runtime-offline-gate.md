# Runtime release offline gate

Date: 2026-07-13

Scope: T031-T042 only. This gate validates the pinned GPU runtime source,
compatibility policy, immutable release/SIF handling, container bind policy,
secret scanning, and tamper rejection without claiming that an OCI image has
been built or that an iTiger GPU allocation has executed it.

## Commands and results

```text
python3 tools/ndnsf-di/run_spec110_offline_tests.py \
  --output results/spec110-itiger-qwen-live/release-build/offline-tests.junit.xml
  SPEC110_OFFLINE tests=53 failures=0 errors=0 skipped=0 duration=0.650200s

tests/container/itiger-qwen-live/integration/test_runtime_compatibility.sh
  RUNTIME_COMPATIBILITY_PASS cases=6

tests/container/itiger-qwen-live/integration/test_release_pipeline.sh
  RELEASE_PIPELINE_PASS
```

JUnit SHA-256:
`98f613046e7a658e9301d7ff04a13ac5fe04651897e6f1aa49d9053f3eff5ca4`.

The negative matrix covers CPU-only execution, a missing shared library, an
old driver, ONNX Runtime CPU fallback, PyTorch/ORT CUDA-major mismatch, release
record mutation, SIF mutation, partial materialization failure, and secret
findings. All fail closed with stable reason codes.

## Authority boundary

- `offlineRuntimeGate=PASS`
- `ociBuildPublished=NOT_EXECUTED`
- `itigerRuntimeProbe=NOT_EXECUTED`
- `nextTask=T043`

T043-T049 remain open until GitHub Actions publishes a digest-addressed image,
the corresponding SIF is materialized on iTiger, and the allocated-GPU probe
crosses the execution boundary.
