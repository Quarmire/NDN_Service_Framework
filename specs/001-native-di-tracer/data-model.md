# Data Model: Native DI Tracer

## Tracer Policy

- **Fields**: application, controller, group, service, providers, roles, artifacts, dependencies, runtime user.
- **Relationships**: Generates one Service Manifest and one Native Execution Plan.
- **Validation**: Every role has at least one provider; every dependency scope has a producer and consumer; final role declares `final-response`.

## Native Execution Plan

- **Fields**: service name, role specs, input edges, output edges, role metadata, final-response scope.
- **Relationships**: Consumed by `NativeExecutionPlanJson`; drives `ProviderRoleWorker`, `NativeProviderSession`, and `NativeProviderHandler`.
- **Validation**: C++ loader accepts generated JSON; no manual JSON edits are required.

## Provider Readiness Record

- **Fields**: provider name, role, status (`installing`, `ready`, `failed`), artifact names, failure reason, timestamp.
- **Relationships**: Used by provider ACK/readiness behavior and experiment evidence.
- **Validation**: Providers are selectable only when status is `ready`.

## Artifact Materialization Record

- **Fields**: artifact name, source URI or local path, cache path, expected sha256, observed sha256, status.
- **Relationships**: Feeds Provider Readiness Record.
- **Validation**: Hash mismatch sets readiness to `failed`.

## Role Timing Record

- **Fields**: session id, provider, role, prefetchMs, executeMs, publishMs, endToEndMs, status.
- **Relationships**: Written to timing CSV and summarized after tests or MiniNDN runs.
- **Validation**: Every executed role produces one timing row.

## MiniNDN Result Directory

- **Fields**: generated policy bundle, process logs, timing CSV, summary, success marker.
- **Relationships**: Durable acceptance evidence for User Story 3.
- **Validation**: Summary lists command, result path, success/failure, and timing aggregates.
