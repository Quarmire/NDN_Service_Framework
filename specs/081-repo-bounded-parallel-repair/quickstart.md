# Quickstart

## Focused verification

```bash
./waf build
python3 -m unittest discover -s tests/python -p 'test_ndnsf_repo*.py'
build/NDNSF-DistributedRepo/DistributedRepoSmoke
build/NDNSF-DistributedRepo/DistributedRepoExactPacketTest
build/NDNSF-DistributedRepo/DistributedRepoHaTest
build/NDNSF-DistributedRepo/DistributedRepoTieredCacheTest
build/unit-tests --run_test=GenericDynamicApi/TargetedInvocation
build/unit-tests --run_test=GenericDynamicApi/CryptoAndAuthorization
build/unit-tests --run_test=GenericDynamicApi/AllSelectedAndWorkers
```

## Matched MiniNDN campaigns

Use the Spec 080 command with these additional options:

```text
Baseline:  --campaign-repair-workers 1 --campaign-repair-max-jobs 6
Treatment: --campaign-repair-workers 3 --campaign-repair-max-jobs 6
```

Canonical outputs:

```text
results/repo_targeted_spec081_rf3_quorum_repair_finalized_workers1_20260710/
results/repo_targeted_spec081_rf3_quorum_parallel_repair_finalized_20260710/
```

Each `summary.json` records worker bounds, receipt floors, outage-object repair
coverage, repair latency/throughput, and `invalidRepairEventCount`.
