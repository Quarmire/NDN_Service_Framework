# Historical Execution-Evidence Correction (Spec 091–093)

**Recorded**: 2026-07-12  
**Correction schema**: `ndnsf-di-historical-evidence-correction-v1`  
**Raw-result policy**: append-only correction; no raw result or prior evidence file was edited

## Controlling Finding

The Spec 091–093 MiniNDN providers were launched with
`tracerDeterministicRunner=1`. That runner sleeps when configured and returns
synthetic 1x1 float tensors. The historical aggregate label
`runnerMode=qwen-onnx-native` was selected by the harness and was not observed
from an initialized model backend. Therefore every Spec 091–093 performance
cell is reclassified as `synthetic-delay`, `realCompute=false`.

This correction does not invalidate the measured NDNSF control/data-path
facts. It narrows what the measurements prove.

## Immutable Correction Records

| Record | Source commit | Historical claim surface | Correct execution class | Preserved scope | Withdrawn scope |
|---|---|---|---|---|---|
| spec091 | `30109fe` | child/threaded/process-pool 1 RPS screening summaries | `synthetic-delay` | user-driver scheduling, ACK/selection, MiniNDN transport, dependency exchange, queue observations | Qwen compute latency, Qwen throughput, model capacity |
| spec092 | `dbb880c` | threaded/process-pool correctness repetitions | `synthetic-delay` | scope-key lifecycle fix, measurement-window correctness, threaded scheduling behavior | real Qwen execution and provider compute capacity |
| spec093 | `7dd7de1`, `f950732`, `b9f47ab` | 1/2/4/8 RPS boundary summaries | `synthetic-delay` | offered-load submission, completion, ACK/selection, dependencies, queue/utilization instrumentation | stable Qwen RPS, Qwen p50/p95, Qwen capacity ceiling |

## Preserved Measurements

- Spec 091 retains the observed driver outcomes, including child 60/60,
  threaded failure, and process-pool 60/60. They compare driver behavior under
  the synthetic provider workload only.
- Spec 092 retains the three threaded 1 RPS repetitions (180/180 and mean
  1.013324 RPS) as scheduling and NDNSF-path evidence only.
- Spec 093 retains all raw counts, including 1440/1440 completions across the
  three 8 RPS repetitions, mean 7.984960 achieved RPS, and mean p95 247.552 ms.
  These are synthetic-runner end-to-end measurements, not Qwen model results.

## Gate Consequence

No Spec 091–093 run may satisfy Spec 105 `evidenceIntegrity`, correctness, or
performance gates for real compute. The raw artifacts remain valid diagnostic
inputs and regression references. A real-compute claim requires
`executionEvidence` emitted after backend/session initialization with matching
model, plan, artifact, provider-boot, runtime, and device bindings.

Canonical source records:

- `specs/091-native-di-offered-load-baseline/evidence/screening-results.md`
- `specs/092-native-di-user-driver-correctness/evidence/experiment-validation.md`
- `specs/093-native-di-threaded-rps-boundary/evidence/rps-results.md`
- `specs/105-ndnsf-di-deployment-readiness/evidence/baseline.md`
