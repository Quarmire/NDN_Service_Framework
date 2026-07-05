# Quickstart: DI Runtime-Aware User-Side Planner Validation

This guide describes validation targets for the feature and records the
commands used to validate the implemented path.

## 0. Validate core versus DI layering

Expected scenario:

- NDNSF core sees generic ACK metadata, generic admission lease fields, and
  directed peer telemetry.
- NDNSF-DI sees model fragment keys, fragment residency, KV-cache hints, and
  graph-placement costs through service-defined payloads.

Expected outcome:

- Core tests prove lease validation works without model/GPU concepts.
- DI tests prove model fragment and residency payloads can be carried inside
  the generic core envelopes.

## 1. Validate deterministic planner scoring

Expected scenario:

- Provider A has the requested fragment GPU-loaded but moderate compute.
- Provider B has stronger compute but must fetch the fragment from repo.
- Provider C has the fragment on disk.

Expected outcome:

- The planner selects Provider A when queue and edge costs are otherwise equal.
- Metrics record a GPU-loaded residency hit.

## 2. Validate lease conflict control

Expected scenario:

- Two users request the same single-slot provider role at the same time.
- Provider grants one immediate lease and rejects or delays the other.

Expected outcome:

- Only one selection consumes the immediate lease.
- The second user replans or uses a future-start lease.
- Provider does not execute two immediate roles for the same reserved slot.

## 3. Validate provider-to-provider edge-aware placement

Expected scenario:

- Assignment A has strong compute providers but a poor dependency edge.
- Assignment B uses slightly weaker compute but a much better provider-pair link.

Expected outcome:

- The edge-aware planner chooses the lower estimated end-to-end assignment.
- Metrics include edge cost and provider-pair RTT/bandwidth inputs.

## 4. Validate stale-state replan

Expected scenario:

- Provider grants a lease and then reports fragment eviction before selection.

Expected outcome:

- Selection is rejected with `FRAGMENT_EVICTED`.
- User performs bounded replan.
- Replan record includes failed provider, failed lease, and next assignment.

## 5. Validate MiniNDN campaign evidence

Expected scenario:

- Multi-user workload.
- Asymmetric provider-to-provider links.
- Mixed fragment residency states.

Expected outcome:

- Output includes p50/p95 latency, success rate, selected assignments, lease
  counters, residency counters, edge-cost summary, replan count, and provider
  utilization.

Implemented smoke command:

```bash
sudo -n python3 Experiments/NDNSF_DI_NativeTracer_Minindn.py \
  --runtime-profile examples/di-native-tracer.runtime.json \
  --out /tmp/ndnsf-spec047-minindn-smoke \
  --requests 2 \
  --concurrency 1 \
  --provider-check-timeout 45 \
  --no-local-execution-only
```

Observed result:

```text
status=SUCCESS
miniNDNStatus=available-root
miniNDNRun=started
localExecution=executed
dependencyExecution=local-baseline-executed
plannerMetrics=/tmp/ndnsf-spec047-minindn-smoke/planner-metrics.json
```

Known environment warning:

```text
RuntimeError: module compiled against API version 0xe but this version of numpy is 0xd
```

The warning is printed by an optional Python dependency during MiniNDN startup
in this local environment. The command still returned 0 and completed the
provider-check MiniNDN smoke. Full user/provider `--full-network` campaign
latency evidence remains a heavier follow-up run.

## Final validation commands

```bash
./waf build --target=unit-tests
./build/unit-tests --run_test=GenericAdmissionLease
./build/unit-tests --run_test=NativeProviderAssignmentPayloadValidatesRoleAndFragment
./build/unit-tests --run_test=GenericDynamicApi/TokensAndReplay/TokenModeDisabledKeepsFirstRespondingAckSelectionResponsePath
./build/unit-tests --run_test=GenericDynamicApi/AllSelectedAndWorkers/AllSelectedProvidersExecuteOnlyAfterSelection
PYTHONPATH=NDNSF-DistributedInference python3 tests/python/test_ndnsf_core_admission_metadata.py
PYTHONPATH=NDNSF-DistributedInference python3 tests/python/test_ndnsf_di_runtime_aware_planner.py
PYTHONPATH=NDNSF-DistributedInference python3 tests/python/test_ndnsf_di_runtime_aware_campaign.py
python3 tools/ndnsf_runtime.py di validate
python3 tools/ndnsf_runtime.py di run --dry-run
python3 Experiments/NDNSF_DI_NativeTracer_Minindn.py --runtime-profile examples/di-native-tracer.runtime.json --dry-run
python3 Experiments/NDNSF_DI_NativeTracer_Minindn.py --runtime-profile examples/di-native-tracer.runtime.json --out /tmp/ndnsf-spec047-local --local-execution-only
sudo -n python3 Experiments/NDNSF_DI_NativeTracer_Minindn.py --runtime-profile examples/di-native-tracer.runtime.json --out /tmp/ndnsf-spec047-minindn-smoke --requests 2 --concurrency 1 --provider-check-timeout 45 --no-local-execution-only
git diff --check
```
