# Frozen Fixtures

Existing fixtures are frozen as black-box acceptance inputs:

- `DistributedRepoExactPacketTest.cpp`
- `DistributedRepoTieredCacheTest.cpp`
- `DistributedRepoHaTest.cpp`
- `tests/python/test_ndnsf_repo_exact_packets.py`
- `tests/python/test_ndnsf_repo_tiered_cache.py`
- `tests/python/test_ndnsf_repo_ha.py`
- `tests/python/test_ndnsf_repo_repair_sidecar.py`
- `tests/python/test_ndnsf_repo_core_discovery_selection.py`
- `tests/python/test_ndnsf_repo_campaign_evidence.py`

No migration may weaken their name, byte, restart, quorum, tombstone, repair,
authorization or metrics assertions.
