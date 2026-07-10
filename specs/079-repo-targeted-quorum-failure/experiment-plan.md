## Material Passport

- **Artifact type**: Experiment Plan
- **Verification status**: VERIFIED
- **Date**: 2026-07-10
- **Environment**: MiniNDN AI_Lab topology

# Experiment Plan

## Research Question

Can the secure Targeted Repo control path preserve bounded RF=3/W=QUORUM writes
when one of three desired Repo providers becomes unavailable?

## Hypothesis

After separating desired RF from required W, writes should commit with two
validated receipts. The first request that observes the failure may pay the
bounded Targeted/fallback timeout; later requests should use cooldown state.

## Measurements

- Attempted, succeeded, failed, and achieved RPS.
- Overall and write p50/p95/p99/stddev.
- Reservation and store phase latency.
- Confirmed replica count per successful write.
- Targeted timeout, fallback, normal-call, and max-concurrency counters.
- Pre-failure, overlapping, and post-failure phase summaries.

## Controls

The no-failure and provider-loss runs share every argument except failure
injection. Results are descriptive research-prototype evidence; one run per
condition is not a population-level statistical claim.

## Acceptance

No successful write may have fewer than two valid receipts. W=ALL regression
must still reject two receipts. Any timeout or throughput regression is
reported rather than hidden.
