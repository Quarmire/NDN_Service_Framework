# Research: Native DI Tracer

## Decision: Build a tracer-bullet before full LLM planner work

**Rationale**: The current repo already has Python policy generation and C++ native execution components, but the risky boundary is the generated plan driving the native provider path. A small tracer makes this boundary testable before adding planner/model complexity.

**Alternatives considered**: Starting with the LLM planner was rejected because planner placeholders would mix research questions with runtime validation. Starting with only unit tests was rejected because it would not produce MiniNDN-ready evidence.

## Decision: Use MiniNDN as final validation surface

**Rationale**: Project governance and previous benchmark practice prefer MiniNDN for NDNSF network/security/performance validation until algorithms are stable.

**Alternatives considered**: Host NFD is useful for diagnosis but not final acceptance. Real hardware is premature for this stage.

## Decision: Keep final-response strict

**Rationale**: Native DI must only return the planned final role output. Returning arbitrary intermediate output hides plan errors and weakens the evidence path.

**Alternatives considered**: A fallback response was simpler but was already identified as unsafe for correctness.

## Decision: Treat readiness as an execution gate

**Rationale**: Provider selection should not advertise providers whose required artifacts or runtime state are still missing. Readiness is part of the service evidence, not just a local log.

**Alternatives considered**: Letting request execution discover missing artifacts was rejected because it makes failures late, noisy, and harder to reason about.
