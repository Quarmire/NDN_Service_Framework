# Native/Python Parity Evidence

Implemented one native state engine for producer buffering, consumer reorder and
adaptive fetch decisions. Python value objects convert deterministically to the
native engine; no Python fallback algorithm remains.

Commands:

```bash
./waf build --targets=unit-tests -j1
./build/unit-tests --run_test=Stream --log_level=message
python3 pythonWrapper/setup.py build_ext --inplace --force
PYTHONPATH=pythonWrapper python3 -m unittest discover \
  -s tests/python -p test_ndnsf_core_streaming.py -v
```

Results: C++ 9/9 PASS; Python 8/8 PASS. Tests include exact chunk/FEC conversion,
reorder, duplicate/stale rejection, skip, pending count/bytes, overflow,
adaptive pressure and concurrent producer access.

## UAV Migration

`GroundStationServiceContainer` now delegates generic session, sequence,
reorder, pending, skip and drain state to `StreamConsumerReorderBuffer`. UAV
continues to own H264 decoding, FEC repair, backlog policy, missing-chunk
timeout, MAVLink, mission and authority behavior.

```bash
./waf build --targets=UavGroundStationApp -j2
./build/unit-tests --run_test=UavProtocolState --log_level=message
```

Results: application target builds; 38/38 UAV protocol-state tests PASS.

## Finite-Object Boundary

Planned DI tensor bundles now publish their original bytes through
`publishLargeNamed(...)` and retrieve them through `fetchLarge(...)`. The
former optional StreamChunk envelope, runtime flag, GUI option and comparison
campaign were removed. The LLM pipeline smoke also uses named finite objects.

```bash
python3 Experiments/NDNSF_Transfer_Boundary_Documentation_Regression.py
./build/unit-tests --run_test='DistributedInferenceAsyncRuntime,Stream,UavProtocolState'
PYTHONPATH=pythonWrapper:NDNSF-DistributedInference:. \
  python3 Experiments/NDNSF_DI_LlmPipeline_Smoke.py \
  --stages 3 --layers 12 \
  --out-dir /tmp/ndnsf-di-llm-pipeline-finite-object-smoke
```

Results: boundary regression PASS; 47/47 focused C++ tests PASS; LLM smoke
publishes and fetches two exact-name dependency objects and completes all three
stages.
