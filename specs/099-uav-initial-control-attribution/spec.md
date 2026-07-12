# Feature Specification: UAV Initial Control Attribution

**Feature Branch**: `Experimental` | **Created**: 2026-07-11 | **Status**: Draft

**Input**: Continue from Spec 098 by isolating initial telemetry availability
and Arm timeout behavior without command retry or timeout tuning.

## User Scenarios & Testing

### User Story 1 - Trust Command Terminal State (Priority: P1)

An operator or parser can distinguish command attempt, real termination, and
elapsed time without contradictory time fields.

**Why this priority**: Spec 098 exposed an Arm transport timeout that automation
missed because the stored terminal timestamp was invalid.

**Independent Test**: Construct pending and timeout states and verify zero
initial RTT, monotonic update time, exact elapsed RTT, and timeout budget.

**Acceptance Scenarios**:

1. **Given** an attempt time, **When** pending is recorded, **Then** update time
   equals attempt time and RTT is zero.
2. **Given** a later timeout, **When** terminal state is recorded, **Then**
   update time equals terminal time and RTT equals elapsed time.
3. **Given** terminal state after dispatch, **When** automation observes it,
   **Then** it ends with that outcome rather than a later observer expiry.

---

### User Story 2 - Attribute Initial Control Failures (Priority: P2)

A reviewer can classify failed initial control as telemetry request outcome,
Arm request outcome, or local observer mismatch using retained phase evidence.

**Why this priority**: A sender timeout does not identify packet direction, and
a local evidence bug must not be mislabeled as network failure.

**Independent Test**: Reparse Spec 098 runs 03/04 and reproduce the Arm timeout
plus observer mismatch and the initial telemetry timeout/deadline overlap.

**Acceptance Scenarios**:

1. **Given** Targeted and automation phases, **When** a run is parsed, **Then**
   telemetry and Arm have sender-observable terminal classification and timing.
2. **Given** command timeout followed by later automation expiry, **When**
   aggregated, **Then** it is an observer mismatch, not a second network timeout.
3. **Given** packet direction is not observable, **When** reported, **Then** the
   category remains sender-side timeout.

---

### User Story 3 - Validate Without Policy Tuning (Priority: P3)

The maintainer can run one frozen 5% MiniNDN diagnostic cell that preserves all
failures without changing retry, timeout, security, or safety behavior.

**Why this priority**: Diagnosis must precede reliability treatment.

**Independent Test**: Five 60-second runs produce terminal telemetry/Arm
attribution, zero duplicate command attempts, and zero lifecycle aborts.

**Acceptance Scenarios**:

1. **Given** the frozen command, **When** it runs once, **Then** all five outcomes
   are retained whether positive or negative.
2. **Given** a failed run, **When** inspected, **Then** its earliest supported
   boundary is reported or explicitly marked unknown.

### Edge Cases

- Telemetry may already be in flight when automation begins waiting.
- A later telemetry request may start just before the wait expires; classify it
  without extending the wait.
- Overlapping telemetry and command requests require service/request-ID/time
  correlation rather than log order alone.
- Provider execution may occur without a returned response; sender evidence
  must not infer provider receipt or execution.
- Shutdown remains a terminal outcome.

## Requirements

### Functional Requirements

- **FR-001**: Pending state MUST store attempt time as update time and zero RTT.
- **FR-002**: Timeout state MUST store terminal update time, elapsed RTT, and the
  configured timeout budget.
- **FR-003**: Pending/timeout construction MUST avoid positional ambiguity.
- **FR-004**: Automation MUST recognize matching terminal state after dispatch.
- **FR-005**: Parser MUST correlate Targeted phases by provider, service,
  request ID, timestamp, and terminal phase.
- **FR-006**: Parser MUST summarize initial telemetry attempts, outcomes, and
  request overlap with the automation deadline.
- **FR-007**: Parser MUST compare Arm Targeted and automation terminal outcomes.
- **FR-008**: Attribution MUST distinguish sender timeout, local safety block,
  observer mismatch, convergence expiry, lifecycle abort, and unknown evidence.
- **FR-009**: Diagnostics MUST exclude payloads, tokens, certificates,
  credentials, and private-key material.
- **FR-010**: Flight commands MUST remain single-attempt without automatic retry.
- **FR-011**: Existing timeouts, polling, safety gates, permissions, NAC-ABE,
  and replay protection MUST remain unchanged.
- **FR-012**: Validation MUST use one five-run, 5% loss, 60-second MiniNDN cell
  and preserve failures without replacement runs.

### Key Entities

- **Command Evidence State**: attempt/update times, RTT, timeout budget, outcome.
- **Targeted Attempt**: provider/service/request ID and dispatch/terminal timing.
- **Initial Control Attribution**: earliest supported boundary and linked evidence.

## Success Criteria

### Measurable Outcomes

- **SC-001**: Unit tests prove pending/timeout timestamps and RTTs in all cases.
- **SC-002**: Spec 098 run 03 is reproduced as Arm sender timeout plus observer
  mismatch; run 04 as telemetry timeout plus late-request deadline overlap.
- **SC-003**: Every treatment run has terminal attribution or named missing evidence.
- **SC-004**: Treatment has zero duplicate commands and unterminated states.
- **SC-005**: Treatment has zero known lifecycle abort markers.
- **SC-006**: Evidence separates implemented/executed/measured claims and makes
  no unsupported packet-direction or reliability claim.

## Assumptions

- Existing phase logs are authoritative sender-side evidence.
- Packet-direction localization is out of scope without existing evidence.
- Correct command bookkeeping is not timeout tuning.
- `results/` stays local; conclusions are tracked in feature evidence.
