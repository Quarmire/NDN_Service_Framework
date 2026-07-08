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

Expected evidence:

- `UavMissionPlanDocumentSupportsPersistentOperationalPlan` passes and shows
  mission plan v2 fields round-trip plus save/load through a temporary
  line-oriented config file.
- `UavDataProductCatalogSummarizesQueryableProducts` passes and shows catalog
  product counts and latest object prefix round-trip.
- `VehicleParameterSnapshotCarriesCapabilityView` passes and shows compact and
  full parameter views remain usable.
- `OperatorAuthorityLeaseBlocksConflictingControl` passes and shows valid,
  wrong-drone, expired, and monitor-only decisions.

This validates the state-contract slice only. Future service and GUI wiring
should reuse these models instead of inventing new status strings.
