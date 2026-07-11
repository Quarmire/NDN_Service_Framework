# Research And Experiment Rationale

## Hypotheses

- H1: process-pool or threaded mode will reduce local schedule slip relative to
  the older child-process-per-request path.
- H2: if all modes submit the offered load but completion drops, the first
  boundary is downstream of the driver.
- H3: if 1 RPS is stable for every mode, this screening point is too low to
  reveal capacity and the next step is a matched rate search using the best
  scheduling-capable driver.

## Variables

- Independent variable: open-loop driver mode.
- Primary dependent variables: submitted ratio, schedule slip, local
  backpressure, success rate, achieved RPS, p95.
- Diagnostic dependent variables: negative ACKs, provider utilization,
  dependency completion, queue/worker counters, role timing.
- Controls: commit, topology, Qwen artifacts, plan, target rate, duration,
  concurrency, request cap, security, logging, and timeouts.

## Interpretation Limits

This is a screening experiment with one run per treatment. It can identify a
gross local-driver failure or select a driver for replication. It cannot support
confidence intervals, maximum stable RPS, or a paper-facing superiority claim.

