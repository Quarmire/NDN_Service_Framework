# T094 Frozen 24-hour local soak preflight

Status: **NOT RUN / BLOCK**.

The soak precondition is the immutable T062 fixed 1 RPS gate. Its valid cell
completed 25/60 generations (41.6667%), achieved 0.4167 RPS against the >=0.95
requirement, and measured distributed p95 138,227.90 ms / baseline 6,854.20 ms
= 20.17x against the <=2.0x requirement. T062 verdict is `BLOCK`.

Per the frozen stop rule, the 24-hour, 1 RPS soak was not started, shortened,
reduced, retried or replaced. Correctness, completion, latency, resource growth,
restart interruption and application-security-path soak evidence therefore
remain unmeasured. The controlling record is
`telemetry-performance-check.md`; this honest skip keeps
`minindnCandidateOverall=BLOCK` and `physicalProductionOverall=DEFERRED`.
