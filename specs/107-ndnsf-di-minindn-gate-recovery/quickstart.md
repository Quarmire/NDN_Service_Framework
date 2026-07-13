# Quickstart: Spec 107 MiniNDN Gate Recovery

These commands define the intended operator surface. They become runnable as
the corresponding tasks are implemented. Run from the repository root.

Spec 105 remains frozen as `minindnCandidateOverall=BLOCK`. Spec 107 is a
MiniNDN-only candidate; physical production remains `DEFERRED` to Spec 106.
Do not delete, replace, tune, shorten, pool, or rerun any preregistered campaign
output; retain successful, failed, and invalid-preflight outcomes.

The bound diagnostic command profile projects 256 MiB for warm-single and
512 MiB for four-worker output. The harness adds the mandatory 1 GiB reserve,
verifies the sealed artifact hashes and candidate-bound three-role ONNX plan,
and retains `*.invalid-preflight.json` before policy generation or MiniNDN
startup if any gate fails.
On PASS it atomically creates the sole `*.writer.json` sidecar before creating
the output directory. The claim is retained: an active/stale writer or an
existing invalid-preflight record permanently blocks reuse of that campaign
output.

## 1. Verify frozen lineage and artifact/disk eligibility

```bash
python3 tools/ndnsf-di/spec107_candidate.py lineage verify \
  --lock specs/107-ndnsf-di-minindn-gate-recovery/lineage-lock.json

python3 tools/ndnsf-di/spec107_candidate.py artifact prepare \
  --source /tmp/spec105-qwen-kv-export/qwen-onnx-stage-artifacts \
  --output-root results/spec107-artifacts
```

Expected: exact Spec 105 digests, one read-only three-stage artifact set, no
`.pt` files, and no modification under Spec 105.

After materializing and reviewing that artifact set, set the nine
`SPEC107_*_INPUT` variables to reviewed files for the named candidate
dimensions. The CLI derives their full SHA-256 values and the committed HEAD
source digest mechanically. Before any campaign preregistration, run this
authoritative entry gate exactly as shown:

```bash
export SPEC107_ARTIFACT_STORE=results/spec107-artifacts/7af3d35966c79e4f03eb657c9fbe149d4a4deaef3afb772e5c2484ecf9a80446
export SPEC107_PROFILE_INPUT=specs/107-ndnsf-di-minindn-gate-recovery/diagnostic-campaign-profile.json
export SPEC107_MODEL_INPUT=results/spec107-qwen-reuse-inputs/qwen-onnx-service-manifest.json
export SPEC107_PLAN_INPUT=results/spec107-qwen-reuse-inputs/native-qwen-execution-plan.json
export SPEC107_ARTIFACT_INPUT="$SPEC107_ARTIFACT_STORE/artifact-set.json"
export SPEC107_LINEAGE_INPUT=specs/107-ndnsf-di-minindn-gate-recovery/lineage-lock.json
export SPEC107_WORKLOAD_INPUT=specs/107-ndnsf-di-minindn-gate-recovery/diagnostic-workload.json
export SPEC107_TOKENIZER_INPUT=results/spec107-qwen-reuse-inputs/qwen-pipeline-runtime.json
export SPEC107_TRUST_POLICY_INPUT=results/spec107-qwen-reuse-inputs/llm_pipeline_policy.yaml
export SPEC107_COMMAND_INPUT=specs/107-ndnsf-di-minindn-gate-recovery/diagnostic-command-profile.json
```

```bash
test -z "$(git status --porcelain=v1 --untracked-files=no)"
python3 tools/ndnsf-di/spec107_candidate.py lineage verify \
  --lock specs/107-ndnsf-di-minindn-gate-recovery/lineage-lock.json
python3 tools/ndnsf-di/spec107_candidate.py candidate inputs \
  --profile "$SPEC107_PROFILE_INPUT" --model "$SPEC107_MODEL_INPUT" \
  --plan "$SPEC107_PLAN_INPUT" --artifact "$SPEC107_ARTIFACT_INPUT" \
  --lineage "$SPEC107_LINEAGE_INPUT" --workload "$SPEC107_WORKLOAD_INPUT" \
  --tokenizer "$SPEC107_TOKENIZER_INPUT" \
  --trust-policy "$SPEC107_TRUST_POLICY_INPUT" \
  --command "$SPEC107_COMMAND_INPUT" \
  --output results/spec107-candidate-inputs.json
python3 tools/ndnsf-di/spec107_candidate.py candidate create \
  --digests results/spec107-candidate-inputs.json \
  --output results/spec107-candidate.json
```

