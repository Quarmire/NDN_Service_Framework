# Quickstart: Executing The Simplification Safely

## 1. Inspect State

```bash
git status --short
codegraph status .
codegraph explore "<mechanism and callers>"
```

Record unrelated dirty files before editing. Do not stage them with the phase.

## 2. Select Or Create The Child Feature

Read `contracts/child-feature-map.md`. Spec 084 does not authorize production
edits directly. Resolve the child feature, run its pre-implementation audit,
and stop if the verdict is BLOCK.

## 3. Open A Removal Gate

Copy the template from `contracts/removal-gate.md` into the phase evidence file.
Run both CodeGraph and exact `rg` symbol scans. Mark the gate `BLOCKED` until
callers, compatibility, tests, and rollback are known.

## 4. Migrate Before Delete

1. Add the target-owner implementation or adapter.
2. Add contract tests.
3. Migrate all callers.
4. Run focused tests.
5. Confirm the old path has no callers.
6. Delete the old path in a separate commit.

## 5. Validate

Replace every relevant `DISCOVER` entry in `contracts/regression-matrix.md`
with an exact current command in the child baseline. Run those commands. For
performance-sensitive work, follow `contracts/experiment-gates.md` and save raw
results plus the exact command/environment.

## 6. Stop Conditions

Stop and mark the task blocked when:

- an unexplained external ABI consumer exists;
- a security or lease invariant changes;
- stored Repo data cannot migrate/restart;
- a focused or module regression fails;
- a frozen threshold in `contracts/experiment-gates.md` is violated;
- rollback is not possible without touching unrelated work.
- a child spec has not passed pre-implementation audit;
- a Repo implementation or wire contract is selected before its ADR;
- a lease path lacks a provider authority, epoch, TTL, or partial-failure rule;
- a typed-field deletion has not classified alias versus domain state.

## 7. Completion

Update the child acceptance evidence, ownership matrix, removal gate,
regression evidence, English/Chinese docs, and the corresponding Spec 084
program task. Keep each removal concern independently revertible.
