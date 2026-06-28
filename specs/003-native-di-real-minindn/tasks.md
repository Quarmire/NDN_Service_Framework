# Tasks: Real MiniNDN Native DI Tracer

**Input**: Design documents from `specs/003-native-di-real-minindn/`

**Prerequisites**: `spec.md`, `plan.md`, `research.md`, `data-model.md`,
`contracts/`, `quickstart.md`

**Tests**: Each P-task must be marked complete only after its acceptance command
passes or records a hard environmental blocker.

## Phase 1: MiniNDN Launcher Skeleton

- [x] P1 Implement `Experiments/NDNSF_DI_NativeTracer_Minindn.py` with quick-smoke, argument parsing, output directory setup, policy generation, and summary writers

**Acceptance**: `python3 Experiments/NDNSF_DI_NativeTracer_Minindn.py --quick-smoke` passes.

**Accepted**: Quick-smoke passed and verified topology, policy generator, and provider binary.

---

## Phase 2: Topology And NFD Startup

- [x] P2 Add MiniNDN startup, topology validation, NFD startup, route calculation, and cleanup to `Experiments/NDNSF_DI_NativeTracer_Minindn.py`

**Acceptance**: A sudo run starts MiniNDN/NFD or records a clear MiniNDN blocker.

**Accepted**: Sudo default and alternate runs started MiniNDN, NFD, route calculation, and cleanup. Non-root normal mode recorded the expected blocker.

---

## Phase 3: Role Assignment Evidence

- [x] P3 Generate and validate default and alternate role-to-provider/node assignments in `assignment.csv`

**Acceptance**: Default and alternate evidence runs produce distinct assignment rows covering all plan roles.

**Accepted**: `/tmp/ndnsf-di-real-minindn-default/assignment.csv` and `/tmp/ndnsf-di-real-minindn-alternate/assignment.csv` cover all four roles with distinct provider identities.

---

## Phase 4: Provider Role Checks

- [x] P4 Run role-specific `di-native-provider --check-only` commands on assigned MiniNDN nodes and collect per-provider logs

**Acceptance**: Provider checks pass for `/Backbone`, `/Head/Shard/0`, `/Head/Shard/1`, and `/Merge`.

**Accepted**: Provider checks pass with `--check-only --wiring-check-only` on the assigned MiniNDN nodes. Logs show each role registered one runner.

---

## Phase 5: Security Bootstrap Status

- [x] P5 Record security bootstrap status for controller, user, group, and provider identities without bypassing NDNSF security assumptions

**Acceptance**: `summary.json` records `securityBootstrap.status` as `executed`, `blocked`, or `not-required-for-check-only`, with a reason.

**Accepted**: `summary.json` records `securityBootstrap.status=not-required-for-check-only` with an explicit reason.

---

## Phase 6: User And Dependency Execution Gate

- [x] P6 Record full user request and dependency execution as executed only if an actual native tracer request path runs; otherwise gate it with the missing driver/artifact reason

**Acceptance**: `summary.json` includes honest `userExecution` and `dependencyExecution` fields.

**Accepted**: `summary.json` records `userExecution.status=gated` and `dependencyExecution.status=gated` with missing driver/artifact reasons.

---

## Phase 7: Final Validation And Documentation

- [x] P7 Validate quick-smoke, default assignment, alternate assignment, focused tests, full unit tests, and update quickstart accepted evidence

**Acceptance**: All validation commands pass or record hard blockers, and this file marks P1-P7 complete.

**Accepted**: Quick-smoke, non-root blocker, default MiniNDN run, alternate MiniNDN run, focused tests, and full unit tests passed or recorded the expected blocker.

---

## Dependencies & Execution Order

- P1 creates the command and summary surface.
- P2 depends on P1.
- P3 depends on generated policy from P1.
- P4 depends on P2 and P3.
- P5 can complete after P2 because this feature records status honestly.
- P6 depends on P4 and current driver/artifact availability.
- P7 completes after P1-P6 acceptance.

## Implementation Strategy

1. Keep the launcher focused and evidence-driven.
2. Reuse feature 002 policy generation.
3. Prefer real MiniNDN startup for acceptance, with explicit hard blockers.
4. Do not claim full inference while native tracer artifacts remain placeholders.
5. Preserve the same result-file contract shape for future LLM planner work.
