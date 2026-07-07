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

## Phase 5 - GUI Automation

- [x] T019 Add Xvfb-backed Tk widget tests that instantiate the real
      `DistributedInferenceGui`.
- [x] T020 Verify the first three operational tabs are `User`, `Provider`, and
      `Controller`.
- [x] T021 Verify profile application updates editable tab fields and the
      profile can be read back from the GUI.
- [x] T022 Verify Run All starts fake Controller, Provider, and User runtimes
      through the same button/controller path used by the GUI.
- [x] T023 Verify the User Send Request button updates the response pane and
      sends the configured payload to the fake runtime.
- [x] T024 Add optional PyAutoGUI visual screenshot smoke that skips when
      PyAutoGUI is unavailable.
- [x] T025 Fix GUI response routing and Tk shutdown issues found by the widget
      tests.

## Phase 6 - Qwen MiniNDN Headless Experiment

- [x] T026 Add a `qwen-minindn` headless experiment mode that launches the
      canonical NativeTracer MiniNDN harness from the GUI CLI entrypoint.
- [x] T027 Force the real full-network Qwen proportional path through
      `llm-proportional`, `--no-local-execution-only`, and `--full-network`.
- [x] T028 Record harness status, runner mode, execution status, utilization,
      result directory, command, and stdout tail in the headless JSON summary.
- [x] T029 Add unit coverage for argument parsing and command construction.
- [x] T030 Run dry-run validation.
- [x] T031 Run the real Qwen MiniNDN experiment.
- [x] T032 Run compile checks, helper tests, diff check, and CodeGraph sync.

## Phase 7 - Shared Non-Headless Qwen MiniNDN GUI

- [x] T033 Add a non-headless `Qwen MiniNDN` tab with editable experiment
      parameters.
- [x] T034 Reuse the same `build_qwen_minindn_command()` helper used by
      `--headless-experiment qwen-minindn`.
- [x] T035 Run the experiment in a background subprocess, stream output into
      the GUI, support Stop, and write compact GUI summary JSON.
- [x] T036 Add Xvfb widget tests for command construction and dry-run button
      execution without starting MiniNDN.
- [x] T037 Update the runtime workflow documentation.

## Phase 8 - GUI Qwen MiniNDN Sweep

- [x] T038 Add editable target-RPS sweep list and repeat count fields to the
      `Qwen MiniNDN` tab.
- [x] T039 Expand sweep values into sequential qwen-minindn commands using the
      same `build_qwen_minindn_command()` path as single run and headless mode.
- [x] T040 Write each sweep run to a distinct output subdirectory.
- [x] T041 Add a `Run Sweep` button, shared Stop behavior, and log headers for
      each sweep item.
- [x] T042 Add Xvfb widget tests for sweep command generation and dry-run
      button execution.
- [x] T043 Update the runtime workflow documentation.

## Phase 9 - GUI Sweep Report CSV

- [x] T044 Extract compact per-run metrics from each Qwen MiniNDN
      `summary.json` after single run or sweep completion.
- [x] T045 Write a sibling CSV report next to the GUI aggregate JSON.
- [x] T046 Include latency, throughput, success rate, dependency status,
      dependency roles, provider count, mean provider utilization, and total
      provider busy handler time.
- [x] T047 Add Xvfb widget coverage for CSV generation from representative
      run summaries.
- [x] T048 Validate CSV generation against the real GUI Qwen MiniNDN sweep
      evidence directory.

## Evidence