Candidate creation is exclusive. The `source` digest must match the committed
HEAD tree; a dirty tracked tree or mismatched digest stops here before any role
starts. The MiniNDN harness repeats both checks at execution time, before
campaign/artifact preflight or writer claim, so changing tracked source or HEAD
after candidate creation cannot consume a cell.
Campaign preregistration also embeds the canonical digest of the entire
candidate manifest in its ID. Changing even a non-display digest such as
workload, tokenizer, trust policy, or command invalidates the campaign.

## 2. Run attribution before implementation acceptance

```bash
export SPEC107_EXPECTED_TOKEN_IDS=2025,271,785,5055,9965,5007,320,2448,37,8,702,7228,264,17708,2025,311,10517,264,501,79528,49601,3922,315,4237,24231,7798,311,1824,3412,304,279,5671
export SPEC107_ATTRIBUTION_COMMAND_DIGEST="sha256:$(sha256sum specs/107-ndnsf-di-minindn-gate-recovery/diagnostic-command-profile.json | cut -d' ' -f1)"
python3 tools/ndnsf-di/spec107_candidate.py campaign preregister \
  --kind diagnostic --ordinal 1 \
  --candidate results/spec107-candidate.json \
  --command-digest "$SPEC107_ATTRIBUTION_COMMAND_DIGEST" \
  --campaign-output-root results/spec107-attribution-c1/warm-single \
  --output specs/107-ndnsf-di-minindn-gate-recovery/evidence/attribution-warm-campaign.json
export SPEC107_ATTRIBUTION_WARM_CAMPAIGN_ID="$(python3 -c 'import json; print(json.load(open("specs/107-ndnsf-di-minindn-gate-recovery/evidence/attribution-warm-campaign.json"))["campaignId"])')"

sudo -n -E env PYTHONPATH=NDNSF-DistributedInference:Experiments \
  python3 Experiments/NDNSF_DI_LlmPipeline_Minindn.py \
  --spec107-diagnostic generation-session-attribution \
  --candidate-manifest results/spec107-candidate.json \
  --campaign-manifest specs/107-ndnsf-di-minindn-gate-recovery/evidence/attribution-warm-campaign.json \
  --campaign-id "$SPEC107_ATTRIBUTION_WARM_CAMPAIGN_ID" \
  --runtime qwen-onnx-cpu-native \
  --prompt 'NDNSF deployment pilot' \
  --max-new-tokens 32 --expected-token-ids "$SPEC107_EXPECTED_TOKEN_IDS" \
  --warmup-requests 0 --measured-requests 1 \
  --measured-duration-s 0 --request-interval-ms 0 \
  --ndn-log 'ndn_service_framework.*=INFO' \
  --spec107-timing-sample-rate 1 \
  --spec107-artifact-store "$SPEC107_ARTIFACT_STORE" \
  --spec107-qwen-service-manifest "$SPEC107_MODEL_INPUT" \
  --spec107-qwen-runtime-manifest "$SPEC107_TOKENIZER_INPUT" \
  --spec107-command-profile "$SPEC107_COMMAND_INPUT" \
  --output-dir results/spec107-attribution-c1/warm-single
```

Expected: `bottleneck-decision.json` with `SELECTED`, >=99% timing coverage,
reconciliation within the contract, and one >=25% dominant branch. This result
is labeled diagnostic and cannot pass performance.

Only after retaining the original warm result, preregister and execute the
four-worker diagnostic exactly once:

```bash
python3 tools/ndnsf-di/spec107_candidate.py campaign preregister \
  --kind diagnostic --ordinal 2 \
  --candidate results/spec107-candidate.json \
  --command-digest "$SPEC107_ATTRIBUTION_COMMAND_DIGEST" \
  --campaign-output-root results/spec107-attribution-c1/four-worker \
  --output specs/107-ndnsf-di-minindn-gate-recovery/evidence/attribution-four-worker-campaign.json
export SPEC107_ATTRIBUTION_FOUR_WORKER_CAMPAIGN_ID="$(python3 -c 'import json; print(json.load(open("specs/107-ndnsf-di-minindn-gate-recovery/evidence/attribution-four-worker-campaign.json"))["campaignId"])')"

sudo -n -E env PYTHONPATH=NDNSF-DistributedInference:Experiments \
  python3 Experiments/NDNSF_DI_LlmPipeline_Minindn.py \
  --spec107-diagnostic generation-session-attribution \
  --candidate-manifest results/spec107-candidate.json \
  --campaign-manifest specs/107-ndnsf-di-minindn-gate-recovery/evidence/attribution-four-worker-campaign.json \
  --campaign-id "$SPEC107_ATTRIBUTION_FOUR_WORKER_CAMPAIGN_ID" \
  --runtime qwen-onnx-cpu-native \
  --prompt 'NDNSF deployment pilot' \
  --max-new-tokens 32 --expected-token-ids "$SPEC107_EXPECTED_TOKEN_IDS" \
  --warmup-requests 0 --measured-requests 4 \
  --measured-duration-s 4 --request-interval-ms 1000 \
  --ndn-log 'ndn_service_framework.*=INFO' \
  --spec107-timing-sample-rate 1 \
  --spec107-artifact-store "$SPEC107_ARTIFACT_STORE" \
  --spec107-qwen-service-manifest "$SPEC107_MODEL_INPUT" \
  --spec107-qwen-runtime-manifest "$SPEC107_TOKENIZER_INPUT" \
  --spec107-command-profile "$SPEC107_COMMAND_INPUT" \
  --output-dir results/spec107-attribution-c1/four-worker
```

