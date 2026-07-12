# Quickstart

```bash
python3 tests/python/test_ndnsf_uav_stream_control_isolation_campaign.py
./waf build --targets=UavGroundStationApp,unit-tests -j4
./build/unit-tests --run_test=UavProtocolState --report_level=short
python3 Experiments/NDNSF_UAV_Stream_Control_Isolation_Campaign.py \
 --out results/spec100-uav-armed-visibility-loss05-final \
 --workload-modes control-only --runs 5 --loss-percent 5 --auto-stop-seconds 60
```

Run the MiniNDN command once; do not replace failures.
