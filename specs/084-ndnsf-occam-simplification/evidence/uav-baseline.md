# UAV Baseline

Executed:

```bash
./build/unit-tests --run_test=Stream,UavProtocolState --log_level=test_suite
PYTHONPATH=pythonWrapper:NDNSF-DistributedInference:NDNSF-DistributedRepo/pythonWrapper \
  python3 -m unittest discover -s tests/python -p 'test_ndnsf_core_streaming.py' -v
python3 Experiments/NDNSF_UAV_GUI_Minindn.py --quick-smoke
```

Results: 46 C++ Stream/UAV tests passed, six Python Core streaming tests passed,
and launcher validation printed `NDNSF_UAV_GUI_MININDN_QUICK_SMOKE_OK`.

Existing network evidence:
`results/uav_core_stream_smoke_20260707_171937/` contains controller, drone,
ground-station and route logs. The documented run passed with 29 published live
packets, `GS_DECODED_FRAMES count=30`, 88 decoded frames before stop, and both
streaming/stopped markers. It has no structured summary JSON, so child 089 must
produce structured matched evidence before stream migration acceptance.
