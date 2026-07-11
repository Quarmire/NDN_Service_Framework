# ADR: Remove V1 ABI Without Compatibility Aliases

**Decision**: Accepted for the Experimental branch.

Removed surfaces:

- `ServiceUser::PublishRequest(...)`
- split `ServiceName, FunctionName` request APIs used only by V1
- V1 request-name builders/parsers and provider fallback
- Bloom-filter request targeting
- legacy NDNSD permission-token decode callbacks
- Direct aliases if any remain

Retained surfaces:

- unified `RequestService` and V2 name helpers
- `RequestServiceTargeted`
- normal ACK/Selection/Response and collaboration APIs
- encrypted PermissionResponse
- NAC-ABE attribute routing
- UserToken/ProviderToken and replay protection

Deferred source-only adapter:

- `LegacyAckStrategyHandler` remains temporarily because it adapts an old
  callback return type to `AckDecision`; it does not select the V1 wire-name
  grammar or restore a V1 invocation path. Its removal belongs to a separately
  inventoried API-cleanup feature after downstream callback migration. No new
  call site may use it.

Rationale: keeping aliases would preserve two protocols and the hidden fallback
that this feature exists to remove. All in-repository callers are migrated and
verified before deletion. External users must migrate to unified V2 APIs.
