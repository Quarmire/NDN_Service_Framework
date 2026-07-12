# Feature Specification: UAV Armed Visibility Boundary

**Branch**: `Experimental` | **Created**: 2026-07-12 | **Status**: Draft

## User Scenarios & Testing

### User Story 1 - Distinguish Drone And Ground Visibility (Priority: P1)

A reviewer can tell whether the drone became armed before the Ground Station's
armed wait expired and whether Ground Station telemetry exposed that state.

**Independent Test**: Reparse Spec 099 runs 02/05 using both logs.

**Acceptance Scenarios**:
1. **Given** drone armed before expiry but no Ground Station observation, **Then** classify `ground-telemetry-not-visible`.
2. **Given** Ground Station receives armed telemetry before expiry but polling misses it, **Then** classify `final-observation-missed`.

### User Story 2 - Honor State At The Deadline (Priority: P2)

Automation performs one final local state read before expiring, without another
request, timeout extension, or command retry.

**Independent Test**: A state arriving after the last poll but before deadline is accepted exactly once.

**Acceptance Scenarios**:
1. **Given** qualifying cached state before expiry, **When** the loop reaches its deadline, **Then** the final read satisfies the wait.
2. **Given** absent/nonqualifying state, **Then** the wait expires as before.

### User Story 3 - Measure The Treatment (Priority: P3)

One frozen five-run 5% MiniNDN cell retains all outcomes.

**Independent Test**: Five 60-second configured runs have terminal attribution and no retries or aborts.

**Acceptance Scenarios**:
1. **Given** the frozen cell, **When** run once, **Then** all results remain retained.

### Edge Cases

- Drone and Ground Station logs share the MiniNDN host clock but not log ordering.
- Armed state after expiry must not count.
- Final observation must not send a telemetry request.
- Missing drone log produces explicit unknown evidence.

## Requirements

### Functional Requirements

- **FR-001**: Parser MUST read drone headless armed timestamps.
- **FR-002**: Parser MUST correlate Arm response, armed wait, drone armed, Ground Station armed telemetry, and expiry.
- **FR-003**: Attribution MUST distinguish drone-not-armed, ground-telemetry-not-visible, final-observation-missed, satisfied, and unknown.
- **FR-004**: Only observations at or before expiry MUST count.
- **FR-005**: Automation MUST perform one final cached-state evaluation before expiry.
- **FR-006**: Final evaluation MUST reuse the same safety predicate as normal polling.
- **FR-007**: Final evaluation MUST NOT issue a telemetry request.
- **FR-008**: Flight commands MUST remain single-attempt without retry.
- **FR-009**: Existing timeout, polling interval, safety, Targeted, and security behavior MUST remain unchanged.
- **FR-010**: Diagnostics MUST contain no payload/token/credential/key material.
- **FR-011**: Validation MUST use MiniNDN and preserve negative results.
- **FR-012**: One five-run 5% 60-second-configured cell MUST be executed once.

### Key Entities

- **Armed Visibility Timeline**: Arm response, drone armed, GS armed, and expiry timestamps.
- **Final Observation**: one non-network cached-state evaluation at the deadline.

## Success Criteria

### Measurable Outcomes

- **SC-001**: Spec 099 run 02 classifies ground telemetry invisible and run 05 final observation missed.
- **SC-002**: Tests prove final cached state is evaluated once without request dispatch.
- **SC-003**: Treatment reports visibility attribution for 5/5 runs.
- **SC-004**: Zero duplicate commands or unterminated automation states.
- **SC-005**: Zero lifecycle abort markers.
- **SC-006**: Results make no unsupported general reliability claim.

## Assumptions

- MiniNDN processes use the same host clock.
- Existing headless status logs are observation evidence, not request receipt proof.
