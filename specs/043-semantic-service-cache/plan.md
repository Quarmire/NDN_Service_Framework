# Plan: Provider-Local Semantic Service Cache

## Design

The semantic cache is a provider-local service optimization layer:

1. The LLM app computes a semantic pattern ID and confidence using its chosen
   embedding/similarity method.
2. The provider checks `SemanticServiceCacheManager`.
3. A high-confidence hit returns a cached final service response.
4. A miss runs normal NDNSF-DI inference, then optionally admits the final
   response based on estimated saved output tokens and reuse likelihood.

The framework never treats semantic hits as exact forward-cache hits. A stage is
still reusable only through the exact forward cache when token prefix and stage
definition match exactly.

## SCALM Fit Audit

The first helper implementation covered provider-local semantic response reuse,
coarse ACK hints, and saved-token-aware admission. A closer audit against SCALM
found two missing pieces:

1. SCALM explicitly ranks semantic patterns by token-saving value, while the
   first NDNSF-DI helper only carried a `semantic_pattern_id`.
2. SCALM evaluates cache value with token saving ratio, not only hit ratio or
   absolute saved tokens.

The final design keeps clustering and embedding outside NDNSF-DI, but accepts
application-produced `SemanticPatternMeta` records. NDNSF-DI ranks those
patterns, attaches rank/round/token-saving metadata to provider-local cache
entries, uses rank and token-saving ratio in admission/eviction scoring, and
exports only coarse rank/ratio buckets in ACK metadata.

## Data Model

- `SemanticServiceCacheKey`: service, model, tokenizer, policy epoch, semantic
  pattern, response schema, app namespace.
- `SemanticServiceCacheEntry`: response payload, confidence threshold,
  estimated prompt/output/saved tokens, byte size, reuse likelihood, timestamps.
- `SemanticServiceCacheManager`: provider-local cache with admission, lookup,
  token-saving-aware eviction, and telemetry counters.
- `SemanticCacheAckHint`: coarse ACK metadata for user selection.
- `SemanticPatternMeta`: application-produced pattern summary with conversation
  round, query count, estimated saved tokens, token saving ratio, and rank.

## Validation

- Focused Python unit tests in `tests/python/test_ndnsf_di_runtime_v1.py`.
- Minimal LLM semantic-cache demo in
  `examples/python/NDNSF-DistributedInference/llm_semantic_cache_demo.py`.
- llama-server provider semantic-cache smoke harness in
  `examples/python/NDNSF-DistributedInference/llama_server/run_semantic_cache_provider_smoke.py`;
  this uses the real provider handler path with a fake llama-server backend so
  cache hits can be measured without loading a model.
- single-host llama-server semantic-cache network smoke in
  `examples/python/NDNSF-DistributedInference/llama_server/run_semantic_cache_network_smoke.py`;
  this starts a fake OpenAI-compatible backend plus real NDNSF-DI
  controller/provider/user processes and checks that similar prompts reduce
  backend calls.
- multi-provider semantic-cache selection campaign in
  `examples/python/NDNSF-DistributedInference/llama_server/run_semantic_cache_selection_campaign.py`;
  this compares first-provider selection against coarse ACK hint based
  cache-aware selection for warmed and cold providers.
- `py_compile` for runtime and tests.
- `git diff --check`.
- `codegraph sync . && codegraph status .`.
