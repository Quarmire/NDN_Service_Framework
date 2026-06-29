# Tasks: Dynamic Network Telemetry

## Phase 1: Data Model And Store

- [X] N001 Add `NetworkTelemetrySnapshot` and sample types in `ndn-service-framework/NetworkTelemetry.hpp`.
- [X] N002 Add EWMA update/query logic in `ndn-service-framework/NetworkTelemetry.cpp`.
- [X] N003 Add unit tests for EWMA, TTL/staleness, confidence, and goodput calculation in `tests/unit-tests/network-telemetry.t.cpp`.

## Phase 2: ACK Selection Telemetry

- [X] N004 Extend `AckSelectionCandidate` in `ndn-service-framework/ServiceUser.hpp` with an optional telemetry snapshot.
- [X] N005 Record request publish time and ACK receive time in `ServiceUser.cpp`.
- [X] N006 Populate candidate telemetry before built-in or custom ACK selection runs.
- [X] N007 Add a custom lower-RTT selection unit test without changing existing strategy behavior.

## Phase 3: Collaboration Large-Data Telemetry

- [X] N008 Convert `CollaborationLargeFetchTiming` completion into telemetry samples in `ServiceProvider.cpp`.
- [X] N009 Record dependency edge identity: consumer provider, producer provider, key scope, Data name.
- [X] N010 Add logs/JSON export for large-data goodput samples when telemetry export is enabled.

## Phase 4: DI Planner Integration

- [X] N011 Add a Python telemetry-log parser that writes dynamic network profile JSON.
- [X] N012 Teach NativeTracer/Yolo planner tools to prefer dynamic profile values when confidence is high.
- [X] N013 Run planner-only validation comparing static vs dynamic profile decisions.
- [X] N014 Run a MiniNDN smoke that records telemetry and exports a dynamic profile.

## Phase 5: API Gap Closure

- [X] N015 Expose ACK telemetry snapshots through the Python ACK selection API.
- [X] N016 Add first-class DI provider worker capacity/queue snapshots.
- [X] N017 Include live provider capacity in native DI readiness ACK payloads and logs.
- [X] N018 Add focused unit coverage for Python-facing telemetry shape and provider capacity snapshots.

## Phase 6: Capacity-Aware High-Concurrency Evidence

- [X] N019 Parse `NDNSF_DI_PROVIDER_CAPACITY` rows into NativeTracer campaign summaries.
- [X] N020 Fix rate-sweep MiniNDN invocation so role delay and activation padding are passed correctly.
- [X] N021 Run 10 RPS / 100 RPS MiniNDN auto-assignment smoke with capacity telemetry.
- [X] N022 Record whether capacity telemetry shows real mitigation or only queue visibility.

## Phase 7: Runtime Capacity-Aware Role Selection

- [X] N023 Rank collaboration ACK candidates by advertised provider capacity before role assignment.
- [X] N024 Add a MiniNDN `capacity-pool` mode that launches a replicated Backbone provider candidate.
- [X] N025 Make concurrent user submission spacing configurable for high-concurrency stress.
- [X] N026 Validate that capacity-aware selection can split a bottleneck role across provider replicas.

## Phase 8: Capacity-Pool Campaign Evidence

- [X] N027 Extend the NativeTracer layout campaign runner to compare arbitrary assignment sets such as `default,capacity-pool`.
- [X] N028 Export provider-role allocation metrics, including Backbone default/replica execution counts and replica share.
- [X] N029 Isolate MiniNDN provider keychain homes so multiple providers can run on the same node without PIB/TPM conflicts.
- [X] N030 Run a small default-vs-capacity-pool MiniNDN campaign and record whether capacity pooling improves latency or only proves load steering.

## Phase 9: Burst Admission Steering

- [X] N031 Add per-child burst admission hints so simultaneous child-process users do not all tie-break to the first provider.
- [X] N032 Let the collaboration role-assignment policy read provider admission bias and optional role/provider preference from the child environment.
- [X] N033 Pass capacity-pool Backbone provider hints from the MiniNDN harness into the NativeTracer user driver.
- [X] N034 Validate that zero-spacing high-concurrency requests can steer work to the replicated Backbone provider.

