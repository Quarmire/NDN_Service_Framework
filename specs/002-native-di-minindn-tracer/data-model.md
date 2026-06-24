# Data Model: Native DI MiniNDN Tracer

## MiniNDN Tracer Run

- **Fields**: command, git commit, result directory, MiniNDN status, marker, started/finished timestamps.
- **Relationships**: Owns policy bundle, logs, timing rows, summary JSON, summary text.
- **Validation**: Has exactly one marker: `SUCCESS` or `FAILURE`.

## Tracer Assignment

- **Fields**: assignment name, role, provider label, provider identity.
- **Relationships**: Used to attribute role evidence and to build provider launch commands.
- **Validation**: Every role in the native plan has a provider; no provider assignment references an unknown role.

## Role Evidence Row

- **Fields**: sessionId, provider, role, inputBytes, outputBytes, prefetchMs, executeMs, publishMs, endToEndMs, status.
- **Relationships**: One row per role execution.
- **Validation**: Required columns are present and numeric timing fields parse.

## Run Summary

- **Fields**: status, gitCommit, command, resultDir, policyBundle, nativePlan, serviceManifest, timingCsv, logs, miniNDNStatus, miniNDNRun, assignment.
- **Relationships**: Points to all evidence files.
- **Validation**: JSON and text summaries agree on status and paths.

## LLM Planner Gate

- **Fields**: gate name, required evidence, next allowed LLM task.
- **Relationships**: References accepted tracer result.
- **Validation**: LLM planner follow-up says to reuse the native tracer path.
