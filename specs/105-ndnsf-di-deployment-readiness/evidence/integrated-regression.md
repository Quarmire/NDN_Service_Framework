# T095 Integrated regression

Status: **PASS** for the maintained regression surface. This does not override
the T062 performance or T078 live-fault BLOCKs.

## Full C++ suite

Final command:

```bash
/usr/bin/time -v ./build/unit-tests --catch_system_errors=no \
  --report_level=detailed --log_level=nothing
```

Result: 242/242 test cases and 45,688/45,688 assertions PASS; exit 0; elapsed
73.69 s; peak RSS 43,268 KiB; zero swap.

The first execution retained a real failure: 241/242 passed because
`DependencyWaitSchedulerRejectsOverflowExpiresAndCancels` observed completion
before its terminal counter. A 200-iteration minimization reproduced it at
iteration 181 (`deadlineExpired` 0 instead of 1). Commit `17380fa` publishes
terminal counters before completion callbacks while keeping the job active
until callback return. The regression then passed 500/500 iterations, its three
focused scheduler cases (1,043 assertions), and the full suite above.

## Maintained Python suite

```bash
/usr/bin/time -v python3 -m unittest discover -s tests/python -p 'test_*.py'
```

Result: 405 tests PASS, 1 environment-gated skip; exit 0; elapsed 11.63 s;
peak RSS 118,316 KiB; zero swap.

## Security regressions

```bash
/usr/bin/time -v ./examples/run_security_regressions.sh
```

Result: all six maintained scripts PASS and final marker
`NDNSF_SECURITY_REGRESSIONS=PASS`; exit 0; elapsed 70.18 s; peak RSS 28,608
KiB; zero swap. The suite used its bounded host-NFD helper and cleanup trap.
This is regression coverage, not the final network acceptance source.

## MiniNDN, Repo, DI and UAV checks

```bash
/usr/bin/time -v python3 Experiments/NDNSF_Run_Minindn_Quick_Checks.py
```

Result: `NDNSF_QUICK_CHECK_SUITE_OK case=all`; exit 0; elapsed 91.56 s;
peak RSS 456,620 KiB; zero swap. Covered:

- script syntax and no-network quick-smoke branches;
- Python HELLO through MiniNDN (`PYTHON_HELLO_MININDN_OK`);
- authoritative DistributedRepo object insert/fetch through MiniNDN
  (`GENERIC_DISTRIBUTED_REPO_QUICK_MININDN_OK`);
- runtime compatibility, staged LLM, tiny Transformers, readiness, llama-server
  and YOLO local DI checks;
- unchanged UAV launcher/config check
  (`NDNSF_UAV_GUI_MININDN_QUICK_SMOKE_OK`).

MiniNDN reported its dummy keychain patch, so this is correctly classified as
application-auth-path execution, not physical cryptographic-strength evidence.
