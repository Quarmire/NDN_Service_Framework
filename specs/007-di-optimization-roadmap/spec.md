# Feature 007: DI Optimization Evidence Roadmap

Status: Accepted

## Goal

Move NDNSF-DI beyond "real network plus real NativeTracer ONNX execution" by
adding explicit planner optimization evidence: provider/network profiles,
candidate layouts, cost estimates, selected-candidate reasoning, and validation
outputs.

The NativeTracer model remains the existing smallest Qwen-derived artifact set.
This feature does not replace the model, enlarge the model, or change the ONNX
runtime contract.

## Scope

- Preserve the current `/Inference/NativeTracer` full-network execution path.
- Preserve the current Qwen-derived ONNX artifacts:
  `qwen-native-tracer-backbone.onnx`, `qwen-native-tracer-head0.onnx`,
  `qwen-native-tracer-head1.onnx`, and `qwen-native-tracer-merge.onnx`.
- Add a `di-plan-v2` optimization evidence contract alongside the existing
  compatibility metadata.
- Generate candidate layouts for the current four-role graph.
- Score candidates with a simple compute, transfer, and queue/load cost model.
- Attach the selected candidate and optimization evidence path to policy
  generation and MiniNDN harness summaries.

## Non-Goals

- Full Qwen tokenizer or autoregressive decoder execution.
- Larger Qwen artifacts.
- New NDNSF wire protocol.
- Replacing the current executable NativeTracer role graph.

## Acceptance

P1-P9 are complete when:

- Policy generation emits `planner-optimization.json` and
  `planner-optimization.csv`.
- The evidence declares `contractVersion=di-plan-v2`.
- The evidence states the source model is still
  `Qwen/Qwen2.5-0.5B-Instruct` and the NativeTracer minimal model is unchanged.
- At least five candidate layouts are listed and scored.
- The selected candidate is explained by a deterministic selection rule.
- The MiniNDN harness summary includes the optimization evidence path and
  selected candidate.
- Python validation passes for the new optimizer, plan tracer, and harness.

## Accepted Evidence

Validation completed with the existing smallest Qwen NativeTracer artifacts.

```text
contractVersion=di-plan-v2
sourceModel=Qwen/Qwen2.5-0.5B-Instruct
modelUnchanged=true
candidateCount=5
selectedCandidate=shared-backbone-current
runnerMode=qwen-onnx-native
localExecution=executed
securityBootstrap=executed
userExecution=executed
dependencyExecution=executed
```

Canonical full-network output:

```text
/tmp/ndnsf-di-optimization-full-network/summary.txt
```
