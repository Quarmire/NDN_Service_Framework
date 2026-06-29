# Feature 010: DI Threshold Campaign

Status: Threshold Campaign Accepted

## Goal

Find the first controlled threshold experiment after the 10x2 layout campaign:
keep the same smallest Qwen NativeTracer ONNX model, but increase dependency
payload size in controlled steps to see whether cross-provider shared-backbone
execution begins to close the gap with single-provider serial execution.

## Scope

- Do not replace or enlarge the Qwen-derived ONNX model artifacts.
- Add an activation-size knob that pads the Backbone encoded output bundle with
  an ignored tensor so Head providers still consume the real `features` tensor.
- Propagate the knob through the NativeTracer policy bundle, MiniNDN harness, and
  layout campaign runner.
- Run a small smoke threshold campaign with at least two activation sizes and
  both executable layouts.
- Emit per-size JSON/CSV summaries that can feed slides/paper later.

## Non-Goals

- Full Qwen token/pipeline execution.
- Larger downloaded Qwen checkpoints.
- Making estimated-only layouts executable in this feature.
- Claiming a final threshold from one smoke campaign.

## Acceptance

- [x] C++ ONNX runner can add optional padding to encoded output bundles without
  changing the named tensor consumed by downstream roles.
- [x] NativeTracer bundle generation can set Backbone activation padding and
  adjusted dependency expected bytes/segments.
- [x] MiniNDN harness accepts an activation padding parameter and records it in
  summaries.
- [x] Campaign runner can iterate over multiple activation padding values.
- [x] A smoke threshold campaign completes with all runs executed.
- [x] Results and interpretation are recorded here.

## Accepted Smoke

Command:

```bash
python3 examples/python/NDNSF-DistributedInference/native_di_tracer/run_layout_campaign.py \
  --runs 1 \
  --out-root /tmp/ndnsf-di-threshold-smoke \
  --provider-check-timeout 60 \
  --activation-pad-bytes-list 0,65536
```

Artifacts:

- Threshold summary: `/tmp/ndnsf-di-threshold-smoke/threshold-summary.json`
- Pad 0 CSV: `/tmp/ndnsf-di-threshold-smoke/pad-0/campaign-runs.csv`
- Pad 65536 CSV: `/tmp/ndnsf-di-threshold-smoke/pad-65536/campaign-runs.csv`

Smoke results:

| Activation padding | Layout | Runs | Mean ms | p50 ms | p95 ms | Candidate |
| ---: | --- | ---: | ---: | ---: | ---: | --- |
| 0 | `default` | 1 | 253.303 | 253.303 | 253.303 | `shared-backbone-current` |
| 0 | `single-provider` | 1 | 201.471 | 201.471 | 201.471 | `single-provider-serial` |
| 65536 | `default` | 1 | 256.534 | 256.534 | 256.534 | `shared-backbone-current` |
| 65536 | `single-provider` | 1 | 191.265 | 191.265 | 191.265 | `single-provider-serial` |

All four full-network MiniNDN runs reported `userExecution=executed` and
`dependencyExecution=executed`.

Interpretation: the threshold mechanism is working, but this smoke is not a
statistical threshold result. At 64 KiB of added Backbone activation payload,
the default shared-backbone layout still did not beat the single-provider
layout in a single run. The next campaign should use more padding levels and
multiple runs per level, for example `0,65536,262144,1048576` with 3--5 runs per
layout.

## Accepted Threshold Campaign

Command:

```bash
python3 examples/python/NDNSF-DistributedInference/native_di_tracer/run_layout_campaign.py \
  --runs 3 \
  --out-root /tmp/ndnsf-di-threshold-campaign-3 \
  --provider-check-timeout 60 \
  --activation-pad-bytes-list 0,65536,262144,1048576
```

Artifacts:

- Threshold summary: `/tmp/ndnsf-di-threshold-campaign-3/threshold-summary.json`
- Per-pad CSVs: `/tmp/ndnsf-di-threshold-campaign-3/pad-*/campaign-runs.csv`

Results:

| Activation padding | Layout | Runs | Mean ms | Stddev ms | p50 ms | p95 ms | Candidate |
| ---: | --- | ---: | ---: | ---: | ---: | ---: | --- |
| 0 | `default` | 3 | 218.907 | 63.219 | 183.626 | 291.893 | `shared-backbone-current` |
| 0 | `single-provider` | 3 | 211.313 | 11.369 | 208.419 | 223.850 | `single-provider-serial` |
| 65536 | `default` | 3 | 292.584 | 9.477 | 294.453 | 300.988 | `shared-backbone-current` |
| 65536 | `single-provider` | 3 | 178.912 | 17.856 | 170.590 | 199.410 | `single-provider-serial` |
| 262144 | `default` | 3 | 355.196 | 26.782 | 358.705 | 380.050 | `shared-backbone-current` |
| 262144 | `single-provider` | 3 | 197.816 | 12.224 | 204.292 | 205.440 | `single-provider-serial` |
| 1048576 | `default` | 3 | 3709.943 | 2754.404 | 5297.460 | 5302.937 | `shared-backbone-current` |
| 1048576 | `single-provider` | 3 | 199.368 | 15.552 | 194.518 | 216.767 | `single-provider-serial` |

All 24 full-network MiniNDN runs reported the expected executable candidate.
The campaign did not find a payload-size threshold where
`shared-backbone-current` beats `single-provider-serial`. Larger activation
padding made the shared-backbone layout worse because the current full-network
path pays real NDNSF large-data dependency exchange cost, while the
single-provider layout avoids cross-provider dependency transfer.

Interpretation: for the current smallest Qwen NativeTracer artifacts,
multi-provider splitting is useful as a correctness and mechanism validation
path, but it is not the low-latency choice. The next step should not be to make
the payload artificially larger forever; it should be to introduce real
Qwen-like pipeline work or provider-capacity constraints where parallelism can
pay for the dependency exchange overhead.
