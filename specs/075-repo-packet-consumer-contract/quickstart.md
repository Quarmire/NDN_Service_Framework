# Quickstart: Repo Packet Consumer Contract

## Focused verification

```bash
./build/examples/distributed-repo-exact-packet-test
PYTHONPATH=pythonWrapper:NDNSF-DistributedInference \
  python3 -m pytest -q tests/python/test_ndnsf_repo_exact_packets.py
```

Expected: packet order, exact names, wire identity, and all negative boundary
cases pass.

## Compatibility verification

```bash
./build/examples/distributed-repo-smoke
./build/examples/distributed-repo-tiered-cache-test
PYTHONPATH=pythonWrapper:NDNSF-DistributedInference \
  python3 -m pytest -q tests/python/test_ndnsf_repo_tiered_cache.py
```

Expected: opaque objects, DI artifact-style payloads, and persistent cache
behavior are unchanged.

## MiniNDN acceptance

```bash
sudo -n -E python3 Experiments/NDNSF_DistributedRepo_Generic_Minindn.py \
  --exact-packet-smoke \
  --nlsr-wait-s 10 \
  --repo-start-wait-s 15 \
  --tiered-restart-wait-s 10 \
  --output-dir results/distributed_repo_exact_packets_minindn
```

Expected: `exact-packet-summary.json` reports `passed: true`, exact names under
`/data/.../v=.../seg=N`, byte-identical wires, restart persistence, and no Repo
aliases.
