# NDNSF-DI Runtime Workflow

Use this document as the normal entry point for NDNSF-DI experiments. The
canonical source of runtime configuration is the runtime profile; avoid calling
individual experiment scripts directly unless you are debugging one script.

## Canonical Profile

The default DI profile is:

```bash
examples/di-native-tracer.runtime.json
```

It records the NativeTracer harness, topology, Qwen tiny proportional model
artifacts, 2GB/4GB/8GB provider profiles, runtime knobs, token settings,
requests, concurrency, target RPS, and timeouts.

## Normal Flow

1. Validate the profile before a long run:

```bash
python3 tools/ndnsf_runtime.py di validate
```

2. Print the resolved profile when you need to inspect absolute paths and
   defaults:

```bash
python3 tools/ndnsf_runtime.py di print
```

3. Run the DI doctor and save the resolved configuration:

```bash
python3 tools/ndnsf_runtime.py di doctor \
  --event-log /tmp/ndnsf-di-runtime-events.jsonl \
  --write-resolved /tmp/ndnsf-di-runtime-resolved.json
```

4. Dry-run the experiment command before spending MiniNDN time:

```bash
python3 tools/ndnsf_runtime.py di run --dry-run -- --out /tmp/ndnsf-di-run
```

5. Run the experiment from the saved resolved profile:

```bash
python3 tools/ndnsf_runtime.py di run \
  --resolved /tmp/ndnsf-di-runtime-resolved.json \
  -- --out /tmp/ndnsf-di-run
```

Arguments before `--` belong to the wrapper. Arguments after `--` are passed to
the underlying experiment script and override profile defaults.

For a runtime-aware smoke run that starts MiniNDN but does not run the full
user/provider request path:

```bash
sudo -n python3 Experiments/NDNSF_DI_NativeTracer_Minindn.py \
  --runtime-profile examples/di-native-tracer.runtime.json \
  --out /tmp/ndnsf-spec047-minindn-smoke \
  --requests 2 \
  --concurrency 1 \
  --provider-check-timeout 45 \
  --no-local-execution-only
```

The default profile keeps `local_execution_only=true` so routine checks stay
fast. Use `--no-local-execution-only` when you intentionally want MiniNDN.

For the short full-network evidence path that exercises controller,
providers, user driver, ACK/Selection/Response, and dependency exchange:

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

## Common Commands

Tk operator console:

```bash
PYTHONPATH=NDNSF-DistributedInference:pythonWrapper \
  python3 Experiments/NDNSF_DI_GUI.py
```

The GUI's first three tabs are `User`, `Provider`, and `Controller`. Loading a
profile does not start anything. A role starts only when its tab's Run button
is clicked, and all three roles may run at the same time. The User tab includes
a request/response panel for normal service requests and collaboration request
JSON inputs. The default reusable profile is:

```bash
examples/python/NDNSF-DistributedInference/gui_three_role_profile.json
```

The GUI is an operator entrypoint. CLI commands below remain the reproducible
evidence path for MiniNDN campaigns and paper-quality measurements.

Headless GUI automation uses the same profile and runtime controller as the Tk
tabs, but does not create a display window. Use fake mode for CI-style logic
checks:

```bash
PYTHONPATH=NDNSF-DistributedInference:pythonWrapper \
  python3 Experiments/NDNSF_DI_GUI.py \
    -headless \
    -controller_auto_run \
    -provider_auto_run \
    -user_auto_run \
    -user_config=examples/python/NDNSF-DistributedInference/gui_user_hello.config \
    -provider_config=examples/python/NDNSF-DistributedInference/gui_provider_hello.config \
    -controller_config=examples/python/NDNSF-DistributedInference/gui_controller_hello.config \
    --runtime-mode fake \
    --send-user-request \
    --output-json /tmp/ndnsf-di-gui-headless.json
```

Use `--runtime-mode direct` only inside a prepared NFD or MiniNDN environment,
because it constructs the real `ServiceController`, `ServiceProvider`, and
`ServiceUser` wrapper objects. The `-user_config`, `-provider_config`, and
`-controller_config` files are JSON role overrides; they may also be wrapped
under a top-level `user`, `provider`, or `controller` key.

MiniNDN GUI preflight without opening the window:

