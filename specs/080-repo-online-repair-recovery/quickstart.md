# Quickstart

## Focused verification

```bash
./waf build
python3 -m unittest discover -s tests/python -p 'test_ndnsf_repo*.py' -v
build/NDNSF-DistributedRepo/DistributedRepoSmoke
build/NDNSF-DistributedRepo/DistributedRepoExactPacketTest
build/NDNSF-DistributedRepo/DistributedRepoHaTest
build/NDNSF-DistributedRepo/DistributedRepoTieredCacheTest
build/unit-tests --run_test=GenericDynamicApi/TargetedInvocation
build/unit-tests --run_test=GenericDynamicApi/CryptoAndAuthorization
build/unit-tests --run_test=GenericDynamicApi/AllSelectedAndWorkers
```

## MiniNDN online-repair campaign

```bash
sudo -n python3 Experiments/NDNSF_DistributedRepo_Generic_Minindn.py \
  --ha-campaign --campaign-duration-s 60 --campaign-rps 0.5 \
  --campaign-concurrency 4 --campaign-read-ratio 0.1 \
  --campaign-object-bytes 2048 --campaign-replication-factor 3 \
  --campaign-write-consistency QUORUM --campaign-seed 78004 \
  --campaign-control-mode targeted --campaign-request-timeout-ms 5000 \
  --campaign-fail-repo repoA --campaign-fail-at-s 20 \
  --campaign-restart-after-s 12 --campaign-auto-repair \
  --output-dir results/repo_targeted_spec080_rf3_quorum_recovery_20260710
```

Canonical output:

```text
results/repo_targeted_spec080_rf3_quorum_recovery_20260710/
  campaign-c4-rps0.5-seed78004/
    summary.json
    request-lifecycle.csv
    minindn-metadata.json
  catalogA-recovered-repair.log
  repoA-restart.log
```

The summary's `faultInjection.recovery` object correlates writes completed
while RepoA was offline with validated repair events on RepoA. The catalog log
is authoritative only after `catalog_repair` receives a successful target
store response.
