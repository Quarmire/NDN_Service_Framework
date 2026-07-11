# Tasks: Native DI Threaded RPS Boundary

## Phase 1: Freeze And Preflight

- [ ] T001 [US1] Record the stable 1 RPS anchor, commit, controls, gates, exact command template, and resource limits.
- [ ] T002 [US1] Dry-run 2/4/8 RPS commands and verify request caps 120/240/480 with only rate/output differences.
- [ ] T003 [US1] Confirm clean runtime source, no stale MiniNDN process, and sufficient disk/memory.

## Phase 2: Coarse Search

- [ ] T004 [US1] Run 2 RPS for 60 seconds and classify all gates.
- [ ] T005 [US1] If 2 RPS passes, run 4 RPS and classify; otherwise stop coarse search.
- [ ] T006 [US1] If 4 RPS passes, run 8 RPS and classify; otherwise stop coarse search.

## Phase 3: Boundary Refinement

- [ ] T007 [US1] Bisect the highest-stable/first-unstable bracket until width is at most 0.25 RPS or record a stop condition.
- [ ] T008 [US1] Obtain three matched runs total at the highest tested stable point.
- [ ] T009 [US1] Aggregate throughput, p50/p95, slip, completion, provider, dependency, and failure counters without dropping failed runs.

## Phase 4: Closure

- [ ] T010 [US1] Produce ARS Material Passport, reproducibility report, and 11/11 fallacy scan.
- [ ] T011 [US1] Update the canonical DI runtime workflow with the bounded conclusion and raw paths.
- [ ] T012 [US1] Run Spec Kit analyze/audit/converge, CodeGraph, GSD health, tests/diff checks, and close only when evidence is reproducible.
