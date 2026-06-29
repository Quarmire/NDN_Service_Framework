# Feature 008: Executable DI Layout Comparison

Status: Accepted

## Goal

Turn one estimated NativeTracer planner candidate into a real executable
MiniNDN layout and compare its measured latency against the current
`shared-backbone-current` layout.

The model remains the existing smallest Qwen-derived NativeTracer ONNX artifact
set.

## Scope

- Make `single-provider-serial` executable through the existing NDNSF
  collaboration path.
- Keep `/Inference/NativeTracer` and the current four-role execution plan.
- Let one provider advertise and serve all four roles.
- Preserve the existing shared-backbone layout as the baseline.
- Record measured latency from the user driver for both layouts.

## Non-Goals

- Larger Qwen model artifacts.
- New NDNSF wire protocol.
- A new C++ model runner.
- Replacing the existing shared-backbone validation path.

## Acceptance

- `--assignment single-provider` runs successfully in local and full-network
  modes.
- Full-network `single-provider` summary reports
  `selectedCandidate=single-provider-serial`.
- Full-network `default` summary still reports
  `selectedCandidate=shared-backbone-current`.
- Both summaries report `runnerMode=qwen-onnx-native`,
  `userExecution=executed`, and `dependencyExecution=executed`.
- A comparison artifact records measured user elapsed latency for both layouts.

## Accepted Evidence

Both layouts ran with the same smallest Qwen NativeTracer artifacts and the same
MiniNDN topology.

```text
baseline assignment=default
baseline selectedCandidate=shared-backbone-current
baseline elapsedMs=236.82666099921335

alternative assignment=single-provider
alternative selectedCandidate=single-provider-serial
alternative elapsedMs=179.01252299998305

deltaMs=-57.814
alternativeOverBaselineRatio=0.7559
```

Evidence files:

```text
/tmp/ndnsf-di-layout-default/summary.txt
/tmp/ndnsf-di-layout-single-provider/summary.txt
/tmp/ndnsf-di-layout-comparison.json
/tmp/ndnsf-di-layout-comparison.csv
```

The single-provider log confirms all four roles executed locally in one
provider and the final response was produced from the local full-plan path:

```text
local_full_plan=true
role=/Backbone
role=/Head/Shard/0
role=/Head/Shard/1
role=/Merge
```
