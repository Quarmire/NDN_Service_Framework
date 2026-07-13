# Operator CLI contract

The canonical implementation is repository-local at `tools/ndnsf-di/ndnsf-di-qwen`. The optional iTiger Skill may call it but is not an authority or runtime dependency.

```text
ndnsf-di-qwen snapshot-source --repo ROOT --output SNAPSHOT
ndnsf-di-qwen validate-predecessors --manifest MANIFEST
ndnsf-di-qwen discover --deployment-profile SPEC108_PROFILE --output DIR
ndnsf-di-qwen plan --matrix MATRIX --quota-record RECORD --output PLAN
ndnsf-di-qwen validate --schema TYPE --input FILE --semantic
ndnsf-di-qwen transfer --model-entry ENTRY --run-id ID [--submit]
ndnsf-di-qwen oracle --cell CELL [--submit]
ndnsf-di-qwen export --cell CELL [--submit]
ndnsf-di-qwen staged-baseline --cell CELL [--submit]
ndnsf-di-qwen candidate --cell CELL [--submit]
ndnsf-di-qwen status|wait|cancel --job-id JOB
ndnsf-di-qwen evidence --run-id ID --output DIR
ndnsf-di-qwen cleanup --deployment-profile SPEC108_PROFILE --dry-run
ndnsf-di-qwen aggregate --matrix MATRIX --output DIR
```

Rules:

- Source snapshot and exact predecessor validation precede candidate locking.
- Deployment resources resolve from the Spec 108 profile/release digest; Spec 109 flags cannot override account/QOS/CPU/memory/walltime/GRES.
- `discover`, `plan`, `validate`, and cleanup dry-run submit no jobs. Live commands require explicit `--submit`; render is default.
- Every acceptance run ID is unique and exactly once. Bundled jobs still write an independent terminal record for each cell.
- `oracle` produces correctness/capacity authority. `staged-baseline` is the only performance denominator. `candidate` cannot compare against oracle timing.
- Acceptance rejects incomplete source/predecessors, insufficient storage, missing license, mutable revisions, unmatched fingerprints, or unresolved model/systemic gates.
- Status/wait/cancel address one exact job ID. No command auto-retries or repairs evidence.
- Output excludes credentials, private keys, MFA data, registry tokens, raw NDNSF security tokens, and unrestricted prompt/tensor content.
