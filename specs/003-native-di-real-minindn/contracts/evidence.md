# Contract: Real MiniNDN Native DI Evidence

## Required Files

- `policy-bundle/`
- `logs/`
- `assignment.csv`
- `summary.json`
- `summary.txt`
- `SUCCESS` or `FAILURE`

## Required Assignment Columns

```text
assignment,role,provider,node,service
```

## Required Summary JSON Fields

```text
status
gitCommit
command
resultDir
policyBundle
nativePlan
serviceManifest
assignmentCsv
logs
miniNDNStatus
miniNDNRun
securityBootstrap
providerChecks
userExecution
dependencyExecution
failureReason
```

## Status Rules

- `SUCCESS` means MiniNDN topology setup and role-specific provider checks were
  accepted.
- `SUCCESS` does not imply full ONNX inference unless `userExecution.status` is
  `executed`.
- `FAILURE` must include `failureReason`.
