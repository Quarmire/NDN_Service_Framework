# Quickstart

```bash
PYTHONPATH=pythonWrapper:NDNSF-DistributedInference \
  python3 -m unittest discover -s tests/python \
    -p 'test_ndnsf_di_policy_isolation.py' -v
```

Default runtime code imports only `ndnsf_distributed_inference`. Semantic-cache
research explicitly imports `experimental.semantic_cache`; advisory
coordination was deleted after failing the frozen experiment gate.
