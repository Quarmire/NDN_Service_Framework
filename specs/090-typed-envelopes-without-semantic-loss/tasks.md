# Tasks: Typed Envelopes Without Semantic Loss

## Phase 1 - Inventory And Gates

- [x] T001 [US3] Inventory typed envelopes, duplicate aliases, domain state, stored
  state, producers, consumers, tests, and external-impact decisions.
- [x] T002 [US1] [US2] Freeze compatibility epoch, typed-authority rule, alias map,
  counters, failure behavior, exit criteria, and fixtures.
- [x] T003 Run strict structure, analyze, and pre-implementation audit; resolve
  every blocker.

## Phase 2 - Core Compatibility Decoder

- [x] T004 [US1] [US2] Add typed-only/legacy/matching/conflict/malformed/unknown tests first in `tests/python/test_ndnsf_ack_compatibility_v2.py`.
- [x] T005 [US1] [US2] Add v2 schema validation, compatibility mode, decode result, and
  thread-safe counters in `pythonWrapper/ndnsf/runtime_telemetry.py`.
- [x] T006 [US1] [US2] Export the shared API from `pythonWrapper/ndnsf/__init__.py` and prove typed authority/fail-closed behavior.

## Phase 3 - Producer And Consumer Migration

- [x] T007 [US1] [US3] Switch Repo ACK producers/consumers in `NDNSF-DistributedRepo/pythonWrapper/py_repoclient/{orchestration.py,__init__.py}` to typed-only envelope fields.
- [x] T008 [US1] [US3] Switch DI Python producers/consumers in `NDNSF-DistributedInference/ndnsf_distributed_inference/{provider.py,runtime_v1.py,deployment.py}` to typed-only envelope fields.
- [x] T009 [US1] [US3] Switch `NDNSF-DistributedInference/cpp/ndnsf-di/NativeProviderReadiness.cpp` and NativeTracer diagnostics to typed-only fields.
- [x] T010 [US3] Preserve GenericAckMetadata and all classified domain/stored fields.
- [x] T011 [US2] [US3] Add forbidden flat-emission scans to `tests/python/test_ndnsf_ack_compatibility_v2.py` and run existing Repo/DI/UAV restart/persistence regressions.

## Phase 4 - Acceptance

- [ ] T012 Run focused and full Core/DI/Repo/UAV Python and C++ regressions.
- [ ] T013 Run security and exact-wire/persistence regressions.
- [ ] T014 [US1] [US2] Run mixed-reader and typed-only MiniNDN smokes and record counters.
- [ ] T015 Verify independent rollback and compatibility deadline documentation.
- [ ] T016 Run post-audit/analyze/converge, update parent T052-T058, and commit
  implementation/closure separately.
