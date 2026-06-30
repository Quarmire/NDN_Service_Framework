# Tasks: Exact Forward Cache For NDNSF-DI

- [x] T001 Create Spec Kit documents under `specs/041-exact-forward-cache/`.
- [x] T002 Add `ExactForwardCacheKey`, `ExactForwardCacheEntry`, and `ExactForwardCacheManager` in `NDNSF-DistributedInference/ndnsf_distributed_inference/runtime_v1.py`.
- [x] T003 Add strict token-prefix digest and stage-definition helpers in `NDNSF-DistributedInference/ndnsf_distributed_inference/runtime_v1.py`.
- [x] T004 Extend `KvCacheTelemetry` with provider-local exact cache key digests for telemetry/evidence while keeping ACK metadata free of cache keys.
- [x] T005 Update `NDNSF-DistributedInference/ndnsf_distributed_inference/runtime_v1_evidence.py` to record exact cache fields.
- [x] T006 Export new Runtime v1 exact-cache symbols in `NDNSF-DistributedInference/ndnsf_distributed_inference/__init__.py`.
- [x] T007 Add Python tests in `tests/python/test_ndnsf_di_runtime_v1.py` for exact hit, token miss, stage miss, and plan/layout miss.
- [x] T008 Run Python tests, py_compile, `git diff --check`, CodeGraph sync/status, and record evidence.
- [x] T009 Refine cache security boundary: keep Exact Forward Cache provider-local, remove in-network forward-state reuse fields, and document NDN cache use for artifacts/input/output Data.
- [x] T010 Add C++ provider-local memoization in `ProviderRoleWorker` so repeated identical local role inputs skip `NativeModelRunner::run`.
- [x] T011 Add C++ tests proving identical provider-local inputs hit and changed inputs miss.

## Evidence

- `PYTHONPATH=NDNSF-DistributedInference python3 tests/python/test_ndnsf_di_runtime_v1.py`: 14 tests passed.
- `PYTHONPYCACHEPREFIX=/tmp/ndnsf_pycache python3 -m py_compile ...`: passed for runtime v1, evidence, package exports, and tests.
- `./waf build --targets=unit-tests`: passed.
- `./build/unit-tests --run_test=ProviderRoleWorkerUsesProviderLocalExactForwardCache`: passed.
- `./build/unit-tests --run_test=ProviderRoleWorker*`: 9 test cases passed.
- `git diff --check`: passed.
- `codegraph sync . && codegraph status .`: index is up to date.
- Exact-cache tests verify identical key hit plus token-prefix, stage-definition, and plan/layout misses.
- Exact Forward Cache is provider-local by default. NDN in-network cache remains
  in the design for model artifacts, tokenizer/config files, input/output Data,
  telemetry objects, and video segments, not for default KV/forward-state reuse.
- Provider-local cache keys are not exposed in ACK metadata.
- C++ provider role worker memoization is implemented in the local execution hot
  path; repeated identical inputs reuse cached outputs and changed inputs miss.
