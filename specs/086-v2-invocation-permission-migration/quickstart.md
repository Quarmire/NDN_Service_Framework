# Quickstart

```bash
python3 .agents/skills/speckit-audit/scripts/audit_speckit_structure.py \
  specs/086-v2-invocation-permission-migration --strict

./waf build -j$(nproc)
./build/unit-tests --log_level=message
python3 -m unittest discover -s tests/python -p 'test_ndnsf_core*.py' -q
examples/run_security_regressions.sh
```

Run the exact MiniNDN commands recorded in `evidence/entry-baseline.md` and
`evidence/minindn-acceptance.md`; do not substitute host NFD for final network
acceptance.
