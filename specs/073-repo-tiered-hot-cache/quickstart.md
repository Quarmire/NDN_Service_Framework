# Quickstart: Tiered DistributedRepo Validation

## Build

```bash
./waf build --targets=ndnsf-distributed-repo,DistributedRepoSmoke,DistributedRepoTieredCacheTest,DistributedRepoNodeApp
```

## C++ Tests

```bash
./build/NDNSF-DistributedRepo/DistributedRepoTieredCacheTest
./build/NDNSF-DistributedRepo/DistributedRepoSmoke
```

Expected markers:

```text
DISTRIBUTED_REPO_TIERED_CACHE_TEST_OK
DISTRIBUTED_REPO_SMOKE_OK
```

## Python Tests

```bash
PYTHONPATH=NDNSF-DistributedInference:pythonWrapper \
  python3 tests/python/test_ndnsf_repo_tiered_cache.py -v
PYTHONPATH=NDNSF-DistributedInference:pythonWrapper \
  python3 tests/python/test_ndnsf_repo_core_discovery_selection.py -v
PYTHONPATH=NDNSF-DistributedInference:pythonWrapper \
  python3 tests/python/test_ndnsf_app_core_envelope_migration.py -v
```

## Node Configuration Smoke

```bash
./build/NDNSF-DistributedRepo/DistributedRepoNodeApp \
  --config NDNSF-DistributedRepo/configs/repo-node.conf \
  --deployment-mode embedded \
  --local-smoke \
  --dry-run
```

## MiniNDN Acceptance

```bash
sudo -n python3 Experiments/NDNSF_DistributedRepo_Generic_Minindn.py \
  --tiered-cache-smoke \
  --tiered-cache-bytes 8192 \
  --tiered-cache-object-bytes 4096 \
  --output-dir results/distributed_repo_tiered_cache_minindn
```

Expected marker and evidence:

```text
GENERIC_DISTRIBUTED_REPO_TIERED_CACHE_MININDN_OK
results/distributed_repo_tiered_cache_minindn/tiered-cache-summary.json
```

The JSON summary must show restart success, byte-integrity success, `hits >= 1`, `misses >= 1`, `evictions >= 1`, `backingReads >= 1`, and `usedBytes <= budgetBytes`.

Verified 2026-07-10 on `Experiments/Topology/AI_Lab.conf`: three 4096-byte
objects, an 8192-byte shared cache, Repo A restart, and access order
`A,A,B,C,A` produced `hits=1`, `misses=4`, `evictions=4`,
`backingReads=4`, `usedBytes=6200`, and `passed=true`.
