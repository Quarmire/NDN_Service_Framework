# Quickstart: DI Runtime-Aware User-Side Planner Validation

This guide describes validation targets for the feature. Commands are placeholders
for the implementation phase and should be wired to concrete scripts as tasks
are completed.

## 1. Validate deterministic planner scoring

Expected scenario:

- Provider A has the requested fragment GPU-loaded but moderate compute.
- Provider B has stronger compute but must fetch the fragment from repo.
- Provider C has the fragment on disk.

Expected outcome:

- The planner selects Provider A when queue and edge costs are otherwise equal.
- Metrics record a GPU-loaded residency hit.

## 2. Validate lease conflict control

Expected scenario:

- Two users request the same single-slot provider role at the same time.
- Provider grants one immediate lease and rejects or delays the other.

Expected outcome:

- Only one selection consumes the immediate lease.
- The second user replans or uses a future-start lease.
- Provider does not execute two immediate roles for the same reserved slot.

## 3. Validate provider-to-provider edge-aware placement

Expected scenario:

- Assignment A has strong compute providers but a poor dependency edge.
- Assignment B uses slightly weaker compute but a much better provider-pair link.

Expected outcome:

- The edge-aware planner chooses the lower estimated end-to-end assignment.
- Metrics include edge cost and provider-pair RTT/bandwidth inputs.

## 4. Validate stale-state replan

Expected scenario:

- Provider grants a lease and then reports fragment eviction before selection.

Expected outcome:

- Selection is rejected with `FRAGMENT_EVICTED`.
- User performs bounded replan.
- Replan record includes failed provider, failed lease, and next assignment.

## 5. Validate MiniNDN campaign evidence

Expected scenario:

- Multi-user workload.
- Asymmetric provider-to-provider links.
- Mixed fragment residency states.

Expected outcome:

- Output includes p50/p95 latency, success rate, selected assignments, lease
  counters, residency counters, edge-cost summary, replan count, and provider
  utilization.
