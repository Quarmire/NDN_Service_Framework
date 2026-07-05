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

Runtime-aware multi-user MiniNDN RPS sweep:

```bash
sudo -n PYTHONPATH=NDNSF-DistributedInference:pythonWrapper:Experiments \
  python3 Experiments/NDNSF_DI_RuntimeAware_RpsSweep.py \
  --out /tmp/ndnsf-di-runtime-aware-rps-sweep \
  --rps 0.2,0.4 \
  --requests 2 \
  --concurrency 2 \
  -- --provider-check-timeout 60
```

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

Admission leases are opt-in. Existing non-lease services keep the current
ACK/Selection/Response path and still rely on ProviderToken, UserToken,
NAC-ABE, provider permissions, and replay protection. A lease is only an
admission-control proof; it is not a replacement for those security checks.

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

The campaign harness scans provider logs for
`NDNSF_ADMISSION_LEASE_ACCEPTED` and `NDNSF_ADMISSION_LEASE_REJECTED`. If the
specific NativeTracer run does not enable generic admission leases, these
counters are expected to stay at zero; the residency and latency fields still
describe the runtime-aware assignment that was exercised.

The current NativeTracer full-network path has runtime-aware assignment,
provider admission/backpressure, and provider-local inventory evidence. Generic
admission lease validation exists in NDNSF core, but NativeTracer does not yet
publish lease offers in ACKs and echo lease proofs in Selection. Treat
`leaseCounters=0` as a NativeTracer integration gap, not as a failure of the
inventory/RPS sweep machinery.

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
