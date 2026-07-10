# Quickstart

```bash
./waf build --targets=distributed-repo-exact-packet-test
./build/NDNSF-DistributedRepo/examples/distributed-repo-exact-packet-test

PYTHONPATH=pythonWrapper:NDNSF-DistributedInference \
python3 -m unittest tests.python.test_ndnsf_repo_exact_packets

sudo -E python3 Experiments/NDNSF_DistributedRepo_ExactPackets_Minindn.py
```

Acceptance requires exact Data names and wire hashes to match before and after
Repo restart, with no `/ndn-data/N` or logical `<object>/seg/N` aliases created
by the packet API.
