# Feature 029: LLM Proportional Capacity Split

## Goal

Add and validate an LLM planner mode that can divide linear pipeline stages
across heterogeneous 2GB, 4GB, and 8GB providers in proportion to usable
capacity and compute. Compare it against the existing greedy mode and produce
RPS evidence.

## Requirements

1. Add a 2GB/4GB/8GB provider resource profile.
2. Add an ACK-candidate sample carrying the same 2GB/4GB/8GB resource metadata.
3. Keep the existing greedy mode as the default.
4. Add `proportional` mode:
   - compute each provider's planning weight from
     `min(memory capacity ratio, compute capacity ratio)`;
   - for 2/4/8GB with matching compute, target an approximate 1:2:4 layer split;
   - keep the split as linear pipeline stages;
   - only use shards when a minimum stage cannot fit a provider.
5. Add a reproducible RPS search tool that compares greedy vs proportional:
   - max stable planner-derived RPS;
   - p50/p95 estimated latency;
   - failure rate;
   - provider utilization.

## MiniNDN Scope Note

The current full-network MiniNDN harness executes `/Inference/NativeTracer`,
not a real LLM role graph. Therefore this feature can prove the planner and RPS
search logic for LLM provider profiles, and can optionally run the existing
NativeTracer MiniNDN harness for compatibility, but it cannot honestly claim
that MiniNDN has executed the LLM proportional roles until an LLM execution
adapter is added to the harness.

## Validation

- Greedy 2/4/8 profile demonstrates current imbalance.
- Proportional 2/4/8 profile produces approximately 1:2:4 layer allocation.
- ACK-derived proportional planning produces the same class of allocation.
- RPS search writes CSV/JSON comparing greedy and proportional.
- Python compile and `git diff --check` pass.

