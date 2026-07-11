# Child 086 Acceptance

Spec 086 completed all 26 tasks. V1 split-name/Bloom-filter invocation,
token-name permission installation, `UserPermissionTable`, and Direct aliases
were removed. V2 authorization now preserves canonical provider/service name,
unified service name, permission kind, and policy epoch in the thread-safe
`ServiceAuthorizationTable`.

Acceptance evidence:

- full C++: 215 tests passed;
- focused Core Python: 29 tests passed;
- security aggregate: all six suites passed;
- forbidden production symbols/build entries: zero relevant matches;
- matched MiniNDN normal: 10/10, p50 59.432 ms, p95 95.507 ms;
- matched MiniNDN Targeted: 10/10, p50 32.541 ms, p95 59.941 ms;
- parent-baseline p95 change: normal -0.91%, Targeted +0.76%, both within 15%;
- detached rollback: implementation and reverted states both built.

Implementation commit: `b3acfd1`. Detailed evidence is under
`specs/086-v2-invocation-permission-migration/evidence/`.
