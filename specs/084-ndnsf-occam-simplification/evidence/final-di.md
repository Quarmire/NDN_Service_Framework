# Final Distributed Inference Acceptance

The final coordinator-off network path used the Spec 090 typed-only command:

```bash
sudo -n timeout 300s python3 Experiments/NDNSF_DI_NativeTracer_Minindn.py \
  --runtime-profile examples/di-native-tracer.runtime.json \
  --out results/spec090-typed-envelope/typed-only \
  --requests 1 --concurrency 1 --provider-check-timeout 60 \
  --no-local-execution-only --full-network
```

Result: 2/2 Qwen ONNX NativeTracer requests completed, all 8 dependency
exchanges completed, p50 was 200.185 ms, p95 was 357.711 ms, and measured
throughput was 3.579 RPS. The fixture used two users, no coordinator, typed
capability ACKs, and real NativeTracer execution. It is a network acceptance
smoke, not a 60-second capacity claim.

Child 087 supplies the independent policy decision: coordinator-off completed
2/2 at p50 324.35 ms and p95 332.64 ms, while ten matched advisory pairs failed
the frozen retention gate. The advisory implementation was therefore deleted
instead of being defended as an improvement.
