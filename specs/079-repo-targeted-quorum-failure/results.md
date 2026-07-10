## Material Passport

- **Artifact type**: Experiment Result
- **Verification status**: VERIFIED
- **Date**: 2026-07-10
- **Environment**: MiniNDN AI_Lab, three Repo providers
- **Measured window**: 60 seconds per condition
- **Configuration**: 0.5 RPS, concurrency 4, 10% reads, 2,048-byte objects,
  RF=3, W=QUORUM, Targeted with bounded fallback, 5,000 ms control timeout

# Spec 079 Results

## Matched Campaigns

| Condition | Success | Achieved RPS | Write p50 | Write p95 | Minimum successful receipts |
|---|---:|---:|---:|---:|---:|
| Three providers available | 30/30 | 0.500 | 192.938 ms | 2,231.614 ms | 3 |
| RepoA stopped at 20 s | 29/30 | 0.483 | 219.818 ms | 5,613.498 ms | 2 |

The one failed request in the provider-loss run started and completed before
RepoA was stopped. Its reservation received zero responses because all three
Targeted deliveries timed out. It is a pre-failure bounded-delivery event, not
a failure caused by removing RepoA.

## Failure Phases

| Phase | Requests | Success | Successful writes | Write p50 | Write p95 | Receipt range |
|---|---:|---:|---:|---:|---:|---:|
| Before failure | 10 | 9/10 | 9 | 2,145.502 ms | 3,220.377 ms | 3--3 |
| Overlapping failure | 1 | 1/1 | 1 | 7,597.168 ms | 7,597.168 ms | 2--2 |
| After failure | 19 | 19/19 | 17 | 178.457 ms | 1,649.912 ms | 2--2 |

The overlapping request paid the first failure-detection cost. After both
Targeted and Normal fallback failed for RepoA, the stronger cooldown prevented
repeated selection for most of the remaining window. All post-failure writes
met W=QUORUM without claiming a third receipt.

## Control Metrics

- Baseline measured window: 173 Targeted completions, zero timeout, fallback,
  or Normal calls; maximum replica-call concurrency 6.
- Provider-loss measured window: 136 Targeted submissions, 130 completions, 6
  timeouts, 5 bounded fallbacks/Normal calls; maximum concurrency 6.
- Warmup counters were reset after the seed readiness barrier.
- Seed readiness is bounded to three attempts and records every pre-window
  error under `seedAttempts` and `seedErrors`.

## Result Paths

- Baseline: `results/repo_targeted_spec079_rf3_quorum_baseline_20260710/campaign-c4-rps0.5-seed77903`
- Provider loss: `results/repo_targeted_spec079_rf3_quorum_repoA_loss_20260710/campaign-c4-rps0.5-seed77903`

## Interpretation and Limits

The implementation now preserves quorum correctness under a real Repo process
loss: RF remains the desired repair target, while W determines whether the
current write can commit. The experiment also confirms that a first request
overlapping failure can consume the bounded detection deadline, and unrelated
SVS/Targeted delivery loss can still fail a request even before node failure.

This is one deterministic campaign pair on one topology, not a production SLO
or a statistical population claim. Automatic restoration of the third replica
was deliberately outside this experiment and remains the next boundary.
