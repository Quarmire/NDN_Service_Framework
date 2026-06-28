# Feature 006: Qwen ONNX NativeTracer

Status: Accepted

## Goal

Replace the full-network NativeTracer deterministic runner with real C++ ONNX
Runtime execution while preserving the same MiniNDN, ServiceController,
ACK/Selection/Response, and `NdnsfCollaborationDependencyIo` dependency path.

## Scope

- Generate minimal ONNX artifacts for the existing NativeTracer role graph:
  `/Backbone`, `/Head/Shard/0`, `/Head/Shard/1`, and `/Merge`.
- Use weights sliced from `Qwen/Qwen2.5-0.5B-Instruct`, the smallest Qwen
  checkpoint already cached in the local environment.
- Keep the current C++ native runner contract: float32 tensor-bundle inputs and
  float32 ONNX outputs.
- Run the same full-network MiniNDN harness without
  `--tracer-deterministic-runner`.

## Non-Goals

- Full Qwen tokenizer execution.
- Decoder-layer pipeline splitting.
- int64 `input_ids` / `attention_mask` support in the C++ runner.
- KV-cache or autoregressive decode.

## Acceptance Evidence

Canonical command:

```bash
sudo -n python3 Experiments/NDNSF_DI_NativeTracer_Minindn.py \
  --full-network \
  --core-trace \
  --out /tmp/ndnsf-di-qwen-full-network \
  --assignment default \
  --provider-check-timeout 45
```

Expected `summary.txt` lines:

```text
status=SUCCESS
runnerMode=qwen-onnx-native
localExecution=executed
securityBootstrap=executed
userExecution=executed
dependencyExecution=executed
```

## Follow-Up

The next gate should use true Qwen token/pipeline artifacts instead of the
float32 NativeTracer adapter. That requires richer runtime support for token
inputs and hidden-state payloads.
