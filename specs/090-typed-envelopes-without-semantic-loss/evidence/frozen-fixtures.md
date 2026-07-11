# Frozen Fixtures And Gates

Required fixtures: typed-only v2, typed v1, legacy-only in mixed mode, matching
dual, conflicting dual, malformed typed plus valid legacy, unknown typed plus
valid legacy, typed-only mode rejecting legacy, counter reset/snapshot,
producer restart, Repo persistence, DI plan/cache reload, and UAV state round
trip.

Removal passes only when current producer scans show no inventoried top-level
alias, typed-only network smoke succeeds, mixed-reader smoke reports zero
unexplained conflicts, full security/persistence/exact-wire tests pass, and the
implementation commit reverts independently.
