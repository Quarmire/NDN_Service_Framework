# Quickstart: UAV QGC-Parity Boundary Slice

Build and test the foundational protocol contracts:

```bash
./waf build --targets=unit-tests
./build/unit-tests --run_test=UavProtocolState
PYTHONPATH=.:pythonWrapper:NDNSF-DistributedInference:NDNSF-DistributedRepo/pythonWrapper \
  python3 tests/python/test_ndnsf_app_core_envelope_migration.py
git diff --check
```

The next runtime slice should expose the new contracts through NDNSF-UAV-APP
services:

- `/UAV/MAVLink/ParameterEdit` (implemented for mock backend; UDP/serial
  backends conservatively return unsupported until safe MAVLink read-back is
  wired)
- `/UAV/Preflight/Checklist` (implemented from current telemetry, readiness,
  and camera status)
- `/UAV/MAVLink/AnalyzeSnapshot` (implemented as a compact message-rate and
  vehicle-state summary for QGC-like Analyze/Inspector panels)

MiniNDN parameter-edit smoke:

```bash
xvfb-run -a sudo -E python3 Experiments/NDNSF_UAV_GUI_Minindn.py \
  --auto-parameter-edit-test \
  --no-start-jmavsim --no-cli --no-xhost
```

Expected marker: `NDNSF_UAV_PARAMETER_EDIT_MININDN_SMOKE_OK`.

MiniNDN preflight checklist smoke:

```bash
xvfb-run -a sudo -E python3 Experiments/NDNSF_UAV_GUI_Minindn.py \
  --auto-preflight-checklist-test \
  --no-start-jmavsim --no-cli --no-xhost
```

Expected marker: `NDNSF_UAV_PREFLIGHT_CHECKLIST_MININDN_SMOKE_OK`.

MiniNDN MAVLink analyze snapshot smoke:

```bash
xvfb-run -a sudo -E python3 Experiments/NDNSF_UAV_GUI_Minindn.py \
  --auto-analyze-snapshot-test \
  --no-start-jmavsim --no-cli --no-xhost
```

Expected marker: `NDNSF_UAV_ANALYZE_SNAPSHOT_MININDN_SMOKE_OK`.
