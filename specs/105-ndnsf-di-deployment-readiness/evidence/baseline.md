# T001 — Controlling Evidence Baseline

**Recorded**: 2026-07-12  
**Current source commit**: `13da7fb61e0066825ec29b0fda1c11b7e8cda8e0`  
**Historical run commits**: `7dd7de1`, `f950732`, `b9f47ab`

## Preserved Fact

Spec 093 summaries label the tested runtime `runnerMode=qwen-onnx-native`, for
example:

```text
results/spec093-native-di-threaded-rps-boundary/rps-8-run3/summary.txt:13
runnerMode=qwen-onnx-native
```

The same run's provider commands include `--tracer-deterministic-runner`, and
provider startup records:

```text
NDNSF_DI_NATIVE_PROVIDER_BACKENDS_READY onnxruntime=1 wiringCheckOnly=0 tracerDeterministicRunner=1
```

The controlling code facts are:

- `Experiments/NDNSF_DI_NativeTracer_Minindn.py:3130-3135` forces the
  deterministic runner whenever `policy_bundle == "llm-proportional"`;
- the same harness assigns `summary["runnerMode"] = "qwen-onnx-native"` at
  lines 3262-3268;
- `examples/DI_NativeProviderExecutable.cpp:235-262` implements the deterministic
  runner as an optional sleep followed by synthetic 1x1 float tensors;
- the provider registers that runner under the `onnxruntime` backend name at
  lines 787-806.

## Correct Interpretation

The Spec 093 1/2/4/8 RPS results remain measured evidence for the threaded user
driver, ACK/selection, collaboration scheduling, dependency publication/fetch,
provider queues and MiniNDN application-security path. They are **not** measured
Qwen model-compute throughput and cannot establish model capacity.

Preserved measured values include 1440/1440 completed requests across the three
8 RPS repetitions, mean 7.9850 achieved RPS and mean p95 247.552 ms. These
numbers are not deleted or changed; their compute classification is corrected.

Canonical evidence:

- `specs/093-native-di-threaded-rps-boundary/evidence/rps-results.md`
- `specs/093-native-di-threaded-rps-boundary/evidence/experiment-validation.md`
- `results/spec093-native-di-threaded-rps-boundary/rps-2-run1`
- `results/spec093-native-di-threaded-rps-boundary/rps-4-run1`
- `results/spec093-native-di-threaded-rps-boundary/rps-8-run1`
- `results/spec093-native-di-threaded-rps-boundary/rps-8-run2`
- `results/spec093-native-di-threaded-rps-boundary/rps-8-run3`

This record is immutable input to T027. Raw result directories are not edited.
