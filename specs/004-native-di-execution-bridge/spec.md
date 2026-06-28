# Feature Specification: Native DI Execution Bridge

**Feature Branch**: `004-native-di-execution-bridge`

**Created**: 2026-06-24

**Status**: Accepted

**Input**: User request: "根据这些文档，你觉得NDNSF-DI下一个大计划应该是什么，该如何安排TASK并更新文档；循环执行任务，直到所有TASK完成并验证。"

## User Scenarios & Testing

### User Story 1 - Preserve Executed Native Baseline (Priority: P1)

As an NDNSF-DI developer, I need the real MiniNDN native tracer harness to also
run the local native execution baseline, so the evidence directory separates
"native plan executes" from "native providers are placed on MiniNDN nodes."

**Why this priority**: Feature 003 proves real MiniNDN provider wiring but gates
full execution. The old shell tracer proves local execution but does not start a
real topology. The next plan should bridge those two evidence surfaces before
claiming full networked inference.

**Independent Test**: Run the Python tracer in local-execution-only mode and
verify `local-execution-timing.csv` plus `summary.json.localExecution`.

### User Story 2 - Keep Full Network Execution Honest (Priority: P2)

As an NDNSF-DI developer, I need summary fields that distinguish local baseline
execution, MiniNDN provider wiring, and full user/dependency execution, so
future reports cannot overstate the result.

**Why this priority**: The native tracer artifacts are still placeholders, and
the full `/Inference/NativeTracer` user request path is still the next gate.

**Independent Test**: Inspect `summary.json` after a local-only and normal run.
`localExecution.status` must be `executed` when the C++ smoke runs, while
`userExecution.status` remains `gated` until the real user driver exists.

### User Story 3 - Document The Next Big Gate (Priority: P3)

As an NDNSF-DI developer, I need the roadmap and build docs to name the next
gate precisely, so implementation does not jump prematurely into LLM planner
work.

**Why this priority**: The project needs a clear runway from local native
execution to real MiniNDN user request execution.

**Independent Test**: Read `docs/native-di-roadmap.md`,
`docs/experiments.md`, and `docs/build-and-test.md`; they must expose the new
bridge command and the remaining full-execution gate.

## Requirements

- **FR-001**: The Python MiniNDN tracer MUST support `--local-execution-only`.
- **FR-002**: Local execution MUST run the C++ schema smoke, manifest smoke, and
  provider-session smoke against the generated `/Inference/NativeTracer`
  bundle.
- **FR-003**: Local execution MUST write and validate
  `local-execution-timing.csv`.
- **FR-004**: `summary.json` MUST include a `localExecution` object.
- **FR-005**: The harness MUST keep `userExecution.status=gated` until an
  actual native tracer user request driver submits `/Inference/NativeTracer`.
- **FR-006**: The harness MUST keep network dependency execution gated until
  dependencies flow through `NdnsfCollaborationDependencyIo` in a real NDNSF
  request.
- **FR-007**: Docs MUST describe the bridge result and the remaining gate.

## Key Entities

- **Local Native Execution Baseline**: C++ plan/manifest execution using
  in-memory dependency I/O, recorded in `local-execution-timing.csv`.
- **MiniNDN Provider Wiring Evidence**: Real topology run that places
  role-specific native providers on MiniNDN nodes.
- **Full Network Execution Gate**: Future evidence where a user driver submits
  `/Inference/NativeTracer` and dependencies are exchanged through NDNSF large
  data.

## Success Criteria

- **SC-001**: `python3 Experiments/NDNSF_DI_NativeTracer_Minindn.py --quick-smoke`
  passes.
- **SC-002**: `python3 Experiments/NDNSF_DI_NativeTracer_Minindn.py --local-execution-only`
  passes and writes valid local execution timing.
- **SC-003**: Existing focused native DI unit tests pass.
- **SC-004**: Docs name the bridge as complete and identify the next full
  network execution task.

## Accepted Evidence

- Local execution bridge: `/tmp/ndnsf-di-execution-bridge-local`
- Default MiniNDN bridge: `/tmp/ndnsf-di-execution-bridge-default`
- Alternate MiniNDN bridge: `/tmp/ndnsf-di-execution-bridge-alternate`
- Focused native DI unit tests: passed
- Full unit tests: `build/unit-tests` passed 158 test cases
