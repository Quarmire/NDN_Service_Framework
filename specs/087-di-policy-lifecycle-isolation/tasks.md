# Tasks: DI Policy And Lifecycle Isolation

## Phase 1 - Gates And Tests

- [x] T001 Record entry imports, callers, tests, branch, HEAD, dirty boundary,
  and CodeGraph status in `evidence/entry-inventory.md`.
- [x] T002 Freeze coordinator-off and advisory-retention commands/thresholds in
  `evidence/experiment-gate.md` before implementation.
- [x] T003 Run strict structure analysis and pre-implementation Spec Kit audit;
  resolve all blockers.
- [x] T004 [US1] [US2] Add default-import, planner-registry, deployment-authority, and
  experimental-disabled tests.
- [x] T005 [US3] Add typed retry/idempotency tests before replacing string inference.

## Phase 2 - Default Runtime

- [x] T006 [US1] Prove coordinator-free planning and admission paths are the default.
- [x] T007 [US1] Make `default_planner_registry()` executable-only and update callers.
- [x] T008 [US2] Move advisory coordination under the explicit experimental package;
  remove default exports and update research callers/tests.
- [x] T009 [US2] Move semantic cache under the explicit experimental package; remove
  default exports and update examples/tests.
- [x] T010 [US1] Delete unused Merge `DeploymentManager` ref-count authority after an
  exact caller scan; preserve descriptive deployment publication.
- [x] T011 [US2] Preserve provider-local Exact Forward Cache and its strict-key tests.
- [x] T012 [US3] Replace retry string inference with typed reason plus explicit
  idempotency metadata; update all callers.

## Phase 3 - Acceptance

- [x] T013 Run focused and full DI Python regressions plus Core build/security.
- [x] T014 Run coordinator-off NativeTracer/Qwen/multi-user MiniNDN acceptance.
- [x] T015 Run ten matched advisory campaigns and retain/delete advisory using
  the frozen statistical gate.
- [x] T016 Verify independent rollback, run post-audit/analyze/converge, update
  parent T027-T034 and commit implementation/closure separately.
