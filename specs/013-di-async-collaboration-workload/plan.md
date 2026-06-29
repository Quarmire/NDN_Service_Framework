# Plan: DI Async Collaboration Workload

## Approach

Expose the existing C++ asynchronous `ServiceUser::RequestCollaboration` path to
Python instead of adding a new protocol. The binding should mirror
`request_service_async`: start the background Face loop, post a collaboration
request into the Face io_context, and call Python callbacks on response or
timeout.

`user_driver.py` uses this binding when `--concurrency > 1`. It submits up to C
outstanding requests, records per-request latency from submission to callback,
and emits the same aggregate workload JSON as the closed-loop path.

## Steps

1. Factor or duplicate native collaboration plan construction in pybind.
2. Add `request_collaboration_async` to `NativeServiceUser` and bind it.
3. Add Python wrapper method.
4. Extend NativeTracer user driver with `--concurrency`.
5. Thread concurrency through the MiniNDN harness and campaign runner.
6. Validate with py_compile/build/dry-run and full-network smoke.
7. Run a small async campaign under 75 ms per-role capacity pressure.

## Expected Interpretation

If shared-backbone improves throughput or p95 under outstanding requests, this
supports the DI claim more directly than closed-loop sequential requests. If it
fails, inspect provider worker queues and SVS publication fetch behavior before
changing planner logic.
