# MiniNDN Acceptance

Both runs used the real Qwen ONNX NativeTracer path and the same AI Lab
topology. The profile resolves to two requests despite the command-line smoke
override, so each result completed two network requests.

```bash
sudo -n timeout 300s python3 Experiments/NDNSF_DI_NativeTracer_Minindn.py \
  --runtime-profile examples/di-native-tracer.runtime.json \
  --out results/spec090-typed-envelope/typed-only \
  --requests 1 --concurrency 1 --provider-check-timeout 60 \
  --no-local-execution-only --full-network

sudo -n env NDNSF_ACK_COMPATIBILITY_MODE=mixed timeout 300s \
  python3 Experiments/NDNSF_DI_NativeTracer_Minindn.py \
  --runtime-profile examples/di-native-tracer.runtime.json \
  --out results/spec090-typed-envelope/mixed-reader \
  --requests 1 --concurrency 1 --provider-check-timeout 60 \
  --no-local-execution-only --full-network
```

| Reader | Requests | Typed | Legacy | Dual conflicts | Malformed | Unknown | Result |
|---|---:|---:|---:|---:|---:|---:|---|
| typed-only | 2/2 | 9 | 0 | 0 | 0 | 0 | PASS |
| mixed | 2/2 | 9 | 0 | 0 | 0 | 0 | PASS |

Both runs executed security bootstrap, user execution, and dependency exchange.
Each reported eight successful dependency-object events and nine v2 capability
envelopes. Raw local results remain under `results/spec090-typed-envelope/`.

