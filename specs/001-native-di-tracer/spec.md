# Feature Specification: Native DI Tracer

**Feature Branch**: `001-native-di-tracer`

**Created**: 2026-06-24

**Status**: Draft

**Input**: User description: "Update documentation with the design and task list, then execute T001-T009 until every task is complete and accepted."

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Generate a Runnable Native DI Plan (Priority: P1)

As the NDNSF-DI developer, I want one small distributed inference policy to generate a service manifest and native execution plan that are stable enough for C++ to consume, so the remaining work is grounded in a real plan instead of slide-level architecture.

**Why this priority**: Without a concrete policy bundle, the C++ execution path cannot be validated against the same data the Python planning layer produces.

**Independent Test**: Generate the tracer policy bundle and run schema/plan smoke checks against the produced `service-manifest.json` and `native-execution-plan.json`.

**Acceptance Scenarios**:

1. **Given** a clean checkout, **When** the tracer policy generator is run, **Then** it produces controller policy, service manifest, native execution plan, and sha256 sidecars.
2. **Given** the generated native plan, **When** C++ plan loaders read it, **Then** role assignments, source inputs, role-to-role dependency edges, and final-response metadata are available without fallback guesses.

---

### User Story 2 - Execute One Native Provider Session (Priority: P1)

As the NDNSF-DI developer, I want the C++ native provider handler to execute the tracer plan role-by-role, publish non-final outputs, and return only the final role response, so the native hot path becomes the default target for future DI work.

**Why this priority**: This is the smallest end-to-end proof that NDNSF-DI can move beyond Python orchestration and use the generic NDNSF collaboration service with native role execution.

**Independent Test**: Run focused unit/smoke tests that drive `NativeProviderHandler`, `ProviderRoleWorker`, and `NativeProviderSession` with the generated tracer plan.

**Acceptance Scenarios**:

1. **Given** provider roles and a runner factory, **When** a collaboration request arrives, **Then** each provider executes only its assigned role.
2. **Given** intermediate outputs, **When** non-final roles finish, **Then** outputs are published for dependent roles and are not returned as final responses.
3. **Given** a final role with `final-response` output, **When** it completes, **Then** only that final payload is returned to the NDNSF user.

---

### User Story 3 - Collect MiniNDN Evidence (Priority: P2)

As the researcher, I want a MiniNDN tracer run to launch controller, user, providers, and native DI execution, then save logs and timing evidence, so proposal and paper claims have a repeatable validation path.

**Why this priority**: MiniNDN remains the default validation loop for NDNSF network/security/performance work until the algorithm is stable.

**Independent Test**: Run the tracer MiniNDN script and inspect its result directory for success marker, role timing CSV, summary, and process logs.

**Acceptance Scenarios**:

1. **Given** the built examples and Python wrapper, **When** the MiniNDN tracer script runs, **Then** the result directory contains controller, user, provider, and plan evidence.
2. **Given** a completed run, **When** the timing summary is checked, **Then** it reports prefetch, execute, publish, end-to-end time, provider name, and role.

---

### User Story 4 - Prepare the LLM Planner Follow-Up (Priority: P3)

As the researcher, I want the LLM planner work separated from the tracer MVP, so native execution becomes stable before adding model-specific planner complexity.

**Why this priority**: The current LLM planner entries are placeholders; making them first would mix planner research with runtime validation.

**Independent Test**: Review the task list and documentation to confirm the LLM planner is explicitly staged after the native tracer acceptance path.

**Acceptance Scenarios**:

1. **Given** the tracer tasks, **When** T009 is reviewed, **Then** it defines the LLM planner stage as a follow-up gated by tracer success.

### Edge Cases

- The tracer policy must fail early if a service role has no provider coverage.
- Provider readiness must stay `failed` or `installing` until required artifacts are materialized and verified.
- Missing `final-response` metadata on the final role must be treated as an error, not as permission to return an arbitrary output.
- MiniNDN may be unavailable on a development host; in that case unit/smoke tests are still required, and MiniNDN acceptance remains open.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The project MUST contain a Spec Kit feature directory documenting the native DI tracer design, acceptance criteria, and task list.
- **FR-002**: The tracer MUST define a minimal policy bundle with service name, roles, providers, artifacts, dependency edges, source inputs, and final-response metadata.
- **FR-003**: The generated native execution plan MUST be consumed by the existing C++ JSON loader without manual edits.
- **FR-004**: Provider readiness MUST distinguish `installing`, `ready`, and `failed`, and provider selection evidence MUST only treat ready providers as candidates.
- **FR-005**: Artifact materialization MUST fetch or locate model/runtime artifacts, verify declared hashes when present, and cache successful materialization.
- **FR-006**: Native provider execution MUST run assigned roles through the C++ runner path, publish intermediate outputs, and return only final-role output.
- **FR-007**: The tracer MUST include a MiniNDN-oriented run script or quickstart path that collects controller, provider, user, and timing evidence.
- **FR-008**: The tracer MUST write an evidence matrix containing provider, role, prefetch time, execution time, publish time, and end-to-end time.
- **FR-009**: The LLM planner MUST remain a second-stage task until the native tracer acceptance path is complete.

### Key Entities *(include if feature involves data)*

- **Tracer Policy**: Minimal distributed inference policy that declares the service, roles, providers, artifacts, and dependencies.
- **Service Manifest**: Planning-plane output used by runtime scripts and deployment tools.
- **Native Execution Plan**: C++-consumable execution description with role specs, role-to-role dependencies, scopes, and final-response metadata.
- **Provider Readiness Record**: Provider-local status that records whether artifacts and runtime prerequisites are ready.
- **Role Timing Record**: Evidence row for a provider role execution.
- **MiniNDN Result Directory**: Durable output containing logs, generated plan files, timing CSV, and summary.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: A fresh tracer policy generation produces all required bundle files and sha256 sidecars in one command.
- **SC-002**: Existing C++ DI smoke/unit tests plus any new tracer tests pass against the generated native plan.
- **SC-003**: A native provider session returns exactly one final response and does not return non-final intermediate outputs.
- **SC-004**: Readiness evidence shows providers are not selectable until required artifacts are materialized.
- **SC-005**: A MiniNDN tracer run, when MiniNDN is available, creates a result directory with logs, timing CSV, summary, and success marker.
- **SC-006**: T001-T009 are marked complete in `tasks.md` only after their acceptance checks pass or a documented environmental blocker is recorded.

## Assumptions

- The first tracer uses a deterministic toy/native runner or tiny existing model path; full LLM planning is out of scope until T009.
- MiniNDN is the preferred final validation surface for NDNSF network/security/performance behavior.
- Host NFD may be used for narrow diagnosis only, not as final acceptance.
- The generic dynamic NDNSF API, NAC-ABE permission path, one-time tokens, and collaboration service flow remain unchanged.