## 3. Validate the generation session

```bash
PYTHONPATH=tests/python:NDNSF-DistributedInference:. \
  python3 -m unittest discover -s tests/python \
  -p 'test_ndnsf_di_spec107_*.py' -v

./build/unit-tests --run_test='DiQwenGenerationSession/*' --log_level=message
```

Expected: exact 1/2/32-token outputs, bounded state, security/attempt negatives,
one final response, and full-context rebuild after compatible replacement.

## 4. Execute the frozen performance campaign

```bash
python3 tools/ndnsf-di/spec107_candidate.py campaign preregister \
  --kind performance --candidate results/spec107-candidate.json \
  --output specs/107-ndnsf-di-minindn-gate-recovery/evidence/performance-preregistration.json

sudo -n -E env PYTHONPATH=NDNSF-DistributedInference:Experiments \
  python3 Experiments/NDNSF_DI_LlmPipeline_Minindn.py \
  --spec107-performance-campaign \
  specs/107-ndnsf-di-minindn-gate-recovery/evidence/performance-preregistration.json
```

Expected: exactly three unique 60-second 1 RPS cells, no replacement run, each
independently evaluated against the unchanged thresholds.

## 5. Execute live faults independently

```bash
python3 tools/ndnsf-di/spec107_candidate.py campaign preregister \
  --kind fault --candidate results/spec107-candidate.json \
  --command-digest "$SPEC107_FAULT_COMMAND_DIGEST" \
  --campaign-output-root results/spec107-c1-live-faults-r1 \
  --output specs/107-ndnsf-di-minindn-gate-recovery/evidence/fault-campaign.json

python3 tools/ndnsf-di/run_spec107_live_faults.py preregister \
  --candidate results/spec107-candidate.json \
  --campaign specs/107-ndnsf-di-minindn-gate-recovery/evidence/fault-campaign.json \
  --output specs/107-ndnsf-di-minindn-gate-recovery/evidence/fault-matrix-lock.json

sudo -n -E env PYTHONPATH=NDNSF-DistributedInference:Experiments \
  python3 tools/ndnsf-di/run_spec107_live_faults.py run-cell \
  --candidate results/spec107-candidate.json \
  --campaign specs/107-ndnsf-di-minindn-gate-recovery/evidence/fault-campaign.json \
  --matrix-lock specs/107-ndnsf-di-minindn-gate-recovery/evidence/fault-matrix-lock.json \
  --cell positive-control
```

Expected: one positive control plus eight once-only cells with
`networkInjection=true`, one authoritative outcome, and clean child-process
state after every cell.

## 6. Local operations and soak

```bash
packaging/ndnsf-di-systemd/run-local-supervised.sh canary \
  --config /tmp/spec107-supervisor.json \
  --staging-root /tmp/spec107-canary-1/staging \
  --output /tmp/spec107-canary-1/canary.json --restart

packaging/ndnsf-di-systemd/run-local-supervised.sh operations \
  --root /tmp/spec107-operations/root \
  --release-n /tmp/spec107-release-n \
  --release-n1 /tmp/spec107-release-n1 \
  --output /tmp/spec107-operations/operations.json
```

The T064 soak runner is intentionally unavailable until T063 verifies every
predecessor gate and disk projection. Do not substitute a manual shortened run.

## 7. Generate and inspect the release decision

```bash
python3 tools/ndnsf-di/spec107_candidate.py gate generate \
  --feature specs/107-ndnsf-di-minindn-gate-recovery \
  --output specs/107-ndnsf-di-minindn-gate-recovery/release-gate.json
```

Expected: honest local PASS/BLOCK, Spec 105 retained as predecessor BLOCK, and
physical production always DEFERRED.
