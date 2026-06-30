# Tasks: Provider-Local Semantic Service Cache

- [x] T001 Create Spec Kit documents for provider-local semantic service cache.
- [x] T002 Add semantic cache key, entry, manager, and lookup/admission logic.
- [x] T003 Add coarse ACK metadata helpers without exposing prompt/embedding/key data.
- [x] T004 Add cache-aware provider selection helper based on ACK hints.
- [x] T005 Export new helpers from package root.
- [x] T006 Add unit tests for hit/miss/admission/eviction/ACK privacy/selection.
- [x] T007 Run validation and record evidence.
- [x] T008 Audit SCALM fit and identify missing pattern-rank/token-saving-ratio support.
- [x] T009 Add semantic pattern metadata, rank classification, and token-saving-ratio helpers.
- [x] T010 Feed pattern rank/ratio into admission, eviction, telemetry, ACK hints, and selection.
- [x] T011 Add tests for pattern ranking, token saving ratio, pattern-derived entries, and ACK buckets.
- [x] T012 Add a minimal LLM semantic-cache provider demo with app-generated pattern/confidence.
- [x] T013 Make the demo emit per-request latency/cache/token metrics and summary JSON/CSV.
- [x] T014 Add tests for repeated/similar prompt cache hits and token-saving metrics.
- [x] T015 Run the demo as a small experiment and record evidence.
- [x] T016 Run validation and update CodeGraph.
- [x] T017 Add optional semantic cache integration to the real llama-server provider handler.
- [x] T018 Add provider-level tests proving first request calls llama-server and similar second request hits cache.
- [x] T019 Run llama-server provider dry-run and focused tests.
- [x] T020 Run final validation and update CodeGraph.
- [x] T021 Add a reproducible llama-server provider semantic-cache smoke harness that exercises the real handler path.
- [x] T022 Make the smoke harness emit per-request CSV metrics and summary JSON.
- [x] T023 Add tests proving the smoke harness reduces backend llama-server calls for similar prompts.
- [x] T024 Run final validation and update CodeGraph.
- [x] T025 Add a llama-server controller entrypoint for full NDNSF-DI network runs.
- [x] T026 Add a single-host network smoke harness with fake OpenAI backend, real controller/provider/user processes, and semantic-cache hit checks.
- [x] T027 Add focused tests for the fake backend used by the network smoke harness.
- [x] T028 Run compile, dry-run, focused tests, and network-smoke validation.
- [x] T029 Add a multi-provider semantic-cache selection campaign comparing first-provider and cache-aware policies.
- [x] T030 Add tests proving cache-aware selection prefers a warmed provider using only coarse ACK hints.
- [x] T031 Run the selection campaign and record latency/backend-call evidence.
- [x] T032 Run final validation and update CodeGraph.

## Evidence

