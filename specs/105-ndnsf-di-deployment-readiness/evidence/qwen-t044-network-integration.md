# T044 Qwen Native MiniNDN Integration

**Executed**: 2026-07-12  
**Profile**: local MiniNDN, three C++ ONNX Runtime 1.26 CPU providers  
**Physical status**: deferred to Spec 106

## Passing integration cells

The canonical `qwen-onnx-cpu-native` path starts one real
`di-native-provider` process for each contiguous Qwen stage. It does not pass
the deterministic NativeTracer runner switch. The user accepts a bounded
`--max-new-tokens` value from 1 through 32, computes a local three-stage ONNX
oracle for every token, and compares the distributed result token-by-token.

| Cell | Result | Observed evidence |
|---|---|---|
| one token | PASS | network token `2025`; local token `2025`; 3,080,992-byte final typed bundle |
| two tokens with KV | PASS | tokens `[2025, 271]`; exact oracle match; total 2,172.76 ms |
| stage-local cache transition | PASS | all three providers stored epoch 1 and reported cache-hit lookup for epoch 1 |

The second response was 658,184 bytes, versus 3,080,992 bytes for the prefill
response. Each runner emits a separate `kv-state` bundle containing only its
declared `present_key.*` and `present_value.*` tensors. The next request maps
those tensors to the corresponding `past_*` inputs and binds them to logical
session, stage, context epoch, model and plan digests, provider identity and
boot, and security epoch.

The passing raw evidence is retained locally under:

```text
/tmp/spec105-qwen-kv-export/retained/t044-2token/
```

## Preserved boundary failure for T046

The first 32-token attempt completed 17 token steps with exact oracle matches
and continuous KV hits, then the user did not receive the Stage 0 ACK for step
18 even though Stage 0 logged an ACK decision. Selection failed closed and the
request timed out. No retry or timeout increase was used. Raw evidence is
retained under:

```text
/tmp/spec105-qwen-kv-export/retained/t046-32token-ack-failure/
```

This does not reopen T044's wiring work. It is a required T046 correctness and
reliability finding that must be resolved before the 32-token cell can pass.

## Verification

```bash
NDNSF_DI_TEST_ONNX_TYPED_MODEL=/tmp/ndnsf-spec105-typed-pilot.onnx \
  ./build/unit-tests \
  --run_test='OnnxRuntimeBackendRunsDynamicPilotDtypesAndReportsDeviceEvidence,NativeTensorBundleCodecRoundTripsPilotDtypesDynamicShapesAndKvOutputs,KvStateStoreBindsReplacesEvictsAndInvalidatesOnBoot'

PYTHONPATH=NDNSF-DistributedInference:. \
  python3 tests/python/test_ndnsf_di_deployment_readiness.py

sudo -n -E env PYTHONPATH=NDNSF-DistributedInference:Experiments \
  python3 Experiments/NDNSF_DI_LlmPipeline_Minindn.py \
  --output-dir /tmp/spec105-qwen-kv-export --reuse-existing-policy \
  --runtime qwen-onnx-cpu-native --stages 3 --layers 24 \
  --prompt 'NDNSF deployment pilot' --max-new-tokens 2 \
  --warmup-requests 0 --measured-requests 1 \
  --provider-start-timeout-s 60 --timeout-ms 120000
```
