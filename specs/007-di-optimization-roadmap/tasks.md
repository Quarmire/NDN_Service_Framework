# Tasks: DI Optimization Evidence Roadmap

- [x] P1 Add a `di-plan-v2` evidence contract while preserving existing
  NativeTracer compatibility fields.
- [x] P2 Add provider and network profile inputs for NativeTracer planning.
- [x] P3 Add a deterministic compute, transfer, and queue/load cost model.
- [x] P4 Generate multiple candidate layouts for the four-role graph.
- [x] P5 Select and explain the executable candidate used by the current
  runtime.
- [x] P6 Validate that the smallest Qwen NativeTracer artifacts remain the
  active model.
- [x] P7 Integrate optimization evidence into policy generation and MiniNDN
  summaries.
- [x] P8 Run validation commands for optimizer, plan generation, and evidence.
- [x] P9 Record accepted evidence and remaining next-step work.

## Validation

```bash
python3 -m py_compile \
  examples/python/NDNSF-DistributedInference/native_di_tracer/optimize_native_tracer_plan.py \
  examples/python/NDNSF-DistributedInference/native_di_tracer/plan_tracer.py \
  Experiments/NDNSF_DI_NativeTracer_Minindn.py

PYTHONPATH=NDNSF-DistributedInference:pythonWrapper \
  python3 examples/python/NDNSF-DistributedInference/native_di_tracer/plan_tracer.py \
  --out /tmp/ndnsf-di-optimization-policy \
  --summary-json /tmp/ndnsf-di-optimization-policy-summary.json

./waf configure --with-examples
./waf build --targets=di-native-provider,di-native-plan-schema-smoke,di-native-plan-manifest-smoke,di-native-provider-session-smoke

python3 Experiments/NDNSF_DI_NativeTracer_Minindn.py --quick-smoke

./build/examples/di-native-plan-schema-smoke \
  /tmp/ndnsf-di-optimization-policy/native-execution-plan.json \
  /Inference/NativeTracer yolo-onnx onnx yolo-detect-auto

./build/examples/di-native-plan-manifest-smoke \
  /tmp/ndnsf-di-optimization-policy/native-execution-plan.json \
  /tmp/ndnsf-di-optimization-policy/service-manifest.json \
  /Inference/NativeTracer

python3 Experiments/NDNSF_DI_NativeTracer_Minindn.py \
  --local-execution-only \
  --out /tmp/ndnsf-di-optimization-local

sudo -n python3 Experiments/NDNSF_DI_NativeTracer_Minindn.py \
  --full-network \
  --out /tmp/ndnsf-di-optimization-full-network \
  --assignment default \
  --provider-check-timeout 45
```

Accepted full-network result:

```text
status=SUCCESS
runnerMode=qwen-onnx-native
localExecution=executed
securityBootstrap=executed
userExecution=executed
dependencyExecution=executed
optimizationContractVersion=di-plan-v2
selectedCandidate=shared-backbone-current
candidateCount=5
```

## Next Step

The next large task should turn estimated candidates into executable planner
variants. In particular, implement one alternative placement, then compare its
measured MiniNDN latency against the current shared-backbone candidate.
