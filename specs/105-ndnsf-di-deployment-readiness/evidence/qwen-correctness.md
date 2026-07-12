# Qwen Pilot Correctness

**Executed**: 2026-07-12  
**Task**: T046  
**Verdict**: PASS, with one preserved transient ACK-delivery failure

## Frozen identity

- Model/tokenizer: `Qwen/Qwen2.5-0.5B-Instruct`
- Prompt: `NDNSF deployment pilot`
- Generation: batch one, greedy, maximum 32 new tokens
- Distributed runtime: C++ ONNX Runtime 1.26.0 CPU
- Observed native manifest digest:
  `sha256:7AEAD238C83F5E57CDB3491FD215E5B00414234312BB21BBEB3A3825D40D9486`
- Observed native plan digest:
  `sha256:688B46AF40300C56D206B12C3E704B62636B572F23F28D75CD824CD58C4B40C2`
- Stage artifact digests:
  - Stage 0: `8d4d8716b499f375634087f56e9011e6c65b70bd9c370fa9d0967055efd74006`
  - Stage 1: `154d56424b46f527c9fbf9ed59877cd92f42ad331a0f61e6bb026a404ab70bf0`
  - Stage 2: `9349b111492efc55c0e6c6586c9ebe087902992be2c85923d2f16d5a3d1ee05c`

## Token cells

| Output bound | Distributed tokens | Frozen baseline | Result |
|---:|---|---|---|
| 1 | `[2025]` | `[2025]` | PASS |
| 2 | `[2025,271]` | `[2025,271]` | PASS |
| 32 | `[2025,271,785,5055,9965,5007,320,2448,37,8,702,7228,264,17708,2025,311,10517,264,501,79528,49601,3922,315,4237,24231,7798,311,1824,3412,304,279,5671]` | exact same sequence | PASS |

The successful 32-token MiniNDN request took 25,320.02 ms. Every generated
token was compared immediately with the local three-stage ONNX oracle; no
tolerance or post-hoc prefix comparison was used.

## Admission and cache cells

| Cell | Expected | Observed | Result |
|---|---|---|---|
| 512 input tokens | admit | Python contract admits | PASS |
| 513 input tokens | reject | bounded request validation rejects | PASS |
| 32 output tokens | admit | network generation completes 32/32 | PASS |
| 33 output tokens | reject | CLI and request validation reject | PASS |
| full-context, empty cache | rebuild | prefill stores epoch 1 on all three providers | PASS |
| cache hit | use stage-local KV | 31 successive decode epochs hit on all three providers | PASS |
| delta-only, empty cache | terminal failure | all three providers report `CACHE_MISS_FULL_CONTEXT_REQUIRED` | PASS |
| wrong binding / provider boot | reject | C++ binding/store tests reject and boot change invalidates | PASS |

The delta-only network negative used a 5-second observation timeout. It did not
fall back to full context, execute ONNX, or publish a successful response. Raw
logs are retained at:

```text
/tmp/spec105-qwen-kv-export/retained/t046-delta-only-miss/
```

## Retained negative result

An earlier 32-token attempt completed 17 correct tokens and then the user did
not receive Stage 0's ACK, despite Stage 0 logging its ACK decision. Selection
failed closed and the request timed out. The raw failure remains at:

```text
/tmp/spec105-qwen-kv-export/retained/t046-32token-ack-failure/
```

A 20-token TRACE diagnostic then passed 20/20, and the unchanged INFO,
1,500-ms ACK configuration subsequently passed 32/32. Therefore this is not a
deterministic KV or token-capacity boundary, but it is a real reliability
observation that must remain visible in T047 completion-rate evidence.

## Verification commands

```bash
PYTHONPATH=NDNSF-DistributedInference:. \
  python3 tests/python/test_ndnsf_di_deployment_readiness.py

NDNSF_DI_TEST_ONNX_TYPED_MODEL=/tmp/ndnsf-spec105-typed-pilot.onnx \
  ./build/unit-tests \
  --run_test='OnnxRuntimeBackendRunsDynamicPilotDtypesAndReportsDeviceEvidence,NativeTensorBundleCodecRoundTripsPilotDtypesDynamicShapesAndKvOutputs,KvStateStoreBindsReplacesEvictsAndInvalidatesOnBoot'

sudo -n -E env PYTHONPATH=NDNSF-DistributedInference:Experiments \
  python3 Experiments/NDNSF_DI_LlmPipeline_Minindn.py \
  --output-dir /tmp/spec105-qwen-kv-export --reuse-existing-policy \
  --runtime qwen-onnx-cpu-native --stages 3 --layers 24 \
  --prompt 'NDNSF deployment pilot' --max-new-tokens 32 \
  --warmup-requests 0 --measured-requests 1 \
  --provider-start-timeout-s 60 --timeout-ms 120000
```
