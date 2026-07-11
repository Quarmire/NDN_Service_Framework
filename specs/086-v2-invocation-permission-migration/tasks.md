# Tasks: V2 Invocation And Permission Migration

## Phase 1 - Entry gate

- [x] T001 Capture branch, HEAD, worktree, CodeGraph status, exact V1 symbols,
  registrations, callers, tests, docs, and build entries in
  `evidence/entry-inventory.md`.
- [x] T002 Record the external ABI decision and rollback boundary in
  `evidence/entry-baseline.md`.
- [x] T003 Run the focused build/unit/security/Python baseline and record exact
  commands and results in `evidence/entry-baseline.md`.
- [x] T004 Run strict structure scan, Spec Kit analysis, and pre-implementation
  audit; resolve all blocking findings and save the audit.

## Phase 2 - Authorization model

- [x] T005 [US2] Add `ndn-service-framework/ServiceAuthorizationTable.hpp` with the
  record, permission kind, epoch-aware replacement, exact lookup, deterministic
  snapshot, and thread safety.
- [x] T006 [US2] Add unit tests covering invalid records, user/provider kinds,
  same/newer/lower epoch, replacement, exact lookup, and concurrent reads.
- [x] T007 [US2] Migrate `ServiceUser.hpp/.cpp` PermissionResponse application and
  permission checks to `ServiceAuthorizationTable`.
- [x] T008 [US2] Migrate `ServiceProvider.hpp/.cpp` PermissionResponse application and
  permission checks to `ServiceAuthorizationTable`.
- [x] T009 [US2] Migrate `tests/unit-tests/encrypted-permission-response.t.cpp` to
  assert permission kind and policy epoch, then prove the focused tests pass.

## Phase 3 - V1 invocation removal

- [x] T010 [US1] Prove no in-repository callers require
  `ServiceUser::PublishRequest`, then delete its declaration/definition.
- [x] T011 [US1] Remove the V1 fallback from `ServiceProvider::OnRequest`; add a
  malformed/legacy-name fail-closed regression.
- [x] T012 [US1] Delete V1 request helpers and regex declarations/definitions from
  `utils.hpp/.cpp` after exact caller proof.
- [x] T013 [US1] Remove legacy V1 decrypt/preprocess handlers only when their caller
  and callback-registration sets are empty.
- [x] T014 [US1] Remove `searchByFunctionName` and delete
  `UserPermissionTable.hpp` after all callers use the new table.
- [x] T015 [US1] Delete BloomFilter sources/includes and every exact wscript/build
  entry; verify clean configure/build from metadata.

## Phase 4 - Legacy permission discovery removal

- [x] T016 [US2] Remove NDNSD token-name permission installation and its registered
  success/error callbacks while retaining encrypted controller permissions.
- [x] T017 [US2] Remove `parsePermissionTokenName` and related token-name builders
  only if the final caller scan proves they are V1-only.
- [x] T018 [US1] Scan and remove Direct aliases/terminology from production API while
  retaining Targeted APIs.

## Phase 5 - Validation and closure

- [x] T019 [US3] Run full C++ build/unit tests and focused Core Python tests.
- [x] T020 [US3] Run the full security aggregate plus normal, Targeted, selection,
  replay, bootstrap, NAC-ABE, and collaboration regressions.
- [x] T021 [US3] Run forbidden-symbol/build scans and CodeGraph sync/caller audit;
  record results in `evidence/final-structural-audit.md`.
- [x] T022 [US3] Run matched MiniNDN normal and Targeted smoke and record completion,
  p50, p95, and comparison in `evidence/minindn-acceptance.md`.
- [x] T023 Verify independent rollback in a detached worktree and record it.
- [x] T024 Run post-implementation Spec Kit audit, analyze, and converge; map
  FR/SC/task/evidence in `traceability.md`.
- [x] T025 Update Spec 084 T018-T026 and child acceptance evidence, then commit
  implementation and closure independently.

## Phase 6 - Convergence

- [x] T026 [SC-004] Reconstruct the missing pre-migration normal/Targeted
  MiniNDN latency baseline at parent commit `419cd2b` using benchmark-only
  instrumentation, then evaluate the NFR-003 15 percent p95 gate and record
  the matched commands and results in `evidence/pre-migration-minindn-baseline.md`
  (partial).
