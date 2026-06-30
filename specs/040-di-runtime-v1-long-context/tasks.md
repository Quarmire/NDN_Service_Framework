# Tasks: NDNSF-DI Runtime v1 With Long-Context Management

## Phase 1: Planning Artifacts

- [x] T001 Create the Runtime v1 long-context feature spec.
- [x] T002 Create the Runtime v1 long-context implementation plan.
- [x] T003 Create this task list.
- [x] T004 Add the full Runtime v1 roadmap to `docs/ndnsf-di-runtime-v1-roadmap.md`.

## Phase 2: Baseline And Contracts

- [x] T005 Collect specs 029-039 evidence into a current-status section.
- [x] T006 Define Runtime v1 goals: model scope, request duration, target RPS,
  p95 latency, and failure semantics.
- [x] T007 Define provider profile, model manifest, plan manifest, telemetry
  snapshot, and context-state schemas.
- [x] T008 Add planner invariant tests for memory feasibility, linear LLM split,
  shard-only-when-needed behavior, and resolvable dependency names.

## Phase 3: Provider Capability And Telemetry

- [x] T009 Extend provider ACK/capability payload with typed static fields.
- [x] T010 Extend dynamic telemetry with queue, active workers, free memory,
  model cache state, and runtime backend state.
- [x] T011 Add provider-to-provider, provider-to-repo, and user-to-provider
  RTT/bandwidth EWMA.
- [x] T012 Add KV-cache telemetry: cache memory budget, used memory, resident
  session/prefix IDs, hits, misses, and evictions.
- [x] T013 Persist telemetry to JSON/CSV for MiniNDN campaigns.

## Phase 4: Reusable Plan Lifecycle

- [x] T014 Define PlanKey, PlanLease, PlanVersion, and invalidation rules.
- [x] T015 Add plan cache and plan reuse in the user driver.
- [x] T016 Include cache-placement assumptions in plan validity.
- [x] T017 Add plan explanation JSON.
- [x] T018 Test plan reuse and invalidation under provider, network, and
  cache-state changes.

## Phase 5: Planner v1.5

- [x] T019 Convert the LLM planner into a stable library API.
- [x] T020 Add compute, memory, queue, RTT/bandwidth, dependency-transfer, and
  cache-placement costs.
- [x] T021 Implement linear stage split optimizer.
- [x] T022 Preserve proportional 2GB/4GB/8GB allocation by effective capacity.
- [x] T023 Generate fallback plans.
- [x] T024 Add target-RPS admission prediction.

## Phase 6: Role-Level Pipelined Runtime

- [x] T025 Make provider queues role-level rather than request-barrier-level.
- [x] T026 Allow a provider to start request N+1 when its role for request N is
  finished.
- [x] T027 Replace one waiter thread per pending dependency with a bounded
  dependency-wait scheduler.
- [x] T028 Add dependency prefetch and publish queues with bounded windows.
- [x] T029 Add cancellation and stale dependency cleanup.
- [x] T030 Test that different providers can be at different request indices in
  the same pipeline.

## Phase 7: Long-Context Runtime

- [x] T031 Define PromptChunk, PrefixState, SessionState, KvBlock, and
  GenerationChunk metadata.
- [x] T032 Extend model manifests with context window, tokenizer identity,
  KV-cache bytes per token/layer, and prefill/decode capabilities.
- [x] T033 Extend provider profiles with max context length and KV-cache memory.
- [x] T034 Add cache-placement planner logic for prefix/session reuse.
- [x] T035 Split long-context planning into prefill and decode phases.
- [x] T036 Add provider-local KV state references so NDNSF-DI can name and
  secure state without always transferring KV tensors.
- [x] T037 Add cache lease, pin, eviction, and invalidation policy.
- [x] T038 Add streaming GenerationChunk response path.
- [x] T039 Add MiniNDN long-context smoke: shared prefix, two follow-up prompts,
  verify prefix/cache reuse.
- [x] T040 Add MiniNDN cache-pressure test: cache hit, cache miss, eviction, and
  replan.

## Phase 8: Data Plane Performance

- [x] T041 Standardize tensor and context-object metadata.
- [x] T042 Add adaptive segment size for large tensors and long prompt chunks.
- [x] T043 Add optional compression/quantization hook for intermediate tensors.
- [x] T044 Measure reference wait, object fetch, publish time, and bytes
  transferred.
- [x] T045 Benchmark dependency and context-object transfer under MiniNDN RTT
  and bandwidth sweeps.

## Phase 9: Robustness

- [x] T046 Add role retry and fallback policy.
- [x] T047 Add straggler detection.
- [x] T048 Add provider failure recovery.
- [x] T049 Add hash mismatch, missing dependency, and stale context-state
  negative tests.
- [x] T050 Add long-session timeout and lease-expiration tests.

## Phase 10: Real LLM Execution

