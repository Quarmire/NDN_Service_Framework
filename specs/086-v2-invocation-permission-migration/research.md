# Research And Code-Reality Notes

## Current V1 path

- `ServiceUser::PublishRequest` builds a Bloom filter and split-name request.
- `utils::{makeRequestName,makeRequestNameWithoutPrefix,parseRequestName}`
  implement the V1 grammar.
- `ServiceProvider::OnRequest` handles V2 first, then parses V1 and checks the
  Bloom filter before invoking legacy decryption callbacks.
- `BloomFilter.cpp` is still listed by example and test build targets.

## Current V2 security path

- V2 request names carry requester, unified service name, and request ID.
- PermissionResponse is controller-signed and encrypted to the target identity.
- Permission kind and policy epoch are already present on the wire.
- UserToken and ProviderToken are fresh one-time invocation values and are not
  the deprecated PermissionEntry token.

## Migration conclusion

The V1 dispatch branch and old permission-token discovery are removable, but
the authorization table itself is live. Replace its representation before
deleting split-name/token helpers.
