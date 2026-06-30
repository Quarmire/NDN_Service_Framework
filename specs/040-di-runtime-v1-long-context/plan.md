# Feature 040 Plan: NDNSF-DI Runtime v1 With Long-Context Management

## Goal

Update the NDNSF-DI roadmap so the next runtime design explicitly supports
long-context LLM serving. The design should explain why the current runtime is
not enough, add context/KV-cache management to the architecture, and define a
task list that can be implemented in later phases.

## Current Findings

- The current LLM adapter is non-streaming.
- The current LLM pipeline path can describe ordered stage execution but does
  not manage KV cache or stream tokens.
- Existing MiniNDN evidence proves useful pieces: proportional layer planning,
  provider utilization summaries, process-pool open-loop drivers, and planner
  prediction alignment.
- Long-context support requires new runtime concepts, not just larger payloads
  or longer request timeouts.

## Design Direction

Runtime v1 should add these architecture pieces:

1. Typed provider capability and telemetry, including KV-cache capacity.
2. Reusable PlanKey/PlanLease lifecycle.
3. Capacity-aware LLM planning with linear stage split by default.
4. Role-level pipelined execution.
5. Named dependency and context objects.
6. Prefill/decode split.
7. Provider-local KV state references.
8. Cache lease, pin, eviction, and invalidation policy.
9. Streaming GenerationChunk output.
10. MiniNDN long-context and cache-pressure validation.

## Validation

This feature is a design/planning feature. Validation is documentation-level:

- Verify the roadmap names the current limitation accurately.
- Verify tasks include long-context management as a first-class runtime module.
- Verify no proposal slides are modified.
- Verify Markdown formatting and git diff are clean for the touched docs.

## Implementation Notes For Later Features

- Start with metadata and telemetry before optimizing KV movement.
- Prefer provider-local KV references first; moving raw KV tensors should be a
  fallback because it can be very expensive.
- Keep smallest Qwen as the first real model target.
- Use MiniNDN for final validation, with one short-context baseline, one
  shared-prefix long-context smoke, and one cache-pressure experiment.
