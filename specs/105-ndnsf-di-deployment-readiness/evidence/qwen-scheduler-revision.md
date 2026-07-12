# Qwen Generation Scheduler Revision

**Task**: T051  
**Date**: 2026-07-12  
**Revision**: R1  
**Campaign ID**: `spec105-r1-qwen-scheduler-v1`  
**Measurement status**: PREREGISTERED, NOT RUN

## Retained Predecessor

The three original directories remain the complete evidence for the failed
predecessor campaign:

- `results/spec105-qwen-pilot-run1-20260712-0450`
- `results/spec105-qwen-pilot-run2-20260712-0454`
- `results/spec105-qwen-pilot-run3-20260712-0500`

They remain `BLOCK` with 0/180 completed generations. They are not rerun,
deleted, pooled with Revision R1, or relabeled. The scheduler revision changes
the tested system and therefore requires the distinct campaign identity above.

## Deterministic Driver Validity

Command executed before preregistration:

```bash
PYTHONPATH=NDNSF-DistributedInference \
  python3 tests/python/test_ndnsf_di_deployment_readiness.py
```

Result: 13/13 PASS. The generation-scheduler fixtures establish:

- one worker owns every ordered token step of an admitted generation;
- a later generation cannot interleave token work ahead of that owned job on
  the same worker;
- active plus queued generations have an explicit bound and overflow reason;
- token progress is positive and monotonic per session;
- completed, failed, unfinished, current active/queued and peak active/queued
  counts are observable;
- the actual `llm_pipeline/user.py` open-loop path completes two sessions with
  two ordered tokens each, uses no `on_result`/`on_error` token resubmission,
  and emits both summary and per-session progress records;
- user and MiniNDN harness reject an absent/unsafe campaign ID for native
  open-loop measurement.

The fixture is deterministic and contains no MiniNDN, ONNX compute, sleeps,
retry, timeout adjustment, or performance claim.

## Frozen Runtime Identity

- Model: `Qwen/Qwen2.5-0.5B-Instruct`
- Runtime: C++ ONNX Runtime CPU Execution Provider
- Stage 0 SHA-256:
  `8d4d8716b499f375634087f56e9011e6c65b70bd9c370fa9d0967055efd74006`
- Stage 1 SHA-256:
  `154d56424b46f527c9fbf9ed59877cd92f42ad331a0f61e6bb026a404ab70bf0`
- Stage 2 SHA-256:
  `9349b111492efc55c0e6c6586c9ebe087902992be2c85923d2f16d5a3d1ee05c`
- Prompt: `NDNSF deployment pilot`
- Expected greedy tokens:
  `[2025,271,785,5055,9965,5007,320,2448,37,8,702,7228,264,17708,2025,311,10517,264,501,79528,49601,3922,315,4237,24231,7798,311,1824,3412,304,279,5671]`
- Generation workers: 4
- Generation queue capacity: all 60 prespecified offered generations, bounded
- Client workers: 4
- Provider workers: 1 per stage
- Retries/replacements: none
- ACK timeout: 1,500 ms
- request timeout: 120,000 ms
- measured window: 60 seconds
- offered schedule: 1 generation/second, 60 generations maximum
- logging: INFO; TRACE disabled

## Preregistered T062 Command

T062 must first record the then-current clean source commit and prove that
T052-T061 telemetry artifacts exist. It then executes the following loop once.
All three repetitions execute even if an earlier repetition fails, because they
are the three prespecified cells; no fourth repetition is allowed.

```bash
set +e
STAMP="$(date -u +%Y%m%dT%H%M%SZ)"
TOKENS='2025,271,785,5055,9965,5007,320,2448,37,8,702,7228,264,17708,2025,311,10517,264,501,79528,49601,3922,315,4237,24231,7798,311,1824,3412,304,279,5671'
for RUN in 1 2 3; do
  OUT="results/spec105-r1-qwen-pilot-run${RUN}-${STAMP}"
  test ! -e "$OUT" || exit 90
  sudo -n env PYTHONPATH="$PWD/NDNSF-DistributedInference" \
    python3 Experiments/NDNSF_DI_LlmPipeline_Minindn.py \
      --topology-file Experiments/Topology/AI_Lab.conf \
      --output-dir "$OUT" \
      --runtime qwen-onnx-cpu-native \
      --stages 3 \
      --qwen-model Qwen/Qwen2.5-0.5B-Instruct \
      --qwen-dtype float32 \
      --prompt 'NDNSF deployment pilot' \
      --max-new-tokens 32 \
      --expected-token-ids "$TOKENS" \
      --native-first-kv-mode full-context \
      --warmup-requests 0 \
      --measured-requests 1 \
      --measured-duration-s 60 \
      --request-interval-ms 1000 \
      --ack-timeout-ms 1500 \
      --timeout-ms 120000 \
      --ndn-log 'ndn_service_framework.*=INFO' \
      --campaign-id spec105-r1-qwen-scheduler-v1
  printf '%s\n' "$?" >"$OUT/outer-exit-code.txt"
done
```

## Stop and Interpretation Rules

- Thresholds remain SC-002/SC-003: >=99% completion, >=0.95 achieved RPS,
  exact tokens, and distributed p95 <=2.0x matched single-node p95.
- Every summary must contain the exact campaign ID and generation queue/progress
  fields; missing identity or metrics makes the cell invalid and BLOCK.
- No timeout, retry, load, worker, token, model, artifact, logging, or topology
  change is allowed after the first cell starts.
- Failure is retained as `BLOCK`; it does not authorize a fourth run or a lower
  capacity class.
- This preregistration authorizes no measurement before T052-T061 complete.
- Physical status remains `DEFERRED` to Spec 106.

