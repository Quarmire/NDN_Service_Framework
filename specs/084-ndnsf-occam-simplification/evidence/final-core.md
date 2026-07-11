# Final Core Acceptance

Commands:

```bash
./waf build --targets=unit-tests -j2
./build/unit-tests --log_level=message
python3 -m unittest discover -s tests/python -p 'test_*.py'
sudo -n timeout 600s examples/run_security_regressions.sh
python3 Experiments/NDNSF_Transfer_Boundary_Documentation_Regression.py
```

Results:

- C++: 214/214 passed.
- Python after dead coordination removal: 336 passed, one expected display skip.
- Security: HELLO authorization, ACK payload, custom selection, NAC-ABE routing,
  token-negative, and certificate-bootstrap suites all passed.
- Exact finite transfer boundary passed.
- Normal and Targeted invocation are covered by child 086's matched 10/10
  MiniNDN runs; collaboration is covered by child 090's Qwen 2/2 run.
- The active V1 production scan reports zero findings.

Rollback proof for the last parent-owned deletion:

```bash
git worktree add --detach /tmp/ndnsf-spec084-coordination-rollback HEAD
git -C /tmp/ndnsf-spec084-coordination-rollback revert --no-commit f714c99
PYTHONPATH=/tmp/ndnsf-spec084-coordination-rollback/pythonWrapper \
  python3 -m unittest discover \
  -s /tmp/ndnsf-spec084-coordination-rollback/tests/python \
  -p 'test_ndnsf_core_coordination.py' -v
```

The revert applied cleanly and the restored coordination suite passed 6/6.

