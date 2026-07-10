# Quickstart

1. Confirm every target dirty file has an owner/commit decision.
2. Run the parent Core/DI/Repo baselines.
3. Add tests from `contracts/test-matrix.md` before implementation.
4. Implement Core lease table, then DI service/transaction.
5. Migrate one boundary symbol group at a time.
6. Run focused tests and exact caller scan before deleting old exports.
7. Run security and MiniNDN gates under the parent frozen thresholds.
8. Record one rollback command per concern.

Stop on unknown external caller, changed security behavior, synthetic lease,
partial execution, stale-epoch acceptance, dirty-file ownership conflict, or a
parent performance-gate violation.