```bash
PYTHONPATH=NDNSF-DistributedInference:pythonWrapper \
  python3 Experiments/NDNSF_DI_GUI_Minindn.py --preflight-only
```

This preflight verifies imports, policy loading, and the headless fake
Controller/Provider/User request path. Add `--run-minindn --case yolo-2x2` when
you also want the existing MiniNDN regression before launching or skipping the
GUI.

GUI headless Qwen NativeTracer MiniNDN experiment:

```bash
sudo -n PYTHONPATH=NDNSF-DistributedInference:pythonWrapper \
  PYTHONPYCACHEPREFIX=/tmp/ndnsf_pycache \
  python3 Experiments/NDNSF_DI_GUI.py \
    -headless \
    --headless-experiment qwen-minindn \
    --experiment-runtime-profile examples/di-native-tracer.runtime.json \
    --experiment-out /tmp/ndnsf-di-gui-qwen-headless-minindn \
    --experiment-requests 1 \
    --experiment-concurrency 1 \
    --experiment-provider-check-timeout 60 \
    --output-json /tmp/ndnsf-di-gui-qwen-headless-minindn/gui-headless-summary.json
```

This entrypoint keeps the GUI profile as the operator-facing configuration
surface, but delegates the network run to the canonical
`NDNSF_DI_NativeTracer_Minindn.py` harness. It forces the Qwen proportional
planner path with `--assignment llm-proportional`, `--policy-bundle
llm-proportional`, `--llm-planner-mode proportional`, `--full-network`, and
`--no-local-execution-only`. Use `--experiment-dry-run` first when checking a
configuration without starting MiniNDN.

The non-headless GUI exposes the same path in the `Qwen MiniNDN` tab. Edit the
runtime profile, output directory, request count, concurrency, provider-check
timeout, target RPS, open-loop duration, and extra harness arguments there,
then click `Preview Command` or `Run Qwen MiniNDN`. The tab uses the same
`build_qwen_minindn_command()` helper as headless mode; the only difference is
that the GUI lets an operator edit fields and click Run instead of passing
`--headless-experiment qwen-minindn`. Keep `Wrap with sudo -n env` enabled when
starting MiniNDN from a normal desktop session.

After a run, the tab displays the decoded core envelope summary from
`summary.json`: provider readiness, reason codes, service-payload schemas,
operation states, latest provider queue/active-work values, and the legacy ACK
runtime hint counters. Use `Refresh Summary` to reload these fields from the
current output directory without rerunning MiniNDN.

For a small GUI-driven campaign, set `Target RPS sweep list` to a comma-separated
list such as `0.2,0.4,0.8`, set `Sweep repeats`, then click `Run Sweep`. The GUI
runs the same Qwen MiniNDN command sequentially for each RPS/repeat and writes
each run under a separate subdirectory such as `rps-0_4-run-1`. Use `Dry run
only` first to verify the expanded command list without starting MiniNDN. When
`Output JSON` is set, the GUI also writes sibling CSV, Markdown, and SVG
reports. The CSV includes status, runner mode, target RPS,
request/success/failure counts, p50/p95/mean/makespan latency, throughput,
dependency status, dependency roles, provider count, mean provider utilization,
and total provider busy handler time. The Markdown report summarizes the run
count, failed runs, best p50, best throughput, provider utilization, and the
per-run `summary.json` paths. It also embeds the SVG plot, which shows latency,
throughput, and provider utilization for the sweep runs.

GUI tests are layered:

```bash
PYTHONPATH=NDNSF-DistributedInference:pythonWrapper \
  python3 tests/python/test_ndnsf_di_tk_gui.py

xvfb-run -a env PYTHONPATH=NDNSF-DistributedInference:pythonWrapper \
  python3 tests/python/test_ndnsf_di_tk_widgets.py

xvfb-run -a env PYTHONPATH=NDNSF-DistributedInference:pythonWrapper \
  python3 tests/python/test_ndnsf_di_gui_visual_smoke.py
```

The first command tests the non-display headless/runtime helpers. The second
creates a real Tk window under Xvfb and checks the `User`, `Provider`, and
`Controller` tabs, editable fields, Run buttons, and User request panel through
the same fake runtime factory. The third is optional and only captures a real
window screenshot when PyAutoGUI is installed; it is not the source of truth for
NDNSF-DI behavior.

Single NativeTracer harness run:

