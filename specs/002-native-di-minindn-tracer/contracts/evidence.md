# Contract: Native DI Tracer Evidence

## Required Files

- `policy-bundle/`
- `logs/`
- `timing.csv`
- `summary.json`
- `summary.txt`
- `SUCCESS` or `FAILURE`

## Required Timing Columns

```text
sessionId,provider,role,inputBytes,outputBytes,prefetchMs,executeMs,publishMs,endToEndMs,status
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
timingCsv
logs
miniNDNStatus
miniNDNRun
assignment
assignmentCsv
llmPlannerGate
```
