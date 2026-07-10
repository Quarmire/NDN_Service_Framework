# Ownership Contract

## Core May Own

- V2 normal and Targeted service invocation.
- Authentication, authorization, NAC-ABE, bootstrap, tokens, replay protection.
- Generic ACK selection and negative rejection reasons.
- Generic admission/execution lease envelopes and provider-local lease table.
- Generic operation status and data references.
- Exact-name segmented large-data publishing/fetching.
- Continuous stream framing, reorder, health, and adaptive state.
- Generic service discovery, provider runtime/capability hints, and pair metrics.
- Generic collaboration transport.

## Core Must Not Own

- Model family, model fragment, stage/shard, backend, prompt, cache-pattern, or
  DI deployment policy.
- Repo storage schema, catalog, repair, replica placement, or Repo producer.
- MAVLink, H264, FEC codec, ROI, mission, preflight, or operator policy.
- Automatic retries whose safety depends on application idempotency.
- Mandatory advisory scheduling or a global resource coordinator.

## Application Responsibilities

- **DI**: planning, deployment lifecycle, model/runtime artifacts, fragment
  residency, execution policy, exact/semantic cache policy, bounded replanning.
- **Repo**: storage/catalog/replication/repair and public Repo contract.
- **UAV**: flight/video/mission/operator semantics and GUI behavior.

## Promotion Rule

An application mechanism may move to Core only when:

1. names and fields are application-neutral;
2. enforcement belongs at the framework boundary rather than in application
   policy;
3. Core can implement it without importing an application package;
4. security and lifecycle semantics are stable;
5. focused tests prove every current consumer uses the shared implementation.

Two independent applications needing identical semantics is strong promotion
evidence, but not an absolute prerequisite for a safety or correctness primitive
that only Core can enforce.
