# Execution Evidence Contract

Provider readiness carries `servicePayload.executionEvidence` using the existing
typed provider-capability envelope.

```json
{
  "schema": "ndnsf-di-execution-evidence-v1",
  "providerName": "/NDNSF-DI/Tracer/provider/llm-2gb",
  "providerBootId": "uuid",
  "evidenceEpoch": 1,
  "runnerKind": "onnxruntime-cuda",
  "realCompute": true,
  "device": {"kind": "cuda", "id": "GPU-uuid"},
  "runtimeVersion": "onnxruntime=...;cuda=...",
  "modelDigest": "sha256:...",
  "planDigest": "sha256:...",
  "artifactDigests": {"/LLM/Stage/0": "sha256:..."},
  "roles": ["/LLM/Stage/0"],
  "createdAtMs": 0
}
```

Rules:

- `realCompute` is produced by the initialized runner factory, not CLI intent.
- `synthetic-delay`, `wiring-only`, and `unknown` always imply false.
- summary gates compare provider identity, boot, plan, model and artifact digests.
- mixed/absent evidence yields `invalid-evidence` and release BLOCK.
- no token, key, prompt, tensor, or payload bytes appear in evidence.
