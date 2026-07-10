# Regression And Coordinator-On Baseline

**Date**: 2026-07-10
**Baseline commit**: `49d6f36`

## Parent Regression Baseline

The parent 084 baseline was rerun before 085 production edits:

| Area | Command | Result |
|---|---|---|
| Core C++ | `./waf build --targets=unit-tests -j4 && ./build/unit-tests --log_level=test_suite` | 199 passed; environment-gated real ONNX fixtures skipped |
| Core Python | `python3 -m unittest discover -s tests/python -p 'test_ndnsf_core*.py' -v` | 24 passed |
| Security | `examples/run_security_regressions.sh` | six suites passed |
| DI Python | `python3 -m unittest discover -s tests/python -p 'test_ndnsf_di*.py' -v` | 152 passed, one optional `pyautogui` skip |
| Repo Python | `python3 -m unittest discover -s tests/python -p 'test_ndnsf_repo*.py' -v` | 80 passed |
| Repo C++ | `DistributedRepoExactPacketTest`, `DistributedRepoTieredCacheTest`, `DistributedRepoHaTest` | all three passed |
| Targeted | `./build/unit-tests --run_test='GenericDynamicApi/TargetedInvocation/*'` plus `test_ndnsf_targeted_python_api.py` | 14 C++ and 2 Python passed |

The exact parent command index and per-area details remain in
`specs/084-ndnsf-occam-simplification/evidence/`.

## Frozen Coordinator-On Campaign

Three matched MiniNDN runs used:

```bash
sudo -n env PYTHONHASHSEED=0 \
  PYTHONPATH=NDNSF-DistributedInference:pythonWrapper:Experiments \
  python3 Experiments/NDNSF_DI_RuntimeAware_RpsSweep.py \
    --out results/spec085-coordinator-baseline-20260710/run-N \
    --rps 0.2 --requests 12 --concurrency 2 \
    --open-loop-duration-s 60 \
    --capacity-pool --advisory-coordinator-only -- \
    --provider-check-timeout 60 --role-execution-delay-ms 200
```

Frozen inputs:

- topology: `Experiments/Topology/AI_Lab.conf`;
- workload: `runtime_aware_fixtures/multi_user_requests.json`;
- assignment: `capacity-pool`;
- planner: runtime-aware with one `FRAGMENT_EVICTED` replan;
- native admission lease: enabled;
- runner: Qwen NativeTracer ONNX deterministic execution;
- seed control: `PYTHONHASHSEED=0` for all runs;
- measured open-loop window: 60 seconds.

## Results

| Run | Success | p50 ms | p95 ms | Lease granted/consumed/rejected | Throughput RPS |
|---|---:|---:|---:|---:|---:|
| 1 | 12/12 | 5638.338 | 5898.041 | 48/48/0 | 0.126590 |
| 2 | 12/12 | 5639.705 | 5855.916 | 48/48/0 | 0.126656 |
| 3 | 12/12 | 5648.534 | 5901.011 | 48/48/0 | 0.126458 |

Aggregate: 36/36 successful, zero timeout/negative ACK/rejection/expiry,
median p50 5639.705 ms, median p95 5898.041 ms, p50 sample standard deviation
5.534 ms, p95 sample standard deviation 25.222 ms, and 144/144 admission
leases consumed.

The T003 frozen test also confirms that the generic Python deployment lease API
is currently broken before it can grant a fallback: the advisory service
constant is missing, and after supplying that constant the coordinator-failure
path imports a nonexistent `ExecutionLease`. The `GRANTED_LOCAL` code is present
but unreachable in the current module state. This is the baseline T026 must
replace with a fail-closed DI-owned transaction.

Raw summaries and logs are under
`results/spec085-coordinator-baseline-20260710/`. Each run contains
`rps-sweep-commands.json`, `rps-sweep-summary.json`, the underlying
`summary.json`, planner metrics, policy bundle, and process logs.

## Environment Note

The launcher printed `module compiled against API version 0xe but this version
of numpy is 0xd` while probing an optional Python module. All three runs still
started MiniNDN, executed the Qwen ONNX NativeTracer path, completed dependency
exchange, produced successful summaries, and cleaned up the topology. This is
recorded as an environment warning, not suppressed or counted as a run failure.
