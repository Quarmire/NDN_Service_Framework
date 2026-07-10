# DI Baseline

Executed local tests:

```bash
PYTHONPATH=pythonWrapper:NDNSF-DistributedInference:NDNSF-DistributedRepo/pythonWrapper \
  python3 -m unittest discover -s tests/python -p 'test_ndnsf_di*.py' -v
python3 -m unittest discover -s tests/python -p 'test_ndnsf_native_tracer_runtime_profile.py' -v
python3 -m unittest discover -s tests/python -p 'test_ndnsf_llm_campaign_runtime_profile.py' -v
python3 Experiments/NDNSF_DI_GUI_Minindn.py --preflight-only --no-gui
```

Results: 152 DI tests passed, one visual test skipped because `pyautogui` is not
installed; both runtime-profile tests passed; GUI preflight printed
`NDNSF_DI_GUI_PREFLIGHT_OK`.

One initial invocation used `python3 -m unittest tests.python...`, which failed
because `tests` is not an import package. The corrected `discover` commands
above passed and are canonical.

Existing network evidence usable as a behavior reference:
`results/streamchunk_mode_campaign_3rep_20260707/` contains three raw and three
StreamChunk full-network NativeTracer runs. It is not Qwen lease-authority
evidence. Child 085 must create a new coordinator-off multi-user authority
campaign; no current result is accepted as a substitute.

Canonical future command shape:

```bash
sudo -n python3 Experiments/NDNSF_DI_NativeTracer_Minindn.py \
  --runtime-profile examples/di-native-tracer.runtime.json \
  --out <child-085-result> --requests 2 --concurrency 1 \
  --provider-check-timeout 60 --no-local-execution-only --full-network
```
