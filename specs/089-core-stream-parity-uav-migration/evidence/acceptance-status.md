# Acceptance Status

## Verified Locally

```bash
./waf build --targets=unit-tests -j2
./build/unit-tests --run_test='DistributedInferenceAsyncRuntime,Stream,UavProtocolState' --log_level=message
./build/unit-tests --log_level=message
PYTHONDONTWRITEBYTECODE=1 PYTHONPATH=pythonWrapper \
  python3 tests/python/test_ndnsf_core_streaming.py
PYTHONDONTWRITEBYTECODE=1 \
  PYTHONPATH=pythonWrapper:NDNSF-DistributedInference:Experiments:. \
  python3 tests/python/test_ndnsf_di_tk_gui.py
python3 Experiments/NDNSF_Transfer_Boundary_Documentation_Regression.py
```

Results on 2026-07-11:

- build PASS;
- focused C++ stream/UAV/DI: 47/47 PASS;
- full C++: 214/214 PASS;
- Python Core stream: 8/8 PASS;
- headless GUI contract: 20/20 PASS;
- transfer-boundary regression: PASS.

Tk widget tests cannot initialize `Tk()` in this managed sandbox because Xvfb
cannot accept a local display socket. All nine failures occur before application
construction.

## Open Network Gate

The required three matched UAV MiniNDN loss campaigns remain unexecuted in this
session. MiniNDN and the security aggregate require root-capable network setup,
but the managed sandbox strips the sudo setuid bit. `nfd-start` also cannot
create `/run/nfd/nfd.sock`. Therefore T012-T014 and parent T051 remain open;
the feature is implemented but not network-accepted.

Canonical command shape retained for a root-capable session:

```bash
sudo -E timeout 160s xvfb-run -a \
  python3 Experiments/NDNSF_UAV_GUI_Minindn.py \
  --no-cli --no-xhost --drone-headless --camera-mode file \
  --auto-video-test --auto-stop-seconds 6 --auto-start-delay-ms 1000 \
  --video-bitrate-kbps 1200 --video-width 320 --output-dir <out>
```

Run three matched loss seeds and retain structured completion, stale-session,
FEC recovery, pending bytes/count, gap/drop, p50 and p95 evidence before closing
T013 or parent T051.