```bash
python3 tools/ndnsf_runtime.py di run -- --out /tmp/ndnsf-di-run
```

LLM full-network campaign:

```bash
python3 tools/ndnsf_runtime.py di campaign -- --runs 1 --workloads c1:1:1
```

NativeTracer rate sweep:

```bash
python3 tools/ndnsf_runtime.py di sweep -- --target-rps-list 0,1,2
```

For measured open-loop work, use the NativeTracer harness directly with a
60-second window, a request cap of `ceil(rate * 60)`, and the `threaded` driver.
Apply the Spec 093 scheduling, completion, throughput, dependency, and malformed
trace gates to every point. The former runtime-aware sweep was removed because
its deterministic runner and success-only stability check could label an
invalid offered-load point as stable.

Planner-only LLM proportional RPS search:

```bash
python3 tools/ndnsf_runtime.py di search -- --target-rps-list 1,5,10
```

Use `--dry-run` with `di run`, `di campaign`, `di sweep`, or `di search` when
you only want to inspect the generated command.

## What Each Step Catches

- `di validate`: misspelled keys, wrong scalar types, unsupported enum values,
  and missing DI sections.
- `di print`: the effective profile after default resolution.
- `di doctor`: missing artifacts, missing topology, missing binaries, NFD socket
  status, and the ready-to-run MiniNDN command.
- `di run`: one NativeTracer execution path.
- `di campaign`: repeated full-network LLM workload runs.
- `di sweep`: NativeTracer request-rate sweep.
- `di search`: planner-side greedy versus proportional RPS search.

## Runtime-Aware User-Side Planner Boundary

Spec047 keeps planning in the user process, but separates reusable NDNSF core
metadata from DI-specific inference semantics.

NDNSF core metadata is service-neutral:

- `GenericAckMetadata`: an ACK envelope for structured provider state.
- `GenericProviderRuntimeHint`: queue length, active work, wait estimate,
  capacity hints, confidence, and directed peer metrics.
- `PeerNetworkMetric`: directed RTT, bandwidth, loss, jitter, staleness, and
  confidence for provider-to-provider edges.
- `GenericAdmissionLease`: an optional, short-lived admission/resource proof.

NDNSF-DI interprets the service payload:

- `ModelFragmentKey`: digest-based model/split/stage/shard identity.
- `DiFragmentRuntimeState`: GPU, CPU, disk, repo, or missing residency state.
- `DiLeaseResourceBinding`: DI role plus fragment binding inside a generic
  lease.
- `ProviderNetworkMatrix`: graph-placement edge-cost view over directed peer
  metrics.

Runtime-aware NativeTracer planning can consume a matrix from an earlier probe
or run:

```bash
sudo -n python3 Experiments/NDNSF_DI_NativeTracer_Minindn.py \
  --runtime-profile examples/di-native-tracer.runtime.json \
  --runtime-aware-user-planner \
  --provider-network-matrix-json /path/to/previous-summary.json \
  --out /tmp/ndnsf-di-network-aware-run
```

The input may be a raw `ProviderNetworkMatrix` JSON file or a previous
NativeTracer `summary.json` containing `providerPairTelemetry.matrix`.
Telemetry from the current run is written after execution, so it is evidence
for the next planning run or campaign phase, not retroactive input to the plan
that already started.

Full-network NativeTracer runs collect this evidence by default with a
dependency-edge ndnping probe after provider provisioning and before the user
workload. The resulting file is:

```text
<run-out>/dependency-edge-ndnping-rtt-stats.json
```

Use `--skip-provider-pair-telemetry-probe` only for fast smoke runs where
provider-pair telemetry is not needed.

Admission leases are opt-in. Existing non-lease services keep the current
ACK/Selection/Response path and still rely on ProviderToken, UserToken,
NAC-ABE, provider permissions, and replay protection. A lease is only an
admission-control proof; it is not a replacement for those security checks.

For multi-user contention, each user plans from current typed ACK/runtime hints
and then acquires provider-owned execution leases. Rejection is explicit and
bounded; the user replans from fresh provider state. There is no DI coordinator
process, cross-user assignment authority, or generic Core coordination
envelope. The earlier advisory path failed its matched retention experiment and
was removed. Provider-owned admission remains the single execution authority.

## Runtime-Aware Campaign Outputs

Spec047 runs now write these stable files in the result directory:

