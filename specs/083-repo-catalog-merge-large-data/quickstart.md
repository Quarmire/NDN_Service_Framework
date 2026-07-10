# Quickstart

## Verification

```bash
python3 -m py_compile \
  NDNSF-DistributedInference/ndnsf_distributed_inference/repo.py \
  examples/python/NDNSF-DistributedRepo/generic_object_store/catalog_sync.py \
  Experiments/repo_campaign_evidence.py
python3 -m unittest discover -s tests/python -p 'test_ndnsf_repo*.py'
./waf build
```

Run the focused C++/Targeted/security/worker commands from the Spec 082
quickstart serially when they share the default PIB.

## MiniNDN

```bash
sudo -n python3 Experiments/NDNSF_DistributedRepo_Generic_Minindn.py \
  --ha-campaign --campaign-duration-s 60 --campaign-rps 0.5 \
  --campaign-concurrency 4 --campaign-read-ratio 0.1 \
  --campaign-object-bytes 2048 --campaign-replication-factor 3 \
  --campaign-write-consistency QUORUM --campaign-seed 78004 \
  --campaign-control-mode targeted --campaign-request-timeout-ms 5000 \
  --campaign-fail-repo repoA --campaign-fail-at-s 20 \
  --campaign-restart-after-s 12 --campaign-auto-repair \
  --campaign-repair-workers 3 --campaign-repair-max-jobs 6 \
  --output-dir results/repo_targeted_spec083_rf3_quorum_catalog_merge_pull_workers3_20260710
```

Canonical summary:

```text
results/repo_targeted_spec083_rf3_quorum_catalog_merge_pull_workers3_20260710/
  campaign-c4-rps0.5-seed78004/summary.json
```
