# Data Model: Simplification Governance

This feature mainly removes code, but durable migration records are required so
deletion decisions remain reproducible.

## OwnershipDecision

| Field | Meaning |
|---|---|
| `mechanism` | Stable mechanism identifier |
| `currentOwner` | Current module/project |
| `targetOwner` | Final module or `none` |
| `disposition` | `keep`, `move`, `hide`, `experimental`, `remove` |
| `invariants` | Security/correctness behavior that must survive |
| `callers` | Repository and known external callers |
| `replacement` | New API or explicit no-replacement decision |
| `phase` | Migration wave |
| `status` | `planned`, `migrating`, `ready-to-remove`, `removed`, `blocked` |

Allowed transitions:

```text
planned -> migrating -> ready-to-remove -> removed
   |           |               |
   +---------> blocked <-------+
```

## CompatibilityEpoch

| Field | Meaning |
|---|---|
| `contract` | API/wire/schema being migrated |
| `oldVersion` / `newVersion` | Supported versions |
| `typedAuthority` | Conflict resolution rule |
| `startCondition` | Preconditions for dual support |
| `endCondition` | Evidence required to remove old form |
| `conflictCounter` | Number of contradictory dual values |
| `legacyUseCounter` | Old-only messages observed |
| `deadline` | Release or phase boundary |

## RemovalGate

| Field | Blocking rule |
|---|---|
| `callerScan` | Must contain no unexplained callers |
| `externalAbiDecision` | Must be approved or adapter defined |
| `securityReview` | Must show no weakened invariant |
| `persistenceMigration` | Required for stored-state changes |
| `focusedTests` | Must pass |
| `moduleRegressions` | Must pass |
| `miniNdnEvidence` | Required for network-visible behavior |
| `performanceComparison` | Required for hot-path behavior |
| `rollbackPoint` | Must identify revert boundary |

Any failed or missing blocking field sets status to `blocked`.

## CanonicalRuntimeDecision

| Field | Meaning |
|---|---|
| `concern` | Invocation, Repo, stream, status, etc. |
| `canonicalImplementation` | Final source of truth |
| `referenceImplementation` | Temporary behavior oracle |
| `adapters` | Thin retained language/client bindings |
| `duplicateImplementations` | Sources scheduled for deletion |
| `parityEvidence` | Test/campaign results proving convergence |

No `canonicalImplementation` value is valid for Repo until the Repo ADR has
evaluated every criterion in `contracts/repo-decision-gate.md`.

## ProviderLeaseAuthority

| Field | Meaning |
|---|---|
| `provider` | Sole authority for the represented local resource |
| `leaseEpoch` | Provider generation; changes after authority restart |
| `leaseId` | Provider-issued lease identity |
| `planId` / `planDigest` | Immutable proposed plan identity |
| `requestId` | Execution request identity |
| `roleIds` | Roles consuming this provider's resources |
| `state` | `prepared`, `committed`, `released`, `expired`, `aborted` |
| `expiresAt` | Final bounded cleanup deadline |
| `idempotencyKey` | Stable key for duplicate prepare/commit/abort/release |

Allowed transitions:

```text
prepared -> committed -> released
    |           |
    +-> aborted +-> expired
```

A stale `leaseEpoch`, conflicting `planDigest`, or unknown transition is
rejected. A user-side deployment record never substitutes for this state.

## FieldDisposition

| Field | Meaning |
|---|---|
| `module` / `field` | Producer and field being classified |
| `semanticOwner` | Core, DI, Repo, or UAV |
| `kind` | `legacy-alias`, `domain-state`, `transport-metadata`, `unknown` |
| `typedReplacement` | Replacement field when kind is `legacy-alias` |
| `storedOrWireImpact` | Migration and compatibility impact |
| `removalDecision` | `retain`, `migrate`, `remove`, `blocked` |

Only `legacy-alias` fields with passing compatibility evidence may be removed.

## RegressionMatrixEntry

| Field | Meaning |
|---|---|
| `invariant` | Behavior being protected |
| `modules` | Affected projects |
| `command` | Exact reproducible command |
| `expected` | Pass criteria |
| `evidencePath` | Result/log path |
| `severity` | `blocking` or `advisory` |

## ChildFeatureDecision

| Field | Meaning |
|---|---|
| `childId` | Reserved feature number and concern |
| `entryGate` | Contracts/evidence required before implementation |
| `dependencies` | Other child features that must complete first |
| `auditVerdict` | `BLOCK`, `CONDITIONAL PASS`, or `PASS` |
| `rollbackPoint` | Independent rollback boundary |
| `acceptanceEvidence` | Final tests/campaigns proving completion |
