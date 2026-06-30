# Feature 041: Exact Forward Cache For NDNSF-DI

## Goal

Implement Exact Forward Cache as a strict Runtime v1 cache layer for distributed
LLM inference. A cache hit must mean the token prefix and the forward-compute
scope are exactly identical. Semantic or approximate service-result reuse is
explicitly out of scope for this feature.

## Requirements

- Exact Forward Cache keys must include the token-prefix digest, tokenizer,
  model identity, model artifact hash, plan/layout identity, stage definition,
  runtime/export identity, position state, dtype/quantization, and security
  epoch.
- Stage identity must be scoped by stage definition, not by stage name alone.
  `/LLM/Stage/1` is only reusable when the model, layer range, plan/layout,
  exported artifact, and runtime context are identical.
- Cache manager lookup must return a hit only for byte-for-byte equal cache
  keys. Similar text, similar requests, or similar results must miss.
- Runtime telemetry and evidence may record resident exact forward cache key
  digests for local debugging and evaluation, but ACK metadata must not expose
  provider-local cache keys in the minimum implementation.
- Exact Forward Cache must remain a provider-local optimization by default.
  Reusable forward/KV state is sensitive runtime state, so it should not be
  published as generally reusable in-network cached Data unless a later feature
  adds explicit encryption, key distribution, revocation, and lifetime policy.
- Exact Forward Cache must be transparent to NDNSF request naming and selection.
  It must not require a new request name, message type, selection strategy, or
  ACK field for cache keys.
- Provider runtime must check local Exact Forward Cache before local forward
  computation. On an exact hit, it must reuse cached outputs and skip the model
  runner. On a miss, it must run the model and store the outputs locally.
- NDN's native in-network Data cache remains useful for less-sensitive or
  explicitly policy-protected objects such as model artifacts, tokenizer/config
  files, input chunks, output Data, telemetry objects, and video segments. That
  cache path is separate from provider-local Exact Forward Cache.
- MiniNDN Runtime v1 evidence must record the exact cache key and whether the
  evidence represents an exact hit or miss.
- Tests must prove that same token prefix hits, changed token prefix misses,
  changed stage definition misses, and changed plan/layout misses.
- Tests must prove that provider-local runtime memoization skips the model
  runner for repeated identical local inputs.

## Non-Goals

- No Semantic Service Cache implementation.
- No approximate embedding similarity lookup.
- No remote KV tensor migration. Provider-local exact cache references are the
  first step.
- No default in-network caching of forward/KV state.
- No proposal-slide changes.

## Success Criteria

- Runtime v1 exposes typed Exact Forward Cache key, entry, and manager objects.
- Existing Runtime v1 tests continue to pass.
- New tests cover exact hit/miss rules and show semantic-looking inputs do not
  hit unless token prefixes and scope are identical.
- C++ provider role worker tests show the second identical role execution reuses
  local cached outputs without invoking the runner again.
- Runtime v1 evidence includes exact cache fields in generated JSON.
- Runtime v1 evidence marks Exact Forward Cache as provider-local.