```text
summary.json
summary.txt
planner-metrics.json
planner-metrics.csv
assignment.csv
runtime-v1/runtime-v1-minindn-evidence-summary.json
```

`planner-metrics.json` is the compact surface for paper or slide evidence. It
records p50/p95/mean latency when the user path runs, success rate, provider
utilization, lease counters, planner-selected residency counters, observed
provider residency counters, edge-cost summary, and bounded replan count. In
local-execution-only and provider-check runs, user latency is zero because the
full user request path is intentionally gated; the local plan, manifest,
runtime-v1 evidence, and MiniNDN provider placement are still validated.

`summary.json` is the broader run record. For core/app-boundary checks, read
`coreEnvelopeSummary`: it decodes provider ACK payloads that carry typed
`ProviderCapabilityHint` and nested `ServiceOperationStatus` envelopes. This
section reports provider readiness, negative/admission reason codes,
service-payload schemas, operation states, and each provider's latest core
runtime view. The older `providerAckRuntimeHints` section remains for legacy
queue/worker fields.

For Qwen NativeTracer MiniNDN runs, the C++ native provider emits both forms:
legacy semicolon ACK fields for old parsers and
`providerCapabilityHint=json64:<json>` for the core envelope summary. A healthy
small run should show `coreEnvelopeSummary.envelopeCounts.providerCapabilityHint`
greater than zero and `providerReadiness.ready` matching the observed ACK
events.

Provider fragment residency should come from `ProviderFragmentInventoryManager`
when the provider runtime can expose local state. The manager treats GPU and CPU
residency as explicit runtime load/evict events, treats disk residency as the
presence of the configured local artifact file, and falls back to
`REPO_AVAILABLE` or `MISSING` when the fragment is not local. The resulting
`DiProviderRuntimeState` is embedded in `GenericAckMetadata.servicePayload`, so
the user-side planner can prefer already-loaded or already-resident fragments
without putting model-specific concepts into NDNSF core.

The native C++ provider also emits provider-local inventory events:

```text
NDNSF_DI_FRAGMENT_INVENTORY event=CPU_RESIDENT provider=/P role=/Backbone \
  fragmentDigest=sha256:... backend=onnx-cpu path=/tmp/stage.onnx \
  residency=CPU_RESIDENT epoch_ms=...
```

`DISK_RESIDENT` is emitted before runner creation, `CPU_RESIDENT` or
`GPU_LOADED` after runner creation depending on the runtime/device metadata,
`EXECUTION_OBSERVED` when the provider actually executes that role, and
`EVICTED` when the provider runtime is released. Current ONNX CPU runs should
normally report CPU residency, not GPU residency. The MiniNDN harness scans
these events into `providerFragmentInventory`, including `eventCounters`,
`residencyCounters`, `latestByProviderRole`, and `latestByFragment`.
`planner-metrics.json.residencyCounters` remains the planner-selected residency
view; `planner-metrics.json.observedResidencyCounters` is the provider-log
observation view.

For multi-user evidence, report both direct lease counters and residency hits:

```text
leaseCounters.granted / rejected / expired / consumed
residencyCounters.GPU_LOADED / CPU_RESIDENT / DISK_RESIDENT / REPO_AVAILABLE / MISSING
observedResidencyCounters.CPU_RESIDENT / GPU_LOADED / DISK_RESIDENT
latencyMs.p50 / latencyMs.p95
maxStableRps from the RPS sweep
```

This is the minimum evidence needed to show whether user-side plans are being
controlled by provider admission leases and whether provider-local model
fragments are actually being reused.

The campaign harness scans provider logs for NativeTracer lease grants plus
`NDNSF_ADMISSION_LEASE_ACCEPTED` and `NDNSF_ADMISSION_LEASE_REJECTED`.
NativeTracer full-network sweeps now enable generic admission leases by default:
providers grant one lease in each successful readiness ACK, the Python
collaboration selector copies `leaseId` and `resourceBindingProof` into each
provider assignment payload, and providers consume the lease before executing
the selected role. Use `--disable-native-admission-lease` on the RPS sweep
wrapper only for an explicit no-lease comparison.

Expected healthy lease-enabled evidence for a two-request four-role smoke is
`granted > consumed`, `consumed = 8`, `rejected = 0`, and
`providerFragmentInventory.eventCounters.EXECUTION_OBSERVED = 8`.