- [x] T051 Keep the smallest Qwen model as the first real target.
- [x] T052 Replace more deterministic delay with real prefill/decode execution.
- [x] T053 Validate output against a single-node baseline.
- [x] T054 Add microbatch and batching knobs.
- [x] T055 Measure time-to-first-token and inter-token latency.

## Phase 11: Usability And Evaluation

- [x] T056 Add `ndnsf-di provider`, `plan`, `run`, `bench`, and `inspect`
  commands.
- [x] T057 Add one-command MiniNDN profiles for short context, long context,
  provider failure, and high RTT.
- [x] T058 Generate Markdown/HTML reports.
- [x] T059 Repeat greedy, proportional, adaptive, and cache-aware layouts
  across RPS and context-length sweeps.
- [x] T060 Produce a decision table: single provider, linear split, sharded
  stage, cached-prefix reuse, or reject/defer.

## Phase 12: MiniNDN Runtime v1 Contract Integration

- [x] T061 Add a reusable Runtime v1 MiniNDN evidence writer that records the
  plan lease, telemetry CSV, report JSON, cache placement, generation timing,
  and decision table beside an experiment run.
- [x] T062 Wire Runtime v1 evidence into the NativeTracer MiniNDN launcher for
  `llm-proportional` policy bundles without changing the network execution
  path.
- [x] T063 Propagate Runtime v1 context/generation/prefix knobs through the LLM
  full-network campaign runner.
- [x] T064 Add regression coverage for the evidence writer and confirm policy
  layer allocation matches Runtime v1 plan allocation.

## Verification

- [x] V001 Markdown formatting/lint check not run because no project Markdown
  linter was found in the current documentation workflow.
- [x] V002 Run `git diff --check`.
- [x] V003 Confirm proposal slides were not modified.

## Evidence

- Python contract tests:
  `PYTHONPATH="$PWD/NDNSF-DistributedInference:$PWD/pythonWrapper" python3 tests/python/test_ndnsf_di_runtime_v1.py`
  passed with `10` tests.
- Runtime v1 long-context smoke:
  `python3 examples/python/NDNSF-DistributedInference/native_di_tracer/run_runtime_v1_long_context_smoke.py --out-dir /tmp/ndnsf-di-runtime-v1-smoke-final`
  produced cache-aware 2GB/4GB/8GB allocation and prefix-cache placement.
- Runtime v1 cache-pressure smoke:
  `python3 examples/python/NDNSF-DistributedInference/native_di_tracer/run_runtime_v1_cache_pressure_smoke.py --out-dir /tmp/ndnsf-di-runtime-v1-cache-pressure-final`
  produced cache hit/miss/eviction evidence.
- Runtime v1 profile campaign:
  `python3 examples/python/NDNSF-DistributedInference/native_di_tracer/run_runtime_v1_profile_campaign.py --out-dir /tmp/ndnsf-runtime-v1-profile-campaign`
  produced short-context, long-context, provider-failure, and high-RTT profile rows.
- Runtime v1 context sweep:
  `python3 -m ndnsf_distributed_inference.runtime_v1 context-sweep --model examples/python/NDNSF-DistributedInference/native_di_tracer/llm_model_spec_qwen_tiny_proportional.json --providers examples/python/NDNSF-DistributedInference/native_di_tracer/llm_provider_profiles_2_4_8.json --out-dir /tmp/ndnsf-runtime-v1-context-sweep --cache-aware`
  produced RPS/context-length rows.
- Runtime v1 MiniNDN contract integration:
  `PYTHONPATH="$PWD/NDNSF-DistributedInference:$PWD/pythonWrapper" python3 tests/python/test_ndnsf_di_runtime_v1.py`
  verifies the reusable evidence writer and now runs `11` contract tests.
- Runtime v1 full-network MiniNDN smoke:
  `PYTHONPATH="$PWD/NDNSF-DistributedInference:$PWD/pythonWrapper" sudo -n -E python3 Experiments/NDNSF_DI_NativeTracer_Minindn.py --policy-bundle llm-proportional --assignment llm-proportional --llm-planner-mode proportional --full-network --requests 1 --concurrency 1 --provider-check-timeout 80 --runtime-v1-context-tokens 8192 --runtime-v1-generated-tokens 8 --runtime-v1-prefix-id shared-uav-prefix --out /tmp/ndnsf-runtime-v1-minindn-full-network-smoke`
  passed with `status=SUCCESS`, `runnerMode=qwen-onnx-native`,
  `securityBootstrap=executed`, `userExecution=executed`,
  `dependencyExecution=executed`, `successCount=1`, `failureCount=0`,
  `meanMs=231.895`, Runtime v1 `2GB/4GB/8GB = 4/8/16` layer allocation,
  `allocationMatchesPolicy=true`, and cache provider
  `/NDNSF-DI/Tracer/provider/llm-8gb`.
- Validation scope: these are Runtime v1 contract checks, local profile
  campaigns, and one small real MiniNDN full-network smoke. They do not yet
  claim a long-running or statistically repeated real-network campaign result.
