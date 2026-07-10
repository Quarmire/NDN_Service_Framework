## Material Passport

- **Artifact type**: Experiment Result
- **Verification status**: VERIFIED
- **Date**: 2026-07-10
- **Environment**: MiniNDN AI_Lab topology, three Repo providers, RF=2, W=ALL
- **Measured window**: 60 seconds per campaign
- **Object size**: 2,048 bytes

# Spec 078 Results

## Matched Campaigns

| Campaign | Control | Success | Write p50 | Write p95 | Write mean | Achieved RPS |
|---|---|---:|---:|---:|---:|---:|
| c16, 2 RPS, 90% read | Spec 077 normal | 119/120 | 24,475.806 ms | 39,838.543 ms | 22,935.450 ms | 1.450 |
| c16, 2 RPS, 90% read | Spec 078 Targeted | 120/120 | 149.134 ms | 243.134 ms | 270.962 ms | 2.000 |
| c4, 0.5 RPS, 10% read | Spec 077 normal | 28/30 | 5,452.653 ms | 10,583.983 ms | 5,793.408 ms | 0.440 |
| c4, 0.5 RPS, 10% read | Spec 078 Targeted | 30/30 | 145.058 ms | 192.855 ms | 145.067 ms | 0.500 |

The Targeted runs disabled normal fallback. The c16 run recorded 76 Targeted
async completions, zero timeout/fallback/normal calls, and maximum replica
concurrency 3. The write-heavy run recorded 118 completions, zero
timeout/fallback/normal calls, and maximum replica concurrency 2.

For successful write-heavy requests, reserve p50/p95 was 50.326/96.609 ms and
store p50/p95 was 88.885/132.447 ms. Receipt validation still required both
RF=2 replicas for W=ALL.

## Result Paths

- Baseline c16: `results/repo_ha_spec077_final_read_c16_20260710/campaign-c16-rps2-seed77716`
- Targeted c16: `results/repo_targeted_spec078_c16_20260710/campaign-c16-rps2-seed77716`
- Baseline write-heavy: `results/repo_ha_spec077_write_20260710/campaign-c4-rps0.5-seed77802`
- Targeted write-heavy: `results/repo_targeted_spec078_write_20260710/campaign-c4-rps0.5-seed77802`

## Correctness Incident

The first RF=2 Targeted smoke exposed a native race in NAC-ABE/OpenABE. GDB
located the crash at `librelic::ep_param_set` from `ABESupport::kpDecrypt`.
OpenABE/RELIC initialization and curve state are bound to one thread, but
NDNSF decrypts can arrive on different Face/worker threads. NAC-ABE now routes
all OpenABE operations through one process-wide worker initialized on that
same thread. An eight-caller concurrent KP-ABE regression passes, and the
original RF=2 Targeted-only MiniNDN smoke completes without a crash.

## Interpretation

This is a real control-plane improvement: known Repo providers avoid repeated
Request/ACK/Selection round trips, and independent reserve/store calls overlap.
It does not relax permissions, NAC-ABE, one-time provider tokens, replay
checks, operation IDs, receipts, or W. The remaining boundary is that ABE
operations are serialized inside each process because the current
OpenABE/RELIC backend is not thread-independent.

## Residual Risks and Next Boundary

- OpenABE work is correct but serialized per process, so ABE can become the
  next bottleneck at substantially higher provider concurrency.
- A depleted Targeted token pool still requires a normal bootstrap/refill
  round. The configurable batch amortizes that cost but does not remove it.
- The bounded Normal fallback is intentionally sequential and has not yet been
  measured under an injected provider failure.
- Evidence currently covers one MiniNDN topology, three Repo providers, RF=2,
  W=ALL, and 2,048-byte opaque objects. It is research-prototype evidence, not
  a production SLO.

The next useful campaign is Targeted RF=3/W=QUORUM with one provider stopped
during the measured window. It should quantify fallback and partial-receipt
behavior before adding more control-plane concurrency.
