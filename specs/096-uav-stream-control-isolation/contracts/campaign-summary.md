# Campaign Output Contract

Each run records:

```text
runId, workloadMode, videoRequired, controlRequired, fecParityShards,
repetition, returncode, processCompletion, videoCompletion,
controlCompletion, completion, accepted, elapsedSeconds
```

Video runs additionally carry the canonical Spec 095 stream, FEC, buffer,
stale, decoded-frame, and RTT metrics. Control-only uses `fecParityShards=-1`,
does not require stream metrics, and is accepted only when process exit and all
three Targeted control markers succeed.

The summary contains constants, all runs, and per-cell aggregates. A failed run
is never silently removed or replaced.
