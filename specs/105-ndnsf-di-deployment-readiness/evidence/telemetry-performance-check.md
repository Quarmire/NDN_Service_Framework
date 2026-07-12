# Revision R1 Telemetry And Performance Check

**Task**: T062  
**Date**: 2026-07-12  
**Source commit**: `efaaa41ffc95bad13aa6b716341556f2867bf257`  
**Campaign ID**: `spec105-r1-qwen-scheduler-v1`  
**Verdict**: **BLOCK**

## Execution Integrity

T052-T061 were checked complete before measurement. The exact three-repetition
loop preregistered in `qwen-scheduler-revision.md` was invoked once. The three
prespecified cells all started with the same UTC stamp; no fourth run was
executed and no runtime parameter changed.

| Repetition | Result directory | Harness exit | Classification |
|---|---|---:|---|
| 1 | `results/spec105-r1-qwen-pilot-run1-20260712T111141Z` | 2 | valid load cell; acceptance failed |
| 2 | `results/spec105-r1-qwen-pilot-run2-20260712T111141Z` | 1 | invalid infrastructure cell; disk full during stage-2 export |
| 3 | `results/spec105-r1-qwen-pilot-run3-20260712T111141Z` | 1 | invalid infrastructure cell; disk full during stage-0 export |

The frozen shell initially could not write `outer-exit-code.txt` because sudo
created each result directory as root. The observed harness exit codes above
were added afterward with root permission; this did not rerun or modify a
measurement. Repetitions 2 and 3 emitted `free space ... is 0.00 GiB` from
`_safe_torch_save` and never reached MiniNDN load.

## Valid Repetition 1

The open-loop summary recorded:

```text
offered=60 completed=25 failed=0 unfinished=35 completionRate=0.416667
offeredRps=1.000000 p50_ms=93094.66 p95_ms=138227.90 p99_ms=155265.31
campaignId=spec105-r1-qwen-scheduler-v1 generationWorkers=4
activeAtCutoff=4 queuedAtCutoff=31 maxActiveObserved=4 maxQueuedObserved=48
schedulerCompleted=25 schedulerFailed=0
```

Queue accounting is internally consistent: 25 completed + 4 active + 31
queued = 60 offered. The four active sessions had monotonic progress of
31/30/29/0 tokens at cutoff; the 25 completed sessions each reached 32 tokens.

### Fixed Acceptance Thresholds

| Check | Required | Observed | Result |
|---|---:|---:|---|
| Exact generated tokens | every completed generation | 25/25 exact; zero `TOKEN_MISMATCH`; `schedulerFailed=0` | PASS |
| Completion | >=99% | 41.6667% | FAIL |
| Achieved throughput | >=0.95 generation/s | 25/60 = 0.4167 generation/s | FAIL |
| Distributed p95 / matched baseline p95 | <=2.0x | 138,227.90 / 6,854.20 = 20.17x | FAIL |

The matched baseline remains the previously frozen local ONNX CPU p95 of
6,854.20 ms; no new baseline was selected after seeing this result.

## Measured Telemetry Perturbation

ACK payloads kept configured capability separate from typed measured telemetry.
The measured source was `linux-proc`, not a configured 2/4/8 GB profile and not
physical GPU evidence.

| Provider stage | Samples decoded | First -> last sequence | Host available bytes | Process RSS bytes | Completed stages | Final service EWMA |
|---|---:|---:|---:|---:|---:|---:|
| 0 | 930 | 21 -> 205 | 6,274,949,120 -> 2,222,968,832 | 1,479,229,440 -> 1,586,397,184 | 0 -> 890 | 1 ms; 1000/s |
| 1 | 930 | 18 -> 203 | 6,279,401,472 -> 2,220,900,352 | 934,985,728 -> 1,133,735,936 | 0 -> 890 | 35.3297 ms; 28.8086/s |
| 2 | 933 | 16 -> 200 | 6,276,513,792 -> 2,221,383,680 | 1,479,073,792 -> 2,698,297,344 | 0 -> 890 | 62.3205 ms; 16.1247/s |

This proves telemetry changed during real MiniNDN CPU execution. It does not
prove physical GPU behavior.

## Artifact Retention And Disk Failure

Repetition 1's manifest retained the exact ONNX hashes:

- stage 0: `8d4d8716b499f375634087f56e9011e6c65b70bd9c370fa9d0967055efd74006`
- stage 1: `154d56424b46f527c9fbf9ed59877cd92f42ad331a0f61e6bb026a404ab70bf0`
- stage 2: `9349b111492efc55c0e6c6586c9ebe087902992be2c85923d2f16d5a3d1ee05c`

After all three cells ended, the hashes were recomputed and matched the
manifest. To recover from a 100% full root filesystem, only reproducible binary
artifacts were removed: repetition 1's `.pt` intermediates and hashed ONNX
copies, and repetition 2's incomplete artifact directory. All manifests,
policy/plan files, logs, CSV, exit records, identities and the three separate
formal result directories remain. Available space recovered to 2.7 GiB.

## Final Interpretation

Revision R1 fixed the breadth-first starvation failure (0/60 became 25/60), so
the generation scheduler is functionally effective. It did not make the fixed
1 generation/s capacity class deployable on this local CPU MiniNDN system. The
campaign also lacks three valid repetitions because two cells were invalidated
by disk exhaustion. Both facts independently require `BLOCK`; results must not
be pooled with the original failed campaign, rerun as a fourth cell, or
relabelled as acceptance.
