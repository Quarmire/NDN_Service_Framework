# Spec 107 Artifact Reuse Preflight

Date: 2026-07-12

This is pre-campaign preparation evidence. It does not complete T025, T026, or
T037 and is not release-eligible.

## Disk recovery

The repository had approximately 4.6 GiB free. The validated build retained
about 2.64 GiB of reproducible `.o` intermediates. The following targeted
cleanup retained `build/unit-tests`, `build/examples/di-native-provider`,
`build/examples/di-native-fault-provider`, and all source/result evidence:

```bash
find build -type f \( -name '*.o' -o -name '*.o.*' \) -delete
```

Available space increased to approximately 7.0 GiB before artifact
materialization.

## Preserved invalid result

The first materialization produced artifact-set digest
`sha256:e45037b22429cff73b678ffc50676cda53539993caa8bee1f6794a849daa93b3`
but labeled roles `/LLM/Stage/0..2`. The real Qwen runtime uses
`/LLM/Pipeline/Stage/0..2`. This was classified
`INVALID_UNBOUND_ARTIFACT_STORE`; it was never referenced by a candidate or
campaign. The derived 2.52 GB copy was deleted only after a RED public-CLI test
reproduced the mismatch. The failure and digest remain recorded here.

## Correct materialization

Command:

```bash
python3 tools/ndnsf-di/spec107_candidate.py artifact prepare \
  --source /tmp/spec105-qwen-kv-export/qwen-onnx-stage-artifacts \
  --output-root results/spec107-artifacts
```

Result: `CREATED`, store:

```text
results/spec107-artifacts/7af3d35966c79e4f03eb657c9fbe149d4a4deaef3afb772e5c2484ecf9a80446
```

Manifest artifact-set digest:
`sha256:7af3d35966c79e4f03eb657c9fbe149d4a4deaef3afb772e5c2484ecf9a80446`.
All three ONNX files and `artifact-set.json` are mode `0444`; the store is
`0555`; no `.pt` file exists. Stage SHA-256 values are:

- Stage 0: `8d4d8716b499f375634087f56e9011e6c65b70bd9c370fa9d0967055efd74006`
- Stage 1: `154d56424b46f527c9fbf9ed59877cd92f42ad331a0f61e6bb026a404ab70bf0`
- Stage 2: `9349b111492efc55c0e6c6586c9ebe087902992be2c85923d2f16d5a3d1ee05c`

## No-export binding

`plan_pipeline.py` now accepts a sealed store plus reviewed Qwen service and
runtime manifests. The prepared input bundle is:

```text
results/spec107-qwen-reuse-inputs/
```

It contains only policy and JSON files. A real harness `prepare_policy` call in
`/tmp/spec107-policy-bytecheck` produced byte-identical
`llm_pipeline_policy.yaml` and `native-qwen-execution-plan.json`; neither
directory contained an ONNX file. Candidate validation binds:

```text
artifact   -> artifact-set.json
model      -> qwen-onnx-service-manifest.json
tokenizer  -> qwen-pipeline-runtime.json
plan       -> native-qwen-execution-plan.json
trustPolicy -> llm_pipeline_policy.yaml
command    -> diagnostic-command-profile.json
profile    -> diagnostic-campaign-profile.json
workload   -> diagnostic-workload.json
```

The actual warm-single and four-worker argparse values both passed the same
in-memory candidate/campaign/command validators. No MiniNDN process or
once-only output directory was started.

The MiniNDN diagnostic entry now invokes the campaign preflight before policy
generation, `pkill`, `Minindn.cleanUp()`, dependency verification, or any role
start. The command profile reserves 256 MiB for warm-single output and 512 MiB
for four-worker output, in addition to the mandatory 1 GiB reserve. Provider
capability input is derived from the candidate-bound native plan: exactly three
roles with the `onnxruntime` backend. Artifact preflight compares the candidate
artifact digest with the raw `artifact-set.json` digest and then verifies every
listed artifact hash.

