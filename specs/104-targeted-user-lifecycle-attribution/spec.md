# Feature Specification: Targeted User Lifecycle Attribution

**Branch**: `Experimental` | **Created**: 2026-07-12 | **Status**: Complete

## User Scenarios & Testing
### User Story 1 - Trace Both Ends (Priority: P1)
An explicit campaign flag enables existing ServiceUser and ServiceProvider TRACE
without changing defaults or core source.

**Independent Test**: Environment tests verify exact categories and default silence.

### User Story 2 - Locate User-Side Stop (Priority: P2)
Each telemetry timeout reports request-published/no-ACK, ACK-received/no-Selection,
Selection-published/no-response, or user-response.

**Independent Test**: Classifier fixtures cover every category by request ID.

### User Story 3 - Frozen Diagnostic (Priority: P3)
One five-run 5% trace cell preserves all results.

### Edge Cases
- Targeted bootstrap uses ACK/Selection; cached-token fast path does not.
- TRACE overhead prevents performance comparison.
- ACK_RECEIVED does not imply decrypted/validated ACK unless later stages exist.

## Requirements
### Functional Requirements
- **FR-001**: New flag MUST enable both ServiceUser and ServiceProvider TRACE.
- **FR-002**: Provider-only flag MUST retain its prior behavior.
- **FR-003**: Default logging MUST remain unchanged.
- **FR-004**: Parser MUST collect user TRACE events by exact request ID.
- **FR-005**: Parser MUST distinguish request-published-no-ACK.
- **FR-006**: Parser MUST distinguish ACK-received-Selection-not-published.
- **FR-007**: Parser MUST distinguish Selection-published-no-response.
- **FR-008**: Parser MUST distinguish user-response and incomplete evidence.
- **FR-009**: Core source/security/wire MUST remain unchanged.
- **FR-010**: Summaries MUST exclude payload/token/key material.
- **FR-011**: Failures/lifecycle evidence MUST be retained.
- **FR-012**: One five-run 5% cell MUST run once.

### Key Entities
- **User Core Events**: request, ACK, decrypt, token, Selection, response stages.
- **User Attribution**: bounded category per Targeted telemetry attempt.

## Success Criteria
### Measurable Outcomes
- **SC-001**: Trace/default environment tests pass.
- **SC-002**: Classifier covers all declared categories.
- **SC-003**: 100% treatment telemetry timeouts have user attribution.
- **SC-004**: Zero duplicate/unterminated automation.
- **SC-005**: Zero lifecycle aborts.
- **SC-006**: No performance/reliability claim from TRACE cell.

## Assumptions
- User and provider request IDs are stable across lifecycle logs.
