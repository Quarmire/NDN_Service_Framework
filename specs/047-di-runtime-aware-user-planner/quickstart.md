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
- Provider runtime exposes local fragment inventory: GPU/CPU load events,
  actual disk artifact presence, and repo/missing fallback.

Expected outcome:

- Output includes p50/p95 latency, success rate, selected assignments, lease
  counters, residency counters, edge-cost summary, replan count, and provider
  utilization.
- `residencyCounters` count selected fragment residency values, not provider
  names.
- `maxStableRps` records the highest RPS sweep point that meets the campaign
  stability threshold.

Inventory unit validation:

```bash
PYTHONPATH=NDNSF-DistributedInference python3 tests/python/test_ndnsf_di_runtime_v1.py
```

Campaign metric aggregation validation:

```bash
PYTHONPATH=NDNSF-DistributedInference:Experiments python3 tests/python/test_ndnsf_di_runtime_aware_campaign.py
```

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
provider-check MiniNDN smoke.

Implemented full-network evidence command:

```bash
sudo -n python3 Experiments/NDNSF_DI_NativeTracer_Minindn.py \
  --runtime-profile examples/di-native-tracer.runtime.json \
  --out /tmp/ndnsf-spec047-full-network-audit \
  --requests 2 \
  --concurrency 1 \
  --provider-check-timeout 60 \
  --no-local-execution-only \
  --full-network \
  --tracer-deterministic-runner
```

Observed result:

```text
status=SUCCESS
miniNDNStatus=available-root
miniNDNRun=started
runnerMode=qwen-onnx-native
securityBootstrap=executed
userExecution=executed
dependencyExecution=executed
plannerMetrics=/tmp/ndnsf-spec047-full-network-audit/planner-metrics.json
```

Observed planner metrics:

```text
requestCount=2
successCount=2
successRate=1.0
p50Ms=163.16181999900436
p95Ms=343.0532000002131
meanMs=253.10750999960874
makespanMs=506.4827599999262
meanEstimatedUtilization=0.287673
replanCount=0
```

## 6. Runtime-aware multi-user RPS sweep

Use the sweep wrapper when the question is maximum stable request rate under
multi-user contention:

```bash
sudo -n PYTHONPATH=NDNSF-DistributedInference:pythonWrapper:Experiments \
  python3 Experiments/NDNSF_DI_RuntimeAware_RpsSweep.py \
  --out /tmp/ndnsf-spec047-runtime-aware-rps-sweep \
  --rps 0.2,0.4 \
  --requests 2 \
  --concurrency 2 \
  -- --provider-check-timeout 60
```

The wrapper writes:

```text
rps-sweep-commands.json
rps-sweep-summary.json
rps-sweep-summary.csv
rps-<value>/summary.json
rps-<value>/planner-metrics.json
```

Provider-local artifact events are collected from provider logs as
`providerFragmentInventory`. For current ONNX CPU NativeTracer runs, expect
`CPU_RESIDENT` and `DISK_RESIDENT` events. `GPU_LOADED` is only expected when a
GPU runtime/backend is configured. In `planner-metrics.json`,
`residencyCounters` is the planner-selected residency view, while
`observedResidencyCounters` is the provider-log observation view.

NativeTracer RPS sweeps enable generic admission leases by default. Each
successful provider readiness ACK carries a lease offer; the collaboration
selector echoes the selected provider's `leaseId` and `resourceBindingProof` in
the Selection assignment payload; the provider consumes that lease before role
execution. Use `--disable-native-admission-lease` only when comparing against a
no-lease baseline. A healthy two-request four-role smoke should report
`leaseCounters.consumed = 8`, `leaseCounters.rejected = 0`, and
`providerFragmentInventory.eventCounters.EXECUTION_OBSERVED = 8`.

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
python3 Experiments/NDNSF_DI_RuntimeAware_RpsSweep.py --dry-run --out /tmp/ndnsf-spec047-rps-sweep-dry-run --rps 0.2,0.4 --requests 2 --concurrency 2 -- --provider-check-timeout 60
sudo -n PYTHONPATH=NDNSF-DistributedInference:pythonWrapper:Experiments python3 Experiments/NDNSF_DI_RuntimeAware_RpsSweep.py --out /tmp/ndnsf-spec047-lease-smoke --rps 0.2 --requests 1 --concurrency 1 -- --provider-check-timeout 60
python3 tools/ndnsf_runtime.py di validate
python3 tools/ndnsf_runtime.py di run --dry-run
python3 Experiments/NDNSF_DI_NativeTracer_Minindn.py --runtime-profile examples/di-native-tracer.runtime.json --dry-run
python3 Experiments/NDNSF_DI_NativeTracer_Minindn.py --runtime-profile examples/di-native-tracer.runtime.json --out /tmp/ndnsf-spec047-local --local-execution-only
sudo -n python3 Experiments/NDNSF_DI_NativeTracer_Minindn.py --runtime-profile examples/di-native-tracer.runtime.json --out /tmp/ndnsf-spec047-minindn-smoke --requests 2 --concurrency 1 --provider-check-timeout 45 --no-local-execution-only
sudo -n python3 Experiments/NDNSF_DI_NativeTracer_Minindn.py --runtime-profile examples/di-native-tracer.runtime.json --out /tmp/ndnsf-spec047-full-network-audit --requests 2 --concurrency 1 --provider-check-timeout 60 --no-local-execution-only --full-network --tracer-deterministic-runner
git diff --check
```
