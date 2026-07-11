# Entry Inventory

**Branch**: `Experimental`
**HEAD**: `2bf8d44b5401430830ffb742f1d5b2e5010bb181`
**Entry source state**: clean before Spec086 artifacts were created
**CodeGraph**: indexed and up to date (2,150 files; 47,650 nodes)

## Live V1 invocation

- `ServiceUser.hpp/.cpp`: `PublishRequest` declaration/definition, BloomFilter
  construction, `searchByFunctionName`, V1 name builders.
- `ServiceProvider::OnRequest`: V2-first parse followed by active V1 fallback,
  BloomFilter target check, legacy decrypt callback registration.
- `utils.hpp/.cpp`: `requestRegexString`, `parseRequestName`,
  `makeRequestName`, `makeRequestNameWithoutPrefix`.
- `BloomFilter.hpp/.cpp`: implementation and logging category.
- `examples/wscript`: four BloomFilter source entries.
- `tests/wscript`: one BloomFilter source entry.

## Live permission representation

- `UserPermissionTable` is used by ServiceUser and ServiceProvider.
- V2 `applyPermissionResponse` inserts provider/service permissions but drops
  response permission kind and policy epoch from stored state.
- `searchByFunctionName` is called only by V1 `PublishRequest`.
- `processNDNSDServiceInfoCallback` still registers legacy token-name parsing
  and `OnPermissionTokenDecryptionSuccessCallback`.

## Retained security mechanisms

- Direct controller permission Interests and encrypted PermissionResponse.
- Controller signature validation, target identity and permission-kind checks.
- NAC-ABE REQUEST/SELECTION and ACK/RESPONSE attributes.
- One-time UserToken/ProviderToken and replay checks.
- Targeted bootstrap/refill token batches.

Historical specs/tests are inventoried but are not rewritten by wildcard. The
final forbidden scan targets production source, current examples, current
tests, and build metadata.
