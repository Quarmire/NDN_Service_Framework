# Quickstart: NDNSF-DI iTiger Qwen Scaling

This validates the intended workflow; it does not authorize live submissions. Canonical commands are repository-local, model downloads are explicit tasks, and acceptance failures are never retried automatically.

## 1. Create output roots and run read-only discovery

```bash
mkdir -p results/spec109-itiger-qwen/{discovery,validation}
uofm-vpn-status
ssh -o BatchMode=yes itiger 'hostname; whoami'
tools/ndnsf-di/ndnsf-di-qwen discover \
  --deployment-profile packaging/ndnsf-di-container/adapters/slurm-apptainer/profiles/itiger-gpu.json \
  --output results/spec109-itiger-qwen/discovery
```

Expected: identity, target-path quota, GRES, Apptainer, and egress facts. Shared `df` alone is not admission PASS.

## 2. Seal source and exact predecessors

```bash
tools/ndnsf-di/ndnsf-di-qwen snapshot-source \
  --repo . --output results/spec109-itiger-qwen/source-snapshot.json
tools/ndnsf-di/ndnsf-di-qwen validate-predecessors \
  --manifest results/spec109-itiger-qwen/predecessor-gate.json
```

The predecessor manifest must enumerate Spec 107 `T027,T028-T038` and Spec 108 `T091-T102`. Dirty source is acceptable only as `SEALED_DIRTY` with resolvable diff/untracked archive digests.

## 3. Validate contracts before transfer

```bash
tools/ndnsf-di/ndnsf-di-qwen validate \
  --schema profile --input experiment/0.5b-profile.json --semantic
tools/ndnsf-di/ndnsf-di-qwen validate \
  --schema matrix --input experiment/scale-matrix.json --semantic
```

Expected: deterministic PASS including source, predecessor, deployment, workload, keyed-cell/run, and gate-scope checks. Validation submits zero jobs.

## 4. Durable and scratch layout

```text
/project/$USER/ndnsf-di/{src,images,models,cache,manifests,evidence}
/tmp/$USER/ndnsf-di/$SLURM_JOB_ID
```

Inside each job, bind project data read-only where possible and set `HF_HOME`, `APPTAINER_CACHEDIR`, `TMPDIR`, and `APPTAINER_TMPDIR` to the admitted project/scratch locations. Bulk data never enters the workstation or `/home`.

## 5. Execute the 0.5B MVP in authority order

1. Storage admission and exactly-once sealed model transfer.
2. Diagnostic smoke under a diagnostic identity.
3. Lock source, predecessor, Spec 108 deployment, model, workload, stage, and run-order digests.
4. Full-model Transformers/PyTorch 1/2/32-token oracle.
5. Export graph/tensor/KV validation and preregistered numerical-equivalence checkpoints.
6. Matched staged ONNX Runtime baseline with node-level execution-provider profile.
7. MiniNDN security/generation regression.
8. iTiger NDNSF-DI 1/2/32-token correctness.
9. Three randomized/interleaved baseline/candidate performance pairs, each with excluded warmup and a 60-second window.
10. Promote original evidence and finalize each keyed cell without retrying failures.

The overhead calculation is always `NDNSF-DI candidate - matched staged ONNX baseline`. Full-model oracle timing is excluded.

## 6. Statistical validity

Report completed counts and confidence intervals. p50/p95/p99 require 20/100/1000 observations; otherwise record `UNAVAILABLE_INSUFFICIENT_N`. Three repetitions remain separate. Reproduction is exact-token plus confidence-interval engineering equivalence, not point-estimate tuning.

## 7. Advance sizes with scoped gates

Schedule `0.5B -> 1.5B -> 3B -> 7B -> 14B`, but a model-local license/fit/export failure blocks only that model. A source, predecessor, deployment, security, shared-storage, or evidence-validator failure is systemic and blocks its dependants. Every block records scope, ID, and digest.

32B/72B require dedicated live admission before transfer. One-node/multi-GPU precedes multi-node; multi-node remains deferred until the independent NFD network gate passes.

## 8. Claim and cleanup boundaries

- Oracle: correctness/capacity only.
- Staged ONNX baseline: performance reference only.
- Secured NDNSF-DI path plus node-level CUDA proof: candidate authority.
- Three matched pairs: candidate performance evidence.
- Physical production: always `DEFERRED`, owner Spec 106.

Cleanup starts with dry-run and rejects source snapshots, identities, active jobs, accepted evidence, referenced models/exports, and current/prior releases.
