# Data Model

## Command Evidence State

Fields: drone, command, accepted/ACK outcome, detail, attempt/update time, RTT,
and timeout budget.

Invariants: pending has zero RTT and `updatedMs=attemptMs`; timeout has
`updatedMs=terminalMs`, elapsed RTT, and unchanged timeout budget.

## Targeted Attempt

Fields: provider, service, request ID, dispatch/terminal time, terminal phase,
status, and elapsed time. A dispatched request has at most one selected terminal
sender outcome.

## Initial Control Attribution

Fields: telemetry attempts, Arm attempt, automation phases, earliest boundary,
evidence completeness, and missing evidence.

Categories: `telemetry-sender-timeout`, `telemetry-convergence-expired`,
`arm-local-block`, `arm-sender-timeout`, `arm-response`,
`armed-convergence-expired`,
`command-observer-mismatch`, `lifecycle-abort`, and `unknown`.
