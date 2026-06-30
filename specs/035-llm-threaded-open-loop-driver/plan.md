# Feature 035: LLM Threaded Open-Loop Driver

## Goal

Remove the current open-loop campaign's artificial child-process driver
bottleneck. Feature 034 showed that above 1 RPS the robust child-process user
driver submits at most `concurrency` requests and records the rest as local
backpressure. That does not measure provider or layout capacity. This feature
first tested a lighter in-process threaded driver, then moved to a long-lived
process-pool driver after threaded ServiceUser reuse failed in MiniNDN.

## Design

- Add `--open-loop-driver-mode child|threaded|process-pool` to `user_driver.py`.
- Keep `child` as the default for compatibility and robustness.
- Implement `threaded` mode as an experimental diagnostic using synchronous
  `request_collaboration` calls:
  - create `concurrency` worker `ServiceUser` objects with identities
    `<base-user>/worker/<n>`;
  - reuse each worker for many sequential requests;
  - schedule requests by `target-rps` and `open-loop-duration-s`;
  - dispatch only to idle workers;
  - count no-idle-worker events as `local-open-loop-backpressure`.
- Implement `process-pool` mode for campaign use:
  - create one long-lived worker process per concurrency slot;
  - give each worker a stable `<base-user>/worker/<n>` identity and copied NDN
    client state;
  - assign fixed-rate request indices across workers;
  - start all workers before the schedule begins;
  - parse per-request JSON records from worker logs and keep missing records as
    explicit worker failures.
- Thread the mode through the MiniNDN harness and LLM campaign runner.
- Validate process-pool with a short MiniNDN 2 RPS smoke, then rerun a small RPS
  sweep.

## Validation

- Compile changed Python scripts.
- Run a dry CLI sanity check.
- Run a short MiniNDN open-loop threaded smoke and record the failure mode.
- Run a short MiniNDN open-loop process-pool smoke.
- Compare child vs process-pool behavior at the same rate to confirm the driver
  bottleneck moves or to identify the next bottleneck.

## Interpretation

Threaded mode is a diagnostic, not a production API promise. The MiniNDN smoke
showed that multiple in-process ServiceUsers did not produce a final workload
record. The process-pool mode keeps the useful reuse property while isolating
each ServiceUser in its own process, which is a better fit for the current
native binding and NDN client state.