- `PYTHONPATH=NDNSF-DistributedInference:pythonWrapper PYTHONPYCACHEPREFIX=/tmp/ndnsf_pycache python3 -m py_compile NDNSF-DistributedInference/ndnsf_distributed_inference/gui.py tests/python/test_ndnsf_di_tk_gui.py Experiments/NDNSF_DI_GUI.py Experiments/NDNSF_DI_GUI_Minindn.py`: passed.
- `PYTHONPATH=NDNSF-DistributedInference:pythonWrapper PYTHONPYCACHEPREFIX=/tmp/ndnsf_pycache python3 tests/python/test_ndnsf_di_tk_gui.py`: 16 tests passed.
- `PYTHONPATH=NDNSF-DistributedInference:pythonWrapper PYTHONPYCACHEPREFIX=/tmp/ndnsf_pycache python3 Experiments/NDNSF_DI_GUI.py -headless -controller_auto_run -provider_auto_run -user_auto_run -user_config=examples/python/NDNSF-DistributedInference/gui_user_hello.config -provider_config=examples/python/NDNSF-DistributedInference/gui_provider_hello.config -controller_config=examples/python/NDNSF-DistributedInference/gui_controller_hello.config --runtime-mode fake --send-user-request --output-json /tmp/ndnsf-di-gui-headless.json`: passed, wrote `payload_text=HELLO`.
- `PYTHONPATH=NDNSF-DistributedInference:pythonWrapper PYTHONPYCACHEPREFIX=/tmp/ndnsf_pycache python3 Experiments/NDNSF_DI_GUI_Minindn.py --preflight-only`: passed.
- `sudo -n PYTHONPATH=NDNSF-DistributedInference:pythonWrapper PYTHONPYCACHEPREFIX=/tmp/ndnsf_pycache python3 Experiments/NDNSF_Python_Hello_Minindn.py --output-dir /tmp/ndnsf-gui-headless-python-hello-minindn --startup-wait-s 3 --controller-wait-s 2 --ack-timeout-ms 2000 --timeout-ms 8000`: passed, printed `PYTHON_HELLO_MININDN_OK`.
- Attempted `Experiments/NDNSF_DI_GUI_Minindn.py --run-minindn --case yolo-2x2 --no-gui`; blocked before MiniNDN workload by local `ultralytics/matplotlib` import conflict, so the HELLO MiniNDN smoke was used as the stable GUI-relevant network validation.
- `xvfb-run -a env PYTHONPATH=NDNSF-DistributedInference:pythonWrapper PYTHONPYCACHEPREFIX=/tmp/ndnsf_pycache python3 tests/python/test_ndnsf_di_tk_widgets.py`: 4 tests passed.
- `xvfb-run -a env PYTHONPATH=NDNSF-DistributedInference:pythonWrapper PYTHONPYCACHEPREFIX=/tmp/ndnsf_pycache python3 tests/python/test_ndnsf_di_gui_visual_smoke.py`: passed with 1 skip because PyAutoGUI is not installed.
- GUI automation found and fixed two real GUI issues: synchronous User request results were routed to the log pane instead of the response pane, and background request threads were reading Tk widgets directly.
- `PYTHONPATH=NDNSF-DistributedInference:pythonWrapper PYTHONPYCACHEPREFIX=/tmp/ndnsf_pycache python3 -m py_compile NDNSF-DistributedInference/ndnsf_distributed_inference/gui.py tests/python/test_ndnsf_di_tk_gui.py Experiments/NDNSF_DI_GUI.py`: passed.
- `PYTHONPATH=NDNSF-DistributedInference:pythonWrapper PYTHONPYCACHEPREFIX=/tmp/ndnsf_pycache python3 tests/python/test_ndnsf_di_tk_gui.py`: 18 tests passed.
- `PYTHONPATH=NDNSF-DistributedInference:pythonWrapper PYTHONPYCACHEPREFIX=/tmp/ndnsf_pycache python3 Experiments/NDNSF_DI_GUI.py -headless --headless-experiment qwen-minindn --experiment-runtime-profile examples/di-native-tracer.runtime.json --experiment-out /tmp/ndnsf-di-gui-qwen-headless-dryrun --experiment-requests 1 --experiment-concurrency 1 --experiment-provider-check-timeout 60 --experiment-dry-run --output-json /tmp/ndnsf-di-gui-qwen-headless-dryrun/gui-headless-summary.json`: passed, generated the Qwen NativeTracer MiniNDN command without starting MiniNDN.
- `sudo -n PYTHONPATH=NDNSF-DistributedInference:pythonWrapper PYTHONPYCACHEPREFIX=/tmp/ndnsf_pycache python3 Experiments/NDNSF_DI_GUI.py -headless --headless-experiment qwen-minindn --experiment-runtime-profile examples/di-native-tracer.runtime.json --experiment-out /tmp/ndnsf-di-gui-qwen-headless-minindn --experiment-requests 1 --experiment-concurrency 1 --experiment-provider-check-timeout 60 --output-json /tmp/ndnsf-di-gui-qwen-headless-minindn/gui-headless-summary.json`: passed. `summary.json` reported `status=SUCCESS`, `runnerMode=qwen-onnx-native`, `miniNDNRun=started`, `userExecution.status=executed`, `dependencyExecution.status=executed`, `requestCount=2`, `successCount=2`, `failureCount=0`, `p50Ms=230.49`, `p95Ms=331.43`, `throughputRps=3.55`.
- `git diff --check`: passed.
- `codegraph sync . && codegraph status .`: passed, index up to date.
- `PYTHONPATH=NDNSF-DistributedInference:pythonWrapper PYTHONPYCACHEPREFIX=/tmp/ndnsf_pycache python3 -m py_compile NDNSF-DistributedInference/ndnsf_distributed_inference/gui.py tests/python/test_ndnsf_di_tk_widgets.py`: passed after adding the non-headless Qwen MiniNDN tab.
- `xvfb-run -a env PYTHONPATH=NDNSF-DistributedInference:pythonWrapper PYTHONPYCACHEPREFIX=/tmp/ndnsf_pycache python3 tests/python/test_ndnsf_di_tk_widgets.py`: 6 tests passed, including the `Qwen MiniNDN` tab command preview and dry-run Run button path.
- `PYTHONPATH=NDNSF-DistributedInference:pythonWrapper PYTHONPYCACHEPREFIX=/tmp/ndnsf_pycache python3 -m py_compile NDNSF-DistributedInference/ndnsf_distributed_inference/gui.py tests/python/test_ndnsf_di_tk_widgets.py Experiments/NDNSF_DI_GUI.py`: passed after adding GUI sweep.
- `xvfb-run -a env PYTHONPATH=NDNSF-DistributedInference:pythonWrapper PYTHONPYCACHEPREFIX=/tmp/ndnsf_pycache python3 tests/python/test_ndnsf_di_tk_widgets.py`: 8 tests passed, including sweep command expansion and dry-run Run Sweep execution.
- `xvfb-run -a env PYTHONPATH=NDNSF-DistributedInference:pythonWrapper PYTHONPYCACHEPREFIX=/tmp/ndnsf_pycache python3 <inline GUI Run Sweep driver>`: passed a real non-headless GUI sweep with `Target RPS sweep list=0.2,0.4`, `Sweep repeats=1`, `Dry run only=false`, and `Wrap with sudo -n env=true`. Aggregate summary `/tmp/ndnsf-di-gui-qwen-real-sweep/gui-sweep-summary.json` reported `ok=true`, two runs, both `status=SUCCESS`, both `runnerMode=qwen-onnx-native`, both `successCount=2`, `failureCount=0`; p50/p95 were `186.23/355.94 ms` for `rps=0.2 run=1` and `200.11/321.41 ms` for `rps=0.4 run=1`.
- `xvfb-run -a env PYTHONPATH=NDNSF-DistributedInference:pythonWrapper PYTHONPYCACHEPREFIX=/tmp/ndnsf_pycache python3 tests/python/test_ndnsf_di_tk_widgets.py`: 9 tests passed after adding CSV report generation.
- `xvfb-run -a env PYTHONPATH=NDNSF-DistributedInference:pythonWrapper PYTHONPYCACHEPREFIX=/tmp/ndnsf_pycache python3 <inline GUI CSV export check>`: passed against `/tmp/ndnsf-di-gui-qwen-real-sweep`, wrote `/tmp/ndnsf-di-gui-qwen-real-sweep/gui-sweep-report.csv` with two rows. The rows include `targetRps=0.2/0.4`, `successRate=1.0`, `throughputRps=3.68/3.83`, `dependencyStatus=executed`, `providerCount=3`, and `providerMeanUtilization=0.280372/0.265451`.
