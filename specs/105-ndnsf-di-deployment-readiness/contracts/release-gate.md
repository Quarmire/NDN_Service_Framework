# Release Gate Contract

The final machine-readable report is:

```json
{
  "schema": "ndnsf-di-release-gate-v1",
  "releaseId": "...",
  "sourceCommit": "...",
  "profileDigest": "sha256:...",
  "dimensions": {
    "evidenceIntegrity": {"status": "PASS|BLOCK", "artifacts": []},
    "correctness": {"status": "PASS|BLOCK", "artifacts": []},
    "performance": {"status": "PASS|BLOCK", "artifacts": []},
    "applicationSecurity": {"status": "PASS|BLOCK", "artifacts": []},
    "recovery": {"status": "PASS|BLOCK", "artifacts": []},
    "operations": {"status": "PASS|BLOCK", "artifacts": []}
  },
  "minindnCandidateOverall": "PASS|BLOCK",
  "physicalProductionOverall": "DEFERRED|PASS|BLOCK",
  "physicalAcceptanceSpec": "specs/106-ndnsf-di-physical-pilot",
  "limitations": [],
  "generatedAtMs": 0
}
```

Candidate precedence is mechanical: any missing candidate dimension, missing artifact, invalid
execution evidence, failed correctness/security gate, or unretained failed run
yields `minindnCandidateOverall=BLOCK`. Performance success cannot override
another dimension. Spec 105 emits `physicalProductionOverall=DEFERRED`; only a
completed Spec 106 may change that status.
