# Quickstart: UAV Operational Layer

Build and run the focused C++ validation:

```bash
./waf build --targets=unit-tests
./build/unit-tests --run_test=UavProtocolState
```

Run the Python core/app bridge regression:

```bash
PYTHONPATH=.:pythonWrapper:NDNSF-DistributedInference:NDNSF-DistributedRepo/pythonWrapper \
  python3 tests/python/test_ndnsf_app_core_envelope_migration.py
```

Ground-station GUI wiring:

```bash
./build/examples/UavGroundStationApp \
  --app-config NDNSF-UAV-APP/configs/ground-station.conf \
  --mission-plan-file /tmp/ndnsf-uav-mission-plan.conf
```

The Map / Mission controls expose a mission-plan file path plus `Save Plan`
and `Load Plan` buttons. Saving uses the current map preview when available,
otherwise the currently uploaded runtime mission plan. Loading updates the
runtime mission plan snapshot and refreshes the inspector/map preview. The
`Upload Mission` button now uses the current loaded or preview mission plan
first, preserving per-drone parts from the file; if no editable plan exists, it
falls back to the older patrol-input generation path.

MiniNDN loaded-plan smoke:

```bash
xvfb-run -a sudo -E python3 Experiments/NDNSF_UAV_GUI_Minindn.py \
  --auto-loaded-mission-plan-test \
  --no-start-jmavsim --no-cli --no-xhost
```

Expected marker: `NDNSF_UAV_LOADED_MISSION_PLAN_MININDN_SMOKE_OK`.

MiniNDN repo catalog smoke:

```bash
xvfb-run -a sudo -E python3 Experiments/NDNSF_UAV_GUI_Minindn.py \
  --auto-repo-catalog-browse-test \
  --no-start-jmavsim --no-cli --no-xhost
```

Expected marker: `NDNSF_UAV_REPO_CATALOG_MININDN_SMOKE_OK`. The drone records
camera chunks to its local in-app repo, serves `/UAV/Camera/Repo/Catalog`, and
the ground station summarizes the repo objects as object-level UAV data
products instead of listing every chunk as a separate recording.

MiniNDN MAVLink parameter-cache smoke:

```bash
xvfb-run -a sudo -E python3 Experiments/NDNSF_UAV_GUI_Minindn.py \
  --auto-parameter-cache-test \
  --no-start-jmavsim --no-cli --no-xhost
```

Expected marker: `NDNSF_UAV_PARAMETER_CACHE_MININDN_SMOKE_OK`. The drone serves
`/UAV/MAVLink/Parameters` under its identity, and the ground station fetches
and caches a `VehicleParameterSnapshot` for the selected drone. This is an
operator-visible parameter/capability view, not a full QGroundControl parameter
editor.

MiniNDN operator-authority lease smoke:

```bash
xvfb-run -a sudo -E python3 Experiments/NDNSF_UAV_GUI_Minindn.py \
  --auto-authority-lease-test \
  --no-start-jmavsim --no-cli --no-xhost
```

Expected marker: `NDNSF_UAV_AUTHORITY_LEASE_MININDN_SMOKE_OK`. The GS starts
with a local default `all/control` lease for normal demos, then the smoke test
injects a monitor-only lease and an expired control lease to verify that UAV
control and mission assignment fast-fail before network/MAVLink dispatch.

MiniNDN configured operator-authority smoke:

```bash
xvfb-run -a sudo -E python3 Experiments/NDNSF_UAV_GUI_Minindn.py \
  --auto-authority-config-test \
  --no-start-jmavsim --no-cli --no-xhost
```

Expected marker: `NDNSF_UAV_AUTHORITY_CONFIG_MININDN_SMOKE_OK`. The launcher
starts the GS with a configured monitor-only lease for the selected drone, then
the GS verifies that telemetry remains allowed while control and mission
assignment are blocked by the startup lease.

Expected evidence:

- `UavMissionPlanDocumentSupportsPersistentOperationalPlan` passes and shows
  mission plan v2 fields round-trip plus save/load through a temporary
  line-oriented config file.
- `UavFunctionalityStateTracksImplementedAndMissingCapabilities` reports
  mission files as available once a mission plan exists.
- `UavDataProductCatalogSummarizesQueryableProducts` passes and shows catalog
  product counts, repo object counts, latest object prefix round-trip, and
  chunk-to-recording product folding.
- `VehicleParameterSnapshotCarriesCapabilityView` passes and shows compact and
  full parameter views remain usable.
- `OperatorAuthorityLeaseBlocksConflictingControl` passes and shows valid,
  wrong-drone, expired, and monitor-only decisions.

This validates the state-contract slice only. Future service and GUI wiring
should reuse these models instead of inventing new status strings.
