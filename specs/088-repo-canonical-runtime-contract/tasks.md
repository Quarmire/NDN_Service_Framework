# Tasks: Repo Canonical Runtime And Contract

## Phase 1 - Decision And Fixtures

- [x] T001 [US3] Record C++/Python symbols, names, schemas, callers, ownership,
  persistence and parity in `evidence/entry-inventory.md`.
- [x] T002 [US3] Approve the Repo ADR against semantic parity, security,
  persistence, recovery, concurrency, observability and maintainability.
- [x] T003 [US1] [US2] Freeze black-box exact packet, SQLite/cache, manifest,
  catalog, quorum, tombstone, idempotency, failover, repair, Targeted,
  malformed and metrics fixtures.
- [x] T004 [US1] Prove deployed configuration is SQLite-authoritative with
  bounded/disabled hot cache and no memory-only production mode.

## Phase 2 - Canonical Adapter

- [x] T005 [US1] Fill missing C++/pybind public object contract parity.
- [x] T006 [US3] Move Repo network orchestration into `py_repoclient` in
  independently revertible slices.
- [x] T007 [US3] Migrate DI/UAV callers to the public Repo client.
- [x] T008 [US3] Delete duplicate DI Repo policy and default server exports after
  caller and fixture proof.
- [x] T009 [US1] Validate stored-state upgrade, restart, exact bytes and rollback-open.
- [x] T010 [US2] Enforce private-operation authorization and ordinary-client negatives.
- [x] T011 [US2] Validate reservation, quorum, tombstone, catalog anti-entropy,
  failover, idempotent repair, backpressure and telemetry.

## Phase 3 - Acceptance

- [x] T012 Run C++ and Python Repo regressions plus full Core security.
- [x] T013 Run at least three matched 60-second MiniNDN RF=2/W=ALL campaigns.
- [x] T014 Verify rollback, post-audit/analyze/converge, parent T035-T044,
  traceability and separate implementation/closure commits.

## Phase 4 - Convergence

- [x] T015 [US3] Remove the duplicate unversioned C++ standalone NDNSF network
  registration path and `DistributedRepoNodeApp` standalone target; retain the
  C++ canonical object/local-service library and the Python versioned deployed
  adapter.
- [x] T016 [US3] Add a forbidden-path regression proving no production Repo
  node registers the unversioned `/NDNSF/DistributedRepo/<operation>` surface.
- [x] T017 [US3] Refresh stale entry inventory, quickstart, ADR wording, caller
  scan and rollback notes to describe the one-runtime result.
- [x] T018 Re-run C++/Python/Repo/security focused gates after duplicate runtime
  removal and record the post-convergence verdict.
