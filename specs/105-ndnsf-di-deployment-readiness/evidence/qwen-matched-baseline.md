# Matched Qwen Single-Node Baseline

**Executed**: 2026-07-12  
**Task**: T045  
**Result**: PASS

The baseline reads the same three-stage service manifest and executes the same
ONNX files, tokenizer, prompt, batch-one greedy generation, 32-token limit,
CPU execution provider, and INFO logging profile as the MiniNDN candidate. KV
state is kept locally between decode steps. One warmup generation is excluded
from three measured generations.

## Frozen identity

- Model: `Qwen/Qwen2.5-0.5B-Instruct`
- Prompt: `NDNSF deployment pilot` (5 input tokens)
- Generation: greedy, batch 1, 32 new tokens
- Backend: ONNX Runtime CPU
- Python oracle runtime: ONNX Runtime 1.19.2, `CPUExecutionProvider`
- Service manifest SHA-256:
  `d38588024032c493c623b98d16c7425d8a543fbe7699fe09109b3aa937526780`
- Stage artifact SHA-256 values:
  - Stage 0: `8d4d8716b499f375634087f56e9011e6c65b70bd9c370fa9d0967055efd74006`
  - Stage 1: `154d56424b46f527c9fbf9ed59877cd92f42ad331a0f61e6bb026a404ab70bf0`
  - Stage 2: `9349b111492efc55c0e6c6586c9ebe087902992be2c85923d2f16d5a3d1ee05c`

All three measured generations produced the identical token sequence:

```text
[2025,271,785,5055,9965,5007,320,2448,37,8,702,7228,264,17708,2025,311,10517,264,501,79528,49601,3922,315,4237,24231,7798,311,1824,3412,304,279,5671]
```

## Measurements

| Metric | Count | p50 | p95 | Mean |
|---|---:|---:|---:|---:|
| 32-token total | 3 | 6,787.63 ms | 6,854.20 ms | 6,660.73 ms |
| TTFT | 3 | 177.58 ms | 247.26 ms | 195.80 ms |
| Inter-token | 93 | 204.35 ms | 281.91 ms | 208.55 ms |

The local result artifact is retained at:

```text
results/spec105-qwen-baseline-20260712-0436/qwen-matched-single-node-summary.json
```

`results/` is intentionally ignored, so this document retains the exact
identity, tokens, and summary values needed to audit the later comparison.
The distributed C++ providers report ONNX Runtime 1.26.0; both paths use the
same ONNX Runtime CPU backend and identical artifacts, while the runtime-version
difference remains explicit rather than being described as bit-for-bit runtime
identity.

## Command

```bash
python3 Experiments/NDNSF_DI_QwenFull_OnnxVsTransformers_LocalBenchmark.py \
  --model Qwen/Qwen2.5-0.5B-Instruct \
  --prompt 'NDNSF deployment pilot' \
  --output-dir results/spec105-qwen-baseline-20260712-0436 \
  --qwen-service-manifest \
    /tmp/spec105-qwen-kv-export/qwen-onnx-service-manifest.json \
  --max-new-tokens 32 --warmup 1 --iterations 3 --intra-op-threads 0
```
