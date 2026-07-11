# RED Test Evidence

Command:

```bash
PYTHONPATH=pythonWrapper:NDNSF-DistributedInference:Experiments \
python3 tests/python/test_ndnsf_di_runtime_aware_campaign.py \
  RuntimeAwareCampaignTest.test_user_driver_keeps_base_publisher_started_for_workload \
  RuntimeAwareCampaignTest.test_user_driver_stops_base_publisher_when_workload_raises \
  RuntimeAwareCampaignTest.test_process_pool_schedule_observation_records_actual_slip \
  RuntimeAwareCampaignTest.test_workload_throughput_uses_process_pool_measurement_interval
```

Result: expected failure, 4 tests run and 4 errors. Each error was an
`AttributeError` for one of the not-yet-implemented contract helpers:

```text
run_with_started_user
process_pool_schedule_observation
process_pool_measurement_metadata
```

An earlier invocation through `python3 -m unittest tests.python...` failed at
test discovery because `tests/python` is not a Python package. It is not used as
RED evidence; the direct test-file invocation above reached all four tests.

## Threaded Measurement Addendum

The first post-fix MiniNDN threaded run completed 60/60 requests with 14.559 ms
maximum schedule slip, but the outer summary included about 35 seconds of
`user.stop()` cleanup in its throughput denominator. A fifth focused test was
added and failed with `KeyError: measurementElapsedMs`, proving the threaded
driver did not expose its request measurement interval before cleanup.
