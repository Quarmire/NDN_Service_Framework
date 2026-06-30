# Spec: Provider-Local Semantic Service Cache

## Goal

Add a small NDNSF-DI runtime helper for provider-local semantic response cache.
The feature borrows the useful idea from SCALM-style LLM semantic caching:
optimize for saved output tokens and keep semantic cache separate from exact
forward/KV cache.

## Requirements

- The cache is provider-local by default.
- It stores service-level responses, not internal KV blocks or stage
  activations.
- Exact Forward Cache and Semantic Service Cache remain separate APIs.
- Semantic hits require the same service, model, tokenizer, policy epoch, and
  semantic pattern ID.
- A caller-provided confidence value must meet a threshold before a hit is
  returned.
- Cache admission should prefer entries with higher estimated saved decode
  tokens and reuse likelihood.
- Cache admission should be able to use application-produced semantic pattern
  metadata, including conversation round and token saving ratio.
- Eviction should consider estimated saved tokens, reuse likelihood, entry size,
  and recent use.
- ACK metadata may expose only coarse fields: hit/candidate/miss, confidence
  bucket, estimated saved tokens, policy epoch, pattern rank bucket, and token
  saving ratio bucket.
- Raw prompts, raw embeddings, and full semantic keys must not be exposed in
  ACK metadata.

## Non-Goals

- No new wire protocol.
- No embedding model implementation.
- No in-network semantic response cache.
- No change to stage correctness or dependency execution.

## Acceptance Criteria

- Runtime v1 exposes a semantic cache key, entry, manager, ACK metadata helper,
  and cache-aware provider selection helper.
- Runtime v1 can rank application-produced semantic pattern metadata by token
  saving value and keep raw pattern IDs local.
- Tests cover hit, confidence miss, policy epoch miss, admission, eviction, and
  ACK privacy.
