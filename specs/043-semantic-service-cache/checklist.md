# Checklist: Provider-Local Semantic Service Cache

- [x] Exact and semantic cache APIs are separate.
- [x] Semantic cache is provider-local.
- [x] ACK metadata is coarse and privacy-preserving.
- [x] Admission/eviction use saved-token benefit.
- [x] Semantic pattern rank and token saving ratio are represented without
  exposing raw pattern IDs in ACKs.
- [x] Tests cover both positive and negative hit cases.
- [x] Minimal LLM demo records repeated/similar prompt hits, latency, hit ratio,
  and token saving ratio.
- [x] Real llama-server provider has opt-in semantic cache integration with
  focused tests and dry-run coverage.
