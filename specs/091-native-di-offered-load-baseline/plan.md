# Implementation Plan: Native DI Offered-Load Baseline

## Constitution Check

- **Security**: PASS. The real permission, NAC-ABE, token, replay, and
  collaboration path remains enabled.
- **MiniNDN final validation**: PASS. Every measured treatment uses MiniNDN.
- **Evidence before optimization**: PASS. FR-007 blocks production edits until
  the limiting layer is measured.
- **Honest negative results**: PASS. FR-008 forbids improvement claims from
  failed or inconclusive screening.
- **Core/application boundary**: PASS. The experiment changes no ownership or
  wire contract.

## Design

Use `Experiments/NDNSF_DI_NativeTracer_Minindn.py` directly so all treatments
share one controller/provider/user harness. The screening point is 1 RPS for
60 seconds with concurrency 4 and request cap 60. This point is above the old
0.2 RPS local-backpressure observation while remaining conservative relative
to the current short-run p95.

Fixed controls:

```text
runtime profile: examples/di-native-tracer.runtime.json
topology: Experiments/Topology/AI_Lab.conf
assignment/policy: llm-proportional / proportional
runner: Qwen ONNX NativeTracer
target: 1 RPS
window: 60 seconds
requests: 60
concurrency: 4
provider telemetry probe: skipped equally in all screening runs
```

## Analysis

Classify the first boundary in this order:

1. user scheduling: submitted ratio, schedule slip, local backpressure;
2. NDNSF control: ACK/selection/timeout/negative-ACK counters;
3. provider queue/admission: queue, active workers, rejection reasons;
4. dependency exchange: expected versus completed dependency objects;
5. model execution: role timing and provider busy duration.

One run per mode is a deterministic screening comparison, not statistical
performance evidence. The selected driver must later receive at least three
matched repetitions before any paper-facing claim.

## Stop Conditions

- permission/bootstrap/security failure unrelated to the treatment;
- wrong model, topology, request count, or duration;
- host resource exhaustion or stale MiniNDN process;
- result schema lacks the counters needed to distinguish the boundary.

## Outputs

- `results/spec091-native-di-offered-load-baseline/<mode>/summary.json`
- `evidence/screening-results.md`
- `evidence/experiment-validation.md`
- a go/no-go decision for implementation work.

