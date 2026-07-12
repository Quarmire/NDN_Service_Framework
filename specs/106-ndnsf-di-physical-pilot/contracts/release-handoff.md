# Spec 105 to Spec 106 Release Handoff

Required immutable input:

```json
{
  "schema": "ndnsf-di-spec105-candidate-v1",
  "candidateReleaseId": "...",
  "sourceCommit": "...",
  "minindnCandidateOverall": "PASS",
  "profileDigest": "sha256:...",
  "planDigest": "sha256:...",
  "modelDigest": "sha256:...",
  "artifactDigests": {},
  "evidenceManifestDigest": "sha256:..."
}
```

Spec 106 rejects any non-PASS candidate or digest drift. Its final report adds
physical cluster/profile digests and `physicalProductionOverall=PASS|BLOCK` but
does not rewrite the Spec 105 record.
