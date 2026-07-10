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

Run the same focused C++/Targeted/security/worker commands listed in the Spec
081 quickstart.

## MiniNDN

Use the Spec 081 workers=3 command with this output directory:

```text
results/repo_targeted_spec082_rf3_quorum_repair_fastpath_workers3_20260710
```

The canonical summary is:

```text
results/repo_targeted_spec082_rf3_quorum_repair_fastpath_workers3_20260710/
  campaign-c4-rps0.5-seed78004/summary.json
```
