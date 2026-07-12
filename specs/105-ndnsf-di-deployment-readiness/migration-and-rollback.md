# Migration and Rollback

## Compatibility Sequence

1. Add execution evidence and new summary fields; keep old `runnerMode` as a
   deprecated derived field for one migration slice.
2. Update all maintained readers and tests to consume `executionEvidence` and
   `runnerClassification`.
3. Reject real-compute gates that only contain legacy `runnerMode`.
4. Remove caller-assigned `runnerMode` after repository-wide zero-reader scan.
5. Add capability/telemetry v3 as an additive typed service payload; legacy
   provider hints remain readable until the existing mixed-reader deadline.
6. Introduce attempt epoch as DI payload metadata with epoch 0 default only
   during migration; epoch-aware recovery activates after all participating DI
   providers echo and validate it.

## Persistent State

No new authoritative database is introduced.

- Repo SQLite/catalog remains authoritative and unchanged.
- Release activation records and plan/evidence artifacts are files with atomic
  rename and digest verification.
- Model caches, activation objects and KV caches are disposable.
- New cache formats include schema, model, plan, boot and security bindings;
  mismatches cause deletion/rebuild, never migration-in-place.

## Rollback Procedure

1. stop user submissions;
2. allow bounded drain or cancel active attempts;
3. stop provider/user/controller units in dependency-safe order;
4. atomically switch `current` release/profile symlink to `previous`;
5. discard incompatible provider-local caches;
6. restart controller/providers, wait for evidence/readiness, then users;
7. run doctor, one correctness canary and security bootstrap check;
8. preserve failed-release logs, evidence and rollback record.

Rollback never rewrites Repo authoritative objects. A release that cannot read
the current plan/evidence schema must generate a compatible plan rather than
silently ignoring fields.

## Removal Conditions

- Legacy `runnerMode`: remove when CodeGraph/text scan shows no maintained
  readers and all regression summaries carry evidence v1.
- Simulated `ndnsf-di run/bench/context-sweep`: rename/remove when the real
  production commands and all docs/tests are migrated.
- Configured-only MiniNDN resource profiles: retain as labeled fixtures in Spec
  105; Spec 106 must exclude them from physical production profiles after
  measured cross-host telemetry acceptance.

## Physical Acceptance Handoff

Spec 105 never upgrades local evidence into physical evidence. Its release gate
records the source commit, candidate release ID, artifact/profile digests and all
local evidence paths required by Spec 106. Spec 106 consumes an immutable Spec
105 candidate manifest and owns real identities, cross-host topology, physical
GPU telemetry, second-operator reproduction and the physical soak.
