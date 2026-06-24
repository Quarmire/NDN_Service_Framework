# Quickstart: Native DI Tracer

## Goal

Validate the native DI tracer from policy generation through C++ smoke/unit checks and, when available, MiniNDN evidence.

## Commands

```bash
cd /home/tianxing/NDN/ndn-service-framework

# 1. Generate the tracer policy bundle once T002 is implemented.
python3 examples/python/NDNSF-DistributedInference/native_di_tracer/plan_tracer.py \
  --out /tmp/ndnsf-di-native-tracer

# 2. Validate generated manifest and native execution plan.
build/examples/di-native-plan-schema-smoke \
  /tmp/ndnsf-di-native-tracer/native-execution-plan.json \
  /Inference/NativeTracer yolo-onnx onnx yolo-detect-auto

build/examples/di-native-plan-manifest-smoke \
  /tmp/ndnsf-di-native-tracer/native-execution-plan.json \
  /tmp/ndnsf-di-native-tracer/service-manifest.json \
  /Inference/NativeTracer \
  --timing-csv /tmp/ndnsf-di-native-tracer/timing.csv

# 3. Run focused DI unit tests.
./build/unit-tests -t distributed-inference-async-runtime

# 4. Run tracer evidence harness. By default it records MiniNDN availability
# and validates native plan/provider execution; add --require-minindn to fail
# when MiniNDN/root conditions are not available.
examples/python/NDNSF-DistributedInference/native_di_tracer/run_minindn_tracer.sh \
  --out results/native_di_tracer/latest
```

## Expected Results

- Policy bundle contains generated manifest, native plan, and sha256 sidecars.
- C++ smoke checks accept generated files without manual edits.
- Unit tests pass.
- Result directory contains logs, `timing.csv`, `summary.txt`, policy bundle, and `SUCCESS`.
- `summary.txt` records `miniNDNStatus`; use `--require-minindn` for hard MiniNDN gating.
