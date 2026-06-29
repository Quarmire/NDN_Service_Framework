# Feature Specification: DI Dependency-Ready Provider Scheduling

**Status**: Accepted
**Created**: 2026-06-28

## User Story

As an NDNSF-DI experiment runner, I want each provider to accept the next
request as soon as its own executable role work is done, instead of losing its
single worker while waiting for other providers' dependencies.

## Problem

The current provider worker fetches dependency inputs inside the provider's
compute worker. Under high concurrency, a dependent role can occupy the only
worker while it waits for another provider's output. That creates artificial
head-of-line blocking: the provider is not computing, but it also cannot execute
another ready request.

## Requirements

- **FR-001**: A provider's compute worker must not be occupied while a role is
  only waiting for dependency inputs.
- **FR-002**: Role execution must still preserve the same dependency correctness:
  a role runner starts only after all required input bundles are available.
- **FR-003**: Provider timing logs must continue to expose queue wait, input
  fetch wait, runner/publish time, and total handler time.
- **FR-004**: Existing NativeTracer full-network request behavior must remain
  compatible for concurrency 1 and higher.
- **FR-005**: A regression test must prove that, with one provider worker, a
  dependency-waiting role does not block a later ready role.

## Success Criteria

- **SC-001**: The new unit test fails on the old behavior and passes after the
  dependency-ready scheduler change.
- **SC-002**: Focused distributed-inference unit tests pass.
- **SC-003**: Python NativeTracer scripts compile.
- **SC-004**: A full-network MiniNDN smoke run with concurrent requests reaches
  `userExecution=executed` and `dependencyExecution=executed`.

## Scope

In scope:

- `ProviderRoleWorker` scheduling semantics.
- Unit tests for provider worker head-of-line blocking.
- NativeTracer harness/docs updates that describe the new scheduling behavior.

Out of scope:

- Proposal slides.
- Larger model artifacts.
- Replacing NDNSF request/ACK/selection semantics.
