# T093 Local operations drills

Overall status: **NOT RUN / BLOCK** for live MiniNDN restart/cache drills;
**PASS** for isolated release operations.

## Completed isolated operations

The T091 staging command generated digest-bound release N and N+1, installed N
twice idempotently, activated N+1, rolled back to N, and verified the Repo
sentinel digest stayed
`75a0a93ec792ce49960c0ef009e2fffd238a1c97d4ca3c29d2f12ca996c7542b`.
`current` and `previous` changed atomically. Unit syntax/dependencies and nine
static hardening directives passed without starting host services. Full command
and retained systemd 245 limitation are in `systemd-staging.md`.

## Gated live operations

The controlling T078 evidence is `BLOCK`: its eight deterministic hook cells
and 11/11 fallacy scan passed, but `networkInjection=false`; provider loss,
restart, straggler, missing segment, hash mismatch, stale telemetry, cache
eviction and late output were not injected into a live MiniNDN network.
Consequently these cells were not relabeled as live evidence:

- scheduled provider-process stop/start and new boot-ID observation;
- bounded network recovery during restart;
- live incompatible-cache rebuild or exact failure;
- live late-old-output rejection.

Running only a normal process and claiming a fault would be false evidence.
Revision R2 therefore closes this attempt as `NOT RUN / BLOCK`; the packaging
mechanisms remain implemented and reproducible, while the MiniNDN candidate
operations dimension remains BLOCK.
