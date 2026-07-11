# Implementation Plan: Native DI User Driver Correctness

## Constitution Check

- **Security and network path**: unchanged; the real permission, NAC-ABE,
  token, replay, collaboration, and large-data paths remain enabled.
- **Application/core boundary**: PASS. The defect is in the DI experiment user
  driver, not NDNSF core or provider runtime.
- **Test first**: lifecycle and measurement contracts receive focused failing
  tests before implementation.
- **MiniNDN final validation**: threaded and process-pool use the same real Qwen
  full-network fixture as Spec 091.
- **Honest evidence**: failed or inconclusive reruns remain first-class results.

## Design

### Base Publisher Lifecycle

Introduce one small helper that starts a `ServiceUser`, runs a callback, and
always stops it. Use it around the threaded workload so the base user continues
serving the scope-key Data published before worker creation. Do not start a
second event loop on each synchronous worker user unless evidence requires it.

### Process-Pool Timing

Each worker already receives `schedule_start_epoch` and its request indices.
Immediately before each request, compute:

```text
target = schedule_start_epoch + (request_index - 1) / target_rps
scheduleSlipMs = max(0, actual_start - target) * 1000
```

Attach the value to the request result. The parent aggregates maximum slip and
records:

```text
measurementStartEpoch
measurementElapsedMs = completion_epoch - measurementStartEpoch
maxScheduleSlipMs
```

`summarize_workload` uses `measurementElapsedMs` as the throughput denominator
when present, while retaining the full driver makespan separately.

## Validation

1. Focused unit tests for lifecycle cleanup and deterministic timing math.
2. Existing runtime-aware campaign and runtime-profile tests.
3. Python test suite if focused tests pass.
4. Matched 60-second MiniNDN reruns for threaded and process-pool.
5. Three process-pool repetitions only if the post-fix screening point passes
   the Spec 091 scheduling and stability gates.

## Rollback

The change is local to the experiment driver and tests. Revert the helper and
metadata fields without wire/schema migration. Raw Spec 091 results remain the
pre-fix baseline.
