# Successor Release Gate Contract

Dimensions:

```text
lineage evidenceIntegrity correctness performance recovery
applicationSecurity localOperations
```

Every dimension is `PASS|BLOCK` and lists digest-verified artifacts. Missing,
malformed, unknown, synthetic, mixed, diagnostic-only, invalid-preflight, or
BLOCK evidence yields BLOCK.

```json
{
  "schema": "ndnsf-di-spec107-release-gate-v1",
  "candidateId": "spec107-c1-...",
  "predecessor": {
    "releaseId": "spec105-local-minindn-candidate-r2",
    "minindnCandidateOverall": "BLOCK"
  },
  "dimensions": {},
  "evidenceManifest": [],
  "minindnCandidateOverall": "PASS|BLOCK",
  "physicalProductionOverall": "DEFERRED",
  "physicalAcceptanceSpec": "specs/106-ndnsf-di-physical-pilot"
}
```

Only a PASS Spec 107 gate may update the Spec 106 candidate prerequisite. A
BLOCK gate leaves Spec 106 deferred and cannot alter Spec 105.
