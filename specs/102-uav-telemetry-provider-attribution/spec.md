# Feature Specification: UAV Telemetry Provider Attribution

**Branch**: `Experimental` | **Created**: 2026-07-12 | **Status**: Draft

## User Scenarios & Testing

### User Story 1 - Observe Provider Handler (Priority: P1)
A reviewer can determine whether a timed-out telemetry request entered and
returned from the drone handler using the same request ID.

**Independent Test**: Synthetic GS/drone logs classify request IDs as
`handler-not-observed`, `handler-returned-no-user-response`, or `user-response`.

**Acceptance Scenarios**:
1. **Given** GS timeout and no provider event, **Then** handler-not-observed.
2. **Given** matching handler return and GS timeout, **Then** handler-returned-no-user-response.
3. **Given** GS response, **Then** user-response.

### User Story 2 - Preserve Protocol And Privacy (Priority: P2)
Diagnostics remain application-local metadata and contain no payload or secrets.

**Independent Test**: Source contract verifies only phase/request/service/time/status fields.

**Acceptance Scenarios**:
1. **Given** telemetry invocation, **Then** no wire/security behavior changes.

### User Story 3 - Frozen Validation (Priority: P3)
One five-run 5% MiniNDN cell preserves all outcomes.

**Independent Test**: Every telemetry timeout has a provider attribution category.

### Edge Cases
- Security rejection before handler is indistinguishable from request loss and remains handler-not-observed.
- Handler return does not prove Data publication or delivery.
- Normal and Targeted telemetry both use the same handler contract.

## Requirements
### Functional Requirements
- **FR-001**: Telemetry MUST register a full RequestHandler exposing request ID.
- **FR-002**: Handler enter and return MUST log the same request ID.
- **FR-003**: Events MUST include phase, service, timestamp, and response status.
- **FR-004**: Events MUST exclude payload, tokens, certificates, credentials, and keys.
- **FR-005**: Parser MUST correlate GS and provider events by request ID.
- **FR-006**: Parser MUST distinguish handler-not-observed, handler-returned-no-user-response, and user-response.
- **FR-007**: Missing provider evidence MUST not be called request loss.
- **FR-008**: Handler return MUST not be called response publication/delivery.
- **FR-009**: Telemetry lease and single-in-flight behavior MUST remain unchanged.
- **FR-010**: Command, safety, wire, and security behavior MUST remain unchanged.
- **FR-011**: Validation MUST preserve failures and lifecycle evidence.
- **FR-012**: One five-run 5% MiniNDN cell MUST execute once.

### Key Entities
- **Provider Handler Event**: request ID, phase, service, timestamp, status.
- **End-to-End Attribution**: GS terminal plus provider handler evidence.

## Success Criteria
### Measurable Outcomes
- **SC-001**: Tests cover all three categories.
- **SC-002**: 100% of treatment telemetry timeouts receive a bounded category.
- **SC-003**: No sensitive fields appear in provider events.
- **SC-004**: Zero duplicate commands or unterminated automation.
- **SC-005**: Zero lifecycle aborts.
- **SC-006**: No unsupported packet-direction or reliability claim.

## Assumptions
- RequestHandler request ID equals the Ground Station Targeted request ID.
