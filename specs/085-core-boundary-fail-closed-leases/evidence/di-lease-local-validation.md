# DI Lease Local Validation

Date: 2026-07-10

## Focused validation

```text
./build/unit-tests --run_test='GenericExecutionLeaseTable/*:DiExecutionLeaseService/*'
10 C++ cases passed

Python focused suites:
- Core binding parity: 3 passed
- cross-language codec: 3 passed
- distributed transaction: 4 passed
- restart/renew/release-loss/eviction: 6 passed
- harness integration: 3 passed
- removed fallback: 2 passed
```

## Concurrent stress

```bash
PYTHONPATH=NDNSF-DistributedInference:. \
  python3 tests/python/test_ndnsf_di_execution_lease_stress.py -v
```

Result: 1,000 successful transactions, 16 client threads, 8 conflict-key
slots, 1,000 prepare/commit/release operations, zero overlapping committed
slot use, and zero active pins after completion. Runtime was approximately
0.96 seconds with maximum RSS 70,856 KiB.

## Network smoke

```bash
sudo -n python3 Experiments/NDNSF_DI_NativeTracer_Minindn.py \
  --out results/spec085-lease-smoke-20260710 \
  --assignment default --tracer-deterministic-runner \
  --requests 1 --concurrency 1 --provider-check-timeout 60 \
  --no-local-execution-only --full-network \
  --enable-execution-leases --skip-provider-pair-telemetry-probe
```

Result: `SUCCESS`; four providers committed provider-local leases, all four
NativeTracer roles executed, response succeeded, dependency execution was
executed, and user request elapsed time was 1,163.244 ms. Coordinator was off.
