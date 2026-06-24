# Contract: Tracer Evidence

## Purpose

Tracer evidence makes the NDNSF-DI run reviewable after the fact.

## Required Files

- `policy-bundle/` with generated plan and manifest files
- `logs/` with controller, user, provider, and MiniNDN logs when applicable
- `timing.csv`
- `summary.txt`
- `SUCCESS` or `FAILURE`

## Timing CSV Columns

```text
sessionId,provider,role,prefetchMs,executeMs,publishMs,endToEndMs,status
```

## Acceptance Rules

- Every role execution has a timing row.
- Summary records the command, git commit, plan path, result directory, and final status.
- MiniNDN evidence is preferred for final acceptance; if MiniNDN is unavailable, the summary must record that blocker explicitly.
