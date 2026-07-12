# Feature Specification: Targeted Core Lifecycle Attribution

**Branch**: `Experimental` | **Created**: 2026-07-12 | **Status**: Draft

## User Scenarios & Testing
### User Story 1 - Reveal Existing Core Stages (Priority: P1)
An experiment can temporarily expose existing Targeted accepted/published/failed
TRACE events without changing normal logging.

**Independent Test**: Diagnostic flag sets ServiceProvider TRACE only for child apps.

### User Story 2 - Classify Timeouts (Priority: P2)
Parser correlates Ground Station, handler, and core events by request ID.

**Independent Test**: Fixtures distinguish provider-not-observed,
pre-handler-rejected-response-published, response-published-no-user-response,
publish-failed, and user-response.

### User Story 3 - Frozen Diagnostic (Priority: P3)
One five-run 5% cell retains all outcomes and core traces.

### Edge Cases
- TRACE changes overhead, so completion comparisons are descriptive only.
- Published means handed to NDNSF publication, not delivered to the user.
- Missing accepted event plus published error indicates pre-handler failure but not its security reason.

## Requirements
### Functional Requirements
- **FR-001**: Campaign MUST offer an explicit provider lifecycle trace flag.
- **FR-002**: Flag MUST affect child environment only.
- **FR-003**: Default logging MUST remain unchanged.
- **FR-004**: Parser MUST read existing Targeted accepted and response publish events.
- **FR-005**: Correlation MUST use exact request ID.
- **FR-006**: Parser MUST distinguish five bounded lifecycle categories.
- **FR-007**: Published MUST not be called delivered.
- **FR-008**: Missing provider evidence MUST not be called packet loss.
- **FR-009**: No core runtime/security behavior MUST change.
- **FR-010**: No payload/token/key material MUST enter new summaries.
- **FR-011**: Failures/lifecycle evidence MUST be retained.
- **FR-012**: One five-run 5% MiniNDN cell MUST execute once.

### Key Entities
- **Core Trace Stages**: accepted, execute done, publish attempt/published/failed.
- **Lifecycle Attribution**: bounded category per telemetry attempt.

## Success Criteria
### Measurable Outcomes
- **SC-001**: Default and trace-enabled environment tests pass.
- **SC-002**: 100% treatment telemetry timeouts have bounded categories.
- **SC-003**: No core source modification is required.
- **SC-004**: Zero duplicate commands/unterminated automation.
- **SC-005**: Zero lifecycle aborts.
- **SC-006**: Results note TRACE perturbation and avoid reliability claims.

## Assumptions
- Existing NDNSF TRACE event request IDs match application/provider IDs.
