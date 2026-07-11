# Coordinator-Off Fail-Closed Lease Acceptance

**Date**: 2026-07-10
**Implementation commit**: `3918c98`

## Command

Each `N` in 1, 2, and 3 used:

```bash
sudo -n env PYTHONHASHSEED=0 \
  PYTHONPATH=NDNSF-DistributedInference:pythonWrapper:Experiments \
  python3 Experiments/NDNSF_DI_RuntimeAware_RpsSweep.py \
    --out results/spec085-lease-acceptance-20260710/run-N \
    --rps 0.2 --requests 12 --concurrency 2 \
    --open-loop-duration-s 60 \
    --workload specs/085-core-boundary-fail-closed-leases/fixtures/capacity-overlap-multi-user.json \
    --capacity-pool --disable-native-admission-lease -- \
    --provider-check-timeout 60 --role-execution-delay-ms 200 \
    --enable-execution-leases
```

The old native admission lease was disabled so the campaign measured the new
Core execution lease rather than two overlapping capacity mechanisms.
Advisory coordination remained off.

## Results

| Run | Success | p50 ms | p95 ms | Throughput RPS |
|---|---:|---:|---:|---:|
| 1 | 12/12 | 1159.512 | 7020.292 | 0.163097 |
| 2 | 12/12 | 1204.304 | 5527.735 | 0.174876 |
| 3 | 12/12 | 1172.652 | 5606.488 | 0.175131 |

Aggregate completion was 36/36. Median p50 was 1172.652 ms and median p95 was
5606.488 ms. The frozen coordinator-on thresholds were 6203.676 ms p50 and
6487.845 ms p95 (110% of baseline), so both median gates passed. Completion
remained 100%, a zero percentage-point decrease.

Every accepted run executed 12 sessions on each of Backbone, Head 0, Head 1,
and Merge. Alternate providers executed zero sessions. Measured provider busy
utilization was approximately 0.3% to 3.9% over each 60-second window; the
fixture's advertised GPU/RAM capacities were 12/32 GiB, 8/24 GiB, 8/24 GiB,
and 6/16 GiB. No conflicting execution was observed, no request executed before
all provider commits, and all final active lease counts converged to zero via
provider-local completion, cleanup, or bounded expiry.

## Negative Evidence And Corrections

Superseded diagnostic runs remain under the same result root. They exposed and
were used to fix: missing Repo wrapper path, whole-collaboration slot holding,
redundant admission plus execution leases, transient Targeted control loss,
later-request starvation, and stale absolute reservation expiry. A preserved
diagnostic also contains one SVS collaboration publication timeout; it was not
misclassified as a lease conflict.

The final design uses provider-local role completion to release capacity,
bounded FIFO ordering per conflict key, idempotent control retries, and a fresh
pre-activation TTL on each legitimate capacity retry. The user retains failure
cleanup, while the execution hard deadline remains the crash/lost-release
backstop.

