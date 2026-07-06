# Quickstart: Core Coordination Envelope

Use core coordination envelopes for service-neutral coordination:

```python
from ndnsf.coordination import (
    CoordinationIntent,
    CoordinationSuggestion,
    coordination_suggestion_proof,
    verify_coordination_suggestion,
)

intent = CoordinationIntent(
    intent_id="intent-1",
    request_id="req-1",
    requester_name="/user/A",
    service_name="/Inference/NativeTracer",
    payload_schema="ndnsf-di-plan-intent-v1",
    payload={"templateId": "qwen-stage0-template"},
)
```

Applications put service-specific meaning in `payload`. NDNSF-DI wraps this as
`PlanIntent` and `AdvisorySuggestion`, but provider assignment scoring remains
inside NDNSF-DI.

Run a NativeTracer dry-run with the coordinator service enabled:

```bash
PYTHONPATH=pythonWrapper:NDNSF-DistributedInference:Experiments \
python3 Experiments/NDNSF_DI_NativeTracer_Minindn.py \
  --dry-run \
  --advisory-coordinator \
  --requests 1 \
  --concurrency 1
```

Generate pure user-side versus advisory-coordinator RPS commands:

```bash
PYTHONPATH=pythonWrapper:NDNSF-DistributedInference:Experiments \
python3 Experiments/NDNSF_DI_RuntimeAware_RpsSweep.py \
  --dry-run \
  --compare-advisory-coordinator \
  --out /tmp/ndnsf-di-rps-sweep-advisory-dry-run \
  --rps 0.2 \
  --requests 2 \
  --concurrency 2
```

The measured campaign uses the same command without `--dry-run`. Compare the
`pure/` and `advisory/` result directories for conflict counters, p50/p95
latency, failure rate, and max stable RPS.

Latest measured smoke:

```text
/tmp/ndnsf-di-advisory-vs-pure-rps-sweep-20260705
```

This run used target RPS `0.2,0.4,0.8,1.2`, `requests=10`, and
`concurrency=2`. Both pure user-side planning and advisory-coordinator planning
were stable through the highest tested target RPS, with zero failures, zero
lease rejection, and zero negative ACK events. This is not a saturation result:
it only shows that 1.2 was still stable in this light-load configuration.
Advisory mode added roughly one coordination service round before each request,
so p50 latency was higher in this low-contention range. This result proves the
wire path is working, but it does not yet prove a coordination benefit; the next
campaign should raise concurrency and offered load until provider conflicts,
lease rejections, or sharp latency growth appear.

High-contention follow-up:

```text
/tmp/ndnsf-di-advisory-vs-pure-contention-sweep-20260705
```

This run used open-loop load, `requests=20`, `concurrency=8`,
`--role-execution-delay-ms 500`, `--provider-admission-max-active-workers 1`,
and `--provider-admission-max-queue 1`. It intentionally overloaded the
providers. Both pure and advisory modes failed in the same way:

```text
mode      target RPS  success rate  lease rejections  p95
pure      2.0         15%           17                ~60.5s
pure      4.0         10%           18                ~60.5s
advisory  2.0         15%           17                ~60.6s
advisory  4.0         10%           18                ~60.6s
```

The result is useful but negative: the coordinator service path was working,
but the initial wire-path smoke suggestion carried no effective role assignment
change, so it could not reduce provider conflicts. After this run, the harness
passes `assignment.csv` into the user driver and the coordination intent now
includes role/provider assignments. A MiniNDN smoke at
`/tmp/ndnsf-di-advisory-assignment-wire-smoke-20260705` confirmed that the
advisory service returns non-empty `roleAssignments` over the real NDNSF service
path. The next step is to make the coordinator compute or rewrite assignments
across a multi-user window, and make the user merge valid suggestions into
provider selection.

Assignment preference wire-path smoke:

```text
/tmp/ndnsf-di-advisory-preference-wire-smoke-20260705
```

This run used `--advisory-coordinator-only`, target RPS `0.2`, `requests=1`,
and `concurrency=1`. It confirmed the complete minimal path:

```text
CoordinationIntent payload includes roleAssignments and roleCandidates
Coordinator returns lease-aware rolling-window roleAssignments
Suggestion payload carries windowVersion=1,2
User records appliedRoleProviderPreference
Collaboration call uses NDNSF_COLLAB_ROLE_PROVIDER_PREFERENCE
MiniNDN run succeeds with 2/2 executed requests
```

