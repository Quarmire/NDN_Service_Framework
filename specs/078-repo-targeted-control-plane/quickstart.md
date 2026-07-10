# Quickstart

## Build and focused regressions

```bash
./waf build
python3 -m pip install -e ./pythonWrapper
PYTHONPATH=pythonWrapper:NDNSF-DistributedInference \
  python3 -m unittest discover -s tests/python -p 'test_ndnsf_repo*.py' -v
PYTHONPATH=pythonWrapper:NDNSF-DistributedInference \
  python3 tests/python/test_ndnsf_targeted_python_api.py
build/unit-tests --run_test=GenericDynamicApi/TargetedInvocation
```

The OpenABE thread-affinity regression lives in the local NAC-ABE dependency:

```bash
cd /home/tianxing/NDN/NAC-ABE
cmake -S . -B build-tests -DHAVE_TESTS=True -DCMAKE_BUILD_TYPE=Debug \
  -DBoost_NO_BOOST_CMAKE=ON -DBOOST_ROOT=/usr
cmake --build build-tests -j4
build-tests/tests/unit-tests \
  --run_test=TestAbeSupport/ConcurrentKpDecryptSerializesOpenAbeGlobalState
cmake --build build -j4
sudo -n cmake --install build
sudo -n ldconfig
```

## Matched 60-second MiniNDN campaigns

Read-heavy baseline and Targeted runs:

```bash
sudo -n python3 Experiments/NDNSF_DistributedRepo_Generic_Minindn.py \
  --ha-campaign --campaign-duration-s 60 --campaign-rps 2 \
  --campaign-concurrency 16 --campaign-read-ratio 0.9 \
  --campaign-object-bytes 2048 --campaign-replication-factor 2 \
  --campaign-write-consistency ALL --campaign-seed 77716 \
  --campaign-control-mode normal \
  --output-dir results/repo_ha_spec077_final_read_c16_20260710

sudo -n python3 Experiments/NDNSF_DistributedRepo_Generic_Minindn.py \
  --ha-campaign --campaign-duration-s 60 --campaign-rps 2 \
  --campaign-concurrency 16 --campaign-read-ratio 0.9 \
  --campaign-object-bytes 2048 --campaign-replication-factor 2 \
  --campaign-write-consistency ALL --campaign-seed 77716 \
  --campaign-control-mode targeted --campaign-disable-targeted-fallback \
  --output-dir results/repo_targeted_spec078_c16_20260710
```

Write-heavy baseline and Targeted runs:

```bash
sudo -n python3 Experiments/NDNSF_DistributedRepo_Generic_Minindn.py \
  --ha-campaign --campaign-duration-s 60 --campaign-rps 0.5 \
  --campaign-concurrency 4 --campaign-read-ratio 0.1 \
  --campaign-object-bytes 2048 --campaign-replication-factor 2 \
  --campaign-write-consistency ALL --campaign-seed 77802 \
  --campaign-control-mode normal \
  --output-dir results/repo_ha_spec077_write_20260710

sudo -n python3 Experiments/NDNSF_DistributedRepo_Generic_Minindn.py \
  --ha-campaign --campaign-duration-s 60 --campaign-rps 0.5 \
  --campaign-concurrency 4 --campaign-read-ratio 0.1 \
  --campaign-object-bytes 2048 --campaign-replication-factor 2 \
  --campaign-write-consistency ALL --campaign-seed 77802 \
  --campaign-control-mode targeted --campaign-disable-targeted-fallback \
  --output-dir results/repo_targeted_spec078_write_20260710
```

Accepted summaries are under the four output directories above. Measured
values and interpretation are recorded in `results.md`.
