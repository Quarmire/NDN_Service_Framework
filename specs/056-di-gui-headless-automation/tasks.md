# Tasks: NDNSF-DI GUI Headless Automation

## Phase 1 - CLI Contract

- [x] T001 Add `--headless` / `-headless`.
- [x] T002 Add auto-run flags for User, Provider, and Controller, including
      single-dash underscore aliases.
- [x] T003 Add full profile and per-role config file flags.
- [x] T004 Add fake/direct runtime selection, request trigger, duration, startup
      timeout, and output JSON flags.

## Phase 2 - Runtime Reuse

- [x] T005 Reuse `ThreeRoleGuiProfile` and `RoleRuntimeController`; do not
      instantiate Tk in headless mode.
- [x] T006 Merge role config files into the active profile.
- [x] T007 Start selected roles in Controller, Provider, User order.
- [x] T008 Send configured user request and record status, message, payload,
      elapsed time, role state, and errors.
- [x] T009 Stop roles in reverse order.

## Phase 3 - Tests And Documentation

- [x] T010 Add unit tests for CLI parsing, config merge, fake lifecycle, JSON
      output, and request precondition errors.
- [x] T011 Add example role config files.
- [x] T012 Document GUI headless automation in the runtime workflow.
- [x] T013 Connect the MiniNDN GUI launcher to the headless preflight path.

## Phase 4 - Validation

- [x] T014 Run Python compile checks.
- [x] T015 Run GUI helper unit tests.
- [x] T016 Run headless fake CLI smoke.
- [x] T017 Run MiniNDN GUI launcher preflight/headless smoke.
- [x] T018 Run `git diff --check` and CodeGraph sync/status.

## Evidence

- `PYTHONPATH=NDNSF-DistributedInference:pythonWrapper PYTHONPYCACHEPREFIX=/tmp/ndnsf_pycache python3 -m py_compile NDNSF-DistributedInference/ndnsf_distributed_inference/gui.py tests/python/test_ndnsf_di_tk_gui.py Experiments/NDNSF_DI_GUI.py Experiments/NDNSF_DI_GUI_Minindn.py`: passed.
- `PYTHONPATH=NDNSF-DistributedInference:pythonWrapper PYTHONPYCACHEPREFIX=/tmp/ndnsf_pycache python3 tests/python/test_ndnsf_di_tk_gui.py`: 16 tests passed.
- `PYTHONPATH=NDNSF-DistributedInference:pythonWrapper PYTHONPYCACHEPREFIX=/tmp/ndnsf_pycache python3 Experiments/NDNSF_DI_GUI.py -headless -controller_auto_run -provider_auto_run -user_auto_run -user_config=examples/python/NDNSF-DistributedInference/gui_user_hello.config -provider_config=examples/python/NDNSF-DistributedInference/gui_provider_hello.config -controller_config=examples/python/NDNSF-DistributedInference/gui_controller_hello.config --runtime-mode fake --send-user-request --output-json /tmp/ndnsf-di-gui-headless.json`: passed, wrote `payload_text=HELLO`.
- `PYTHONPATH=NDNSF-DistributedInference:pythonWrapper PYTHONPYCACHEPREFIX=/tmp/ndnsf_pycache python3 Experiments/NDNSF_DI_GUI_Minindn.py --preflight-only`: passed.
- `sudo -n PYTHONPATH=NDNSF-DistributedInference:pythonWrapper PYTHONPYCACHEPREFIX=/tmp/ndnsf_pycache python3 Experiments/NDNSF_Python_Hello_Minindn.py --output-dir /tmp/ndnsf-gui-headless-python-hello-minindn --startup-wait-s 3 --controller-wait-s 2 --ack-timeout-ms 2000 --timeout-ms 8000`: passed, printed `PYTHON_HELLO_MININDN_OK`.
- Attempted `Experiments/NDNSF_DI_GUI_Minindn.py --run-minindn --case yolo-2x2 --no-gui`; blocked before MiniNDN workload by local `ultralytics/matplotlib` import conflict, so the HELLO MiniNDN smoke was used as the stable GUI-relevant network validation.
