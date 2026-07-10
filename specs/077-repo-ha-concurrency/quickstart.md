# Quickstart

## Focused Validation

```bash
PYTHONPATH=pythonWrapper:NDNSF-DistributedInference \
  python3 tests/python/test_ndnsf_repo_ha.py -v

PYTHONPATH=pythonWrapper:NDNSF-DistributedInference \
  python3 tests/python/test_ndnsf_repo_exact_packets.py -v

PYTHONPATH=pythonWrapper:NDNSF-DistributedInference \
  python3 tests/python/test_ndnsf_repo_tiered_cache.py -v
```

## C++ Build and Tests

```bash
./waf build
./build/NDNSF-DistributedRepo/examples/DistributedRepoHaTest
./build/NDNSF-DistributedRepo/examples/DistributedRepoExactPacketTest
./build/NDNSF-DistributedRepo/examples/DistributedRepoTieredCacheTest
```

## MiniNDN Campaign

```bash
sudo -E env PYTHONPATH="$PWD/pythonWrapper:$PWD/NDNSF-DistributedInference" \
  python3 Experiments/NDNSF_DistributedRepo_Generic_Minindn.py \
  --ha-campaign \
  --output-dir results/repo_ha_spec077_read_c1_20260710 \
  --campaign-duration-s 60 --campaign-rps 0.5 \
  --campaign-concurrency 1 --campaign-read-ratio 0.9 \
  --campaign-object-bytes 2048 --campaign-replication-factor 2 \
  --campaign-write-consistency ALL --campaign-seed 77101
```

For a fault-triggered repair run:

```bash
sudo -E env PYTHONPATH="$PWD/pythonWrapper:$PWD/NDNSF-DistributedInference" \
  python3 Experiments/NDNSF_DistributedRepo_Generic_Minindn.py \
  --ha-campaign \
  --output-dir results/repo_ha_spec077_repair_loss_20260710 \
  --campaign-duration-s 60 --campaign-rps 0.2 \
  --campaign-concurrency 2 --campaign-read-ratio 1 \
  --campaign-object-bytes 2048 --campaign-replication-factor 2 \
  --campaign-write-consistency ALL --campaign-seed 77903 \
  --campaign-fail-repo repoA --campaign-fail-at-s 15 \
  --campaign-auto-repair
```

The ready marker is written only after seed replication succeeds, so
`--campaign-fail-at-s` is measured from the start of the workload rather than
from process launch. HA campaigns clear all three experiment stores before
starting. Payloads larger than the bounded control-message shape must use the
exact/segmented Data path rather than inline STORE payloads.

Canonical summaries are under
`results/repo_ha_spec077_canonical_20260710/`. The result directory contains
configuration, request-level CSV, aggregate JSON/CSV, node logs, failure
timestamps, receipts, and failover/repair evidence.