`NDNSF_COLLAB_ROLE_PROVIDER_PREFERENCE` is the existing native selector input
used by the Python wrapper. Its format is a semicolon-separated list of
`<role>=>/<provider/name>;` pairs, for example:

```text
/Backbone=>/NDNSF-DI/Tracer/provider/backbone;/Merge=>/NDNSF-DI/Tracer/provider/merge;
```

The implemented coordinator window is a deterministic rolling window version,
not a full asynchronous batching window. It scores role candidates using
provider reservations, optional lease offers, optional runtime hints, ready
cost, duration, queue wait, and a small fairness penalty. The user accepts only
suggestions that match requested roles, converts them into the existing native
role-provider preference format, and the final native selector still rechecks
real ACK candidates before assigning a provider. This means a bad or stale
suggestion cannot force a provider that did not successfully ACK.

Important limitation: the default NativeTracer assignment has only one provider
candidate per role, so advisory planning cannot reduce contention in that
fixture. To measure a real benefit, run a follow-up campaign with multiple
valid candidates per role or a capacity-pool fixture, then compare pure
user-side planning against advisory-coordinator planning for lease rejections,
p50/p95 latency, and max stable RPS.

Capacity-pool candidate smoke:

```text
/tmp/ndnsf-di-advisory-capacity-pool-child-smoke-20260705
```

This run used `--assignment capacity-pool`, `--advisory-coordinator-only`,
target RPS `1.0`, `requests=2`, `concurrency=2`, and a small role execution
delay. The harness generated eight assignment rows: the normal primary
provider plus one alternate provider for each role. The open-loop child workers
now receive the same `assignment.csv` as the parent process, so their
coordination intents include `roleCandidates`.

The smoke succeeded with 2/2 executed requests. The logs confirmed that the
first request used the primary role providers, while the second request used
the alternate role providers. Both requests recorded
`appliedRoleProviderPreference`, so the advisory assignment was carried through
the real NDNSF coordination service path and into the native collaboration
selector.

Capacity-pool pure versus advisory sweep:

```text
/tmp/ndnsf-di-advisory-vs-pure-capacity-pool-sweep-20260705
```

This run used `--compare-advisory-coordinator`, `--assignment capacity-pool`,
target RPS `1.0,2.0`, `requests=8`, `concurrency=4`, open-loop duration `4s`,
`--role-execution-delay-ms 200`,
`--provider-admission-max-active-workers 1`, and
`--provider-admission-max-queue 1`.

```text
mode      target RPS  success  p50       p95        lease rejections
pure      1.0         4/4      631.27ms  893.74ms   0
advisory  1.0         4/4      690.43ms  998.55ms   0
pure      2.0         2/8      0ms       60517.17ms 6
advisory  2.0         2/8      0ms       60578.79ms 6
```

The capacity-pool fixture proves that advisory planning can issue different
valid provider assignments when multiple candidates exist for each role.
However, this first overloaded sweep is still a negative performance result:
both pure and advisory modes had the same max stable target RPS, the same
success count at 2 RPS, and the same lease rejection count. The current
bottleneck appears to be the bounded execution/admission path and long request
timeout under open-loop overload, not only provider choice. The next design
step should therefore focus on faster overload feedback, shorter failed-request
completion, and a scheduler that can admit work before requests occupy the
full collaboration timeout.

Overload fast-fail boundary smoke:

```text
/tmp/ndnsf-di-fast-fail-capacity-pool-sweep-20260705
```

This run repeated the 2 RPS capacity-pool overload point with
`--overload-fast-fail-timeout-ms 5000`. The knob keeps the normal base timeout
available for ordinary long-running work, but uses a shorter collaboration
timeout for overload-boundary experiments and records `overloadFastFailCount`.

Compared with the previous capacity-pool overload sweep, requests that would
otherwise time out during overload no longer waited for the 60s collaboration
timeout. The failed-timeout p95 dropped from about 60.5s to about 5.5s:

```text
mode      target RPS  success  p95        overload fast-fail count
pure      2.0         3/8      5501.35ms  1
advisory  2.0         2/8      5583.31ms  2
```

