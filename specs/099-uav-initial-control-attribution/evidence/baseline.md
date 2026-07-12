# Baseline Evidence

## Spec 098 Run 03

Telemetry readiness converged in 101 ms. Arm dispatched at `1783831426099` ms
and sender-side Targeted timeout occurred at `1783831436599` ms (10500 ms).
Timeout state carried the terminal epoch as RTT and `updatedMs=0`, so automation
missed it and ended later at `1783831438135` as
`command-state-not-terminal`.

Classification: `arm-sender-timeout` plus `command-observer-mismatch`. This does
not establish request-versus-response loss. Drone state later became armed, but
there is no request-ID-correlated provider receipt evidence.

## Spec 098 Run 04

Initial telemetry dispatched at `1783831455494` ms and timed out after 10501 ms.
A replacement dispatched at `1783831466006`; automation expired 25 ms later;
the replacement response arrived at `1783831467180`.

Classification: `telemetry-sender-timeout` followed by
`telemetry-convergence-expired` with a late request overlapping the deadline.
Spec 099 records rather than repairs this boundary.

## Code Finding

`FlightCommandState` order ends with `detail, rttMs, updatedMs, timeoutMs`.
Pending and timeout aggregate initializers used the wrong semantic positions.
Named shared factories are the smallest unit-testable correction.

## Pre-Implementation Audit

**Verdict: PASS.** The named factories prevent the observed field-order defect
without weakening stale-state rejection. Parser work consumes existing
sender-side diagnostics and does not add protocol or sensitive fields. The
frozen experiment tests attribution completeness, so a flat or negative
completion result remains admissible. No migration is required; rollback is a
source revert.
