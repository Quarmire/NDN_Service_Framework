# iTiger Qwen evidence and authority

Qwen evidence has four independent planes:

1. **Standalone oracle** — full-model deterministic tokens and capacity only.
2. **Artifact correctness** — exported graph, tensor, KV, logit, and exact-token
   equivalence against the oracle.
3. **Matched staged baseline** — the same artifacts, ONNX Runtime settings,
   workload, cache, stage/GPU map, warmup, logging, timeout, and 60-second window
   as the candidate, without NDNSF networking/security/orchestration.
4. **NDNSF-DI candidate** — normal permissions, NAC-ABE, UserToken,
   ProviderToken, replay protection, provider permission, dependency dataflow,
   node-level CUDA assignments, GPU UUIDs, and final exact tokens.

Only plane 4 can establish candidate correctness. Candidate overhead is
candidate minus plane 3; Transformers timing is never used for overhead.
Physical production is always deferred to Spec 106.

## Submission lifecycle

`oracle`, `staged-baseline`, and `candidate` support render, explicit submit,
status, wait, cancel, and evidence operations. Rendering never submits.
Acceptance cells are submitted exactly once under a locked identity; the first
failure and original exit code remain evidence.

```bash
tools/ndnsf-di/ndnsf-di-qwen candidate \
  --job-profile candidate.json --output candidate.sbatch \
  --ledger candidate-ledger.json
```

Every live cell must first pass source, exact predecessor, deployment/SIF,
storage, and semantic gates. A systemic gate blocks all dependants; a
model-local or placement-local gate never censors unrelated sizes.

## Backend and metrics

GPU PASS requires every model node to report `CUDAExecutionProvider`, no CPU
fallback, and a GPU UUID drawn from the Slurm allocation. ORT profiles,
stage/artifact digests, runtime version, and fallback state are promoted with
the run. Missing/incomplete profiles fail closed.

Warmup is excluded. Each accepted performance size retains three original
60-second repetitions separately. p50/p95/p99 require 20/100/1000 observations;
otherwise the tail is `UNAVAILABLE_INSUFFICIENT_N`. Confidence intervals never
rescue an invalid cell.

## Current Spec 109 outcome

Model staging passed only for the sealed 0.5B source. All 105 scaling cells are
terminally blocked because the exact Spec 107/108 GPU predecessors are
incomplete. The MiniNDN fake LLM pipeline also retained a `local deadline`
preflight failure. No GPU inference job ran; reproduction is inconclusive;
physical production remains deferred.
