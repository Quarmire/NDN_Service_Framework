# Quickstart

## Focused verification

```bash
./waf build
PYTHONPATH=pythonWrapper:NDNSF-DistributedInference \
  python3 -m unittest discover -s tests/python -p 'test_ndnsf_repo*.py' -v
build/unit-tests --run_test=GenericDynamicApi/TargetedInvocation
```

## Matched MiniNDN campaigns

```bash
sudo -n python3 Experiments/NDNSF_DistributedRepo_Generic_Minindn.py \
  --ha-campaign --campaign-duration-s 60 --campaign-rps 0.5 \
  --campaign-concurrency 4 --campaign-read-ratio 0.1 \
  --campaign-object-bytes 2048 --campaign-replication-factor 3 \
  --campaign-write-consistency QUORUM --campaign-seed 77903 \
  --campaign-control-mode targeted --campaign-request-timeout-ms 5000 \
  --output-dir results/repo_targeted_spec079_rf3_quorum_baseline_20260710

sudo -n python3 Experiments/NDNSF_DistributedRepo_Generic_Minindn.py \
  --ha-campaign --campaign-duration-s 60 --campaign-rps 0.5 \
  --campaign-concurrency 4 --campaign-read-ratio 0.1 \
  --campaign-object-bytes 2048 --campaign-replication-factor 3 \
  --campaign-write-consistency QUORUM --campaign-seed 77903 \
  --campaign-control-mode targeted --campaign-request-timeout-ms 5000 \
  --campaign-fail-repo repoA --campaign-fail-at-s 20 \
  --output-dir results/repo_targeted_spec079_rf3_quorum_repoA_loss_20260710
```

Each output contains `summary.json`, `request-lifecycle.csv`, logs, the failure
epoch, and pre/overlap/post phase metrics.
