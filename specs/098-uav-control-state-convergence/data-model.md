# Data Model

## AutomationSequence

- `droneId`: target drone
- `startedMs`: sequence start timestamp
- `currentStep`: arm, takeoff, land, emergency-stop, complete
- `terminalReason`: completed, command-failed, convergence-expired, shutdown

Transitions are monotonic. A command step can dispatch once, then waits for a
terminal command outcome and any required telemetry state before advancing.

## StateConvergenceObservation

- `prerequisite`: telemetry-ready, armed, airborne, disarmed
- `phase`: wait-begin, satisfied, expired, shutdown
- `timestampMs` and `elapsedMs`
- `observedState`: redacted state label
- `reason`: stable machine-readable outcome

## CampaignRunExtension

- `automationPhases`: counts by phase
- `stateConvergenceStages`: latest stage by prerequisite
- `stateConvergenceComplete`: all begun waits terminal
- `unterminatedAutomationWaits`: prerequisites without satisfied/expired/shutdown

No entity is serialized onto the NDNSF wire or persisted outside experiment
logs and summaries.
