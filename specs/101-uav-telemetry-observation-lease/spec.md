# Feature Specification: UAV Telemetry Observation Lease

**Branch**: `Experimental` | **Created**: 2026-07-12 | **Status**: Draft

## User Scenarios & Testing

### User Story 1 - Bound Observation Ownership (Priority: P1)
Telemetry observation cannot monopolize an entire convergence window.

**Independent Test**: A 10-second wait permits at most two sequential 5-second
read-only attempts while retaining one in-flight request per drone.

**Acceptance Scenarios**:
1. **Given** a lost telemetry attempt, **Then** its lease terminates before the observation window ends.
2. **Given** an active attempt, **Then** concurrent polls do not dispatch another.

### User Story 2 - Preserve Command Policy (Priority: P2)
Flight commands remain single-attempt and use their unchanged timeout.

**Independent Test**: Source and runtime evidence show only telemetry requests use the observation lease.

**Acceptance Scenarios**:
1. **Given** Arm/Takeoff/Land, **Then** their Targeted timeout and retry behavior are unchanged.

### User Story 3 - Measure Once (Priority: P3)
Run one frozen five-run 5% MiniNDN treatment and retain every outcome.

**Independent Test**: Five terminal runs report telemetry attempts and convergence.

**Acceptance Scenarios**:
1. **Given** the frozen cell, **Then** no result is replaced or tuned.

### Edge Cases
- Late response after an observation timeout must follow existing Targeted semantics.
- Shutdown clears ownership through existing callbacks.
- The second attempt may also fail; no third attempt fits the 10-second window.

## Requirements

### Functional Requirements
- **FR-001**: Telemetry Targeted requests MUST use a 5-second observation lease.
- **FR-002**: The general Targeted default timeout MUST remain unchanged.
- **FR-003**: Flight-command Targeted requests MUST retain the general timeout.
- **FR-004**: At most one telemetry request per drone may be in flight.
- **FR-005**: Timeout and response MUST both release telemetry ownership.
- **FR-006**: The 10-second convergence window MUST permit a second observation after first timeout.
- **FR-007**: Flight commands MUST remain single-attempt.
- **FR-008**: Safety predicates and final cached read MUST remain unchanged.
- **FR-009**: Wire names, permissions, tokens, NAC-ABE, and replay checks MUST remain unchanged.
- **FR-010**: Parser MUST report telemetry attempt counts/outcomes during armed wait.
- **FR-011**: Validation MUST preserve failures and lifecycle evidence.
- **FR-012**: One five-run 5% 60-second-configured MiniNDN cell MUST run once.

### Key Entities
- **Observation Lease**: request timeout budget and single-in-flight ownership.
- **Armed Wait Attempts**: telemetry requests dispatched/terminated during armed wait.

## Success Criteria
### Measurable Outcomes
- **SC-001**: Tests prove telemetry uses 5000 ms and commands keep the configured timeout.
- **SC-002**: Treatment reports armed-wait attempt outcomes for 5/5 runs.
- **SC-003**: Ground-telemetry-not-visible runs receive a second observation opportunity.
- **SC-004**: Zero duplicate flight commands or unterminated automation.
- **SC-005**: Zero lifecycle aborts.
- **SC-006**: Completion is reported without unsupported improvement claims.

## Assumptions
- Five seconds preserves the observed successful telemetry latency envelope (maximum retained example about 3.2 seconds) while fitting two attempts.
- Telemetry observation retry is read-only and distinct from flight-command retry.