- `PYTHONPATH=NDNSF-DistributedInference PYTHONPYCACHEPREFIX=/tmp/ndnsf_pycache python3 -m py_compile NDNSF-DistributedInference/ndnsf_distributed_inference/runtime_v1.py NDNSF-DistributedInference/ndnsf_distributed_inference/__init__.py tests/python/test_ndnsf_di_runtime_v1.py`: passed.
- `PYTHONPATH=NDNSF-DistributedInference PYTHONPYCACHEPREFIX=/tmp/ndnsf_pycache python3 tests/python/test_ndnsf_di_runtime_v1.py`: 21 tests passed.
- `git diff --check`: passed.
- `codegraph sync . && codegraph status .`: index is up to date.
- `PYTHONPATH=NDNSF-DistributedInference PYTHONPYCACHEPREFIX=/tmp/ndnsf_pycache python3 tests/python/test_ndnsf_di_semantic_cache_demo.py`: 2 tests passed.
- `PYTHONPATH=NDNSF-DistributedInference PYTHONPYCACHEPREFIX=/tmp/ndnsf_pycache python3 examples/python/NDNSF-DistributedInference/llm_semantic_cache_demo.py --compute-delay-ms 5 --cache-delay-ms 0.5 --metrics-csv /tmp/ndnsf-di-semantic-cache-demo.csv --summary-json /tmp/ndnsf-di-semantic-cache-demo-summary.json`: 8 requests, 5 hits, 3 misses, hit ratio 0.625, token saving ratio 0.538.
- `PYTHONPATH=pythonWrapper:NDNSF-DistributedInference PYTHONPYCACHEPREFIX=/tmp/ndnsf_pycache python3 examples/python/NDNSF-DistributedInference/llama_server/provider.py --dry-run --enable-semantic-cache --semantic-cache-budget-mb 8`: passed.
- `PYTHONPATH=pythonWrapper:NDNSF-DistributedInference PYTHONPYCACHEPREFIX=/tmp/ndnsf_pycache python3 tests/python/test_ndnsf_di_llama_semantic_cache_provider.py`: 2 tests passed.
- `PYTHONPATH=pythonWrapper:NDNSF-DistributedInference PYTHONPYCACHEPREFIX=/tmp/ndnsf_pycache python3 -m py_compile ...`: passed for runtime, package exports, semantic-cache demo, llama-server provider, and focused tests.
- `PYTHONPATH=pythonWrapper:NDNSF-DistributedInference PYTHONPYCACHEPREFIX=/tmp/ndnsf_pycache python3 examples/python/NDNSF-DistributedInference/llama_server/run_semantic_cache_provider_smoke.py --backend-delay-ms 5 --metrics-csv /tmp/ndnsf-di-llama-semantic-cache-smoke.csv --summary-json /tmp/ndnsf-di-llama-semantic-cache-smoke-summary.json`: 8 requests, 5 hits, 3 misses, 3 backend calls, hit ratio 0.625, token saving ratio 0.572, p50 0.123 ms, p95 13.883 ms.
- `tmpdir=$(mktemp -d /tmp/ndnsf-di-llama-controller-dry.XXXXXX); PYTHONPATH=pythonWrapper:NDNSF-DistributedInference PYTHONPYCACHEPREFIX=/tmp/ndnsf_pycache python3 examples/python/NDNSF-DistributedInference/llama_server/plan_llama_server.py --policy "$tmpdir/policy.yaml" --providers 1 --predeployed-only && PYTHONPATH=pythonWrapper:NDNSF-DistributedInference PYTHONPYCACHEPREFIX=/tmp/ndnsf_pycache python3 examples/python/NDNSF-DistributedInference/llama_server/controller.py --dry-run --config "$tmpdir/policy.yaml" --generated-policy-dir "$tmpdir/generated"`: passed.
- `PYTHONPATH=pythonWrapper:NDNSF-DistributedInference PYTHONPYCACHEPREFIX=/tmp/ndnsf_pycache python3 examples/python/NDNSF-DistributedInference/llama_server/run_semantic_cache_network_smoke.py --start-local-nfd --backend-delay-ms 2 --controller-wait-s 2 --provider-wait-s 6 --user-timeout-s 80 --ack-timeout-ms 1500 --timeout-ms 60000`: 4 requests, 4 user successes, 2 provider cache hits, 2 provider misses, 2 backend calls, passed; logs in `/tmp/ndnsf-di-llama-semantic-network.moapxyez`.
- `PYTHONPATH=pythonWrapper:NDNSF-DistributedInference PYTHONPYCACHEPREFIX=/tmp/ndnsf_pycache python3 examples/python/NDNSF-DistributedInference/llama_server/run_semantic_cache_selection_campaign.py --backend-delay-ms 5 --cache-delay-ms 0.25 --metrics-csv /tmp/ndnsf-di-llama-semantic-selection.csv --summary-json /tmp/ndnsf-di-llama-semantic-selection-summary.json`: first-provider 5 hits, 3 misses, 3 backend calls, hit ratio 0.625, avg 4.222 ms; semantic-cache-aware 8 hits, 0 misses, 0 backend calls, hit ratio 1.000, avg 2.948 ms.
