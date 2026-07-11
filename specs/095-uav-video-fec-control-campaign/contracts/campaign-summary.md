# Campaign Output Contract

`campaign-summary.json` contains `status`, frozen constants, all run records,
and aggregates keyed by `lossPercent` and `fecParityShards`.

Each run records at least:

```text
runId, lossPercent, fecParityShards, repetition, durationSeconds,
returncode, completion, decodedFrames, mavlinkArm, mavlinkTakeoff, mavlinkLand,
fecRecoveredChunks, maxTimeouts, maxNacks, maxDuplicates,
maxDecodedFrameGap, maxPendingChunks, maxPendingBytes, rttP50Ms, rttP95Ms
```

Missing or malformed mandatory metrics make the run non-accepted. Failed runs
remain in JSON/CSV and are not automatically replaced.

