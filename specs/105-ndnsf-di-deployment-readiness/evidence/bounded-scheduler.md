# Bounded Dependency Wait Scheduler

**Task**: T077  
**Date**: 2026-07-12  
**Verdict**: PASS

## Frozen Command

```bash
/usr/bin/time -v ./build/unit-tests \
  --run_test='DependencyWaitSchedulerBoundsOneThousandWaitsAndCompletesOnce,DependencyWaitSchedulerRejectsOverflowExpiresAndCancels,DependencyWaitSchedulerShutdownCancelsPendingWork' \
  --log_level=error
```

Exit status: 0; 3/3 cases passed.

## Results

| Property | Observed |
|---|---:|
| Submitted waits | 1,000 |
| Fixed wait workers | 4 |
| Peak active waiters | <=4 |
| Active + queued after admission | 1,000 |
| Completed callbacks | 1,000, each unique and exactly once |
| Remaining active after release | 0 |
| Remaining queued after release | 0 |
| Queue overflow fixture | explicit `DEPENDENCY_WAIT_SCHEDULER_OVERLOAD` |
| Cancellation fixture | 2/2 cancelled |
| Deadline fixture | 1/1 `DEADLINE_EXPIRED` |
| Shutdown fixture | 16/16 terminal callbacks; zero active/queued |
| Post-shutdown admission | `SHUTTING_DOWN` |
| Wall time | 0.06 s |
| Maximum RSS | 21,836 KiB |
| Voluntary / involuntary context switches | 122 / 34 |
| Swaps | 0 |

The test checks scheduler-owned thread count rather than inferring it from host
process totals: exactly four workers serve all 1,000 pending waits. The old
one-thread-per-wait vector no longer exists. Every completion path removes its
job from the authority map before idle is reported, so the final zero state is
also the state-cleanup assertion.

This is a local deterministic C++ stress result. It validates bounded runtime
growth; it is not a physical-network or throughput result.
