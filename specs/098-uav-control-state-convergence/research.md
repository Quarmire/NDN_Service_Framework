# Research: UAV Control State Convergence

## Code And Evidence Finding

In the Spec 097 5% baseline, the only successful Takeoff run received a fresh
telemetry response 0.601 seconds after the accepted Arm response and before
Takeoff. The other four runs received no telemetry response in that interval;
Takeoff was locally blocked as `not-armed`.

The current code confirms the split ownership:

- Arm response updates `FlightCommandState` and logs `fc_state=mock-armed`.
- Takeoff readiness reads fresh `TelemetryState`, not command response fields.
- The auto sequence waits two wall-clock seconds from Arm dispatch, not from
  Arm completion or telemetry convergence.
- The safety gate is behaving correctly; the automation advances before its
  observable precondition is satisfied.

## Decisions

### Decision: Preserve the production safety gate

**Rationale**: `not-armed` is a correct local rejection when telemetry has not
confirmed armed state. Copying `fc_state` into telemetry or bypassing readiness
would create a second source of truth.

**Alternatives considered**: Treat the command response as telemetry; weaken
the gate for automation; dispatch Takeoff unconditionally. All were rejected.

### Decision: Sequence by observed command and telemetry state

**Rationale**: The automated operator should wait until Arm is terminal and
fresh telemetry reports armed before posting Takeoff. This matches the real UI
safety contract and removes fixed-clock scheduling as a confound.

**Alternatives considered**: Increase the fixed delay; add Takeoff retry; raise
timeouts. These mask the race or change offered behavior and were rejected.

### Decision: Permit telemetry polling but never command retry

**Rationale**: Telemetry is an observation path already polled periodically.
Repeated observation within a bounded convergence window is not replaying a
flight command.

**Alternatives considered**: Wait only on passive periodic polling. Explicit
bounded observation is easier to test and diagnose while preserving security.

### Decision: Use a five-run functional treatment

**Rationale**: The preserved five-run baseline is sufficient to falsify the
specific scheduling hypothesis. Exact confidence intervals will expose the
large uncertainty; the result cannot support a general reliability claim.

**Alternatives considered**: Larger inferential campaign. Deferred until the
functional race is removed and the next research question is defined.
