# Final Distributed Repo Acceptance

Canonical performance evidence is the three matched 60-second exact-packet,
RF=2, W=ALL MiniNDN campaigns under:

```text
results/spec088-rf2-wall-20260711/pass-1
results/spec088-rf2-wall-20260711/pass-2
results/spec088-rf2-wall-20260711/pass-3
```

Each run completed 30/30 requests, required two write receipts, and recorded
zero rejection. Child 088 also passed the C++ object/local contracts, 89 Repo
Python tests, private-operation authorization negatives, persistence, repair,
catalog merge, and stop/restart validation.

Final integration smoke:

```bash
sudo -n timeout 300s python3 Experiments/NDNSF_DistributedRepo_Generic_Minindn.py \
  --output-dir results/spec084-final/repo-integration \
  --ha-campaign --campaign-duration-s 10 --campaign-rps 0.5 \
  --campaign-concurrency 4 --campaign-read-ratio 0.8 \
  --campaign-object-bytes 4096 --campaign-object-mode exact \
  --campaign-replication-factor 2 --campaign-write-consistency ALL \
  --campaign-control-mode targeted --campaign-request-timeout-ms 30000 \
  --campaign-fail-repo repoA --campaign-fail-at-s 3 \
  --campaign-restart-after-s 3 --campaign-auto-repair \
  --campaign-repair-workers 2 --campaign-repair-max-jobs 4 \
  --campaign-seed 88401
```

The smoke completed 5/5 with repoA stop/restart and repair/catalog sidecars.
Its p50 5,507 ms and p95 22,695 ms include deliberate failure downtime and are
not compared with healthy-path performance. This short seeded run happened to
select no writes; write-quorum acceptance comes from the three matched child
campaigns above.

