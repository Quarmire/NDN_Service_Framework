# Research Decisions

## R1 - Authority

Provider admission leases are authoritative. Advice may rank candidates but
cannot reserve resources, authorize execution, or override a rejection.

## R2 - Cache boundary

Exact Forward Cache uses exact token/model/stage/runtime/security identity and
is a provider-local runtime optimization. Semantic cache depends on
application-produced similarity metadata and therefore remains experimental.

## R3 - Retry safety

Retryability is a property of the operation and typed failure, not English
error text. The caller must explicitly opt into retrying an idempotent action.

## R4 - Advisory evidence

The primary metric is provider lease-conflict rate. Retention requires >=10%
paired improvement and a paired bootstrap 95% CI excluding zero, with no
completion or p95 threshold violation over at least ten matched runs.
