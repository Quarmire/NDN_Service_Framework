# Acceptance Status

**Verdict**: PASS

## Regressions

```bash
PYTHONPATH=pythonWrapper:NDNSF-DistributedInference:NDNSF-DistributedRepo/pythonWrapper:NDNSF-UAV-APP/pythonWrapper \
  python3 -m unittest discover -s tests/python -p 'test_*.py'
./waf build --targets=unit-tests -j2
./build/unit-tests --log_level=message
sudo -n timeout 600s examples/run_security_regressions.sh
python3 Experiments/NDNSF_Transfer_Boundary_Documentation_Regression.py
```

- Python: 342 passed, one expected display skip.
- C++: 214/214 passed.
- Security: all six regressions passed.
- Static/finite transfer boundary passed.
- Producer scan: no current flat capability aliases.
- Typed-only and mixed-reader Qwen MiniNDN: both 2/2 requests passed.

## Rollback

In detached worktree `/tmp/ndnsf-spec090-rollback`,
`git revert --no-commit 72dc052` applied cleanly. The restored deployment ACK
suite passed 5/5. The temporary worktree was then removed.

The bounded mixed reader deadline is the next major release or 2026-12-31,
whichever is earlier. No persisted-state rollback or rewrite is required.

