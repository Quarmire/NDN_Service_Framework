# Tasks: Tk GUI For NDNSF-DI Operators

- [x] T001 Create Spec Kit documents for the Tk GUI feature.
- [x] T002 Add role profile serialization helpers in `gui.py`.
- [x] T003 Add a role process supervisor with status transitions and stop/restart support.
- [x] T004 Extend Controller/User/Provider tabs with Start, Stop, Restart, Show Command, and status text.
- [x] T005 Add Deployment Runner profile controls: Load Profile, Save Profile, Start All, Stop All, Clear Logs.
- [x] T006 Use `shlex.split` for role extra args and keep commands inspectable.
- [x] T007 Add non-display Python tests for profiles, command building, and process state.
- [x] T008 Run py_compile, focused tests, diff check, CodeGraph sync/status, and record evidence.

## Evidence

- `PYTHONPATH=NDNSF-DistributedInference PYTHONPYCACHEPREFIX=/tmp/ndnsf_pycache python3 -m py_compile NDNSF-DistributedInference/ndnsf_distributed_inference/gui.py tests/python/test_ndnsf_di_tk_gui.py`: passed.
- `PYTHONPATH=NDNSF-DistributedInference PYTHONPYCACHEPREFIX=/tmp/ndnsf_pycache python3 tests/python/test_ndnsf_di_tk_gui.py`: 6 tests passed.
- `git diff --check`: passed.
- `codegraph sync . && codegraph status .`: index is up to date.
