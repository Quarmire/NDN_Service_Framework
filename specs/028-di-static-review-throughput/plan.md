# Feature 028: NDNSF-DI Static Review and Throughput Cleanup

## Goal

Audit NDNSF-DI statically for logic issues and low-risk throughput/latency
improvements, then complete the fixes that can be validated without changing
the proposal slides or widening the runtime protocol.

## Review Findings

### Finding 1: ACK Capacity Pressure Double Counts Queue State

`NativeProviderReadinessState::makeAckDecision()` publishes:

```text
queue = pendingWorkCount = readyQueue + waitingInputs + activeWorkers
```

The Python wrapper's role assignment selector then scored provider pressure as:

```text
queue + readyQueue + waitingInputs + activeWorkers
```

That double counts load in the current native provider ACK format. Under
high-concurrency admission, this can distort provider choice and reduce
multi-provider throughput.

Decision: treat `queue` as aggregate pressure when component fields are present.
Use component fields only when aggregate `queue` is absent.

### Finding 2: Ready Dependency Futures Still Create Waiter Threads

`ProviderRoleWorker::executeAsync()` prefetches all dependency inputs. If any
inputs exist, it always calls `scheduleWhenInputsReady()`, even when all futures
are already ready. That creates one waiter thread per dependent role. Fast
local/cache dependency paths should enter the ready queue immediately.

Decision: add a fast path that drains already-ready dependency futures and
enqueues the role directly when all inputs are available.

### Finding 3: Remaining Optimization Opportunities

These are worth future work, but are not required for this low-risk cleanup:

- Replace one waiter thread per pending role with a bounded dependency-wait
  scheduler.
- Feed NetworkTelemetry snapshots into role assignment once MiniNDN evidence is
  available.
- Add runtime integration for ACK-derived reusable LLM plans.

## Validation

- Unit test for ready dependency fast path.
- Build `pythonWrapper` extension so the ACK selection pressure change compiles.
- Build `unit-tests` target and run the DI async runtime test binary.
- Run `git diff --check` and CodeGraph sync/status.

