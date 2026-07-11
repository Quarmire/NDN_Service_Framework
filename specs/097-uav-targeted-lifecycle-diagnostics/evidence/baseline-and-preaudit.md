# Baseline And Pre-Implementation Audit

## Reproduction

Spec 096 preserves two exact post-exit aborts:

```text
control-only-run-01: GS_GUI_EXIT rc=0 -> terminate called without an active exception
combined-fec1-run-03: GS_GUI_EXIT rc=0 -> terminate called without an active exception
```

The control-only MiniNDN mode is the fastest correct feedback seam and retains
the real ServiceController, provider, user, Targeted request, GUI automation,
and shutdown path.

## Code Reality

- `shutdownRuntime()` sets `m_done` but checks/joins YOLO, decoder, playback,
  and refresh workers before stopping/joining the face thread.
- The face thread creates `m_yoloPrewarmThread` after runtime initialization.
- A late creation can therefore occur after the worker join check; later
  shutdown calls return early because `m_done` is already true.
- The existing command path records final UI status but does not correlate
  queue/dispatch/response/timeout stages with request ID and elapsed time.

## Audit Verdict

**PASS**. The face-first quiescence change is minimal and preserves protocol,
security, and application behavior. Diagnostics are metadata-only and exclude
payload/token/certificate/key material. The 0%/5% control-only matrix separates
lifecycle correctness from remaining network delivery failures.