A read-only warm-single projection against the real sealed store returned:

```text
verdict=PASS
roleStartAllowed=true
projectedNewBytes=268435456
reserveBytes=1073741824
requiredBytes=1342177280
freeBytes=4892766208
providerCount=3
requiredCapability=onnxruntime
```

This was a projection only: a synthetic in-memory candidate/campaign was used,
the PASS path wrote no file, and T025 was not consumed. A RED/GREEN harness
test also proves that an insufficient-space verdict exclusively retains the
sibling `*.invalid-preflight.json` record without creating the campaign output
directory.

PASS is now an atomic transition, not only a check: the harness creates one
`*.writer.json` sidecar with `O_EXCL` before output creation. A competing writer
cannot claim the cell, and later preflight classifies the live PID as
`OUTPUT_ACTIVE_WRITER`. The claim remains after exit so a crash is observed as
a stale writer instead of authorizing a replacement run. An existing retained
`*.invalid-preflight.json` is also a terminal condition even if disk or other
environment facts later recover.

The candidate profile/workload dimensions no longer reference the older
`/LLM/Stage/*` campaign layout or an unrelated p32 prompt. Dedicated Spec 107
manifests bind `/LLM/Pipeline/Stage/*`, the actual topology/runtime, `NDNSF
deployment pilot`, all 32 expected token IDs, the no-retry rule, and both
diagnostic schedules. The harness recomputes both raw digests and compares this
content with the parsed command before preflight.

The bound output identities now match T025/T026 literally:
`results/spec107-attribution-c1/warm-single` and
`results/spec107-attribution-c1/four-worker`. Command binding rejects a flat or
otherwise differently shaped path even when candidate, campaign, and command
digests all agree, preventing an internally consistent but spec-ineligible
campaign from starting.

Source identity is also revalidated at execution time. The harness requires a
clean tracked tree and recomputes Git's canonical committed HEAD tree digest
before campaign/artifact validation. A temporary-repository integration test
proves the frozen commit passes, a tracked edit fails
`SPEC107_SOURCE_TREE_DIRTY`, and a clean different HEAD fails
`SPEC107_SOURCE_CANDIDATE_DIGEST_MISMATCH`.

Campaign identity now includes `candidateDigest`, the canonical digest of the
complete candidate manifest. This closes the gap where the readable candidate
ID stayed unchanged after modifying workload, tokenizer, trust-policy, command,
or candidate metadata. Identity tests preserve the same candidate ID while
tampering the workload digest and prove campaign validation rejects it with
`CAMPAIGN_CANDIDATE_DIGEST_MISMATCH`.

Campaign validation now also recomputes the campaign ID from all six binding
fields and derives eligibility from kind. Tests prove an altered ordinal with
the old ID fails `CAMPAIGN_ID_MISMATCH`, and a diagnostic relabeled as
release-eligible fails `CAMPAIGN_ELIGIBILITY_INVALID`.

Candidate validation is strict at the top level: tests reject an unknown field,
`state=DRAFT`, malformed creation time, and an empty generator version. Only an
exact `FROZEN` candidate schema can reach campaign validation.

Campaign validation is likewise strict at the top level. An unknown field fails
`CAMPAIGN_FIELD_UNKNOWN`, and a missing generated field fails
`CAMPAIGN_FIELD_MISSING`; neither can be hidden behind an otherwise canonical
campaign ID.

Both shared identity validators now reject non-object manifest roots with
stable `CANDIDATE_OBJECT_INVALID` or `CAMPAIGN_OBJECT_INVALID` errors. This
prevents malformed JSON values from escaping the fail-closed boundary as raw
Python type exceptions.

## Verification

```text
Spec 107 Python: 84/84 PASS
Artifact focused: 9/9 PASS
Attribution focused: 13/13 PASS
Identity focused: 16/16 PASS
Preflight focused: 9/9 PASS
Strict Spec Kit structure: PASS
```
