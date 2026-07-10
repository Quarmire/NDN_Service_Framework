# Repo Baseline

Executed local tests:

```bash
PYTHONPATH=pythonWrapper:NDNSF-DistributedInference:NDNSF-DistributedRepo/pythonWrapper \
  python3 -m unittest discover -s tests/python -p 'test_ndnsf_repo*.py' -v
build/NDNSF-DistributedRepo/DistributedRepoExactPacketTest
build/NDNSF-DistributedRepo/DistributedRepoTieredCacheTest
build/NDNSF-DistributedRepo/DistributedRepoHaTest
```

Results: 80 Python tests passed; all three C++ contracts passed with markers
`EXACT_DATA_PACKET_STORAGE_OK`, `DISTRIBUTED_REPO_TIERED_CACHE_TEST_OK`, and
`DISTRIBUTED_REPO_HA_CONTRACT_TEST_OK`.

Canonical existing MiniNDN evidence:

- `results/repo_ha_spec077_canonical_20260710/campaign-summary.json`: nine-run
  sweep, with each row preserving its own completion/latency outcome; it includes
  both stable and deliberately overloaded points and must not be summarized as
  all-pass performance.
- `results/distributed_repo_exact_packet_failover_minindn/`: exact names and
  wire identity passed; failover latency was 42,735 ms and total 50,772 ms.
- `results/distributed_repo_tiered_cache_minindn/`: SQLite authority, restart,
  bounded LRU, cold/hot/fallback and budget checks passed.

The exact commands are documented in `NDNSF-DistributedRepo/README.md` and are
indexed in `evidence/regression-command-index.md`.
