# Quickstart

```bash
./build/NDNSF-DistributedRepo/DistributedRepoSmoke
./build/NDNSF-DistributedRepo/DistributedRepoExactPacketTest
./build/NDNSF-DistributedRepo/DistributedRepoTieredCacheTest
./build/NDNSF-DistributedRepo/DistributedRepoHaTest
PYTHONPATH=pythonWrapper:NDNSF-DistributedRepo/pythonWrapper:NDNSF-DistributedInference \
  python3 -m unittest discover -s tests/python -p 'test_ndnsf_repo_*.py'
```

Network Repo nodes are started through
`examples/python/NDNSF-DistributedRepo/generic_object_store/repo_node.py`.
There is no second C++ standalone network daemon.
