# Spec 107 Lineage Baseline

**Executed:** 2026-07-12T18:39:12Z
**Repository HEAD:** `48877b5854aa9231d7b28f423160e5695388fce4`
**Verdict:** `PASS`

## Verification command

```bash
python3 tools/ndnsf-di/spec107_candidate.py lineage verify \
  --lock specs/107-ndnsf-di-minindn-gate-recovery/lineage-lock.json

git diff --exit-code \
  48877b5854aa9231d7b28f423160e5695388fce4 -- \
  specs/105-ndnsf-di-deployment-readiness/
```

The lineage verifier returned `verifiedFileCount=4`,
`verifiedIdentifierCount=5`, predecessor local verdict `BLOCK`, and predecessor
physical verdict `DEFERRED`. The Git comparison exited zero with no output.

## Locked identifiers

| Classification | Path | SHA-256 |
|---|---|---|
| Frozen commit | repository | `48877b5854aa9231d7b28f423160e5695388fce4` |
| Task closure | `specs/105-ndnsf-di-deployment-readiness/tasks.md` | `4dff3d74337b35fba0677b933ecf9b8ac6d745f64bb0d6ab453bb5d1916a26bf` |
| Release decision | `specs/105-ndnsf-di-deployment-readiness/release-gate.json` | `2752ca1853b5243099dd40dd07ef86d80f24d34dbe6e6c91d567e13ecef296f9` |
| Performance negative evidence | `specs/105-ndnsf-di-deployment-readiness/evidence/telemetry-performance-check.md` | `1090503b7fe58c127aa83187ea0a15f50053fe77dab5aa746780aaf797d39364` |
| Recovery negative evidence | `specs/105-ndnsf-di-deployment-readiness/evidence/fault-recovery.md` | `2777b17ddc231b910667fb4866222359c51f8d9ceb8043a2fa873f8bee66d257` |

No Spec 105 file, verdict, threshold, task, or retained evidence was rewritten.
The verifier is read-only; mutation guards independently reject paths under
`specs/105-*` and `results/spec105-*`.
