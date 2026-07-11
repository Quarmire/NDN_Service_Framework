# Final Structural Audit

**Date**: 2026-07-11

Production scans reported zero definitions, callers, build entries, or active
documentation references for V1 `PublishRequest`, split request helpers,
BloomFilter, UserPermissionTable, token-name permission callbacks, Direct
invocation aliases, or old environment variables. The sole textual hit was the
negative regression comment describing the removed V1 shape.

`codegraph status .` reports the index up to date. Caller queries found no
`OnPermissionTokenDecryptionSuccessCallback`, `searchByFunctionName`, or
`BloomFilter` symbol. Partial-name results for `PublishRequest` and
`parseRequestName` were inspected and proved to be `PublishRequestV2` and
`parseRequestNameV2` calls only.

`codegraph sync .` parsed the tree to 100% but returned `Maximum call stack size
exceeded`. A subsequent status check reported the index up to date with 2,147
files, 47,598 nodes, and 159,091 edges. This CLI defect is recorded rather than
misclassified as a source failure.

`GenericDynamicApi/CryptoAndAuthorization/LegacySplitNameFailsClosed` builds a
V1-shaped Service/Function/Bloom request. Exact V2 authorization rejects the
expanded service name and the registered handler remains uncalled.
