# Campaign Output Contract

`campaign-summary.json` contains `status`, frozen constants, all run records,
and aggregates keyed by `lossPercent` and `fecParityShards`.

Each run records at least:

```text
runId, lossPercent, fecParityShards, repetition, durationSeconds,
returncode, processCompletion, videoCompletion, controlCompletion, completion,
metricsValid, malformedMetrics, decodedFrames, mavlinkArm, mavlinkTakeoff, mavlinkLand,
fecRecoveredChunks, maxTimeouts, maxNacks, maxDuplicates,
maxDecodedFrameGap, maxPendingChunks, maxPendingBytes, decodedFrameRate,
minimumDecodedFrames, rttP50Ms, rttP95Ms
```

Missing or malformed mandatory metrics make the run non-accepted. Failed runs
remain in JSON/CSV and are not automatically replaced.

`videoCompletion` requires the requested stream duration, a clean GUI exit, the
accepted parity value, and at least `minimumDecodedFrames`. The primary
60-second/30-fps campaign sets this minimum to 900 frames. `controlCompletion`
requires accepted Arm, Takeoff, and Land operations. Overall `completion`
requires process, video, and control completion together.

`metricsValid` is false when no structured adaptive snapshot exists or a
required adaptive metric is absent or malformed. `malformedMetrics` preserves
the exact snapshot/field failures, and a run with invalid metrics cannot be
accepted.
