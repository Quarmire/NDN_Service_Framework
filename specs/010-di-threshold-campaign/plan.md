# Plan: DI Threshold Campaign

## Approach

Use activation padding rather than larger model artifacts. The Backbone ONNX
runner will still execute the same small Qwen-derived stage, then optionally add
a dummy tensor to the encoded output bundle. Downstream Head roles select the
real `features` tensor from the bundle and ignore the padding tensor.

## Implementation Steps

1. Add `outputBundlePadBytes` metadata support in `OnnxRuntimeModelRunner` for
   encoded output bundles.
2. Add `--activation-pad-bytes` to `plan_tracer.py`; patch only Backbone
   dependencies and Backbone artifact metadata in the generated policy bundle.
3. Add `--activation-pad-bytes` to `NDNSF_DI_NativeTracer_Minindn.py` and record
   the value in `summary.json`.
4. Add `--activation-pad-bytes-list` to `run_layout_campaign.py`; output either
   the current flat summary for one value or a threshold summary for many values.
5. Validate with syntax/build checks, one local/full-network smoke, and a small
   multi-size threshold smoke.

## Candidate Values

Start with a smoke set: `0,65536`. If stable, continue with a broader set such
as `0,65536,262144,1048576` and 3-5 runs per layout.
