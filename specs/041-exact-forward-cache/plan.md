# Plan: Exact Forward Cache For NDNSF-DI

## Design

NDNSF-DI will keep two cache concepts separate:

1. Exact Forward Cache: strict forward-compute reuse for identical token prefix
   and identical stage definition.
2. Semantic Service Cache: future application-level approximate result reuse;
   not implemented here.

Exact Forward Cache has one default exact hit path:

1. Provider-local cache: a provider has resident forward/KV state and can reuse
   it locally after validating the full exact key.

NDN's native in-network Data cache is still part of the broader NDNSF-DI design,
but it is not the default home for KV/forward state. In-network cache is better
suited for model artifacts, tokenizer/config files, input chunks, output Data,
telemetry objects, and video segments. Forward/KV state is tied to provider
memory, runtime, layer range, and request security context, so the safer default
is to keep it inside the provider and not advertise exact key digests in ACKs.

The implementation adds `ExactForwardCacheKey`, `ExactForwardCacheEntry`, and
`ExactForwardCacheManager` to Runtime v1. The key is content-addressed with a
stable digest and includes enough scope to prevent accidental reuse across
models, tokenizers, plan layouts, layer ranges, exported artifacts, runtime
backends, and security epochs.

`StageDefinition` is represented inside the key by `role`, `layerStart`,
`layerEnd`, `planHash`, `splitLayoutHash`, and `exportArtifactHash`. A stage
name alone is not authoritative.

Runtime telemetry gains `resident_exact_cache_key_digests`. ACK metadata
does not report these digests in the minimum implementation. The selected
provider checks its local cache before invoking its model runner; a hit returns
cached outputs through the normal dependency publication path, while a miss runs
the model and inserts the outputs into the provider-local cache.

## Task Plan

- T001 Create spec, plan, tasks, and checklist for Exact Forward Cache.
- T002 Add exact cache key/entry/manager data structures in Runtime v1.
- T003 Add token-prefix digest and stage-definition helper builders.
- T004 Extend Runtime v1 telemetry and ACK metadata with exact cache key digests.
- T005 Update Runtime v1 smoke/evidence output with exact cache hit/miss fields.
- T006 Keep Exact Forward Cache provider-local and reserve NDN in-network cache
  for artifacts, input/output objects, and other policy-safe Data.
- T007 Add unit tests for strict hit/miss behavior and provider-local cache
  object names.
- T008 Run Python contract tests, py_compile, git diff check, and CodeGraph sync.
- T009 Remove ACK exposure of provider-local cache keys.
- T010 Add C++ provider-role local memoization so repeated identical local
  inputs skip `NativeModelRunner::run`.
- T011 Add C++ tests proving identical inputs hit and changed inputs miss.

## Validation

Use focused Python tests first. MiniNDN evidence integration is contract-level in
this phase: the existing full-network harness should emit exact cache metadata,
but repeated MiniNDN cache-performance campaigns are a later feature.
