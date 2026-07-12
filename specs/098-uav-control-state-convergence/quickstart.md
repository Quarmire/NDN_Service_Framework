# Quickstart

## Prerequisites

- Build the Ground Station and unit tests.
- Verify `sudo -n true` and ensure no other MiniNDN campaign is active.
- Keep the Spec 097 baseline unchanged.

## Focused Validation

```bash
./waf build -j2 --targets=UavGroundStationApp,unit-tests
./build/unit-tests --run_test=UavProtocolState --log_level=test_suite
python3 tests/python/test_ndnsf_uav_stream_control_isolation_campaign.py
```

## Frozen MiniNDN Treatment

```bash
python3 Experiments/NDNSF_UAV_Stream_Control_Isolation_Campaign.py \
  --out results/spec098-uav-control-state-loss05-current-final \
  --workload-modes control-only --runs 5 --loss-percent 5 \
  --auto-stop-seconds 60
```

Run the treatment exactly once. Do not replace failed repetitions. Acceptance
requires terminal command and convergence stages, zero known lifecycle aborts,
and a report that preserves any negative network result.
