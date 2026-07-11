# Reconstructed Pre-Migration MiniNDN Baseline

**Date**: 2026-07-11
**Baseline commit**: `419cd2b`

The original entry baseline omitted MiniNDN latency. A detached worktree at the
implementation parent commit was therefore instrumented with only the three
benchmark-support diffs from the treatment commit:

- `examples/App_User.cpp`: `--targeted` benchmark dispatch;
- `examples/App_Provider.cpp`: HELLO `NormalAndTargeted` registration;
- `Experiments/NDNSF_NewAPI_Minindn_Perf.py`: option forwarding.

The parent authorization readiness expression remained unchanged. No V2
authorization-table or V1-removal implementation was applied to the baseline.
The instrumented parent built `App_ServiceController`, `App_Provider`, and
`App_User` successfully.

The exact topology, placement, warmup, request count, interval, ACK timeout,
service timeout, logging, and startup parameters match
`evidence/minindn-acceptance.md`. Result directories:

- `results/spec086-pre-migration/normal`
- `results/spec086-pre-migration/targeted`

| Mode | Baseline completion | Baseline p50/p95 | Migrated p50/p95 | p95 change | 15% gate |
|---|---:|---:|---:|---:|---:|
| Normal | 10/10 | 59.244/96.384 ms | 59.432/95.507 ms | -0.91% | PASS |
| Targeted | 10/10 | 33.112/59.486 ms | 32.541/59.941 ms | +0.76% | PASS |

Both baseline and treatment had zero timeouts and zero pending calls at
shutdown. This closes the previously missing historical comparison without
using treatment implementation in the baseline.
