# Child 088 Acceptance

Accepted: 2026-07-11

Child feature: `specs/088-repo-canonical-runtime-contract/`

Acceptance evidence:

- C++ Repo object/local contract binaries all pass.
- Repo Python regressions pass 89/89; full Python passes 343/343 with one
  environment-dependent skip.
- Core C++ passes 214/214 and all six security regressions pass.
- Versioned public/internal services enforce operation/service consistency and
  reject ordinary identities on peer-only handlers.
- Three matched 60-second exact-packet RF=2/W=ALL MiniNDN campaigns complete
  30/30 at 0.5 offered RPS with zero rejections; finalized writes carry both
  required receipts.
- Stop/restart repair validation completes 12/12, runs 16 repair scans and two
  catalog merges, and records zero invalid repair events.
- The unversioned C++ standalone network runtime is removed; C++ remains the
  object/local contract and `py_repoclient` is the sole deployed network
  adapter.
- SQLite schema/version and exact packet bytes are unchanged, so source rollback
  does not require stored-state conversion.

Raw campaign results:
`results/spec088-rf2-wall-20260711/`.

Implementation commit: `5aca321`.
