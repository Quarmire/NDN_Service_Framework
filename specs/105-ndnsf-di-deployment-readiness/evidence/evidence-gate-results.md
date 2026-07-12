# Execution-Evidence Gate Results

**Executed**: 2026-07-12  
**Profile**: local MiniNDN candidate, real ONNX Runtime CPU  
**Physical status**: always `DEFERRED` to Spec 106

## Executed Cells

| Cell | Observed classification | Candidate gate | Expected | Result |
|---|---|---|---|---|
| real CPU ONNX | `onnxruntime-cpu` | PASS | PASS | PASS |
| synthetic delay | `synthetic-delay` | BLOCK | BLOCK | PASS |
| CUDA requested but unavailable | `invalid-evidence` | BLOCK | BLOCK | PASS |
| mixed CPU/synthetic | `invalid-evidence` | BLOCK | BLOCK | PASS |
| missing evidence | `invalid-evidence` | BLOCK | BLOCK | PASS |
| same role, mismatched artifact digest | `invalid-evidence` | BLOCK | BLOCK | PASS |

The real CPU cell initialized and executed an ONNX Runtime 1.26.0 session with
dynamic sequence dimensions and Int64, Bool, and Float16 tensors before checking
backend/device evidence. The unavailable-CUDA cell exercised the actual linked
runtime provider inventory and verified that runner creation throws while CPU
fallback is disabled. No CUDA evidence was fabricated.

## Commands

```bash
NDNSF_DI_TEST_ONNX_TYPED_MODEL=/tmp/ndnsf-spec105-typed-pilot.onnx \
  ./build/unit-tests \
  --run_test='OnnxRuntimeProviderSelectionRequiresExplicitCpuFallback,OnnxRuntimeBackendRunsDynamicPilotDtypesAndReportsDeviceEvidence,NativeTensorBundleCodecRoundTripsPilotDtypesDynamicShapesAndKvOutputs,NativeTensorBundleCodecRejectsMalformedShapesTypesAndPayloadSizes'

PYTHONPATH=NDNSF-DistributedInference:. \
  python3 tests/python/test_ndnsf_di_deployment_readiness.py
```

The six release-gate inputs used distinct release IDs and otherwise identical
PASS dimension fixtures. Output was:

```text
real-cpu          onnxruntime-cpu  PASS   DEFERRED
synthetic         synthetic-delay BLOCK  DEFERRED
unavailable-cuda  invalid-evidence BLOCK  DEFERRED
mixed             invalid-evidence BLOCK  DEFERRED
missing           invalid-evidence BLOCK  DEFERRED
digest-mismatch   invalid-evidence BLOCK  DEFERRED
```

This matrix closes only evidence integrity. Correctness, performance, recovery,
security, and operations dimensions remain subject to their later Spec 105
tasks.
