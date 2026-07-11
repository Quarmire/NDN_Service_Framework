# Baseline And Pre-Implementation Audit

## Baseline

- Worktree was clean at Spec 096 start.
- CodeGraph was current with 2,149 files and 47,499 nodes.
- Spec 095 full regressions passed: 215 C++ tests and 342 Python tests with one
  skip.
- Canonical Spec 095 evidence is 9/12 accepted: both zero-loss cells 3/3,
  5% FEC-off 2/3, and 5% FEC-on 1/3.
- Filesystem had 8.0 GiB available; the prior canonical campaign used 4.7 MiB.

## ARS Experiment Review

- The question is localization, not optimization or proof of causality.
- The five cells isolate required video and control components while retaining
  parity as the existing video treatment.
- Three repetitions are descriptive only; no p-value, confidence claim, or
  causal interaction claim is permitted.
- Control-only is sequence-matched rather than duration-matched. This is an
  explicit limitation, not silently treated as an equal-duration workload.
- No failed run may be retried, replaced, or omitted.

## Architecture And Security Audit

- PASS: all runtime behavior remains in the current UAV apps and canonical
  NDNSF Targeted/stream paths.
- PASS: only experiment helper parameters and a thin campaign are needed.
- PASS: no permission, NAC-ABE, token, replay, wire, persistence, or Core
  semantics change is proposed.
- PASS: finite named objects and SegmentFetcher are out of scope.
- PASS: rollback removes only the campaign and default-preserving helper flags.

## Pre-Implementation Verdict

**PASS**. The design is minimal, falsifiable, bounded, and executable. No
blocking security, ownership, migration, or evidence issue remains.
