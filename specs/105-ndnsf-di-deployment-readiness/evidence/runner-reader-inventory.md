# T002 — Runner Evidence Producer/Reader Inventory

Inventory method: current CodeGraph index first, followed by exact `rg` searches
for `runnerMode`, `tracerDeterministicRunner`, `tracer_deterministic_runner`,
backend registration and provider-ready logs.

## Authoritative Producers Today

| Surface | Current behavior | Migration owner |
|---|---|---|
| `examples/DI_NativeProviderExecutable.cpp` | Selects real ONNX, wiring or deterministic factories; logs intent flags but emits no typed immutable evidence | T021-T023 |
| `NDNSF-DistributedInference/cpp/ndnsf-di/OnnxRuntimeModelRunner.cpp` | Initializes ONNX Runtime runner; backend/device evidence is incomplete | T021, T035-T036 |
| `Experiments/NDNSF_DI_NativeTracer_Minindn.py` | Launches providers and writes caller-selected aggregate `runnerMode` | T019, T024-T025 |
| `tools/ndnsf_runtime.py` | Loads `tracer_deterministic_runner` from runtime profiles | T028-T029 |

## Maintained Readers

- `Experiments/NDNSF_DI_NativeTracer_Minindn.py`: summary generation,
  post-processing and campaign result classification;
- `NDNSF-DistributedInference/ndnsf_distributed_inference/gui.py`: result/profile
  display;
- `NDNSF-DistributedInference/ndnsf_distributed_inference/runtime_v1.py`:
  runtime metadata and future operator surface;
- `tools/ndnsf_runtime.py`: profile validation/printing/launch;
- `examples/python/NDNSF-DistributedInference/native_di_tracer/run_layout_campaign.py`;
- `examples/python/NDNSF-DistributedInference/native_di_tracer/compare_layout_results.py`;
- `examples/python/NDNSF-DistributedInference/native_di_tracer/generate_llm_proportional_native_bundle.py`;
- `tests/python/test_ndnsf_di_tk_gui.py` and `test_ndnsf_di_tk_widgets.py`;
- `tests/unit-tests/distributed-inference-async-runtime.t.cpp`;
- maintained README/runtime workflow documents referenced by T028.

## Build and Smoke Consumers

- `examples/DI_NativeOnnxRuntimeSmoke.cpp`
- `examples/DI_NativePlanOnnxSmoke.cpp`
- `wscript` targets for native DI examples and unit tests

## Historical-Only References

Specs 005-008, 014, 040, 047, 056 and 085 describe earlier behavior. They are
evidence/history, not maintained runtime readers, and must not be rewritten to
pretend the old runs produced new evidence.

## Migration Rule

Add provider-observed `executionEvidence` and derived
`runnerClassification` first. Keep `runnerMode` only as a derived compatibility
field for one slice. T029 may remove it only after CodeGraph plus exact text
search finds no maintained caller-controlled reader.

## T029 Migration Audit — 2026-07-12

CodeGraph was synchronized and queried for current `runnerMode` summary readers.
Exact scans then found and migrated the remaining maintained readers in the GUI,
layout comparison, and layout campaign. The proportional bundle no longer
asserts a configured `runnerMode`; it emits the explicitly non-evidentiary
`configuredRunnerProfile=deterministic-fixture` instead.

Post-migration fixed-string scan (excluding raw results, historical specs,
third-party, build output, and proposal artifacts) leaves only:

- `Experiments/NDNSF_DI_NativeTracer_Minindn.py`: initialization and assignment
  of the deprecated field from the already-derived `runnerClassification`;
- `tests/python/test_ndnsf_runtime_doctor.py`: an adversarial legacy input and
  assertion that the evidence reader ignores it.

There are zero maintained readers and zero caller/configuration assignments.
The derived compatibility output remains for the one-slice compatibility
period required by `migration-and-rollback.md`; it is not accepted by the
release gate or any maintained report reader.
