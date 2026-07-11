# Implementation Plan: Native DI Threaded RPS Boundary

## Constitution Check

- MiniNDN is the final validation environment.
- Security/application path, Qwen artifacts, topology, and placement remain
  fixed.
- Negative results and stopped treatments remain evidence.
- This feature performs no production code change.

## Experimental Design

Independent variable: target offered RPS. Dependent variables: schedule slip,
submission ratio, completion, achieved throughput, latency, provider state,
and dependency completion. Fixed controls are listed in FR-001.

Search algorithm:

1. Reuse Spec 092's three 1 RPS runs as stable anchor.
2. Run 2, 4, 8 RPS in ascending order, stopping at first unstable point.
3. Bisect between highest stable and first unstable to 0.25 RPS width.
4. Obtain three total matched runs at the highest stable tested point.

Every treatment uses:

```bash
sudo -n timeout 300s python3 Experiments/NDNSF_DI_NativeTracer_Minindn.py \
  --runtime-profile examples/di-native-tracer.runtime.json \
  --out results/spec093-native-di-threaded-rps-boundary/RATE-RUN \
  --requests REQUESTS --concurrency 4 --target-rps RATE \
  --open-loop-duration-s 60 --open-loop-driver-mode threaded \
  --provider-check-timeout 60 --no-local-execution-only --full-network \
  --skip-provider-pair-telemetry-probe
```

## Stop Conditions

- stale MiniNDN/NFD process or source-code drift;
- disk below 3 GiB or host resource exhaustion;
- bootstrap/security/model/topology failure unrelated to offered load;
- first unstable coarse point, before midpoint refinement;
- outer 300-second timeout.

## Analysis

Apply gates per point before inspecting higher rates. Report individual runs,
aggregate repeated-point statistics, and the first counter that violates a
gate. Do not substitute adjusted metrics after seeing a failure.

## Rollback

No runtime change exists to roll back. Results and Spec 093 documents can be
removed independently without affecting executable behavior.