Use two RPS modes carefully:

- Closed-loop sweeps with `--requests` and `--concurrency` validate end-to-end
  correctness, lease counters, residency counters, and latency, but
  `--target-rps` is only planner/load evidence unless `--open-loop-duration-s`
  is set. A 2026-07-05 lease/no-lease comparison at target RPS
  `0.2,0.4,0.8,1.2` kept observed throughput near `0.203 RPS` for every
  point, so it did not create a real high-load conflict.
- Open-loop sweeps with `--open-loop-duration-s` do schedule by target rate.
  The same 2026-07-05 comparison with a 20-second window succeeded at `0.2`
  RPS, but `0.4`, `0.8`, and `1.2` failed with
  `local-open-loop-backpressure` in both lease-enabled and no-lease runs. That
  means the current user-side child-process driver hits local backpressure
  before provider admission leases become the bottleneck.

For a defensible high-concurrency lease result, first use or implement a user
driver that can keep the offered load close to the target rate without local
backpressure, then rerun the same lease/no-lease comparison.

Spec 091 screened the three existing drivers at 1 RPS for 60 seconds with
concurrency 4. `child` completed 60/60 but achieved only 0.410 RPS with 77.96 s
maximum schedule slip. `threaded` failed before a complete workload summary
because worker users could not retrieve scope-key large Data while the base
publisher was not kept running. The removed process-pool experiment completed
60/60 with no local
backpressure and all dependency events, but its reported 0.932 RPS includes an
intentional five-second schedule lead and lacks per-worker slip telemetry.
Therefore this is a user-driver/instrumentation boundary, not provider-capacity
evidence. Do not quote a maximum stable RPS from this screening; see
`specs/091-native-di-offered-load-baseline/evidence/`.

Spec 092 fixed the base scope-key publisher lifecycle and open-loop timing
instrumentation. At the same 1 RPS, 60-second, concurrency-4 Qwen point, the
threaded driver passed three matched repetitions: 180/180 requests, 720/720
dependency events, mean 1.0133 RPS, mean p50 211.8 ms, mean p95 1176.4 ms, and
worst maximum schedule slip 14.8 ms. Use `threaded` for the next offered-load
search. The removed process-pool experiment completed 60/60 at 1.012 RPS but
failed the scheduling gate with 4345.5 ms startup slip. These results do not establish a
maximum stable RPS; see `specs/092-native-di-user-driver-correctness/evidence/`.

Spec 093 extends the same threaded Qwen fixture through 2, 4, and 8 offered
RPS. All points pass the scheduling and system gates. At 8 RPS, three matched
runtime treatments complete 1440/1440 requests with mean 7.9850 RPS, mean p50
198.5 ms, mean p95 247.6 ms, and worst schedule slip 16.24 ms. Dependency trace
markers total 5760; one line is explicitly retained as an observability parse
error caused by concurrent plain-text logging. The busiest stages reach about
26% estimated utilization, so no limiting layer is reached within 1-8 RPS.
State this as **stable through the highest tested point of 8 RPS**, never as a
maximum stable RPS. See `specs/093-native-di-threaded-rps-boundary/evidence/`.

The multi-user fixture is:

```bash
examples/python/NDNSF-DistributedInference/native_di_tracer/runtime_aware_fixtures/multi_user_requests.json
```

The directed provider-to-provider metric fixture is:

```bash
Experiments/Topology/AI_Lab_RuntimeAwarePeerMetrics.json
```

It is separate from `AI_Lab.conf` because MiniNDN topology links are symmetric,
while runtime-aware DI planning needs directed overlay metrics such as
provider A to provider B RTT/bandwidth versus provider B to provider A.

## When To Use Lower-Level Scripts

Use the wrapper first. Drop down to lower-level scripts only when you need to
debug script-specific behavior. The lower-level scripts still accept:

```bash
--runtime-profile examples/di-native-tracer.runtime.json
--runtime-resolved /tmp/ndnsf-di-runtime-resolved.json
```

Command-line flags on those scripts override profile defaults.

## Result Hygiene

For each meaningful run, keep the result directory and the summary files it
produces, such as JSON, CSV, lifecycle traces, or campaign summaries. If a run
is only a failed smoke or local troubleshooting attempt, delete it after the
useful finding is documented.
