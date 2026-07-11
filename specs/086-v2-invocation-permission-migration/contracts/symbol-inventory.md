# Symbol-Level V1 Inventory

## Delete after zero-caller proof

| Area | Symbol/path |
|---|---|
| User API | `ServiceUser::PublishRequest` declaration/definition |
| Request names | `requestRegexString`, `parseRequestName`, `makeRequestName`, `makeRequestNameWithoutPrefix` |
| Provider dispatch | V1 fallback in `ServiceProvider::OnRequest` |
| Provider callbacks | legacy V1 request decrypt/preprocess callbacks after caller scan |
| Permission lookup | `UserPermissionTable::searchByFunctionName` |
| Permission discovery | `parsePermissionTokenName`, token-name success/error callbacks after registration removal |
| Bloom filter | `BloomFilter.hpp`, `BloomFilter.cpp`, includes and wscript entries |
| Terminology | remaining Direct API aliases, if exact scan finds any |

## Retain

- `parseRequestNameV2` and all V2 name helpers.
- `makeUnifiedServiceName` only where still needed for non-V1 legacy payload
  decoding; remove separately when its exact caller set is empty.
- PermissionResponse encryption/decryption and validation.
- UserToken/ProviderToken fields, checks, caches, and replay guards.
- V2 ACK/Selection/Response handlers and Targeted token bootstrap/refill.

## Required proof

For each deleted symbol: declaration, definition, callback registration,
callers, tests, docs, and build entries are scanned independently. Historical
spec evidence is not rewritten merely to make a wildcard scan empty.
