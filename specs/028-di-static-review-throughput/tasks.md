# Tasks: NDNSF-DI Static Review and Throughput Cleanup

## Phase 1: Review Artifacts

- [x] T001 Create review plan in `specs/028-di-static-review-throughput/plan.md`.
- [x] T002 Create task list in `specs/028-di-static-review-throughput/tasks.md`.

## Phase 2: Fix Low-Risk Logic and Latency Issues

- [x] T003 Fix ACK pressure scoring in `pythonWrapper/src/ndnsf/_ndnsf.cpp`.
- [x] T004 Add ready-input fast path in `NDNSF-DistributedInference/cpp/ndnsf-di/ProviderRoleWorker.cpp`.
- [x] T005 Add unit coverage in `tests/unit-tests/distributed-inference-async-runtime.t.cpp`.

## Phase 3: Verification

- [x] T006 Build `pythonWrapper` extension.
- [x] T007 Build and run focused DI unit tests.
- [x] T008 Run `git diff --check`, `codegraph sync .`, and `codegraph status .`.

## Evidence

- Architecture review report: `/tmp/architecture-review-20260629-161103.html`.
- `cd pythonWrapper && python3 setup.py build_ext --inplace`: passed.
- `./waf build --targets=unit-tests -j4`: passed.
- `./build/unit-tests --run_test='*ProviderRoleWorker*,*NativeProviderReadiness*' --log_level=test_suite`: passed, 10 test cases, no errors.
- `PYTHONPATH=pythonWrapper python3 -c 'import ndnsf'`: passed.
- `git diff --check`: passed.
- `codegraph sync . && codegraph status .`: passed; index is up to date.
