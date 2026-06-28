# Feature Specification: Real MiniNDN Native DI Tracer

**Feature Branch**: `003-native-di-real-minindn`

**Created**: 2026-06-24

**Status**: Draft

**Input**: User request: "更新文档记录设计和任务列表，然后循环执行P1-P7，直到所有任务完成并验收"

## User Scenarios & Testing

### User Story 1 - Launch Native DI Across MiniNDN Nodes (Priority: P1)

As an NDNSF-DI developer, I need a single command that launches the native DI
tracer in MiniNDN with role-specific providers on separate nodes, so local smoke
evidence is no longer confused with a network experiment.

**Why this priority**: The previous tracer evidence is MiniNDN-aware but still
local. The next dissertation-grade step is a true topology harness.

**Independent Test**: Run the real MiniNDN tracer command and verify it writes a
MiniNDN evidence directory with topology, process, provider-role, and security
bootstrap status.

**Acceptance Scenarios**:

1. **Given** the repo is built with examples, **When** the real MiniNDN tracer is
   run in quick-smoke mode, **Then** it verifies topology files, native tracer
   inputs, and provider binary availability.
2. **Given** MiniNDN is available as root, **When** the tracer is run normally,
   **Then** it starts MiniNDN, NFD, routes, and provider role checks on assigned
   nodes.

---

### User Story 2 - Preserve Security And Assignment Evidence (Priority: P2)

As an NDNSF-DI developer, I need the real MiniNDN run to record controller,
user, provider, group, and role assignment evidence, so later service execution
cannot silently bypass NDNSF security assumptions.

**Why this priority**: NDNSF-DI must remain tied to permissions, NAC-ABE, and
provider identities rather than becoming an isolated dataflow prototype.

**Independent Test**: Inspect `summary.json`, `assignment.csv`, and logs after
a run; every role has a provider identity and node, and the run records whether
security bootstrap was executed or blocked.

**Acceptance Scenarios**:

1. **Given** the default assignment, **When** evidence is written, **Then**
   `/Backbone`, both head shards, and `/Merge` map to distinct provider
   identities and MiniNDN nodes.
2. **Given** the alternate assignment, **When** evidence is written, **Then**
   the provider identities change and the evidence rows reflect that change.

---

### User Story 3 - Gate Full Native Execution Honestly (Priority: P3)

As an NDNSF-DI developer, I need the launcher to report what is truly executed
and what is still gated, so future LLM planner and real artifact work starts
from accurate evidence.

**Why this priority**: The current native tracer uses placeholder artifacts. A
successful topology/provisioning run must not be described as full ONNX
inference unless the user request path and real artifacts are available.

**Independent Test**: Run the launcher and check that `summary.json` records
provider checks, dependency/user path status, and the next gate.

**Acceptance Scenarios**:

1. **Given** placeholder native tracer artifacts, **When** the MiniNDN launcher
   finishes, **Then** it reports topology/provider evidence as accepted and full
   request execution as gated.
2. **Given** a missing provider binary or role, **When** the launcher runs,
   **Then** it fails closed with a `FAILURE` marker and failure reason.

### Edge Cases

- MiniNDN is importable but the command is not run as root.
- The topology file is missing required nodes.
- The provider executable is missing or not built.
- A role is missing from the generated native plan.
- The current placeholder artifacts cannot support real ONNX execution.

## Requirements

### Functional Requirements

- **FR-001**: The system MUST provide a real MiniNDN native tracer command.
- **FR-002**: The command MUST support `--quick-smoke`, `--assignment default`,
  and `--assignment alternate`.
- **FR-003**: The command MUST generate the native tracer policy bundle before
  starting MiniNDN.
- **FR-004**: The command MUST validate required MiniNDN topology nodes.
- **FR-005**: The command MUST start MiniNDN and NFD when run in normal mode
  with root privileges.
- **FR-006**: The command MUST map each native DI role to an explicit provider
  identity and MiniNDN node.
- **FR-007**: The command MUST write `summary.json`, `summary.txt`,
  `assignment.csv`, logs, and a `SUCCESS` or `FAILURE` marker.
- **FR-008**: The command MUST record whether NDNSF security bootstrap was
  executed, skipped, or blocked.
- **FR-009**: The command MUST record whether full user/dependency execution is
  executed or gated.
- **FR-010**: The command MUST fail closed for missing topology nodes, missing
  provider binaries, missing plan roles, or MiniNDN root requirements.

### Key Entities

- **Native Role Assignment**: Role, provider identity, MiniNDN node, assignment
  name.
- **MiniNDN Run Evidence**: Output directory, summary files, logs, marker,
  MiniNDN status, security status, provider check status.
- **Execution Gate**: A named reason why full request/dependency execution is
  not yet claimed.

## Success Criteria

### Measurable Outcomes

- **SC-001**: Quick-smoke completes without starting MiniNDN and verifies all
  static prerequisites.
- **SC-002**: Normal mode writes a complete evidence directory or records a
  clear hard blocker.
- **SC-003**: Default and alternate assignments produce distinct provider
  identity rows in `assignment.csv`.
- **SC-004**: The launcher never records `SUCCESS` unless provider role checks
  completed or a documented hard environmental blocker was intentionally tested.
- **SC-005**: Full unit tests still pass after adding the launcher.

## Assumptions

- The native tracer placeholder artifacts are enough for provider plan and
  runner wiring checks, but not a claim of full ONNX inference.
- Full request execution needs either a native tracer user driver or real
  artifact-ready models; that work remains gated after this feature unless the
  current repo already exposes a runnable driver.
