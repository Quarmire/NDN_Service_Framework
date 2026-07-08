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

MiniNDN operator-authority issuer smoke:

```bash
xvfb-run -a sudo -E python3 Experiments/NDNSF_UAV_GUI_Minindn.py \
  --auto-authority-issuer-test \
  --no-start-jmavsim --no-cli --no-xhost
```

Expected marker: `NDNSF_UAV_AUTHORITY_ISSUER_MININDN_SMOKE_OK`. The GS first
sets its active lease to monitor-only, then requests a control lease from the
GS authority issuer service through the normal NDNSF service path. The returned
lease is applied locally and mission/control validation becomes allowed again.

MiniNDN operator-authority arbitration smoke:

```bash
xvfb-run -a sudo -E python3 Experiments/NDNSF_UAV_GUI_Minindn.py \
  --auto-authority-arbitration-test \
  --no-start-jmavsim --no-cli --no-xhost
```

Expected marker: `NDNSF_UAV_AUTHORITY_ARBITRATION_MININDN_SMOKE_OK`. The GS
requests a control lease for one operator, rejects a second operator's
conflicting control lease for the same drone, accepts renewal by the original
operator, accepts an admin override, and then rejects the original operator
because the admin lease is now active.

MiniNDN operator-authority persistence smoke:

```bash
xvfb-run -a sudo -E python3 Experiments/NDNSF_UAV_GUI_Minindn.py \
  --auto-authority-persistence-test \
  --no-start-jmavsim --no-cli --no-xhost
```

Expected marker: `NDNSF_UAV_AUTHORITY_PERSISTENCE_MININDN_SMOKE_OK`. The GS
uses a configured active-lease state file and admin allowlist, rejects an
unauthorized admin override, accepts an authorized admin override, returns
revoked lease evidence in the response fields, and verifies that the active
issuer table is persisted as a single admin lease.

MiniNDN operator-authority revocation lookup smoke:

```bash
xvfb-run -a sudo -E python3 Experiments/NDNSF_UAV_GUI_Minindn.py \
  --auto-authority-revocation-test \
  --no-start-jmavsim --no-cli --no-xhost
```

Expected marker: `NDNSF_UAV_AUTHORITY_REVOCATION_MININDN_SMOKE_OK`. The GS
creates a control lease, overrides it with an authorized admin lease, then
queries `/UAV/GS/OperatorAuthority/Revocation` through the NDNSF service path
to fetch the revoked lease record. A missing lease lookup returns a typed
`found=false` response instead of timing out.

MiniNDN operator-authority refresh smoke:

```bash
xvfb-run -a sudo -E python3 Experiments/NDNSF_UAV_GUI_Minindn.py \
  --auto-authority-refresh-test \
  --no-start-jmavsim --no-cli --no-xhost
```

Expected marker: `NDNSF_UAV_AUTHORITY_REFRESH_MININDN_SMOKE_OK`. The GS first
acts as operator one and obtains a control lease. Operator two then obtains an
authorized admin lease that revokes the first lease. The original operator
refreshes its active lease against `/UAV/GS/OperatorAuthority/Revocation`,
marks the local lease as revoked, and existing command/mission gates return
`lease-revoked`.

MiniNDN periodic operator-authority refresh smoke:

```bash
xvfb-run -a sudo -E python3 Experiments/NDNSF_UAV_GUI_Minindn.py \
  --auto-authority-refresh-timer-test \
  --no-start-jmavsim --no-cli --no-xhost
```

Expected marker: `NDNSF_UAV_AUTHORITY_REFRESH_TIMER_MININDN_SMOKE_OK`. This
uses `--operator-authority-refresh-interval-ms 500` so the ground-station
runtime detects the revoked lease without a manual refresh call. In the GUI,
the same logic is available through the `Refresh Lease` button; an interval of
`0` leaves periodic refresh disabled and keeps the button/manual path only.

MiniNDN operator-authority alert-history smoke:

```bash
xvfb-run -a sudo -E python3 Experiments/NDNSF_UAV_GUI_Minindn.py \
  --auto-authority-alert-history-test \
  --no-start-jmavsim --no-cli --no-xhost
```

Expected marker: `NDNSF_UAV_AUTHORITY_ALERT_HISTORY_MININDN_SMOKE_OK`. The
ground station records an `admin-override` alert when the issuer revokes an
older exclusive lease, then records `lease-revoked-detected` when the original
operator refreshes and detects that its active lease is no longer valid. The
alerts are persisted in the configured authority state file, reloaded, and
then checked again so the same evidence can support post-mission review. The
Operator Authority inspector displays the most recent alert entries.

MiniNDN operator-authority audit-query smoke:

```bash
xvfb-run -a sudo -E python3 Experiments/NDNSF_UAV_GUI_Minindn.py \
  --auto-authority-audit-query-test \
  --no-start-jmavsim --no-cli --no-xhost
```

Expected marker: `NDNSF_UAV_AUTHORITY_AUDIT_QUERY_MININDN_SMOKE_OK`. The
ground station creates the same admin-override and revoked-lease-detected
alerts, then queries `/UAV/GS/OperatorAuthority/Audit` through the NDNSF
service path and verifies the returned audit fields contain both events. The
same smoke also queries a one-entry window with `offset`, `limit`, and
`from_ms` to verify bounded post-mission review can page through the audit
trail without fetching every entry. It also queries with `redaction=summary`
and verifies that operator identities are hidden while the event type, lease,
drone, scope, and reason remain available for safe review. Finally, it queries
with `redaction=self` and a spoofed `requester_operator` field to verify that
the service prefers the NDNSF requester identity mapping when one is available.

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
