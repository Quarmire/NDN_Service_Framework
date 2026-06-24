# Feature Specification: Native DI MiniNDN Tracer

**Feature Branch**: `002-native-di-minindn-tracer`

**Created**: 2026-06-24

**Status**: Draft

**Input**: User description: "Update documentation with the design and task list, then loop through P1-P5 until every task is complete and accepted."

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Run Native DI In MiniNDN (Priority: P1)

As the NDNSF-DI developer, I want the native tracer to run across a real MiniNDN topology, so controller, user, and providers are validated over NFD/SVS instead of only a local smoke harness.

**Why this priority**: The previous tracer proved the generated plan and native provider execution locally. The next research risk is the distributed network path.

**Independent Test**: Run the MiniNDN tracer command and inspect a result directory containing policy bundle, topology/process logs, timing evidence, and `SUCCESS`.

**Acceptance Scenarios**:

1. **Given** a built checkout with MiniNDN available, **When** the tracer runs, **Then** controller, user, and role providers start in a MiniNDN topology and the run records a success marker.
2. **Given** the tracer cannot start MiniNDN, **When** hard MiniNDN gating is requested, **Then** the run fails with a clear blocker and writes `FAILURE`.

---

### User Story 2 - Harden Native DI Data Path (Priority: P2)

As the runtime maintainer, I want source input, role dependencies, artifact failures, readiness, and final-response handling to fail closed, so MiniNDN evidence cannot pass because of fallback behavior.

**Why this priority**: A network-level tracer is only useful if incorrect plans and missing artifacts fail visibly.

**Independent Test**: Run focused negative tests for missing roles, missing artifacts, hash mismatch, non-ready providers, and missing final-response metadata.

**Acceptance Scenarios**:

1. **Given** a bad artifact hash, **When** a provider materializes artifacts, **Then** readiness stays non-ready and the run fails.
2. **Given** a final role without final-response metadata, **When** the provider executes, **Then** no arbitrary intermediate output is returned as the final answer.

---

### User Story 3 - Produce Research-Grade Evidence (Priority: P3)

As the researcher, I want each run to produce structured timing and summary evidence, so I can compare runs and cite concrete data in the proposal and paper.

**Why this priority**: The tracer must become a repeatable experiment, not just a pass/fail script.

**Independent Test**: Inspect `timing.csv`, `summary.json`, `summary.txt`, and per-process logs after a run.

**Acceptance Scenarios**:

1. **Given** a successful tracer run, **When** `timing.csv` is read, **Then** it contains session id, provider, role, input/output byte counts, prefetch, execution, publish, end-to-end time, and status.
2. **Given** any run, **When** `summary.json` is read, **Then** it records command, git commit, environment, MiniNDN status, artifact/readiness status, and result marker.

---

### User Story 4 - Validate Multi-Provider Cooperation (Priority: P4)

As the framework designer, I want one tracer plan to support different provider assignments, so NDNSF-DI can demonstrate multi-provider cooperation instead of a single fixed layout.

**Why this priority**: Distributed inference is about coordinated providers; role-to-provider binding must be explicit and reviewable.

**Independent Test**: Run local and MiniNDN-ready tracer modes with at least two assignment layouts and compare provider/role rows in the evidence.

**Acceptance Scenarios**:

1. **Given** the default tracer assignment, **When** the run completes, **Then** each role is attributed to its planned provider.
2. **Given** an alternate assignment, **When** the run completes, **Then** the same plan executes with the alternate provider mapping and records it in evidence.

---

### User Story 5 - Gate LLM Planner Work (Priority: P5)

As the researcher, I want real LLM planner work gated on MiniNDN tracer acceptance, so planner work builds on a working runtime instead of hiding runtime gaps.

**Why this priority**: The LLM planner is the next stage, but it should not start until the native tracer has accepted evidence.

**Independent Test**: Review the feature documentation and task state to confirm LLM planner tasks require accepted native tracer evidence.

**Acceptance Scenarios**:

1. **Given** incomplete tracer evidence, **When** LLM planner work is considered, **Then** documentation points back to the MiniNDN tracer gate.
2. **Given** accepted tracer evidence, **When** the next feature is planned, **Then** the first LLM task is a minimal two-stage or prefill/decode plan reusing the tracer data path.

### Edge Cases

- MiniNDN import succeeds but the process is not root.
- Native provider artifacts exist locally but hash metadata is wrong.
- A provider ACK advertises installing or failed status.
- A dependency edge has no producer or consumer.
- `final-response` is present as a dependency edge instead of final role metadata.
- Provider assignment lists a role not declared by the plan.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The project MUST contain a durable design and task list for P1-P5 under `specs/002-native-di-minindn-tracer/`.
- **FR-002**: The tracer command MUST support hard MiniNDN gating and record whether MiniNDN ran or why it did not run.
- **FR-003**: The tracer MUST launch or be able to launch controller, user, and role providers from one command.
- **FR-004**: Native DI evidence MUST include one timing row per role execution with byte counts and timing fields.
- **FR-005**: Native DI evidence MUST include a machine-readable summary JSON and a human-readable summary text.
- **FR-006**: Data-path negative checks MUST prove missing artifacts, bad hashes, non-ready providers, and missing final response fail closed.
- **FR-007**: Provider assignment MUST be explicit in plan/evidence and support at least a default and alternate layout.
- **FR-008**: LLM planner work MUST be documented as gated by accepted native MiniNDN tracer evidence.

### Key Entities *(include if feature involves data)*

- **MiniNDN Tracer Run**: One invocation that prepares policy, starts or checks topology, runs tracer execution, writes evidence, and marks success/failure.
- **Tracer Assignment**: Mapping from role names to provider identities or provider labels.
- **Role Evidence Row**: Timing and byte-count row for one role in one session.
- **Run Summary**: JSON/text record of command, git commit, environment, result, MiniNDN status, and evidence paths.
- **LLM Planner Gate**: Documentation rule that blocks LLM planner expansion until tracer evidence is accepted.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: P1-P5 are represented as completed tasks only after their acceptance checks pass or hard environmental blockers are recorded.
- **SC-002**: A tracer evidence command writes `summary.json`, `summary.txt`, `timing.csv`, process logs, policy bundle, and `SUCCESS` for the accepted path.
- **SC-003**: `timing.csv` contains at least four role rows for `/Backbone`, two `/Head/Shard/*` roles, and `/Merge`.
- **SC-004**: Negative tests for artifact/readiness/final-response failure pass.
- **SC-005**: Full unit tests still pass after the feature changes.
- **SC-006**: Documentation clearly states the next task after this feature: minimal LLM planner using the accepted tracer path.

## Assumptions

- MiniNDN remains the preferred final network validation loop.
- If a full MiniNDN topology cannot run in the current user context, the script must still record the blocker and support a hard failure mode.
- The native tracer from `specs/001-native-di-tracer/` remains the base plan and local evidence path.
- Full LLM model execution is out of scope for this feature.
