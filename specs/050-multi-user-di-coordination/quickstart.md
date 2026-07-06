# Quickstart: Multi-User DI Coordination Hardening

## Fragment-Aware Advisory Coordinator

Start coordinator with fragment state awareness:

```bash
python3 examples/python/NDNSF-DistributedInference/native_di_tracer/advisory_coordinator.py \
  --state-file /tmp/ndnsf-coordinator-state.json
```

## State Persistence

Coordinator persists rolling state after each window:

```bash
cat /tmp/ndnsf-coordinator-state.json
# {"providerUse": {"/P/a": 5}, "providerAvailableAtMs": {...}, "windowVersion": 12, ...}
```

## Priority Mode

Enable intent priority ordering:

```bash
python3 examples/python/NDNSF-DistributedInference/native_di_tracer/advisory_coordinator.py \
  --enable-priority
```

## MiniNDN with Fragment State

```bash
sudo -n PYTHONPATH=NDNSF-DistributedInference:pythonWrapper:Experiments \
  python3 Experiments/NDNSF_DI_NativeTracer_Minindn.py \
  --out /tmp/ndnsf-spec050-run \
  --advisory-coordinator \
  --runtime-aware-user-planner \
  --requests 4 --concurrency 1 \
  --full-network --tracer-deterministic-runner \
  --enable-native-admission-lease \
  --assignment capacity-pool \
  --provider-check-timeout 60
```

## Comparison Sweep

```bash
sudo -n PYTHONPATH=NDNSF-DistributedInference:pythonWrapper:Experiments \
  python3 Experiments/NDNSF_DI_RuntimeAware_RpsSweep.py \
  --compare-advisory-coordinator \
  --out /tmp/ndnsf-spec050-sweep \
  --rps 0.2,0.4,0.8 \
  --requests 10 --concurrency 2 --capacity-pool \
  -- --provider-check-timeout 60
```

## Key Metrics

- `NDNSF_DI_ADVISORY_COORDINATOR_FRAGMENT_STATE` — emitted when fragment state received
- `fragmentStateSize` in `COORDINATOR_REQUEST` log
- `fragmentReadyPenaltyMs` in score detail per candidate
- `providerFragmentInventory` in summary.json
- `fragment-inventory.json` written to result directory
