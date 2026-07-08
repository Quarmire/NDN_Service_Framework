# Tasks: Repo Core Discovery Selection

## Phase 1: Setup

- [x] T001 Point Spec Kit active feature to `specs/059-repo-core-discovery-selection`.
- [x] T002 Document the narrow Repo discovery-selection design in `spec.md` and `plan.md`.

## Phase 2: Implementation

- [x] T003 Add `discovery_record_from_ack` in `NDNSF-DistributedRepo/pythonWrapper/py_repoclient/__init__.py`.
- [x] T004 Add `ready_capability_from_ack` and make `_capacity_selector` skip typed unready/draining providers.
- [x] T005 Keep `capability_from_ack` legacy fallback behavior unchanged for ACKs without core hints.

## Phase 3: Tests

- [x] T006 Add `tests/python/test_ndnsf_repo_core_discovery_selection.py`.
- [x] T007 Cover typed ready, typed draining, typed unready, and legacy-only fallback.
- [x] T008 Cover all-unready selector output.

## Phase 4: Validation

- [x] T009 Run focused Repo core discovery selection test.
- [x] T010 Run existing app core envelope migration test.
- [x] T011 Run core service discovery test.
- [x] T012 Run `git diff --check`.
- [x] T013 Mark all tasks complete.

