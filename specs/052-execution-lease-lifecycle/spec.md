# Spec: Execution Lease + Reference-Counted Deployment Lifecycle

**Branch**: `052-execution-lease-lifecycle` | **Date**: 2026-07-06 | **Status**: Draft

## Summary

Add execution leases and reference counting to deployments. The Merge Provider
(terminal role of a pipeline) is the authority for ref_count and evict/Gc
decisions. Users acquire/release execution leases. A deployment cannot be
evicted while ref_count > 0. Idle deployments can be preempted by higher-priority
new deployments.

## Requirements

- REQ-052-001: `ExecutionLease` type: `lease_id`, `deployment_id`, `user`,
  `acquired_at_ms`, `expires_at_ms`, `released`.
- REQ-052-002: `DeploymentLeaseTable` tracks active leases per deployment,
  computes `ref_count` atomically.
- REQ-052-003: `ServiceUser.acquire_execution_lease(deployment_id) → ExecutionLease`.
- REQ-052-004: `ServiceUser.release_execution_lease(lease_id) → bool`.
- REQ-052-005: Leases auto-expire after TTL; expired leases are not counted in ref_count.
- REQ-052-006: `Deployment` carries `ref_count` field; visible via `discover_deployments`.
- REQ-052-007: Eviction MUST be rejected when ref_count > 0 with reason `DEPLOYMENT_IN_USE`.
- REQ-052-008: Idle deployments (ref_count==0 beyond idle_timeout_s) are eligible for preemption.
- REQ-052-009: Merge Provider GC scans leases periodically, expires stale ones, updates ref_count.

## Non-Goals

- Distributed consensus for ref_count (Merge Provider is single authority per deployment)
- Automatic preemption (manual evict API only; preemption policy is future work)
