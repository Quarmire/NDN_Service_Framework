# Tasks: Qwen ONNX NativeTracer

- [x] T001 Add a reproducible generator for tiny Qwen-derived ONNX artifacts.
- [x] T002 Update the NativeTracer policy to reference generated Qwen ONNX
  artifacts and tensor metadata.
- [x] T003 Switch full-network provider serve mode from deterministic runner to
  real `onnxruntime`.
- [x] T004 Validate local native execution with the generated ONNX artifacts.
- [x] T005 Validate full-network MiniNDN execution with
  `runnerMode=qwen-onnx-native`.
- [x] T006 Update project docs with evidence, commands, and the next gate.

## Validation

```bash
python3 -m py_compile \
  examples/python/NDNSF-DistributedInference/native_di_tracer/generate_qwen_native_tracer_artifacts.py \
  examples/python/NDNSF-DistributedInference/native_di_tracer/plan_tracer.py \
  Experiments/NDNSF_DI_NativeTracer_Minindn.py \
  examples/python/NDNSF-DistributedInference/native_di_tracer/user_driver.py

python3 examples/python/NDNSF-DistributedInference/native_di_tracer/generate_qwen_native_tracer_artifacts.py

PYTHONPATH=NDNSF-DistributedInference:pythonWrapper \
  python3 examples/python/NDNSF-DistributedInference/native_di_tracer/plan_tracer.py \
  --out /tmp/ndnsf-di-qwen-policy-check \
  --summary-json /tmp/ndnsf-di-qwen-policy-check/summary.json

./waf build

python3 Experiments/NDNSF_DI_NativeTracer_Minindn.py --quick-smoke

python3 Experiments/NDNSF_DI_NativeTracer_Minindn.py \
  --local-execution-only \
  --out /tmp/ndnsf-di-qwen-local

sudo -n python3 Experiments/NDNSF_DI_NativeTracer_Minindn.py \
  --full-network \
  --core-trace \
  --out /tmp/ndnsf-di-qwen-full-network \
  --assignment default \
  --provider-check-timeout 45
```

Full-network result:

```text
status=SUCCESS
runnerMode=qwen-onnx-native
securityBootstrap=executed
userExecution=executed
dependencyExecution=executed
```
