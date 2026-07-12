# T003 — Qwen Runtime and Artifact Inventory

## Frozen Model Source Available Locally

- Model: `Qwen/Qwen2.5-0.5B-Instruct`
- Hugging Face revision: `7ae557604adf67be50417f59c2c2f167def9a775`
- `model.safetensors` SHA-256:
  `fdf756fa7fcbe7404d5c60e26bff1a0c8b8aa1f72ced49e7dd0210fe288fb7fe`
- `tokenizer.json` SHA-256:
  `c0382117ea329cdf097041132f6d735924b697924d6f6fc3945713e96ce87539`
- Configuration SHA-256:
  `18e18afcaccafade98daf13a54092927904649e1dd4eba8299ab717d5d94ff45`

The snapshot is local cache state, not a committed model artifact. The candidate
manifest must bind these digests and fail if the snapshot differs.

## Existing Real Qwen Evidence

- `results/_preserved_summaries/qwen_pipeline_minindn_smoke_latest/qwen-pipeline-proof-summary.json`
  records three stages with ranges `[0,8)`, `[8,16)`, `[16,24)`, top token 38444
  matching the full model and `maxDiff=0.0` for the recorded one-token proof.
- `qwen-stage-profile-summary.json` records 128 distributed requests, p50
  460.244 ms and p95 504.56635 ms with stage compute/fetch/publish decomposition.
- `results/qwen_full_onnx_vs_transformers_decode_short/qwen-full-onnx-vs-transformers-summary.json`
  records a full-model ONNX/Transformers top-token match and 8-token decode-like
  recomputation measurements.

These artifacts prove real execution anchors; they do not prove the Spec 105
native CUDA, stage-local KV, 32-token or release-gate requirements.

## Current Data Contracts

- `llm_pipeline_lib.py` splits 24 layers across three stage artifacts.
- Python stage packages use `torch.save`; hidden tensor transport uses compressed
  NPZ via `numpy.savez_compressed`/`numpy.load(allow_pickle=False)`.
- Current ONNX wrappers accept token IDs/hidden state/position IDs and run with
  `use_cache=False`; declared per-stage past/present KV tensors are not yet part
  of the accepted native contract.
- `provider.py` holds per-process session and metadata caches; `user.py` holds a
  full-context cache for append-delta behavior. These are comparison/runtime
  scaffolds, not the final provider-local `KvStateBinding` implementation.

## NativeTracer Artifacts Are Not Qwen Compute Proof

The local `native_di_tracer/artifacts/qwen-native-tracer-*.onnx` files are tiny
synthetic graph fixtures (620-1283 bytes, feature dimensions 8/16). They are
untracked local artifacts and must never satisfy the real Qwen gate.

## Closing Work

T031-T038 must define and validate dtype, dynamic shape, full-model/stage and KV
contracts before T039-T044 integrate bounded generation. Failure at T037/T038 is
a hard re-plan boundary.
