# Data Model: DI Optimization Evidence

## OptimizationEvidence

- `contractVersion`: must be `di-plan-v2`.
- `service`: NDNSF service name.
- `model`: NativeTracer model name.
- `sourceModel`: source checkpoint used for the tiny ONNX artifacts.
- `modelUnchanged`: true when the current minimal Qwen artifacts are preserved.
- `compatibility`: existing plan metadata needed by current C++ smoke tests.
- `providerProfiles`: role/provider/node compute and queue assumptions.
- `networkProfile`: default RTT/bandwidth plus provider-pair overrides.
- `candidates`: scored candidate layouts.
- `selectedCandidate`: candidate selected for current execution.
- `selectionRule`: deterministic explanation of why it was selected.

## Candidate

- `id`: stable candidate identifier.
- `label`: human-readable name.
- `supportedByCurrentRuntime`: whether the current NativeTracer runtime can run
  this candidate without changing the executable graph.
- `rolePlacement`: role-to-provider mapping.
- `dependencies`: dependency edges and expected bytes.
- `cost`: compute, transfer, queue, and total latency estimates.
- `reason`: planner explanation.
