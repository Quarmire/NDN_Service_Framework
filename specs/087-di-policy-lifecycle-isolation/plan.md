# Implementation Plan: DI Policy And Lifecycle Isolation

## Architecture

1. Keep executable planner, provider runtime, admission, exact cache, and long
   context mechanisms in the normal DI package.
2. Isolate advisory coordination for the frozen experiment, then delete its
   implementation and integration when the retention gate fails.
3. Move semantic cache implementation and examples to
   `ndnsf_distributed_inference.experimental.semantic_cache`.
4. Replace text-derived retry decisions with `RetryReason` and an explicit
   `idempotent` argument.
5. Delete the unused Merge-owned `DeploymentManager`; Core/provider leases are
   authoritative and deployment publication stays descriptive.
6. Make `default_planner_registry()` register only handlers that can execute.

## Constitution Check

- Canonical Dynamic Runtime: no wire/API naming changes; V2/Targeted remains.
- Security Is Part Of The Data Path: admission and authorization remain in
  Core; experimental advice cannot bypass either.
- CodeGraph First: callers and default imports were traced before planning.
- Spec-Driven Changes: this child specification owns the multi-file migration.
- Verify With The Right Scope: unit, security, DI, and MiniNDN gates are
  explicit; no host-NFD result is final evidence.

## Validation

- Focused Python unit tests for imports, registry, retry, exact cache, semantic
  cache, and deployment ownership.
- Existing NativeTracer, Qwen, GUI/headless, security and C++ regressions.
- MiniNDN coordinator-off multi-user acceptance.
- Frozen advisory retention campaign; delete advisory if its gate fails.

## Rollback

The implementation and closure are separate commits. Experimental package
moves are independently revertible; no wire schema or stored state changes.
