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

Expected evidence:

- `UavMissionPlanDocumentSupportsPersistentOperationalPlan` passes and shows
  mission plan v2 fields round-trip plus save/load through a temporary
  line-oriented config file.
- `UavFunctionalityStateTracksImplementedAndMissingCapabilities` reports
  mission files as available once a mission plan exists.
- `UavDataProductCatalogSummarizesQueryableProducts` passes and shows catalog
  product counts and latest object prefix round-trip.
- `VehicleParameterSnapshotCarriesCapabilityView` passes and shows compact and
  full parameter views remain usable.
- `OperatorAuthorityLeaseBlocksConflictingControl` passes and shows valid,
  wrong-drone, expired, and monitor-only decisions.

This validates the state-contract slice only. Future service and GUI wiring
should reuse these models instead of inventing new status strings.
