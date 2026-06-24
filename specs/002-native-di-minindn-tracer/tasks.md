# Tasks: Native DI MiniNDN Tracer

**Input**: Design documents from `specs/002-native-di-minindn-tracer/`

**Prerequisites**: `spec.md`, `plan.md`, `research.md`, `data-model.md`, `contracts/`, `quickstart.md`

**Tests**: Each P-task must be marked complete only after the acceptance command passes or records a hard environmental blocker.

## Phase 1: Real MiniNDN Native Tracer

- [x] P1 Document and implement MiniNDN-aware native tracer evidence command in `examples/python/NDNSF-DistributedInference/native_di_tracer/run_minindn_tracer.sh`

**Acceptance**: Command writes policy bundle, logs, `timing.csv`, summaries, marker, and records MiniNDN hard-gate status.

**Accepted**: Default and alternate evidence runs wrote full evidence directories. `--require-minindn` recorded a hard blocker in `/tmp/ndnsf-di-native-tracer-require` when run non-root.

---

## Phase 2: Native DI Data Path Hardening

- [x] P2 Add or verify fail-closed data-path checks for source input, dependency edges, artifacts, readiness, assignments, and final-response metadata

**Acceptance**: Focused negative tests and tracer validations pass.

**Accepted**: Focused Boost tests and tracer assignment/evidence validators passed.

---

## Phase 3: Evidence And Measurement

- [x] P3 Produce research-grade evidence with byte counts, timing columns, `summary.json`, `summary.txt`, process logs, and stable marker files

**Acceptance**: Evidence directory matches `contracts/evidence.md`.

**Accepted**: `timing.csv` includes byte counts and timing columns; `summary.json`, `summary.txt`, `logs/`, and marker files are present.

---

## Phase 4: Multi-Provider Cooperation Semantics

- [x] P4 Support explicit default and alternate role-to-provider assignments in tracer evidence

**Acceptance**: Evidence rows reflect the selected assignment and validation rejects unknown/missing roles.

**Accepted**: `/tmp/ndnsf-di-native-tracer-default` and `/tmp/ndnsf-di-native-tracer-alternate` use distinct provider names and pass assignment validation.

---

## Phase 5: LLM Planner Stage Gate

- [x] P5 Document the LLM planner follow-up as gated on accepted native MiniNDN tracer evidence

**Acceptance**: LLM gate contract is current and referenced by feature docs and summary.

**Accepted**: `contracts/llm-gate.md`, `quickstart.md`, and evidence summaries record the LLM planner gate.

---

## Dependencies & Execution Order

- P1 creates the evidence command used by P3.
- P2 can run after P1 and before final acceptance.
- P3 depends on P1 and uses P2 checks.
- P4 depends on P3 evidence structure.
- P5 completes after P1-P4 evidence is accepted.

## Implementation Strategy

1. Extend the existing native tracer command rather than replacing it.
2. Keep local evidence mode useful, but add hard MiniNDN gating for final validation.
3. Make timing and summary files machine-readable.
4. Add assignment validation before launching or recording a run.
5. Keep LLM planner work explicitly second-stage.
