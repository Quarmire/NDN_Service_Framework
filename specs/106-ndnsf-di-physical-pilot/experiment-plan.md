# Physical Pilot Experiment Plan

## Material Passport

- Origin: Spec 105 physical-acceptance migration
- Date: 2026-07-12
- Verification Status: DEFERRED
- Primary question: Does the immutable passing MiniNDN candidate remain correct,
  secure, bounded and operable on the fixed three-node physical profile?

## Frozen Cells

1. candidate digest and host preflight;
2. positive real-identity bootstrap/inference;
3. forged trust, token replay, provider-token replay, wrong digest and stale
   attempt negative cells;
4. matched physical single-node/distributed canary;
5. provider restart and same-three-node fallback;
6. two upgrade/rollback drills;
7. one 24-hour 1 RPS soak with a scheduled restart.

Every cell uses a unique directory and retains failures. There are no automatic
replacement runs. The soak reports every request plus completion interval,
p50/p95/p99, TTFT, inter-token latency, stage compute/fetch/publish/queue time,
GPU memory/utilization, cache events, restart interruption and resource growth.

## Falsifiers

- candidate digest or profile drift;
- any incorrect token or unexplained baseline mismatch;
- any security bypass or secret/payload leakage;
- telemetry older than two seconds accepted at commit;
- duplicate authoritative outcome or accepted stale boot/attempt/KV state;
- authoritative Repo loss on upgrade/rollback;
- unbounded thread/memory/disk growth;
- completion below 99% or distributed p95 above 2.0x matched single-node p95.

## Evidence Rule

`physicalProductionOverall=PASS` requires every frozen artifact. Missing
hardware, operator, cell or soak interval produces BLOCK. MiniNDN artifacts are
prerequisites, not substitutes for physical evidence.