## Validation Evidence

- `./waf build --targets=unit-tests -j4` passed.
- `./build/unit-tests -t NetworkTelemetry` passed.
- Python syntax check passed for `export_network_telemetry_profile.py`,
  `optimize_native_tracer_plan.py`, and `run_layout_campaign.py`.
- Parser fixture exported 1 synthetic edge and 1 pair override.
- Planner-only validation wrote static and dynamic evidence under
  `/tmp/ndnsf-telemetry-planner.CPbaXR`; the dynamic profile produced one
  high-confidence provider-pair override.
- MiniNDN smoke under `/tmp/ndnsf-026-telemetry-minindn-smoke-env` completed
  default/shared and single-provider NativeTracer runs with
  `userExecution=executed` and `dependencyExecution=executed`; exported profile
  contains 8 dependency edges and 8 pair overrides.
- Gap-closure validation passed:
  - `./waf build --targets=unit-tests -j4`
  - `./waf build -j4`
  - `./build/unit-tests -t NetworkTelemetry`
  - `./build/unit-tests -t ProviderRoleWorkerSnapshotReportsActiveAndQueuedWork`
  - `./build/unit-tests -t NativeProviderReadinessAckControlsSelectionEligibility`
  - `./build/unit-tests -t ProviderRoleWorkerSnapshotReportsActiveAndQueuedWork -t NativeProviderReadinessAckControlsSelectionEligibility -t NetworkTelemetry`
  - `./build/unit-tests` passed 163 test cases.
  - `./build/unit-tests -t NdnSvsSmoke/ServiceUserRequestServiceReachesProviderAndReturnsResponse`
    passed after making the smoke publish the response after the ACK is accepted,
    matching selection-before-response ordering.
  - `PYTHONPYCACHEPREFIX=/tmp/ndnsf_pycache python3 -m py_compile pythonWrapper/ndnsf/service.py`
  - `cd pythonWrapper && python3 setup.py build_ext --inplace`
  - Python import smoke verified `_ndnsf.AckCandidate.telemetry` and the
    Python `AckCandidate.telemetry` dataclass field.
- Capacity-evidence validation passed:
  - `PYTHONPYCACHEPREFIX=/tmp/ndnsf_pycache python3 -m py_compile
    examples/python/NDNSF-DistributedInference/native_di_tracer/run_layout_campaign.py
    examples/python/NDNSF-DistributedInference/native_di_tracer/run_auto_assignment_campaign.py
    examples/python/NDNSF-DistributedInference/native_di_tracer/run_rate_sweep_campaign.py`
  - Planner-only 10/100 RPS sweep under
    `/tmp/ndnsf-di-rate-planner-20260629-020532` selected
    `shared-backbone-current` for both rates and showed sharply higher
    estimated provider-capacity pressure at 100 RPS.
  - MiniNDN 10/100 RPS smoke under
    `/tmp/ndnsf-di-rate-minindn-20260629-020548` completed both rates with
    4/4 successful requests and exported provider capacity columns.
  - MiniNDN 100 RPS stress under
    `/tmp/ndnsf-di-rate-minindn-stress-20260629-020747` completed 8/8
    requests with `workloadP95Ms=5103.086`, `providerQueueWaitMaxMs=507.11`,
    `providerCapacityRows=64`, and `providerCapacityActiveWorkersMax=1.0`.
    This shows the telemetry exposes saturation and queue delay; it is
    visibility evidence, not yet a proven mitigation mechanism.
