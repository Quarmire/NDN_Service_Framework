# Plan: DI Dependency-Ready Provider Scheduling

## Design

Keep the one-worker-per-provider capacity model, but split provider work into
two phases:

1. **Dependency wait phase**: start dependency prefetch outside the compute
   worker and wait until all required input bundles are available.
2. **Compute/publish phase**: enqueue the role on the provider worker only after
   inputs are ready. The worker then runs the model runner and publishes outputs.

This means a provider can have several outstanding requests waiting for remote
dependencies, while its single worker remains available for whichever role
becomes executable next.

## Implementation Steps

1. Add a regression test where a consumer role is submitted before its producer
   with `workerCount=1`. The producer must still run and unblock the consumer.
2. Refactor `ProviderRoleWorker` so `executeAsync()` starts input prefetch before
   compute queue admission.
3. Preserve timing evidence:
   - input timing records dependency wait;
   - worker queue wait measures ready-to-compute queueing;
   - runner/publish timing remains compute-worker timing.
4. Run focused C++ unit tests.
5. Run Python script syntax checks.
6. Run one MiniNDN concurrent NativeTracer smoke.
7. Update experiment docs with the concurrency semantics and evidence path.

## Validation Commands

```bash
./build/unit-tests -t DistributedInference/ProviderRoleWorkerDoesNotOccupyComputeWorkerWhileWaitingForInputs
./build/unit-tests -t DistributedInference
python3 -m py_compile Experiments/NDNSF_DI_NativeTracer_Minindn.py examples/python/NDNSF-DistributedInference/native_di_tracer/*.py
sudo -n python3 Experiments/NDNSF_DI_NativeTracer_Minindn.py --full-network --assignment default --role-execution-delay-ms 75 --requests 4 --concurrency 4 --out /tmp/ndnsf-di-024-ready-scheduler-smoke --provider-check-timeout 60
```

## Risk

The dependency wait phase can create more pending waiters. This is acceptable
for the current NativeTracer campaign sizes, but future production work should
add admission limits for pending dependency waits.
