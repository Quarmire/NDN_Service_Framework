# Contract: planner-optimization.json

`planner-optimization.json` is generated from the policy bundle. It must be
safe to archive with experiment output and read without re-running MiniNDN.

Required top-level fields:

```json
{
  "contractVersion": "di-plan-v2",
  "service": "/Inference/NativeTracer",
  "model": "/Model/NativeTracer/Qwen2.5-0.5B-Minimal/v1",
  "sourceModel": "Qwen/Qwen2.5-0.5B-Instruct",
  "modelUnchanged": true,
  "compatibility": {},
  "providerProfiles": [],
  "networkProfile": {},
  "candidates": [],
  "selectedCandidate": {},
  "selectionRule": ""
}
```

The selected candidate must be one of the candidate IDs and must be supported by
the current runtime.
