# Preflight

Source evidence: `specs/091-native-di-offered-load-baseline/evidence/`.

Final validation source commit: `dbb880c`.

Fixed validation controls:

```text
runtime profile: examples/di-native-tracer.runtime.json
topology: Experiments/Topology/AI_Lab.conf
model path: real Qwen ONNX NativeTracer
assignment: proportional 2/4/8 GB
target: 1 RPS
window: 60 seconds
requests: 60
concurrency: 4
```

Only driver mode and output path differ between matched post-fix treatments.

Command template:

```bash
sudo -n timeout 240s python3 Experiments/NDNSF_DI_NativeTracer_Minindn.py \
  --runtime-profile examples/di-native-tracer.runtime.json \
  --out results/spec092-native-di-user-driver-correctness/OUTPUT \
  --requests 60 --concurrency 4 --target-rps 1 \
  --open-loop-duration-s 60 --open-loop-driver-mode MODE \
  --provider-check-timeout 60 --no-local-execution-only --full-network \
  --skip-provider-pair-telemetry-probe
```

No stale MiniNDN/NFD/user/provider process was present before the treatments.
The result set uses about 51 MiB and does not create a disk-pressure confound.
