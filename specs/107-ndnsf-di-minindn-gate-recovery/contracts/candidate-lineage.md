# Candidate and Lineage Contract

`tools/ndnsf-di/spec107_candidate.py lineage verify` reads `lineage-lock.json`, verifies
every file and digest, checks the predecessor release/physical verdicts, and
emits no file on failure.

Candidate ID format:

```text
spec107-c1-<source12>-<profile12>-<model12>-<plan12>-<artifact12>-<lineage12>
```

All six suffixes are lowercase SHA-256 prefixes from full digests retained in
the manifest. Candidate creation fails on a dirty tracked source tree, missing
digest, Spec 105 output path, mutable artifact set, or unknown workload field.

`candidate inputs` mechanically creates
`ndnsf-di-spec107-candidate-inputs-v1` from nine reviewed input files:
profile, model, plan, artifact-set manifest, lineage lock, workload, tokenizer,
trust policy, and canonical command. It hashes raw file bytes and derives the
tenth `source` digest from Git's canonical committed HEAD tree. It rejects a
dirty tracked tree, unreadable input, Spec 105 input path, or existing output.
`candidate create` consumes this exclusive input manifest and independently
rechecks both source cleanliness and the committed source digest.
Every MiniNDN diagnostic/fault execution repeats those checks against its
checkout before campaign validation, artifact hashing, preflight, writer claim,
cleanup, or role startup. Untracked/ignored evidence outputs do not alter the
canonical committed-tree digest; tracked edits, staged edits, or a different
HEAD fail closed.

For the Qwen diagnostic candidate the dimensions bind concrete prepared inputs:

```text
artifact    artifact-set.json in the sealed /LLM/Pipeline/Stage/0..2 store
model       rebound qwen-onnx-service-manifest.json
tokenizer   qwen-pipeline-runtime.json with frozen prompt/input/token oracle
plan        native-qwen-execution-plan.json
trustPolicy llm_pipeline_policy.yaml after the exact app-root rewrite
command     diagnostic-command-profile.json
profile     diagnostic-campaign-profile.json with exact topology/runtime/roles
workload    diagnostic-workload.json with exact prompt/tokens/cell schedule
```

The MiniNDN harness independently recomputes these digests before process
cleanup or role startup. The candidate command digest, campaign command digest,
campaign ordinal/output root, and actual parsed arguments must all match one
command-profile cell. The candidate profile/workload digests are also recomputed
and their topology, runtime, roles, prompt, exact tokens, retry rule, and cell
schedule must match the parsed command before preflight. Per-campaign model export is forbidden; policy generation
may only rebind the reviewed metadata to the verified read-only artifact store.

The candidate manifest includes full source, lineage, workload, profile, model,
tokenizer, plan, artifact, trust-policy, command, and generator digests.

Every campaign stores `candidateDigest`, the SHA-256 of the canonical complete
candidate manifest, and includes it in campaign-ID derivation. The six-prefix
human-readable candidate ID remains stable, but it is not sufficient by itself
to authorize execution. Changing workload, tokenizer, trust policy, command,
timestamp, generator version, or any other candidate field changes the full
digest; campaign validation rejects the old manifest before preflight.
Validation never trusts the stored campaign ID: it recomputes the ID from
candidate ID/digest, kind, ordinal, command digest, and normalized output root.
It also derives diagnostic/release eligibility from `kind`; changing a binding
field or claiming acceptance eligibility for a diagnostic fails closed. The
campaign manifest must contain exactly the generated top-level fields; missing
or unknown metadata fails closed before campaign execution.

Candidate validation requires the exact top-level field set, `state=FROZEN`, a
UTC second-resolution `createdAt` value, and a non-empty generator version.
Unknown/missing metadata, draft state, or malformed metadata cannot be made
valid merely by including it in the complete candidate digest. Candidate and
campaign manifest roots must be objects; non-object JSON values fail with
stable identity errors rather than escaping as runtime exceptions.
