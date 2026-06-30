# Feature 040: NDNSF-DI Runtime v1 With Long-Context Management

## User Story

As an NDNSF-DI developer, I want the distributed inference runtime to manage
provider capacity, reusable plans, role-level pipelining, and long-context LLM
state so that NDNSF-DI can become a usable and performant distributed language
model serving framework rather than a one-shot experiment harness.

## Motivation

Current NDNSF-DI evidence shows that capacity-aware proportional planning can
spread provider load and improve high-concurrency behavior in MiniNDN. However,
the current LLM path still lacks first-class long-context support. Long prompts,
prefix reuse, conversation state, KV cache, and streaming generation are not
represented as runtime objects.

For practical LLM serving, long-context management is a core performance
requirement. Re-sending a full prompt and rebuilding KV state on every request
can dominate latency and bandwidth. Moving KV tensors blindly between providers
can be worse than keeping a session on the provider that already owns the
state. Therefore, Runtime v1 must include context-state naming, cache-aware
planning, prefill/decode separation, and streaming output.

## Requirements

### Functional Requirements

- FR-001: Providers MUST advertise static resource capacity including memory,
  compute capacity, worker slots, supported runtime backends, and maximum
  context length.
- FR-002: Providers MUST expose dynamic telemetry including queue depth, active
  workers, free memory, model cache state, and KV-cache usage.
- FR-003: The planner MUST treat LLM plans as reusable objects with explicit
  validity conditions.
- FR-004: LLM planning MUST default to linear stage splits and use sharding only
  when a stage cannot fit on one provider.
- FR-005: The planner MUST support proportional layer assignment based on
  effective capacity.
- FR-006: Runtime execution MUST allow role-level pipelining across requests.
- FR-007: The data plane MUST support named large dependency objects with
  digest, byte-count, and segment-count metadata.
- FR-008: Long-context state MUST be represented through named runtime objects:
  PromptChunk, PrefixState, SessionState, KvBlock, and GenerationChunk.
- FR-009: The planner MUST distinguish prefill and decode costs.
- FR-010: The planner MUST account for prefix/session cache residency when
  selecting providers.
- FR-011: Providers MUST expose cache hit, miss, eviction, and resident-state
  counters.
- FR-012: The runtime MUST support streaming generated token chunks instead of
  requiring one final response object.
- FR-013: The runtime MUST define cache lease, pin, eviction, and invalidation
  rules.
- FR-014: MiniNDN validation MUST include at least one long-context smoke test
  and one cache-pressure test.

### Non-Functional Requirements

- NFR-001: Runtime v1 must preserve NDNSF security rules: permissions,
  NAC-ABE routing, one-time tokens, replay protection, and signed/encrypted
  control data.
- NFR-002: New context-state objects must be named and authenticated as data,
  even when the underlying KV bytes remain provider-local.
- NFR-003: Long-running LLM requests should use session/plan leases rather than
  relying only on a very large request timeout.
- NFR-004: MiniNDN remains the default validation surface for network,
  security, and performance evidence.
- NFR-005: The smallest Qwen model remains the first real model target.

## Acceptance Criteria

- AC-001: A documented Runtime v1 design exists and includes long-context
  management.
- AC-002: Task artifacts describe provider telemetry, plan lifecycle,
  role-level pipelining, data-plane optimization, and long-context work.
- AC-003: The long-context task list covers prefill/decode split, KV-cache
  metadata, provider-local KV references, streaming output, cache leases,
  eviction, and MiniNDN validation.
- AC-004: Existing completed feature evidence from specs 029-039 remains
  untouched and is treated as baseline evidence.

## Out Of Scope For This Planning Feature

- Full implementation of the Runtime v1 roadmap.
- Replacing the smallest Qwen model with a larger model.
- Editing proposal slides.
- Reworking NDNSF core security semantics.
