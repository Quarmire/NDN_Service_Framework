# Final Occam Report

## What Became Smaller

- `service.py`: 2,236 to 1,317 lines.
- `ServiceUser.cpp`: 7,950 to 7,289 lines.
- `ServiceProvider.cpp`: 9,156 to 8,628 lines.
- Python `ndnsf.__all__`: baseline 98, temporary pre-cleanup 109, final 90.
- Dead Core coordination implementation and its isolated tests: 575 lines
  removed after the DI advisory path failed its retention experiment.
- Active V1 invocation findings: zero.

## What Did Not Become Smaller

- `GroundStationServiceContainer.inc.hpp`: 8,460 to 8,501 lines.
- Pybind classes: 14 to 27; bound methods: 59 to 106.
- Tracked C/C++/Python source files outside `third_party/`, `specs/`, and agent
  planning directories: 357 at the old baseline and 383 after the program.

The project did not shrink globally. The baseline predates substantial Repo,
DI, UAV, typed-envelope, stream, and lease functionality. Spec 084 reduced
duplicate authority and public ambiguity while adding tests and reusable
contracts. This report makes no total-code-reduction claim.

## Behavior And Performance

- Core: 214/214 C++, 336 Python plus one expected skip, and six security suites.
- DI: final 2/2 typed Qwen run at p50 200.185 ms and p95 357.711 ms. Compared
  with child 087's p95 332.64 ms, the 7.5% difference remains within the frozen
  10% gate; these short runs are not a throughput comparison.
- Repo: three healthy 60-second RF=2/W=ALL runs completed 30/30 each. The final
  failure-injection smoke also completed 5/5, but its latency is intentionally
  worse and is not presented as an optimization.
- UAV: the three-run loss campaign completed 3/3 with zero frame gap and bounded
  pending memory. The final one-run p50 rose from the earlier mean while p95 was
  unchanged; one unmatched run cannot establish a regression or improvement.

## Failure And Resource Evidence

- Lease authority loss fails closed; no synthetic local lease is issued.
- Repo private operations reject ordinary clients and repair remains bounded.
- Typed ACK decoders count legacy, conflict, malformed, and unknown-schema
  cases; accepted MiniNDN runs observed zero in all four categories.
- UAV reports pending bytes/chunks, gap, stale-session rejection, and FEC hits.
- DI reports dependency completion, typed-envelope counts, latency, and
  coordinator state.

Conclusion: the program passes because ownership, authority, wire contracts,
and rollback are simpler and testable, not because every numeric size or
latency metric improved.
