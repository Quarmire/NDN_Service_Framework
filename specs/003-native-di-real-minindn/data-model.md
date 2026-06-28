# Data Model: Real MiniNDN Native DI Tracer

## NativeMiniNdnAssignment

- `assignment`: `default` or `alternate`
- `role`: native DI role name
- `provider`: NDNSF provider identity
- `node`: MiniNDN node name
- `service`: service name

Validation:

- Every generated plan role must appear exactly once.
- Every assignment row must use a known MiniNDN node.
- Default and alternate assignments must use distinct provider identities.

## MiniNdnEvidenceSummary

- `status`: `SUCCESS` or `FAILURE`
- `resultDir`
- `policyBundle`
- `nativePlan`
- `serviceManifest`
- `assignmentCsv`
- `logs`
- `miniNDNStatus`
- `miniNDNRun`
- `securityBootstrap`
- `providerChecks`
- `userExecution`
- `dependencyExecution`
- `failureReason`

Validation:

- `SUCCESS` requires a generated policy bundle, valid assignment, and provider
  checks that reached every role.
- Hard environmental blockers must be recorded as `FAILURE` with a clear
  `failureReason`.

## ExecutionGate

- `name`
- `status`
- `reason`
- `nextStep`

Validation:

- Full user execution must be `executed` only when an actual user request path
  completes.
- Placeholder artifacts must keep full inference gated.
