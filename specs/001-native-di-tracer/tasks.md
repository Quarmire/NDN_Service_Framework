# Tasks: Native DI Tracer

**Input**: Design documents from `specs/001-native-di-tracer/`

**Prerequisites**: `spec.md`, `plan.md`, `research.md`, `data-model.md`, `contracts/`, `quickstart.md`

**Tests**: Each task includes its own acceptance check. Mark a task `[x]` only after its check passes or a documented environmental blocker is recorded.

## Phase 1: Feature Design And Task Contract

- [x] T001 Record native DI tracer design, acceptance criteria, and T001-T009 execution plan in `specs/001-native-di-tracer/`

**Acceptance**: `spec.md`, `plan.md`, `research.md`, `data-model.md`, `contracts/`, `quickstart.md`, `checklists/requirements.md`, and `tasks.md` exist and describe the tracer path.

---

## Phase 2: Planning-Plane Tracer

- [x] T002 Create minimal tracer policy example in `examples/python/NDNSF-DistributedInference/native_di_tracer/` with service, providers, roles, artifacts, dependencies, and final-response scope
- [x] T003 Verify Python plan to C++ plan contract using generated `native-execution-plan.json` and existing C++ plan/schema smoke coverage

**Acceptance**: One command generates the tracer policy bundle, and C++ smoke tests load it without manual edits.

---

## Phase 3: Provider Readiness And Artifacts

- [x] T004 Implement or tighten provider readiness semantics so `installing`, `ready`, and `failed` are observable and only ready providers are selectable
- [x] T005 Add artifact materialization acceptance so repo/local artifacts are fetched or located, hash-verified when declared, cached, and reflected in readiness

**Acceptance**: Tests or smoke scripts prove missing/bad artifacts keep readiness non-ready and valid artifacts allow readiness.

---

## Phase 4: Native Execution Path

- [x] T006 Complete native provider end-to-end handler path so collaboration requests execute assigned roles, publish non-final outputs, and return only final-role `final-response`

**Acceptance**: Focused unit/smoke tests drive `NativeProviderHandler`, `NativeProviderSession`, and `ProviderRoleWorker` with the tracer plan.

---

## Phase 5: MiniNDN Evidence

- [x] T007 Add MiniNDN tracer experiment script for controller, user, and 2-3 providers with native DI plan inputs
- [x] T008 Write timing/evidence matrix with provider, role, prefetchMs, executeMs, publishMs, endToEndMs, status, logs, and summary

**Acceptance**: A MiniNDN run creates a result directory with policy bundle, logs, `timing.csv`, `summary.txt`, and success/failure marker.

---

## Phase 6: Second-Stage Planner Boundary

- [x] T009 Document and gate the LLM planner follow-up so runnable native tracer acceptance remains the prerequisite for real LLM planner work

**Acceptance**: Documentation states the LLM planner is next-stage work and references tracer evidence as the gate.

---

## Dependencies & Execution Order

- T001 blocks all implementation tasks.
- T002 must complete before T003.
- T003 blocks T006 and gives the generated plan used by later tests.
- T004 and T005 can proceed after T002 but must finish before final MiniNDN acceptance.
- T006 blocks T007 and T008.
- T007 and T008 form the MiniNDN evidence loop.
- T009 completes after tracer acceptance criteria are documented.

## Implementation Strategy

1. Finish T002-T003 to lock the policy and generated plan contract.
2. Finish T004-T005 so provider selection has honest readiness evidence.
3. Finish T006 so native provider execution is the real hot path.
4. Finish T007-T008 to produce MiniNDN evidence.
5. Finish T009 by documenting how the LLM planner builds on the accepted tracer.
