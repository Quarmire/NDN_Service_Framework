# Quickstart

```bash
PYTHONPATH=pythonWrapper:NDNSF-DistributedInference \
  python3 -m unittest tests.python.test_ndnsf_di_policy_isolation -v
```

Default runtime code imports only `ndnsf_distributed_inference`. Research
experiments opt in through the explicit `experimental` package paths.
