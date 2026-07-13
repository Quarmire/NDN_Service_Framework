# Spec 110 Offline Foundation Gate

Date: 2026-07-13

## Verdict

`PASS` for T001–T030 offline foundation. This is a necessary gate only.
`liveSubmissionEligible` remains `false` until T031–T049 produce and validate
the runtime release and compute-node probe. No Slurm job was submitted.

## Verification

Final offline suite command:

```bash
python3 tools/ndnsf-di/run_spec110_offline_tests.py \
  --output results/spec110-itiger-qwen-live/offline-foundation/junit.xml
```

Result: 49 tests, 0 failures, 0 errors, 0 skipped. JUnit SHA-256:
`9033460edecead0de742c5aa8f3d85166caaa1985c3126d08ad22ec67a49549c`.

Foundation validation command:

```bash
python3 tools/ndnsf-di/validate_spec110_foundation.py \
  --output results/spec110-itiger-qwen-live/offline-foundation/foundation-validation.json
```

The validator passed 10 cross-contract checks: source baseline, seven-model
ladder, workload, identity contract, cluster contract, campaign freeze,
execution state, storage, positive evidence, and offline JUnit. Report SHA-256:
`25e14c3762aba8b18c64c2e30dc6919cd2041d3ac17050406398f29b163732b4`.

## Preserved failures

1. The initial `python3 -m pytest ...` command did not collect tests because the
   system Python has no `pytest` module. No dependency was installed; the
   repository now has a standard-library JUnit runner.
2. Its first run executed all 49 tests successfully, then failed to write JUnit
   because Python 3.8 lacks `ElementTree.indent`. Removing that cosmetic 3.9+
   call did not change tests or acceptance logic.
3. The first foundation validator run failed on a local variable-name typo.
   The fail-closed result was retained during execution; the corrected command
   generated the PASS report above.

These are offline tooling failures, not once-only measured candidate outcomes.

## Authority and next gate

- Campaign: `spec110-campaign-v1-3c37f488716dfb54d532`.
- Campaign digest:
  `sha256:3c37f488716dfb54d532f15b3a03e87b5da78d81b691332420b4af01bc5abea1`.
- Live submission count: 0.
- Physical production: `DEFERRED`, owner Spec 106.
- Next required gate: T031–T049 runtime release and compute-node probe.
- The current workstation VPN service was unavailable during this phase, so a
  fresh cluster snapshot could not yet be taken. This does not weaken the
  offline PASS and does block any live action that needs mutable iTiger facts.
