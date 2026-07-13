# Data Model: NDNSF-DI iTiger Qwen Scaling

## SourceSnapshot

| Field | Rule |
|---|---|
| `headCommit`, `treeDigest`, `capturedAt` | required and immutable |
| `worktreeState` | `CLEAN` or `SEALED_DIRTY` |
| `binaryDiffDigest`, `untrackedManifestDigest`, `untrackedArchiveDigest` | null only for `CLEAN`; all required for `SEALED_DIRTY` |
| `includedPaths[]`, `excludedPaths[]` | explicit; secrets/results/model weights excluded |
| `snapshotDigest` | digest over the normalized record and referenced bytes |

Only a reconstructable snapshot may create a campaign. A later source change creates a new campaign/candidate.

## PredecessorGate

Contains one keyed entry per required predecessor: Spec 107 `T027,T028-T038` and Spec 108 `T091-T102`. Each entry binds `specId`, `taskId`, required/observed status, schema version, artifact path, artifact digest, candidate/release identity, and validation time. PASS requires every exact key once; ranges and prose references are invalid.

## DeploymentBinding

References the Spec 108 deployment profile and release by SHA-256. It records the resolved account/QOS/partition/CPU/memory/walltime/GRES/node/image values for evidence, but Spec 108 remains their authority. Spec 109 adds only stage-to-logical-GPU mappings and experiment constraints.

## ModelRegistryEntry

| Field | Rule |
|---|---|
| `modelId`, `family`, `sizeClass` | Qwen2.5-Instruct ladder identity |
| `repository`, `revision` | immutable, never branch-only |
| `tokenizerDigest`, `licenseClass`, `licenseDigest` | required before execution |
| `files[]` | relative path, bytes, SHA-256/LFS identity |
| `sourceBytes`, `state` | `PLANNED/STAGING/VERIFIED/SEALED/BLOCKED` |
| `projectPath` | content-addressed project path; never `/home` or local |

Transitions: `PLANNED -> STAGING -> VERIFIED -> SEALED`; validation failure enters `BLOCKED`. Acceptance consumes only `SEALED`.

## StorageAdmissionRecord

Records live target path, quota source/value, used/shared capacity, projected source/export/cache/evidence bytes, reserve, peak, protected paths, and `PASS/BLOCKED`. Shared capacity alone cannot yield PASS.

## WorkloadProfile

Immutable fields: prompt-set digest and ordered prompt IDs, context/output bounds, greedy decode, arrival mode, target RPS, max in-flight, request timeout, cache state, warmup count, measurement duration, repetitions, logging profile, and randomized/interleaved run-order seed. Changing any field creates a new workload digest.

## ExperimentCandidate

Digest over `SourceSnapshot`, `PredecessorGate`, `DeploymentBinding`, model/tokenizer/license, dtype/quantization, exporter/opset, stage artifacts/partition, runtime/session options, workload profile, and fallback policy. Immutable once any acceptance cell is submitted.

## StageArtifactSet and NumericalEquivalence

`StageArtifactSet` binds exporter/version/opset/dtype, graph/external files and hashes, stage roles, input/output/KV tensor contracts, partition map, and total bytes. `NumericalEquivalence` stores exact input/output token arrays/digests plus per-checkpoint tensor name, shape, dtype, `rtol`, `atol`, max absolute/relative error, top-1 token, and logit margin. Artifact PASS requires every preregistered checkpoint PASS.

## MatchedBaselinePair

Links one `staged-onnx-baseline` cell and one `ndnsf-di-performance` cell. `comparisonFingerprint` covers exported bytes, runtime/session options, workload/cache/logging, stage topology, GPU mapping, warmup, timeout, and window. The only allowed difference is the NDNSF network/security/orchestration layer. A mismatch is descriptive and has no overhead authority.

## BackendObservation

Records requested/available/selected providers, fallback policy/use, ORT/CUDA/cuDNN/driver versions, requested/allocated GRES, GPU UUID/model/container mapping, stage role, profile digest, and every executed model node's assigned provider. `fullCuda=true` requires complete node coverage, all model nodes assigned CUDA, zero undeclared CPU nodes, and allocation-correlated GPU identity.

## ExperimentCell

| Field | Rule |
|---|---|
| keyed `cellId`, `candidateId`, `mode` | mode is diagnostic, oracle, artifact, staged baseline, candidate correctness, or candidate performance |
| `modelSize`, `placement`, `workloadDigest`, `repetition` | immutable |
| `runId`, `jobId` | globally unique; several cells may share one bundled job but not a run ID |
| `schedulerState` | separate from experiment authority |
| `state`, `reasonCode` | `NOT_STARTED/SUBMITTED/RUNNING/PASS/FAIL/BLOCKED/DEFERRED/CANCELLED` |
| `gateScope`, `gateId`, `gateDigest` | required for BLOCKED/DEFERRED |
| `evidenceDigest` | required for executed terminal cells |

No transition out of a measured terminal state. A bundled task closes only after every member cell is terminal.

## SlurmRunRecord

Unique run/job identity, submission digest, deployment binding, requested/allocated TRES, node/GPU allocation, scheduler timestamps/state/exit, cancellation/preemption reason, scratch path, durable evidence path, original workload exit, promotion, and cleanup. Run IDs are unique across the campaign.

## ScalingEvidenceBundle

Links source, predecessors, deployment, candidate, cell, model, artifact, oracle, matched baseline, scheduler, container, node-level backend, security regression, numerical correctness, workload, statistics, terminal, checksums, redaction, and authority. Cross-field conditions are validated by Schema and the canonical semantic validator; missing or mixed links block acceptance.

## ScaleMatrix

An object keyed by `cellId`, plus a separate object keyed by `runId`. Keys provide structural uniqueness; the semantic validator checks cross-key references, candidate immutability, terminal conditions, matched pairs, and gate propagation. Every planned cell has one explicit state/reason. The matrix may be finalized only when no cell is `SUBMITTED/RUNNING` and every bundled task member is terminal.

## Authority states

Substrate, oracle, artifact, staged baseline, candidate correctness, and candidate performance are distinct. A candidate PASS requires source/predecessor/deployment validity, exact/numerical correctness, full CUDA node assignment, successful promotion, and a compatible terminal state. Physical production is always `DEFERRED`, owner Spec 106.