This is a latency-boundary improvement, not a throughput improvement. Max
stable RPS did not improve, and advisory mode still did not outperform pure
user-side planning in this overload fixture. The result confirms that the next
real scheduler step is not more candidate providers; it is earlier admission
feedback, faster failed-request completion, and a queue/admission boundary that
prevents doomed requests from occupying the full collaboration response
timeout.

Lease-aware advisory coordinator smoke:

```text
/tmp/ndnsf-di-lease-aware-advisory-smoke-20260705
```

This run used the upgraded wire-path coordinator with `--assignment
capacity-pool`, `--advisory-coordinator-only`, target RPS `1.0`, `requests=2`,
and `concurrency=2`. It completed successfully:

```text
success: 2/2
p50: 388.70ms
p95: 507.23ms
leaseCounters: granted=8 consumed=8 rejected=0
```

The user-worker logs show `advisoryMode=lease-aware-rolling-window`.
One request received the alternate provider set and the other received the
primary provider set. This confirms that the NDNSF service wire path can carry
DI-owned rolling reservation suggestions without adding DI-specific scheduling
fields to NDNSF core.

Runtime hint snapshot:

```text
runtime-hints.json
```

When `--advisory-coordinator` is enabled, the MiniNDN harness now writes a
DI-owned runtime hint snapshot and passes it to the user driver with
`--runtime-hints-json`. The user driver merges the snapshot into each
`roleCandidates` entry before requesting `/NDNSF/Coordination/Advisory`.
The snapshot may include `runtimeHint`, `leaseOffers`, `estimatedDurationMs`,
`readyCostMs`, and fragment `residency`. These fields remain inside the
NativeTracer payload; NDNSF core still treats the coordination payload as
opaque.

In real MiniNDN advisory runs, the harness writes an initial profile-based
snapshot, waits for provider provisioning, parses provider
`NDNSF_DI_FRAGMENT_INVENTORY` logs, and refreshes the same file before the user
driver starts. The refreshed snapshot marks observed records with
`source=provider-runtime-inventory`, so coordinator scoring can distinguish
profile assumptions from provider runtime evidence.

Latest provider-inventory refresh smoke:

```text
/tmp/ndnsf-di-runtime-inventory-refresh-smoke2-20260705
```

This run completed successfully with 2/2 requests. The summary recorded
`runtimeHintSnapshotRefresh.updated=4`: the four primary providers had
provider-tagged runtime inventory before the user driver started, while the
alternate providers kept the profile-based hints because they did not emit
provider-tagged provisioning inventory in this smoke. That is the intended
partial-refresh behavior: observed provider runtime state overrides static
profile assumptions when available, and missing live evidence falls back to the
initial snapshot.

Provider ACK runtime telemetry:

Native provider ACK decision logs include the provider identity and ACK payload.
The MiniNDN harness aggregates those logs into `providerAckRuntimeHints` in
`summary.json`. The fields include queue depth, ready queue, waiting inputs,
active workers, workers, idle workers, runtime status, negative ACK reason, and
lease identifiers. This proves that the provider-side live capacity signal is
available on the real invocation path. The next step is to expose the same ACK
candidate snapshot to the Python user/advisory layer before final role
selection, instead of using it only inside the native selector and summary
evidence.

Latest ACK telemetry smoke:

```text
/tmp/ndnsf-di-ack-telemetry-smoke-20260705
```

This run completed successfully with 2/2 requests. The underlying summary
reported `providerAckRuntimeHints.eventCount=8` across the four primary
providers. Each provider contributed live ACK fields such as `queue=0`,
`activeWorkers=0`, `workers=1`, `idleWorkers=1`, and a fresh `leaseId`.

Collaboration ACK observer:

Python users can now observe collaboration ACK candidates without replacing the
native selector:

```python
snapshots = []

def observe_ack(candidates):
    snapshots.extend(candidates)

response = user.request_collaboration(
    "/Inference/NativeTracer",
    payload,
    roles=roles,
    key_scopes=key_scopes,
    ack_observer=observe_ack,
)
```

NativeTracer records these observations as `ackCandidateSnapshot` in each
request result. This is the bridge from provider ACK runtime telemetry to
future advisory planning windows.

MiniNDN smoke evidence:

```text
/tmp/ndnsf-di-ack-observer-smoke-20260705-fixed/advisory/rps-1p0
status=SUCCESS
userExecution=executed
successCount=2 / requestCount=2
ackCandidateSnapshot=14 candidates per request
providerAckRuntimeHints.eventCount=8
```
