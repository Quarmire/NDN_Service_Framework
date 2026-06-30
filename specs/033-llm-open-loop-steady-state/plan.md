# Feature 033: LLM Open-Loop Steady-State Campaign

## Goal

Add a true open-loop NativeTracer user workload for NDNSF-DI LLM full-network
experiments. The previous capacity-pressure campaign submitted a fixed request
batch and therefore did not create sustained provider backlog. This feature lets
the user driver submit requests at a target offered rate for a fixed duration,
while recording success rate, latency, throughput, local backpressure, and
provider queue evidence.

## Design

- Extend `user_driver.py` with an explicit open-loop mode:
  - `--target-rps` controls the fixed request schedule.
  - `--open-loop-duration-s` controls the measured submission window.
  - `--requests` remains a hard cap when positive.
  - `--concurrency` remains the maximum local outstanding request limit.
- Use the existing child-process ServiceUser request path for open-loop mode.
  The same-process async path was tested first but the full-network user process
  exited after the first async submission without emitting final execution JSON,
  so the robust measurement path keeps the per-request ServiceUser isolation
  already used by concurrent runs.
- Count requests that cannot be submitted because local outstanding requests are
  full as `local-open-loop-backpressure` failures.
- Thread the new knobs through the MiniNDN harness and LLM campaign runner.
- Preserve closed-loop and child-process behavior when open-loop mode is not
  requested.

## Validation

- Compile the changed Python scripts.
- Run a short local/dry syntax path and a small MiniNDN full-network smoke.
- Run at least one greedy/proportional open-loop campaign with a small duration
  and controlled stage delay.
- Record result paths and interpretation in `tasks.md`.

## Interpretation

This feature measures offered-load behavior. Child-process startup is included
in the current latency numbers, so the campaign is best used for end-to-end
stability and offered-load/backpressure evidence. It does not claim proportional
LLM splitting wins by itself; it provides the workload shape needed to test
whether provider backlog appears and whether proportional placement can reduce
that backlog under sustained load.
