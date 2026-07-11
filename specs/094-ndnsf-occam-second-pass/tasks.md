# Tasks: NDNSF Occam Second Pass

## Phase 1: Baseline And Disposition

- [x] T001 [US5] Record CodeGraph status, git state, build/test baseline, disk, and active Occam findings.
- [x] T002 [US5] Produce a Core/DI/UAV/Repo disposition matrix with REMOVE, CONSOLIDATE, KEEP, and DEFER decisions plus exact callers.
- [x] T003 [US5] Record migration, rollback, security, persistence, and deadline constraints for every removal/defer decision.
- [x] T004 [US5] Run Spec Kit structure/analyze and pre-implementation audit; resolve every blocking finding before edits.

## Phase 2: Canonical DI Driver And Evidence Path

- [x] T005 [US1] Remove process-pool CLI choices, runtime-profile values, GUI choices/defaults, campaign forwarding, worker-batch protocol, helpers, and tests.
- [x] T006 [US1] Make threaded the canonical GUI/runtime-profile open-loop default while retaining child diagnostic mode.
- [x] T007 [US1] Remove `Experiments/NDNSF_DI_RuntimeAware_RpsSweep.py`, its dedicated tests, and canonical-doc references; document the strict NativeTracer harness recipe.
- [x] T008 [US1] Run DI parser/profile/GUI/campaign tests and prohibited-symbol scans.

## Phase 3: Canonical DI GUI And Artifact API

- [ ] T009 [US2] Remove legacy runtime profile classes/loaders/writers and duplicate Script Controller/User/Provider tabs while retaining distinct supporting tools.
- [ ] T010 [US2] Remove `repo_manifests` public keywords and the dual-name selector; rename internal artifact-loading helpers and update tests/docs.
- [ ] T011 [US2] Run headless and Tk GUI tests plus DI artifact/deployment/runtime tests.

## Phase 4: Canonical Persistent Repo

- [ ] T012 [US3] Remove public `InMemoryRepoStore`, `makeMemoryRepoStore`, and default memory-backed `RepoCore`/`RepoNode` constructors.
- [ ] T013 [US3] Convert examples/tests to explicit temporary SQLite tiered stores or test-local fakes; retain bounded hot-cache tests and restart persistence tests.
- [ ] T014 [US4] Remove ignored `producer_retention_s` from Repo constructor, CLI, MiniNDN harness, tests, and docs.
- [ ] T015 [US4] Remove ignored `isolated_runtime` from the private request helper and all callers.
- [ ] T016 [US4] Remove redundant `legacyStatus` metadata and rename `legacy_fields` to `capability_fields` without changing typed payload content.
- [ ] T017 [US3] Build and run Repo C++/Python focused tests, exact-packet tests, cache tests, restart tests, and symbol scans.

## Phase 5: Accurate Recurrence Guard

- [ ] T018 [US5] Replace broad false-positive Occam rules with ownership-aware rules for removed mechanisms.
- [ ] T019 [US5] Expand audit fixtures for prohibited active code, allowed app-owned types, tests/docs/history, and internal binding exceptions.
- [ ] T020 [US5] Run the audit with `--fail-on-active` and record zero active prohibited findings.

## Phase 6: Integrated Verification

- [ ] T021 [US5] Run full Python tests and affected C++ Core/DI/UAV/Repo build/test suites.
- [ ] T022 [US1] Run a 60-second threaded NativeTracer MiniNDN validation and retain scheduling/dependency/success/latency evidence.
- [ ] T023 [US5] Run relevant Core, Repo, and UAV MiniNDN quick checks; record environmental skips as missing evidence.
- [ ] T024 [US5] Compare before/after maintained source and public surfaces; verify no proposal file changed.

## Phase 7: Closure

- [ ] T025 [US5] Produce final disposition, adversarial review, traceability, commands, results, and residual-risk evidence.
- [ ] T026 [US5] Run Spec Kit analyze, post-implementation audit, and converge; append any real gap as a new task and execute it.
- [ ] T027 [US5] Update the agent context, GSD state/health, CodeGraph index, git diff/status, and close only after all acceptance criteria are met.
