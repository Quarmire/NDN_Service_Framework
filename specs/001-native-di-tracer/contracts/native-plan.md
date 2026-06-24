# Contract: Native Execution Plan

## Purpose

The native execution plan is the C++ data-path contract. It must be loadable by `NativeExecutionPlanJson` and complete enough to drive provider role execution.

## Required Semantics

- Role specs include role names, input scopes, output scopes, and planned Data names for role-to-role activation edges.
- Provider assignments identify which provider executes which role.
- Non-final roles publish planned outputs for downstream roles.
- Final role returns only the `final-response` scope declared in runner/artifact metadata.

## Acceptance Rules

- `DI_NativePlanSchemaSmoke` accepts the generated plan.
- Existing async-runtime tests can construct provider role workers and sessions from the generated plan shape.
- Missing `final-response` metadata is a failure.
