# Research: Native DI MiniNDN Tracer

## Decision: Extend the existing tracer command instead of creating a separate workflow

**Rationale**: `run_minindn_tracer.sh` already generates policy, runs C++ native smoke checks, writes logs, and records MiniNDN status. Extending it keeps one evidence entry point.

**Alternatives considered**: A standalone MiniNDN-only script was rejected because local smoke evidence remains useful when MiniNDN cannot run in the current shell.

## Decision: Add hard MiniNDN gating as a mode

**Rationale**: The default path should still be usable by non-root development sessions, but acceptance for full network validation needs an explicit failure mode.

**Alternatives considered**: Always requiring root was rejected because it blocks ordinary coding and unit/smoke verification.

## Decision: Treat final response as final-role metadata

**Rationale**: Role-to-role dependencies represent activation exchange. Final response is returned by the final role handler and must not be modeled as a no-consumer dependency edge.

**Alternatives considered**: A no-consumer dependency was rejected because provider-worker tests correctly treat dependencies as publish/fetch activation edges.

## Decision: Gate LLM planner work behind tracer evidence

**Rationale**: The runtime path should be stable before planner complexity is added. The first LLM planner task should reuse accepted tracer evidence and plan shape.

**Alternatives considered**: Starting with LLM planner generation was rejected because it could hide network/runtime gaps behind planner metadata.
