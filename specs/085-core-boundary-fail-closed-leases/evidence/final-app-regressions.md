# Final Application Regression Verification

**Implementation commit**: `3918c98`

Focused Python results:

| Area | Tests | Result |
|---|---:|---|
| DI execution lease codec/transaction/restart/stress/integration | 19 | PASS |
| DI scenarios and runtime-v1 | 64 | PASS |
| Core/application envelope and import boundary | 12 | PASS |
| Repo HA | 47 | PASS |
| Tk GUI and widget preflight | 29 | PASS |

The final MiniNDN campaign below executed the Qwen NativeTracer ONNX path with
real NDNSF control and dependency exchange, so it supersedes a separate local
Qwen smoke. The launcher reported `runnerMode=qwen-onnx-native`,
`userExecution=executed`, and `dependencyExecution=executed` in all accepted
runs.