- Runtime capacity-aware role-selection validation passed:
  - `cd pythonWrapper && python3 setup.py build_ext --inplace`
  - `./waf build --targets=unit-tests -j4`
  - `PYTHONPYCACHEPREFIX=/tmp/ndnsf_pycache python3 -m py_compile
    Experiments/NDNSF_DI_NativeTracer_Minindn.py`
  - MiniNDN capacity-pool log-check under
    `/tmp/ndnsf-di-capacity-pool-logcheck-20260629-022906` completed 4/4
    requests with `p95Ms=2503.882`. The replicated Backbone provider
    `/NDNSF-DI/Tracer/provider/single` executed 3 Backbone sessions while the
    default Backbone provider executed 1, proving runtime ACK capacity can steer
    a bottleneck role across online candidates.
  - A failed all-role pool attempt under
    `/tmp/ndnsf-di-capacity-pool-stress-20260629-022355` showed that arbitrary
    per-role mixing with one all-role provider can create dependency scheduling
    failures. The supported `capacity-pool` mode therefore replicates the
    bottleneck Backbone role only.
- Capacity-pool campaign evidence passed:
  - `PYTHONPYCACHEPREFIX=/tmp/ndnsf_pycache python3 -m py_compile
    Experiments/NDNSF_DI_NativeTracer_Minindn.py
    examples/python/NDNSF-DistributedInference/native_di_tracer/run_layout_campaign.py`
  - A zero-spacing diagnostic under
    `/tmp/ndnsf-di-capacity-campaign-20260629-023843` completed 16/16 requests
    but selected the default Backbone for all requests. This shows simultaneous
    child-process submissions can tie before runtime capacity changes become
    visible.
  - Provider HOME isolation was validated under
    `/tmp/ndnsf-di-capacity-pool-homefix-20260629-024605`: the replicated
    Backbone provider on the same MiniNDN node reached
    `NDNSF_DI_NATIVE_PROVIDER_KEYCHAIN_READY` and no longer failed with a shared
    PIB/TPM default-identity error.
  - A 250 ms spacing sanity run under
    `/tmp/ndnsf-di-capacity-pool-spacing-20260629-024740` completed 4/4
    requests and split Backbone work 2/4 to the default provider and 2/4 to the
    replica.
  - A matched 2-run campaign under
    `/tmp/ndnsf-di-capacity-campaign-spacing-20260629-024848` completed 16/16
    total requests. `default` executed 8 Backbone sessions on the default
    provider; `capacity-pool` executed 4 on the default provider and 4 on the
    replica (`backboneReplicaShareMean=0.5`). Latency did not improve in this
    small Qwen/300 ms-delay campaign: capacity-pool had `workloadP95MeanMs`
    2143.489 vs default 1709.322, so the current evidence proves steering and
    keychain-safe same-node replicas, not a latency win.
- Burst-admission steering validation passed:
  - `cd pythonWrapper && python3 setup.py build_ext --inplace`
  - `PYTHONPYCACHEPREFIX=/tmp/ndnsf_pycache python3 -m py_compile
    Experiments/NDNSF_DI_NativeTracer_Minindn.py
    examples/python/NDNSF-DistributedInference/native_di_tracer/user_driver.py
    examples/python/NDNSF-DistributedInference/native_di_tracer/run_layout_campaign.py`
  - A zero-spacing diagnostic under
    `/tmp/ndnsf-di-capacity-pool-pref2-20260629-030358` completed 4/4 requests
    and steered 3 of 4 Backbone executions to the replica, confirming that
    parent-side burst admission hints can break the simultaneous-ACK tie.
  - A matched zero-spacing 2-run campaign under
    `/tmp/ndnsf-di-capacity-campaign-pref-20260629-030506` completed 16/16
    total requests. `default` executed 8 Backbone sessions on the default
    provider; `capacity-pool` executed 2 on the default provider and 6 on the
    replica (`backboneReplicaShareMean=0.75`). This small campaign showed a
    modest latency improvement: capacity-pool `p95Ms=11384.923` vs default
    `11619.706`, and `workloadP95MeanMs=2226.115` vs default 2315.276.
    Because the sample is only two runs, treat this as evidence that admission
    steering mitigates burst tie-breaking, not yet as a final performance claim.
