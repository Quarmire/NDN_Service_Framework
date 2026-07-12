# Baseline And Pre-Implementation Audit

## Measured Baseline

Source: `results/spec097-uav-targeted-control-loss05-current-final`.

| Run | Arm outcome | Armed telemetry after Arm and before Takeoff | Takeoff outcome |
|---|---|---|---|
| 01 | response accepted | no | blocked `not-armed` |
| 02 | response accepted | no | blocked `not-armed` |
| 03 | response accepted | yes, 0.601 s after Arm response | response accepted |
| 04 | blocked `no-telemetry` | no | blocked `not-armed` |
| 05 | response accepted | no; armed telemetry arrived 0.533 s after Takeoff decision | blocked `not-armed` |

Three of four accepted Arm responses advanced to Takeoff before an armed
telemetry observation. The only accepted Takeoff had an armed observation in
the interval. Run 04 shows the separate initial-telemetry failure path.

## Code Reality

- `GroundStationWindow` posts Arm and Takeoff on a fixed two-second schedule.
- the Arm response updates `FlightCommandState`;
- `validateTakeoffReadiness` independently reads a fresh `TelemetryState` and
  correctly rejects disarmed or absent telemetry;
- periodic telemetry runs every 1.5 seconds, but response time under loss is
  not synchronized with the fixed Takeoff schedule.

The supported hypothesis is therefore an automation sequencing race between
command completion and telemetry convergence. It is not evidence that the
production Takeoff safety gate is wrong.

## Pre-Implementation Audit

**Verdict: PASS.** The smallest safe treatment is an application-owned,
bounded state machine that waits for the existing telemetry-based precondition
and never retries a flight command. A pure shared state object makes monotonic
transitions and dispatch-once behavior unit-testable; the GTK window retains
thread ownership and actual command dispatch. Parser changes are evidence-only.

Security, wire names, tokens, permissions, replay checks, provider checks,
command timeouts, and safety policy remain unchanged. The treatment is
falsified if `not-armed` still follows an accepted Arm without a convergence
expiry, any command is dispatched twice, a wait remains nonterminal, or a
lifecycle abort occurs.
