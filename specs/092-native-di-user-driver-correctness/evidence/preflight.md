# Preflight

Source evidence: `specs/091-native-di-offered-load-baseline/evidence/`.

Fixed validation controls:

```text
runtime profile: examples/di-native-tracer.runtime.json
topology: Experiments/Topology/AI_Lab.conf
model path: real Qwen ONNX NativeTracer
assignment: proportional 2/4/8 GB
target: 1 RPS
window: 60 seconds
requests: 60
concurrency: 4
```

Only driver mode and output path differ between matched post-fix treatments.
