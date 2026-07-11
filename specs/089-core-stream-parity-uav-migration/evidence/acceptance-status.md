# Acceptance Status

**Verdict**: PASS

## Local And Security Regressions

```bash
./waf build --targets=unit-tests,UavDroneApp,UavGroundStationApp -j2
./build/unit-tests --log_level=message
PYTHONDONTWRITEBYTECODE=1 PYTHONPATH=pythonWrapper:NDNSF-DistributedInference:Experiments:. \
  python3 -m unittest discover -s tests/python -p 'test_*.py'
sudo -n timeout 600s examples/run_security_regressions.sh
python3 Experiments/NDNSF_Transfer_Boundary_Documentation_Regression.py
```

- Build PASS.
- Full C++: 214/214 PASS.
- Final full Python: 332 PASS, one expected display skip.
- All six security regressions PASS.
- Static/finite transfer-boundary regression PASS.

## MiniNDN Network Gate

Three matched 5% link-loss UAV video runs completed successfully. All runs
decoded the required 30 frames and exited cleanly. Across the campaign:

- completion: 3/3;
- FEC recoveries: 7;
- maximum pending bytes: 21,600;
- maximum pending chunks at sampled adaptive snapshots: 0;
- maximum decoded frame gap: 0;
- mean run-level RTT p50: 53.5 ms;
- mean run-level RTT p95: 120.0 ms;
- stale-session/stream rejects: 0 unexpected stale packets.

Raw local output is under `results/spec089-uav-stream-parity-loss5/`; the
reproduction command and durable summary are in `uav-minindn-loss-campaign.md`.

## Rollback

`git revert --no-commit 01466f5` applied cleanly in a detached worktree. The
restored Python stream implementation and DI harnesses parsed successfully and
the restored Core streaming Python regression passed 6/6.
