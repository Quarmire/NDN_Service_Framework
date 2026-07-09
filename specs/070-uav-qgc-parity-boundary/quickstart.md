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
- Ground Station operator dashboard snapshot (implemented as an app-owned
  aggregation of telemetry, parameter cache, preflight checklist, Analyze
  snapshot, and action gates)
- Ground Station Vehicle Summary inspector panel (implemented as a GUI consumer
  of the dashboard snapshot)
- Ground Station Preflight Checks and MAVLink Messages inspector panels
  (implemented as GUI consumers of cached checklist and Analyze rows)
- Ground Station Preflight and Analyze refresh buttons (implemented as explicit
  operator workflow actions that refresh the detail panels)
- Ground Station editable parameter panel (implemented as a compact
  QGroundControl-like operator workflow for applying one MAVLink parameter
  through NDNSF and refreshing the parameter cache)

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

MiniNDN operator dashboard snapshot smoke:

```bash
xvfb-run -a sudo -E python3 Experiments/NDNSF_UAV_GUI_Minindn.py \
  --auto-dashboard-snapshot-test \
  --no-start-jmavsim --no-cli --no-xhost
```

Expected marker: `NDNSF_UAV_DASHBOARD_SNAPSHOT_MININDN_SMOKE_OK`.

MiniNDN operator dashboard panel GUI smoke:

```bash
xvfb-run -a sudo -E python3 Experiments/NDNSF_UAV_GUI_Minindn.py \
  --auto-dashboard-panel-test \
  --no-start-jmavsim --no-cli --no-xhost
```

Expected marker: `NDNSF_UAV_DASHBOARD_PANEL_MININDN_SMOKE_OK`.

MiniNDN operator dashboard detail panel GUI smoke:

```bash
xvfb-run -a sudo -E python3 Experiments/NDNSF_UAV_GUI_Minindn.py \
  --auto-dashboard-detail-panel-test \
  --no-start-jmavsim --no-cli --no-xhost
```

Expected marker: `NDNSF_UAV_DASHBOARD_DETAIL_PANEL_MININDN_SMOKE_OK`.

MiniNDN dashboard refresh buttons GUI smoke:

```bash
xvfb-run -a sudo -E python3 Experiments/NDNSF_UAV_GUI_Minindn.py \
  --auto-dashboard-refresh-buttons-test \
  --no-start-jmavsim --no-cli --no-xhost
```

Expected marker: `NDNSF_UAV_DASHBOARD_REFRESH_BUTTONS_MININDN_SMOKE_OK`.

MiniNDN parameter edit panel GUI smoke:

```bash
xvfb-run -a sudo -E python3 Experiments/NDNSF_UAV_GUI_Minindn.py \
  --auto-parameter-edit-panel-test \
  --no-start-jmavsim --no-cli --no-xhost
```

Expected marker: `NDNSF_UAV_PARAMETER_EDIT_PANEL_MININDN_SMOKE_OK`.
