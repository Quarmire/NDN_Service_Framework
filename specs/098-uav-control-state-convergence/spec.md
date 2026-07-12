# Feature Specification: UAV Control State Convergence

**Feature Branch**: `Experimental`

**Created**: 2026-07-11

**Status**: Draft

**Input**: Continue from Spec 097 by diagnosing and removing the auto-control
sequence race that produced Takeoff `not-armed` outcomes at 5% loss, without
adding command retries or weakening flight safety.

## User Scenarios & Testing

### User Story 1 - Explain The Blocked Transition (Priority: P1)

As an experiment operator, I can distinguish a lost command from a command
that was locally blocked because the Ground Station had not yet observed the
state produced by the preceding command.

**Why this priority**: Spec 097 measured four Takeoff `not-armed` outcomes, but
optimizing before identifying the controlling state transition could weaken a
correct safety gate or hide network failures.

**Independent Test**: Reparse the five canonical 5% runs and show, per run,
whether a fresh armed observation occurred between the Arm terminal outcome
and the Takeoff decision.

**Acceptance Scenarios**:

1. **Given** an accepted Arm response followed by no armed-state observation,
   **When** Takeoff is evaluated, **Then** the evidence identifies the missing
   state convergence rather than labeling the Takeoff request as lost.
2. **Given** an armed-state observation before Takeoff, **When** Takeoff is
   evaluated, **Then** the evidence identifies that the safety precondition was
   satisfied and records whether the command was dispatched.

---

### User Story 2 - Sequence Commands By Observed State (Priority: P1)

As an experiment operator, I can run the automated Arm/Takeoff/Land/Emergency
Stop sequence using observed command and telemetry state instead of fixed wall
clock delays, while each flight command is still attempted at most once.

**Why this priority**: The automated sequence must represent the production
safety contract. Waiting for an observed precondition is valid; retrying a
failed command or bypassing the safety gate is not.

**Independent Test**: Run the sequence with controlled delayed and missing
state updates and verify that it dispatches each command once only after its
precondition, or records a bounded terminal reason without dispatch.

**Acceptance Scenarios**:

1. **Given** Arm is accepted and armed telemetry arrives within the bounded
   convergence window, **When** the automated sequence advances, **Then** it
   dispatches Takeoff exactly once.
2. **Given** armed telemetry does not arrive within the convergence window,
   **When** the window expires, **Then** Takeoff is not dispatched and the run
   records `armed-state-not-converged` as a terminal automation outcome.
3. **Given** a flight command times out, is blocked, is busy, or is rejected,
   **When** the sequence evaluates the outcome, **Then** it does not retry that
   command and preserves the measured failure.

---

### User Story 3 - Measure The Treatment Without Overclaiming (Priority: P2)

As a framework researcher, I can compare the canonical Spec 097 5% baseline
with one frozen current-revision 5% treatment matrix and determine whether the
state-convergence race was removed without claiming general lossy-network
reliability.

**Why this priority**: The project treats negative results as real outcomes and
requires matched MiniNDN evidence before retaining a behavioral change.

**Independent Test**: Execute five 5% control-only MiniNDN runs once, without
automatic retries, and compare state convergence, command stages, completion,
and lifecycle aborts against the preserved Spec 097 baseline.

**Acceptance Scenarios**:

1. **Given** the frozen treatment matrix, **When** all five runs complete,
   **Then** every command and automation wait has a terminal classification and
   no repetition is replaced.
2. **Given** the result is worse, flat, or mixed, **When** evidence is written,
   **Then** the result is reported as measured and no improvement claim is made.

### Edge Cases

- Arm can be accepted after the original fixed Takeoff time.
- A command response can report an armed state before the telemetry cache does.
- Telemetry can be fresh but still report disarmed.
- Periodic telemetry responses can arrive out of order.
- The convergence deadline can expire without a command timeout.
- Shutdown can begin while the automated sequence is waiting.
- A clean process exit with a nonterminal automation wait must be rejected.

## Requirements

### Functional Requirements

- **FR-001**: The system MUST correlate Arm terminal outcome, subsequent
  telemetry observations, and the Takeoff decision for one automated sequence.
- **FR-002**: The system MUST distinguish command transport outcomes from local
  safety-gate and state-convergence outcomes.
- **FR-003**: The automated sequence MUST wait for fresh telemetry that reports
  the required state before dispatching a dependent flight command.
- **FR-004**: Each Arm, Takeoff, Land, and Emergency Stop command MUST be
  attempted at most once per run.
- **FR-005**: A missing precondition MUST end in a bounded, machine-readable
  terminal outcome and MUST NOT bypass the production safety gate.
- **FR-006**: Automation diagnostics MUST include sequence phase, drone,
  prerequisite, elapsed time, and reason without payloads, tokens,
  certificates, credentials, or key material.
- **FR-007**: The campaign parser MUST reject a clean launcher exit if a command
  or automation wait remains nonterminal.
- **FR-008**: The treatment MUST preserve existing Targeted authentication,
  authorization, one-time token, replay, and provider-permission checks.
- **FR-009**: The treatment MUST NOT add command retries, change command
  timeouts, change loss configuration, or weaken readiness/safety checks.
- **FR-010**: Validation MUST use MiniNDN with the Spec 097 control-only
  topology, 5% loss, five runs, and automatic retry disabled.
- **FR-011**: All treatment repetitions, including failures, MUST be retained
  and reported once without tuning and rerunning.
- **FR-012**: Result analysis MUST report command completion, state-convergence
  completion, lifecycle aborts, terminal-stage counts, and exact uncertainty
  appropriate for the small sample.

### Key Entities

- **Automation Sequence**: One ordered Arm/Takeoff/Land/Emergency Stop run with
  a unique start time and terminal phase for each step.
- **State Convergence Observation**: A fresh telemetry observation tied to a
  required state, timestamp, elapsed time, and satisfied/expired outcome.
- **Command Outcome**: The existing attempted command's terminal response,
  timeout, blocked, busy, or rejected stage.
- **Treatment Cell**: Five no-retry control-only repetitions under 5% loss.

## Success Criteria

### Measurable Outcomes

- **SC-001**: The five Spec 097 baseline runs are classified with the presence
  or absence of an armed observation between Arm completion and Takeoff.
- **SC-002**: Unit tests demonstrate that each dependent command is dispatched
  at most once and only after its required observed state.
- **SC-003**: All five treatment runs have terminal command and automation-wait
  classifications with zero unterminated attempts.
- **SC-004**: All five treatment runs contain zero known lifecycle abort
  markers.
- **SC-005**: No treatment Takeoff is locally blocked as `not-armed` after an
  accepted Arm response; a convergence expiry is reported separately.
- **SC-006**: The final report gives baseline and treatment completion counts,
  exact binomial confidence intervals, and a clearly bounded causal claim.

## Assumptions

- Spec 097's five 5% runs are the immutable baseline.
- MiniNDN remains the final validation environment until algorithm work is
  complete.
- Existing periodic telemetry polling remains available; the feature changes
  sequencing and observability, not the wire protocol.
- Five treatment runs are a functional falsification cell, not sufficient to
  claim general reliability across loss processes or hardware.
- Proposal slides and paper files are out of scope.
